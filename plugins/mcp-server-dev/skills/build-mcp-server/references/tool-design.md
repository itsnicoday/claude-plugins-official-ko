# 도구 디자인 — Claude가 올바르게 사용하는 도구 작성하기

도구 스케마와 설명은 프롬프트 엔지니어링의 일환입니다. 이 정보들은 Claude의 컨텍스트에 직접 반영되며, Claude가 올바른 인자값을 사용해 올바른 도구를 선택하도록 결정하는 기준이 됩니다. MCP 연동 과정에서 발생하는 버그의 대부분은 모호한 설명이나 느슨한 스케마 때문에 발생합니다.

## Anthropic 디렉토리 등록 필수 조건

이 서버를 Anthropic 디렉토리(Anthropic Directory)에 제출하려는 경우, 다음 조건들은 심사 통과/탈락을 결정하는 핵심 기준이 됩니다 (전체 목록: https://claude.com/docs/connectors/building/review-criteria):

- 모든 도구는 반드시 `readOnlyHint`, `destructiveHint`, `title` 주석(annotations)을 포함해야 합니다 — 이 주석들이 Claude 내에서 자동 권한 승인 여부를 결정합니다.
- 도구 이름의 길이는 64자 이하여야 합니다.
- 읽기(Read)와 쓰기(Write) 작업은 반드시 개별 도구로 분리되어야 합니다. 단일 도구에서 GET과 POST/PUT/PATCH/DELETE 요청을 모두 수용하는 형태는 반려됩니다 — 도구 설명 내에 안전한 호출과 안전하지 않은 호출을 구분해 문서화해 놓는 것만으로는 이 기준을 충족하지 못합니다.
- 도구 설명에 Claude의 행동을 직접 지시해서는 안 됩니다 (예: "항상 X를 수행하십시오", "반드시 Y를 먼저 호출해야 합니다" 등 시스템 지침을 덮어쓰거나 제품을 홍보하는 행위) — 심사 시 프롬프트 인젝션(prompt injection) 시도로 간주됩니다.
- 자유 형식의 API 엔드포인트나 파라미터를 그대로 받는 도구의 경우, 도구 설명 내에 대상 API의 공식 문서 링크를 참조 형식으로 반드시 언급해야 합니다.

---

## 도구 설명 작성법 (Descriptions)

**도구 설명은 계약서와 같습니다.** Claude가 도구를 실행할지 여부를 결정할 때 읽는 유일한 정보입니다. 한 줄짜리 man page 항목처럼 직관적으로 기술하고, 다른 도구와 혼동하지 않도록 힌트를 추가해 명확히 작성하십시오.

### 올바른 예시

```
search_issues — 제목과 본문에서 키워드로 이슈를 검색합니다. 최근순으로 정렬된 매칭 결과 목록을 최대 `limit`개 반환합니다. 댓글이나 PR 내용은 검색하지 않습니다 — 해당 내용을 검색하려면 search_comments / search_prs 도구를 사용하십시오.
```

- 동작을 명확히 기술함
- 반환 값을 설명함
- 이 도구가 *할 수 없는* 동작을 규정함 (잘못된 도구를 선택하는 문제 방지)

### 잘못된 예시

```
search_issues — 이슈를 검색합니다.
```

Claude는 검색과 조금이라도 연관된 요청이 들어오면, 이 도구로 처리할 수 없는 작업임에도 무작정 호출해 버릴 수 있습니다.

### 유사한 도구 간의 구별 힌트 제공

비슷한 기능을 가진 두 도구가 있다면, 각 설명 부분에 상대 도구를 언제 사용해야 하는지 명시해 주는 것이 좋습니다:

```
get_user      — ID로 사용자를 조회합니다. 이메일 정보만 가지고 있는 경우에는 find_user_by_email 도구를 사용하십시오.
find_user_by_email — 이메일 주소로 사용자를 찾습니다. 매칭되는 사용자가 없으면 null을 반환합니다.
```

---

## 매개변수 스키마 (Parameter schemas)

**엄격한 스키마는 잘못된 도구 호출을 차단합니다.** 스키마에 제약 조건을 꼼꼼히 걸어둘수록, 런타임에 에러가 발생할 가능성이 줄어듭니다.

| 기존 작성 방식 | 권장 작성 방식 |
|---|---|
| ID에 `z.string()`만 사용 | `z.string().regex(/^usr_[a-z0-9]{12}$/)` 규칙 부여 |
| limit 값에 `z.number()`만 사용 | `z.number().int().min(1).max(100).default(20)` 제약 |
| 선택 값에 `z.string()` 사용 | `z.enum(["open", "closed", "all"])` 형태로 한정 |
| 설명 없이 optional 설정 | `.optional().describe("기본값은 호출한 사용자의 워크스페이스입니다.")` |

**모든 매개변수에 설명을 적어주십시오.** `.describe()`로 전달한 텍스트는 Claude가 확인하는 스키마 정보에 그대로 노출됩니다. 이를 누락하는 것은 중요한 최적화 기회를 잃는 것과 같습니다.

```typescript
{
  query: z.string().describe("검색할 키워드. 따옴표를 사용한 구절 검색도 지원합니다."),
  status: z.enum(["open", "closed", "all"]).default("open")
    .describe("상태 값 필터. 종료된 항목을 포함하려면 'all'을 사용하십시오."),
  limit: z.number().int().min(1).max(50).default(10)
    .describe("최대 결과 개수. 50개 이하로 제한됩니다."),
}
```

---

## 반환 값 형태 (Return shapes)

Claude는 `content[].text`에 담긴 데이터를 그대로 해석합니다. 파싱하기 쉬운 형태로 전달하십시오.

**권장 사항 (Do):**
- 구조화된 데이터는 JSON 문자열로 반환 (`JSON.stringify(result, null, 2)`)
- 수정 작업(mutation) 시 짧고 명확한 확인 메시지 반환 (`"Created issue #123"`)
- 후속 호출 시 Claude가 참조할 수 있는 식별용 ID 값 포함
- 페이로드가 지나치게 크다면 데이터를 축소하고 안내 메시지 제공 (`"847개 결과 중 10개만 표시 중입니다. 범위를 좁히려면 검색 조건을 더 명확히 하십시오."`)

**금지 사항 (Don't):**
- 가공되지 않은 raw HTML 반환
- 필터링되지 않은 수 메가바이트 크기의 raw API 응답 반환
- 식별자 정보 없이 성공 여부만 반환 (생성 작업 완료 후 단순 `"ok"`만 반환하면 Claude는 방금 무엇을 만들었는지 후속 작업에서 참조할 수 없음)

---

## 도구는 몇 개나 만들어야 하나요?

| 도구 개수 | 권장 가이드라인 |
|---|---|
| 1–15개 | 액션당 하나의 도구 구성. 가장 이상적입니다. |
| 15–30개 | 여전히 무리 없이 작동합니다. 병합할 수 있는 중복 성격의 도구가 없는지 점검하십시오. |
| 30개 이상 | '검색 후 실행(search + execute)' 패턴으로 전환하십시오. 핵심 3~5개만 전용 도구로 승격시키는 편이 좋습니다. |

도구 개수 제한은 프로토콜 규격상의 한계 때문이 아닌, 컨텍스트 창(context-window) 비용 때문입니다. 도구의 스케마 정보는 Claude가 *매 대화 턴마다* 소비하는 토큰이 됩니다. 엄격하고 풍부하게 작성된 스케마를 가진 도구가 30개가 넘어가면, 대화를 시작하기도 전에 3~5k의 토큰이 기초 비용으로 소비됩니다.

---

## 에러 처리 (Errors)

전송 계층(transport) 자체를 중단시키는 예외를 던지지 말고, MCP 도구 규격에 맞는 에러 객체를 반환하십시오. Claude가 문제 상황을 인지하고 우회 시도를 하거나 다른 값을 입력할 수 있도록 상세 정보를 담아야 합니다.

```typescript
if (!item) {
  return {
    isError: true,
    content: [{
      type: "text",
      text: `Item ${id} not found. Use search_items to find valid IDs.`,
    }],
  };
}
```

안내 메시지("use search_items...")를 덧붙여주면 Claude는 작업을 포기하는 대신 다음 작업을 유연하게 이어나갈 수 있습니다.

---

## 도구 주석 (Tool annotations)

호스트가 UI 구성에 활용하는 힌트 정보입니다 — 파괴적인 작업에는 붉은색 확인 버튼을 노출하고, 읽기 전용 작업은 권한 확인 과정을 자동으로 승인하도록 설정합니다. 기본적으로 지정되어 있지 않으면 호스트는 가장 안전하지 않은 동작으로 간주하여 보수적으로 처리합니다.

| 주석 (Annotation) | 의미 | 호스트 동작 |
|---|---|---|
| `readOnlyHint: true` | 부수 효과가 없음 | 자동 승인 처리할 수 있음 |
| `destructiveHint: true` | 삭제/덮어쓰기 동작 | 확인 대화창 노출 |
| `idempotentHint: true` | 재시도해도 안전함 | 일시적인 오류 발생 시 자동 재시도 |
| `openWorldHint: true` | 외부 세계와 통신함 (웹, 외부 API) | 네트워크 연결 아이콘 표시 가능 |

```typescript
server.registerTool("delete_file", {
  description: "Delete a file",
  inputSchema: { path: z.string() },
  annotations: { destructiveHint: true, idempotentHint: false },
}, handler);
```

```python
@mcp.tool(annotations={"destructiveHint": True, "idempotentHint": False})
def delete_file(path: str) -> str:
    ...
```

`build-mcpb/references/local-security.md`에 기재된 읽기/쓰기 분리 원칙과 함께 적용하십시오 — 모든 읽기 관련 도구에는 `readOnlyHint: true`를 지정해 줍니다.

---

## 구조화된 출력 (Structured output)

텍스트 블록에 `JSON.stringify(result)`를 담아 보내는 방식도 잘 동작하지만, MCP 스펙은 `outputSchema` + `structuredContent`라는 격식 있는 타입 정의형 출력을 지원합니다. 클라이언트 측에서 검증에 활용할 수 있습니다.

```typescript
server.registerTool("get_weather", {
  description: "Get current weather",
  inputSchema: { city: z.string() },
  outputSchema: { temp: z.number(), conditions: z.string() },
}, async ({ city }) => {
  const data = await fetchWeather(city);
  return {
    content: [{ type: "text", text: JSON.stringify(data) }],  // 하위 호환성 유지용
    structuredContent: data,                                    // 타입이 정의된 출력값
  };
});
```

아직 `structuredContent` 규격을 해석하지 못하는 호스트들이 존재하므로, 항상 텍스트 형태의 대체 텍스트(fallback text)를 함께 반환하여 주십시오.

---

## 텍스트 이외의 콘텐츠 타입 (Content types beyond text)

도구는 단순 문자열 외에도 다양한 데이터 형식을 반환할 수 있습니다:

| 타입 | 구조 | 주요 사용 사례 |
|---|---|---|
| `text` | `{ type: "text", text: string }` | 기본 텍스트 포맷 |
| `image` | `{ type: "image", data: base64, mimeType }` | 스크린샷, 차트, 다이어그램 등 이미지 정보 |
| `audio` | `{ type: "audio", data: base64, mimeType }` | TTS 음성 출력, 음성 녹음 내용 |
| `resource_link` | `{ type: "resource_link", uri, name?, description? }` | 포인터 정보 — 클라이언트가 나중에 직접 조회하도록 함 |
| `resource` (내장형) | `{ type: "resource", resource: { uri, text\|blob, mimeType } }` | 전체 데이터를 인라인으로 직접 포함 |

**`resource_link` vs 내장형(embedded):** 페이로드가 아주 크거나 클라이언트가 즉시 사용할 필요가 없을 때는 링크 형태가 유리합니다. 반면 데이터 크기가 작고 후속 판단에 항상 필요하다면 내장형 리소스를 사용하는 것이 좋습니다.
