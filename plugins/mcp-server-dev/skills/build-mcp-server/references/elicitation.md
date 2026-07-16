# Elicitation (사용자 입력 유도) — 스펙 표준 기반 사용자 입력

Elicitation(엘리시테이션)은 서버가 도구 호출 중간에 일시 중지하고 사용자에게 구조화된 입력을 요청할 수 있게 해주는 기능입니다. 클라이언트는 iframe이나 HTML 없이 네이티브 폼을 렌더링합니다. 사용자가 입력 양식을 채우면 서버가 동작을 재개합니다.

**단순한 입력값 수집에는 이 방법이 가장 적합합니다.** 위젯(`build-mcp-app`)은 차트, 검색 가능한 목록, 시각적 미리보기와 같은 풍부한 UI가 필요할 때 사용합니다. 단순 확인, 옵션 선택, 혹은 몇 개의 입력 필드만 필요하다면 Elicitation이 더 간결하고, 스펙 표준을 따르며, 이를 지원하는 모든 호스트 환경에서 작동합니다.

---

## ⚠️ 지원 여부 먼저 확인 — 새로운 기능

호스트의 지원 여부는 매우 최근에 추가되었습니다:

| 호스트 | 지원 현황 |
|---|---|
| Claude Code | ✅ v2.1.76부터 지원 (`form` 및 `url` 모드 모두) |
| Claude Desktop | 확인되지 않음 — 아직 지원하지 않거나 매우 최근에 추가되었을 가능성 있음 |
| claude.ai | 알 수 없음 |

**클라이언트가 Elicitation 기능을 제공한다고 알리지 않는 경우, SDK는 `CapabilityNotSupported` 에러를 던집니다.** 기본적으로 제공되는 예외 처리가 없으므로, 반드시 이를 확인하고 대체 동작(fallback)을 구현해야 합니다.

### 모범 패턴

```typescript
server.registerTool("delete_all", {
  description: "Delete all items after confirmation",
  inputSchema: {},
}, async ({}, extra) => {
  const caps = server.getClientCapabilities();
  if (caps?.elicitation) {
    const r = await server.elicitInput({
      mode: "form",
      message: "Delete all items? This cannot be undone.",
      requestedSchema: {
        type: "object",
        properties: { confirm: { type: "boolean", title: "Confirm deletion" } },
        required: ["confirm"],
      },
    });
    if (r.action === "accept" && r.content?.confirm) {
      await deleteAll();
      return { content: [{ type: "text", text: "Deleted." }] };
    }
    return { content: [{ type: "text", text: "Cancelled." }] };
  }
  // Fallback: return text asking Claude to relay the question
  return { content: [{ type: "text", text: "Confirmation required. Please ask the user: 'Delete all items? This cannot be undone.' Then call this tool again with their answer." }] };
});
```

```python
# fastmcp
from fastmcp import Context
from fastmcp.exceptions import CapabilityNotSupported

@mcp.tool
async def delete_all(ctx: Context) -> str:
    try:
        result = await ctx.elicit("Delete all items? This cannot be undone.", response_type=bool)
        if result.action == "accept" and result.data:
            await do_delete()
            return "Deleted."
        return "Cancelled."
    except CapabilityNotSupported:
        return "Confirmation required. Ask the user to confirm deletion, then retry."
```

---

## 스키마 제약 사항 (Schema constraints)

Elicitation 스키마는 의도적으로 제한되어 있습니다. 입력 폼은 가급적 단순하게 유지하십시오:

- **1차원 객체(Flat object)만 지원** — 중첩 불가, 객체 배열 불가
- **원시 타입(Primitive)만 지원** — `string`, `number`, `integer`, `boolean`, `enum`
- String 포맷은 다음으로 제한됨: `email`, `uri`, `date`, `date-time`
- 각 속성에 `title`과 `description`을 반드시 사용하십시오 — 폼의 레이블이 됩니다.

데이터가 이러한 제약 조건에 맞지 않는다면, 위젯(widget)으로 전환해야 한다는 신호입니다.

---

## 3가지 상태의 응답 (Three-state response)

| 액션 (Action) | 의미 | `content` 존재 여부 |
|---|---|---|
| `accept` | 사용자가 폼을 제출함 | ✅ 사용자가 정의한 스키마를 기준으로 검증됨 |
| `decline` | 사용자가 거부 의사를 명확히 함 | ❌ |
| `cancel` | 사용자가 입력 창을 닫음 (Esc 키 입력, 다른 곳 클릭 등) | ❌ |

`decline`과 `cancel`은 의미상 구분해야 합니다. `decline`은 명시적인 거부 의사인 반면, `cancel`은 실수에 의한 닫기일 수 있습니다.

TypeScript SDK의 `server.elicitInput()`은 Ajv를 통해 `accept` 응답을 사용자가 설정한 스키마 기준으로 자동 검증합니다. fastmcp의 `ctx.elicit()`는 타입이 지정된 식별 유니온(`AcceptedElicitation[T] | DeclinedElicitation | CancelledElicitation`)을 반환합니다.

---

## fastmcp response_type 단축 표현 (shorthand)

```python
await ctx.elicit("Pick a color", response_type=["red", "green", "blue"])  # enum
await ctx.elicit("Enter email", response_type=str)                         # string
await ctx.elicit("Confirm?", response_type=bool)                           # boolean

@dataclass
class ContactInfo:
    name: str
    email: str
await ctx.elicit("Contact details", response_type=ContactInfo)             # flat dataclass
```

지원 대상: 원시 타입, `list[str]` (enum으로 변환됨), dataclass, TypedDict, Pydantic BaseModel. 단, 모두 중첩되지 않은 1차원 구조여야 합니다.

---

## 보안 (Security)

**Elicitation을 통해 비밀번호, API 키, 토큰 등을 요청해서는 안 됩니다** — 스펙 권장사항입니다. 민감 정보는 런타임 폼이 아닌 OAuth를 이용하거나, `sensitive: true`로 설정된 `user_config` (MCPB)를 통해 전달해야 합니다.

---

## 위젯(Widgets)으로의 전환 시점

Elicitation이 적합한 작업: 확인 창, enum 선택기, 짧고 단순한 구조의 폼.

다음과 같은 요구사항이 있을 때는 `build-mcp-app` 위젯을 사용하십시오:
- 중첩되거나 복잡한 데이터 구조가 필요할 때
- 스크롤 가능하거나 검색이 필요한 대량의 목록 (100개 이상의 아이템)
- 선택 전 시각적 미리보기 필요 시 (이미지 썸네일, 파일 트리 등)
- 실시간으로 업데이트되는 진행 상태나 스트리밍 콘텐츠 표시
- 커스텀 레이아웃, 차트, 지도 등
