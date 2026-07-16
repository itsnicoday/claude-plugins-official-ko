# Cloudflare Workers에 배포하기

작동 중인 `https://` MCP URL을 가장 신속하게 확보할 수 있는 방법입니다. 무료 플랜을 제공하며 시작할 때 신용카드가 필요 없고, 명령어 두 개만으로 배포할 수 있습니다.

**주의 사항(Trade-off):** 이 방식은 Workers 전용 스캐폴딩이며, `remote-http-scaffold.md`에 설명된 Express 스캐폴딩의 배포 대상이 아닙니다. 런타임 환경이 서로 다릅니다. 다양한 호스트 환경 간의 이식성(portability)이 중요하다면 Express 방식을 고수하십시오. 우선 라이브로 배포해 동작을 보고 싶다면 여기서 시작하십시오.

---

## 프로젝트 초기화 (Bootstrap)

```bash
npm create cloudflare@latest -- my-mcp-server \
  --template=cloudflare/ai/demos/remote-mcp-authless
cd my-mcp-server
```

이 명령어는 적합한 종속성(`agents`, `zod`)과 바로 사용 가능한 `wrangler.jsonc` 설정 파일이 포함된 초소형 템플릿을 내려받습니다.

---

## `src/index.ts`

템플릿에 들어 있는 계산기 예제 코드를 여러분의 도구로 대체하십시오. Express 스캐폴딩과 완전히 동일한 API 형태의 `registerTool()`을 사용합니다 (`McpServer` 인스턴스는 동일함):

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { McpAgent } from "agents/mcp";
import { z } from "zod";

export class MyMCP extends McpAgent {
  server = new McpServer(
    { name: "my-service", version: "0.1.0" },
    { instructions: "Prefer search_items before get_item — IDs aren't guessable." },
  );

  async init() {
    this.server.registerTool(
      "search_items",
      {
        description: "Search items by keyword. Returns up to `limit` matches.",
        inputSchema: {
          query: z.string().describe("Search keywords"),
          limit: z.number().int().min(1).max(50).default(10),
        },
        annotations: { readOnlyHint: true },
      },
      async ({ query, limit }) => {
        const results = await upstreamApi.search(query, limit);
        return { content: [{ type: "text", text: JSON.stringify(results, null, 2) }] };
      },
    );
  }
}

export default {
  fetch(request: Request, env: Env, ctx: ExecutionContext) {
    const url = new URL(request.url);
    if (url.pathname === "/mcp") {
      return MyMCP.serve("/mcp").fetch(request, env, ctx);
    }
    return new Response("Not found", { status: 404 });
  },
};
```

`McpAgent`는 Cloudflare가 제공하는 래퍼 클래스로, streamable-HTTP 전송 계층과 세션 라우팅, Durable Object 인프라 구성을 대신 처리해 줍니다. 개발자 코드는 오직 SDK의 `McpServer` 클래스인 `this.server`만 다루면 됩니다. `tool-design.md`와 `server-capabilities.md`에 설명된 모든 디자인 원칙이 똑같이 적용됩니다.

---

## `wrangler.jsonc`

템플릿에 기본적으로 포함된 파일입니다. Durable Objects 블록은 상용구(boilerplate) 코드로, `McpAgent`가 세션 상태를 저장하기 위해 내부적으로 DO를 사용합니다. 개발자가 이 영역을 직접 건드릴 필요는 없습니다.

```jsonc
{
  "name": "my-mcp-server",
  "main": "src/index.ts",
  "compatibility_date": "2025-03-10",
  "compatibility_flags": ["nodejs_compat"],
  "migrations": [{ "new_sqlite_classes": ["MyMCP"], "tag": "v1" }],
  "durable_objects": {
    "bindings": [{ "class_name": "MyMCP", "name": "MCP_OBJECT" }]
  }
}
```

만약 `MyMCP` 클래스 이름을 바꾸는 경우에는 `new_sqlite_classes`와 `class_name`에 적힌 이름도 함께 수정하십시오.

---

## 실행 및 배포 (Run and deploy)

```bash
npx wrangler dev     # → http://localhost:8787/mcp
npx wrangler deploy  # → https://my-mcp-server.<account>.workers.dev/mcp
```

`wrangler deploy` 명령어가 실행을 완료하면 라이브 URL을 출력합니다. 사용자가 Claude에 등록할 주소가 바로 이 URL입니다.

비밀 키 설정(외부 API 키 등): `npx wrangler secret put UPSTREAM_API_KEY`를 실행한 후, `init()` 메서드 안에서 `env.UPSTREAM_API_KEY` 값으로 읽어오십시오.

---

## OAuth

Cloudflare는 인증 서버 동작(CIMD/DCR 엔드포인트 제공, 토큰 발행, 동의 UI 렌더링)을 간편하게 해주는 `@cloudflare/workers-oauth-provider` 패키지를 제공합니다. 이 패키지는 `McpAgent`를 감싸서 `/mcp` 경로 접근 시 토큰 검증 과정을 적용합니다. 프로토콜 상세 사양은 `auth.md`를 참고하십시오. 구체적인 설정 연결 방식은 Cloudflare의 `cloudflare/ai/demos/remote-mcp-github-oauth` 템플릿을 참고하시면 됩니다.
