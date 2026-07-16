# Resources & Prompts — 나머지 2개의 핵심 요소

MCP는 서버 측의 3대 핵심 요소(primitives)를 정의합니다. Tools(도구)는 모델이 제어하지만(Claude가 호출 시점을 결정), 나머지 2개는 성격이 다릅니다:

- **Resources (리소스)**는 애플리케이션이 제어함 — 호스트가 컨텍스트로 불러올 대상을 결정
- **Prompts (프롬프트)**는 사용자가 제어함 — 슬래시 명령어 또는 메뉴 항목으로 표출됨

대부분의 서버는 Tools만으로 충분합니다. 구현하려는 연동 형태가 "Claude가 특정 함수를 호출한다"는 개념에 어울리지 않을 때 이 기능들을 고려하십시오.

---

## Resources (리소스)

리소스를 식별하는 식별자는 URI입니다. 도구와 달리 리소스는 '호출'되지 않고 '읽기'만 지원합니다. 호스트가 사용 가능한 리소스 목록을 훑어보고 컨텍스트에 로드할 대상을 결정합니다.

**리소스가 도구보다 나은 경우:**
- Claude가 직접 탐색해야 하는 대량의 참조 데이터 (문서, 스케마, 설정 파일 등)
- 대화 흐름과 독립적으로 변경되는 정보 (로그 파일, 실시간 데이터)
- "Claude가 직접 판단해서 데이터를 가져온다"는 모델링이 적절하지 않은 모든 데이터

**도구가 더 나은 경우:**
- 실행 과정에서 부수 효과(side effects)가 발생할 때
- 실행 결과가 Claude가 선택하는 매개변수(parameter)에 영향을 받을 때
- 호스트 UI가 아닌 Claude가 적절한 시점을 판단하여 직접 데이터를 끌어오게 하고 싶을 때

### 정적 리소스 (Static resources)

```typescript
// TypeScript SDK
server.registerResource(
  "config",
  "config://app/settings",
  { name: "App Settings", description: "Current configuration", mimeType: "application/json" },
  async (uri) => ({
    contents: [{ uri: uri.href, mimeType: "application/json", text: JSON.stringify(config) }],
  }),
);
```

```python
# fastmcp
@mcp.resource("config://app/settings")
def get_settings() -> str:
    """Current application configuration."""
    return json.dumps(config)
```

### 동적 리소스 (Dynamic resources — URI 템플릿)

RFC 6570 템플릿을 사용하면 단 한 번의 등록으로 여러 개의 URI를 지원할 수 있습니다:

```typescript
import { ResourceTemplate } from "@modelcontextprotocol/sdk/server/mcp.js";

server.registerResource(
  "file",
  new ResourceTemplate("file:///{path}", { list: undefined }),
  { name: "File", description: "Read a file from the workspace" },
  async (uri, { path }) => ({
    contents: [{ uri: uri.href, text: await fs.readFile(path, "utf8") }],
  }),
);
```

```python
@mcp.resource("file:///{path}")
def read_file(path: str) -> str:
    return Path(path).read_text()
```

### 리소스 구독 (Subscriptions)

리소스의 데이터가 변경되었을 때 클라이언트에 알림을 보낼 수 있습니다. capabilities에 `subscribe: true`를 선언하고, 리소스 변경 시 `notifications/resources/updated` 알림을 발행하면 호스트가 데이터를 다시 읽어옵니다. 로그 추적, 실시간 대시보드, 변경 감지 파일에 유용합니다.

---

## Prompts (프롬프트)

프롬프트는 매개변수화된 메시지 템플릿입니다. 호스트는 이를 슬래시(/) 명령어 또는 메뉴 항목으로 표출합니다. 사용자가 이를 선택하여 인자값을 입력하면, 완성된 메시지가 대화창에 삽입됩니다.

**사용 시점:** 사용자가 반복해서 실행하는 고정된 작업 흐름이 있을 때 사용합니다 (예: `/summarize-thread`, `/draft-reply`, `/explain-error`). 코드가 거의 들어가지 않으면서 UX 가치가 매우 높습니다.

```typescript
server.registerPrompt(
  "summarize",
  {
    title: "Summarize document",
    description: "Generate a concise summary of the given text",
    argsSchema: { text: z.string(), max_words: z.string().optional() },
  },
  ({ text, max_words }) => ({
    messages: [{
      role: "user",
      content: { type: "text", text: `Summarize in ${max_words ?? "100"} words:\n\n${text}` },
    }],
  }),
);
```

```python
@mcp.prompt
def summarize(text: str, max_words: str = "100") -> str:
    """Generate a concise summary of the given text."""
    return f"Summarize in {max_words} words:\n\n{text}"
```

**제약 조건:**
- 인자값은 **문자열(string-only) 타입**으로만 전달됩니다 (숫자, boolean, 객체 불가). 필요한 경우 핸들러 내부에서 직접 타입을 캐스팅하십시오.
- `messages[]` 배열 구조로 반환해야 합니다 — 텍스트뿐만 아니라 포함된 리소스나 이미지도 넣을 수 있습니다.
- 부수 효과가 없어야 합니다 — 핸들러는 메시지 객체를 조립해서 반환하기만 할 뿐, 실제 작업을 직접 실행해서는 안 됩니다.

---

## 요약 의사결정표 (Quick decision table)

| 목적 | 선택할 요소 |
|---|---|
| Claude가 매개변수를 지정해 동적으로 데이터를 가져오게 할 때 | **Tool (도구)** |
| 훑어볼 수 있는 컨텍스트(파일, 문서, 스키마 등)를 제공할 때 | **Resource (리소스)** |
| 동적 URI 패턴을 가진 데이터 세트를 노출할 때 (`db://{table}`) | **Resource template (리소스 템플릿)** |
| 사용자에게 원클릭 작업을 제공하고 싶을 때 | **Prompt (프롬프트)** |
| 도구 실행 중간에 사용자에게 무언가 질문하고 싶을 때 | **Elicitation (사용자 입력 유도)** (자세한 내용은 `elicitation.md` 참고) |
