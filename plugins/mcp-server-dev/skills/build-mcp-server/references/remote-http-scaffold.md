# 원격 Streamable-HTTP MCP 서버 — 스캐폴딩

권장하는 두 가지 프레임워크 환경에서 작동하는 가장 기본적인 서버 구조입니다. 여기서 시작하여 도구를 추가해 나가십시오.

---

## TypeScript SDK (`@modelcontextprotocol/sdk`)

```bash
npm init -y
npm install @modelcontextprotocol/sdk zod express
npm install -D typescript @types/express @types/node tsx
```

**`src/server.ts`**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";
import { z } from "zod";

const server = new McpServer(
  { name: "my-service", version: "0.1.0" },
  { instructions: "Prefer search_items before calling get_item directly — IDs aren't guessable." },
);

// Pattern A: Action당 하나의 도구 구성
server.registerTool(
  "search_items",
  {
    description: "Search items by keyword. Returns up to `limit` matches ranked by relevance.",
    inputSchema: {
      query: z.string().describe("Search keywords"),
      limit: z.number().int().min(1).max(50).default(10),
    },
    annotations: { readOnlyHint: true },
  },
  async ({ query, limit }, extra) => {
    // extra.signal은 AbortSignal입니다 — 무거운 루프 구문에서 취소 요청을 확인하는 데 사용합니다.
    const results = await upstreamApi.search(query, limit);
    return {
      content: [{ type: "text", text: JSON.stringify(results, null, 2) }],
    };
  },
);

server.registerTool(
  "get_item",
  {
    description: "Fetch a single item by its ID.",
    inputSchema: { id: z.string() },
    annotations: { readOnlyHint: true },
  },
  async ({ id }) => {
    const item = await upstreamApi.get(id);
    return { content: [{ type: "text", text: JSON.stringify(item) }] };
  },
);

// Streamable HTTP transport (무상태(stateless) 모드 — 가장 간단한 방식)
const app = express();
app.use(express.json());

app.post("/mcp", async (req, res) => {
  const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: undefined, // stateless 설정
  });
  res.on("close", () => transport.close());
  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});

app.listen(process.env.PORT ?? 3000);
```

**무상태(stateless) vs 유상태(stateful):** 위의 코드 스니펫은 매 요청 시마다 새로운 트랜스포트를 만듭니다(무상태). 단순 API를 래핑하는 대다수의 서버에 이 방식이 적합합니다. 단일 세션 안에서 여러 번의 도구 호출 간에 상태를 공유해야 하는 경우(드문 경우)에는 세션 키 기반의 트랜스포트 맵을 사용하십시오 — SDK 제공 예제인 `examples/server/simpleStreamableHttp.ts`를 참고하십시오.

---

## FastMCP 3.x (Python)

```bash
pip install fastmcp
```

**`server.py`**

```python
from fastmcp import FastMCP

mcp = FastMCP(
    name="my-service",
    instructions="Prefer search_items before calling get_item directly — IDs aren't guessable.",
)

@mcp.tool(annotations={"readOnlyHint": True})
def search_items(query: str, limit: int = 10) -> list[dict]:
    """Search items by keyword. Returns up to `limit` matches ranked by relevance."""
    return upstream_api.search(query, limit)

@mcp.tool(annotations={"readOnlyHint": True})
def get_item(id: str) -> dict:
    """Fetch a single item by its ID."""
    return upstream_api.get(id)

if __name__ == "__main__":
    mcp.run(transport="http", host="0.0.0.0", port=3000)
```

FastMCP는 함수에 지정한 타입 힌트(type hints) 정보로부터 JSON 스키마를 유추하고, docstring은 도구 설명이 됩니다. docstring은 간결하고 동작 지향적으로 작성하십시오 — 해당 내용이 있는 그대로 Claude의 컨텍스트 창에 입력됩니다.

---

## 검색 후 실행 패턴 (대규모 API 집합인 경우)

바인딩해야 할 API 엔드포인트가 50개가 넘어가는 대규모 스케일이라면, 이를 일일이 다 개별 도구로 등록하지 마십시오. 두 개의 도구만 구현하여 이 패턴을 사용합니다:

```typescript
const CATALOG = loadActionCatalog(); // { id, description, paramSchema }[]

server.registerTool(
  "search_actions",
  {
    description: "Find available actions matching an intent. Call this first to discover what's possible. Returns action IDs, descriptions, and parameter schemas.",
    inputSchema: { intent: z.string().describe("What you want to do, in plain English") },
    annotations: { readOnlyHint: true },
  },
  async ({ intent }) => {
    const matches = rankActions(CATALOG, intent).slice(0, 10);
    return { content: [{ type: "text", text: JSON.stringify(matches, null, 2) }] };
  },
);

server.registerTool(
  "execute_action",
  {
    description: "Execute an action by ID. Get the ID and params schema from search_actions first.",
    inputSchema: {
      action_id: z.string(),
      params: z.record(z.unknown()),
    },
  },
  async ({ action_id, params }) => {
    const action = CATALOG.find(a => a.id === action_id);
    if (!action) throw new Error(`Unknown action: ${action_id}`);
    validate(params, action.paramSchema);
    const result = await dispatch(action, params);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  },
);
```

`rankActions` 알고리즘은 시작 단계에서는 단순한 키워드 매칭만으로도 충분합니다. 정확도가 더 필요해지면 텍스트 임베딩 모델(embeddings)을 통한 검색 기법으로 업그레이드하십시오.

---

## 작동 확인 및 테스트 (Test it)

MCP 인스펙터(Inspector) 도구를 사용해 구동 중인 트랜스포트에 연결하고 대화형 UI를 통해 도구를 직접 실행해 볼 수 있습니다.

```bash
# 대화형 테스트 — localhost:6274 경로로 웹 UI를 실행합니다.
npx @modelcontextprotocol/inspector
# → UI 화면에서 "Streamable HTTP" 선택 후, http://localhost:3000/mcp 주소 입력, Connect 버튼 클릭
```

스크립트를 통한 테스트 검증 (CI 검증, 스모크 테스트 용):

```bash
npx @modelcontextprotocol/inspector --cli http://localhost:3000/mcp \
  --transport http --method tools/list

npx @modelcontextprotocol/inspector --cli http://localhost:3000/mcp \
  --transport http --method tools/call --tool-name search_items --tool-arg query=test
```

---

## 사용자 연결 방법 (Connect users)

서버가 외부 웹상에 배포되고 나면, 사용자는 별도의 로컬 설치 과정 없이 해당 URL을 등록하여 바로 사용할 수 있습니다.

| 사용 환경 | 등록 방법 |
|---|---|
| **Claude Code** | `claude mcp add --transport http <name> <url>` 실행 (전역 설정은 `--scope user` 추가, 인증 토큰 전달이 필요하다면 `--header "Authorization: Bearer ..."` 추가) |
| **Claude Desktop / Claude.ai** | 설정 → Connectors → Add custom connector 선택 등록. **주의:** `claude_desktop_config.json`에 원격 서버 주소를 기재하면 무시됩니다. |
| **Connector 디렉토리** | Anthropic은 일반 공개 커넥터 디렉토리에 제출하여 게시하려는 개발자들을 위한 공식 가이드라인을 별도로 제공하고 있습니다. |

---

## 배포 가이드 (Deploy)

**가장 빠른 방법:** Cloudflare Workers 사용 — 무료 티어로 어떠한 카드 결제 등록 없이, 단 두 개의 명령어로 라이브 `https://` 주소를 확보할 수 있습니다. Workers 전용의 스캐폴딩 템플릿을 기반으로 합니다. → `deploy-cloudflare-workers.md` 참고

**위의 Express 스캐폴딩 코드**는 Render, Railway, Fly.io, 독립 가상 머신(VPS) 등 Node.js 런타임을 구동할 수 있는 모든 호스트 환경에서 돌아갑니다. Docker 컨테이너 이미지로 빌드해 (`node:20-slim` 베이스, 빌드 결과 복사, `npm ci`, `node dist/server.js` 구동) 손쉽게 배포할 수 있습니다. FastMCP 역시 파이썬 베이스 이미지를 사용해 동일하게 진행하면 됩니다.

---

## 배포 전 점검 목록 (Deployment checklist)

- [ ] `POST /mcp` 엔드포인트가 `initialize` 호출 시 서버의 capabilities를 정상 반환하는가
- [ ] `tools/list`가 정의된 도구 목록과 완전한 형태의 스키마들을 온전히 반환하는가
- [ ] 오류 상황 발생 시 HTML 바디를 포함한 HTTP 500 에러를 던지지 않고, MCP 표준 규격에 맞는 에러 객체를 반환하는가
- [ ] 브라우저 환경 기반의 클라이언트가 연동되어 들어오는 경우 CORS 설정이 정상 적용되어 있는가
- [ ] `/mcp`로 인입되는 요청에 대해 `Origin` 헤더 검증 과정을 수행하는가 (스펙 MUST 사양 — DNS 바인딩 우회 공격 방지)
- [ ] `MCP-Protocol-Version` 헤더 정보를 파악해 미지원 버전 요청에 대해 400 에러를 내려주는가
- [ ] 도구 호출의 힌트를 제공해야 하는 경우 `instructions` field가 정상 기재되어 있는가
- [ ] 호스트의 헬스체크 폴링을 받기 위한 별도의 헬스체크 엔드포인트가 `/mcp`와 구분되어 설계되어 있는가
- [ ] 외부 비밀 키 정보는 소스코드에 하드코딩되지 않고 환경 변수로 주입받도록 했는가
- [ ] OAuth를 적용하는 경우, CIMD 또는 DCR 엔드포인트가 적절히 구현되었는가 — `auth.md` 참고
