---
name: build-mcp-app
description: This skill should be used when the user wants to build an "MCP app", add "interactive UI" or "widgets" to an MCP server, "render components in chat", build "MCP UI resources", make a tool that shows a "form", "picker", "dashboard" or "confirmation dialog" inline in the conversation, or mentions "apps SDK" in the context of MCP. Use AFTER the build-mcp-server skill has settled the deployment model, or when the user already knows they want UI widgets.
version: 0.1.0
---

# MCP App 구축하기 (대화형 UI 위젯)

MCP app is a standard MCP server that **also serves UI resources** — interactive components rendered inline in the chat surface. Build once, runs in Claude *and* ChatGPT and any other host that implements the apps surface.

UI 레이어는 **추가적인 결합 요소(additive)**입니다. 밑단에서는 여전히 이전과 동일한 도구(tools), 리소스(resources) 및 프로토콜 규격으로 돌아갑니다. 일반적인 형태의 MCP 서버를 개발해 본 적이 없다면, `build-mcp-server` skill을 통해 기초 환경 설계를 먼저 숙지하십시오. 이 skill은 그 기초 위에 화면 위젯을 얹는 지침을 다룹니다.

> **Claude 환경에서의 테스트:** claude.ai 환경에 커넥터로 서버를 등록하십시오 (로컬 개발 환경인 경우 Cloudflare 터널 서비스를 사용해 우회 주소를 확보해 진행). 이를 통해 실제 iframe 샌드박스 정책과 `hostContext` 상속이 어떻게 반영되는지 검증할 수 있습니다. 가이드는 https://claude.com/docs/connectors/building/testing 문서를 참고하십시오.

## Claude 호스트 특화 규격 명세

| `_meta.ui.*` 속성 키 | 기재 위치 | 효과 |
|---|---|---|
| `resourceUri` | tool (도구) | 이 도구의 실행 결과 시각화를 위해 호스트가 로드해야 할 `ui://` 리소스 주소. |
| `visibility: ["app"]` | tool (도구) | Claude의 일반 대화 도구 추천 목록에서 위젯 전용 내부 도우미 도구(예: `callServerTool` 호출로 위젯이 내부적으로 사용하는 이미지/데이터 로더)를 숨김 처리함. |
| `prefersBorder: false` | resource (리소스) | 모바일 화면 등에서 호스트가 위젯 바깥 테두리에 씌우는 카드 보더 선을 제거함. |
| `csp.{connectDomains, resourceDomains, baseUriDomains}` | resource (리소스) | 연동을 위해 외부로 통신이 허용되어야 할 도메인 화이트리스트 주소 선언. 기본값은 모든 외부 요청 차단. (현재 Claude 내에서 `frameDomains` 제어는 제한적임) |

- `hostContext.safeAreaInsets: {top, right, bottom, left}` (px) — 기기별 노치 디자인이나 작문 영역 입력 컴포저 오버레이와의 겹침 방지용 패딩 값으로 이를 준수해야 합니다.
- 공용 디렉토리 등록을 위해서는 OAuth 또는 **인증 없음**(`none`) 사양이 필수입니다 — 고정 static bearer 토큰 방식은 사설 배포만 허용되고 공용 등록이 차단됩니다 — 추가로 도구의 `annotations` 명시와 3~5장의 PNG 스크린샷이 필요합니다. 상세 지침은 `references/directory-checklist.md` 문서를 참고하십시오.

---

## 화면 위젯(Widget)이 텍스트보다 나은 경우

화려함만을 이유로 UI를 억지로 끼워 넣지는 마십시오 — 대다수의 도구는 단순 텍스트나 JSON 응답으로도 매우 잘 돌아갑니다. 아래 사례에 해당하는 경우에만 위젯 연동을 결정하십시오:

| 연동 시그널 | 적합한 위젯 유형 |
|---|---|
| 도구 호출에 필요한 매개변수 구조를 Claude가 스스로 완벽히 유추하기 힘들 때 | 입력 폼 (Form) |
| 파일 일람, 주소록 연락처, 다량의 레코드 중 사용자가 손으로 직접 골라야 할 때 | 피커 / 테이블 뷰 |
| 리소스가 많이 소모되거나 결제 등 파괴적인 액션에 대해 명시적 승인이 요구될 때 | 확인 대화창 (Confirm dialog) |
| 출력 데이터 자체가 지리 정보나 통계 등 시각화가 본질일 때 | 차트 / 지도 / 미리보기 위젯 |
| 시간이 오래 걸리는 연산이 돌아가고 있어 실시간 진행도를 보여주고 싶을 때 | 로딩 상태 바 / 실시간 현황판 |

위 케이스에 해당하지 않는다면 위젯 추가 단계를 건너뛰십시오. 텍스트 방식으로 개발하는 것이 연산 속도도 빠르고 사용자 인터랙션도 더 가볍습니다.

---

## 위젯 vs Elicitation — 올바른 설계 경로 판단

위젯을 새로 그리는 설계를 밟기 전에, 단순 표준 기능인 **elicitation**으로 대체할 수 없는지 먼저 검토해 보십시오. elicitation은 UI 개발 공수가 제로이며 프로토콜 내장 표준 동작입니다.

| 기능 필요 사양 | Elicitation 기능 범위 | 위젯(Widget) 필요 범위 |
|---|---|---|
| 예/아니오 단순 확인 | ✅ 지원 가능 | 위젯을 설계하는 것은 오버스펙 |
| 짧은 리스트 중 단일 값 선택 | ✅ 지원 가능 | 위젯을 설계하는 것은 오버스펙 |
| 1차원 구조의 짧은 양식 기재 (이름, 메일 주소, 날짜 등) | ✅ 지원 가능 | 위젯을 설계하는 것은 오버스펙 |
| 대량이거나 내부 키워드 검색이 필요한 목록 선택 | ❌ (스크롤/검색 미지원) | ✅ 위젯 필요 |
| 선택 전에 시각적 미리보기 필요 (썸네일, 사진 등) | ❌ | ✅ 위젯 필요 |
| 통계 차트 / 지도 데이터 / 코드 디프(diff) 화면 표시 | ❌ | ✅ 위젯 필요 |
| 실시간으로 갱신되는 로딩 진척 표시 | ❌ | ✅ 위젯 필요 |

elicitation 범위 내에서 해결 가능한 구조라면 무조건 elicitation을 쓰십시오. 가이드는 `../build-mcp-server/references/elicitation.md` 문서를 참고하십시오.

---

## 아키텍처: 두 가지 배포 시나리오

### 원격 MCP App 구조 (가장 널리 쓰임)

Hosted streamable-HTTP server. Widget templates are served as **resources**; tool results reference them. The host fetches the resource, renders it in an iframe sandbox, and brokers messages between the widget and Claude.

```
┌──────────┐  tools/call   ┌────────────┐
│  Claude  │─────────────> │ MCP server │
│  호스트  │<── result ────│  (remote)  │
│          │  + widget ref │            │
│          │               │            │
│          │ resources/read│            │
│          │─────────────> │  widget    │
│ ┌──────┐ │<── template ──│  HTML/JS   │
│ │iframe│ │               └────────────┘
│ │widget│ │
│ └──────┘ │
└──────────┘
```

### MCPB 패키징 방식의 MCP App 구조 (로컬 구동 + 대화창 UI 연동)

Same widget mechanism, but the server runs locally inside an MCPB bundle. Use this when the widget needs to drive a **local** application — e.g., a file picker that browses the actual local disk, a dialog that controls a desktop app.

MCPB 패키징 빌드 기법에 대해서는 **`build-mcpb`** skill을 통해 상세 안내를 이어나가십시오. 아래에 다루는 UI 구현 지침은 두 아키텍처 환경 모두에 공통 적용되는 스펙입니다.

---

## 위젯 화면과 도구의 연결 매커니즘

A widget-enabled tool has **two separate registrations**:

1. **The tool** declares a UI resource via `_meta.ui.resourceUri`. Its handler returns plain text/JSON — NOT the HTML.
2. **The resource** is registered separately and serves the HTML.

Claude가 도구를 실행하면 호스트는 `_meta.ui.resourceUri` 메타 정보를 발견해 해당 리소스 주소로 HTML 파일을 조회하고, 이를 iframe 위에 렌더링한 뒤, 도구가 뱉어낸 순수 반환값 데이터를 `ontoolresult` 이벤트 채널을 통해 해당 iframe으로 실어 보냅니다.

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { registerAppTool, registerAppResource, RESOURCE_MIME_TYPE }
  from "@modelcontextprotocol/ext-apps/server";
import { z } from "zod";

const server = new McpServer({ name: "contacts", version: "1.0.0" });

// 1. 도구 등록 — 데이터(DATA)를 반환하고, 보여줄 UI 정보 명시
registerAppTool(server, "pick_contact", {
  description: "Open an interactive contact picker",
  annotations: { title: "Pick Contact", readOnlyHint: true },
  inputSchema: { filter: z.string().optional() },
  _meta: { ui: { resourceUri: "ui://widgets/contact-picker.html" } },
}, async ({ filter }) => {
  const contacts = await db.contacts.search(filter);
  // 순수 JSON 반환 — 위젯이 ontoolresult로 전달받을 데이터
  return { content: [{ type: "text", text: JSON.stringify(contacts) }] };
});

// 2. 리소스 등록 — 위젯 HTML 파일 내용 서빙
registerAppResource(
  server,
  "Contact Picker",
  "ui://widgets/contact-picker.html",
  {},
  async () => ({
    contents: [{
      uri: "ui://widgets/contact-picker.html",
      mimeType: RESOURCE_MIME_TYPE,
      text: pickerHtml,  // HTML 소스 코드 문자열
    }],
  }),
);
```

사용할 URI 스키마명은 관례적으로 `ui://`를 씁니다. 리소스 MIME 타입은 반드시 `RESOURCE_MIME_TYPE` 규격인 `"text/html;profile=mcp-app"` 값을 설정해 주어야만, 호스트가 이것을 단순 코드 텍스트 뷰어로 뿌리지 않고 렌더링용 iframe 창을 띄워 실행시킵니다.

---

## 위젯 런타임 환경 — `App` 클래스 연동

 Inside the iframe, your script talks to the host via the `App` class from `@modelcontextprotocol/ext-apps`. This is a **persistent bidirectional connection** — the widget stays alive as long as the conversation is active, receiving new tool results and sending user actions.

```html
<script type="module">
  /* 빌드 타임에 인라인 치환되어 globalThis.ExtApps 위치에 상주 */
  /*__EXT_APPS_BUNDLE__*/
  const { App } = globalThis.ExtApps;

  const app = new App({ name: "ContactPicker", version: "1.0.0" }, {});

  // 연결(connect)을 수행하기 전에 항상 이벤트 수신 핸들러를 정의해 둠
  app.ontoolresult = ({ content }) => {
    const contacts = JSON.parse(content[0].text);
    render(contacts);
  };

  await app.connect();

  // 사용자가 화면에서 연락처 카드를 골라 클릭했을 때 호출될 함수 예시:
  function onPick(contact) {
    app.sendMessage({
      role: "user",
      content: [{ type: "text", text: `Selected contact: ${contact.id}` }],
    });
  }
</script>
```

정적 HTML 안의 `/*__EXT_APPS_BUNDLE__*/` 주석 영역은 서버 기동 시점에 `@modelcontextprotocol/ext-apps/app-with-deps` 파일 모듈 코드로 자동 치환됩니다 — 자세한 번들 인라이닝 치환 로직과 필요성에 대해서는 `references/iframe-sandbox.md` 문서를 참고하십시오. **절대로** `import { App } from "https://esm.sh/..."`와 같이 외부 주소로 스크립트를 import 호출하지 마십시오; iframe 샌드박스의 CSP 보안 정책이 외부 ESM 연계 종속성 조회를 도중에 차단해 버려 화면에 에러조차 찍히지 않고 하얗게 굳어버리게 됩니다.

| 주요 API | 통신 방향 | 활용 목적 |
|---|---|---|
| `app.ontoolresult = fn` | 호스트 → 위젯 | 도구가 리턴해 준 가공된 연산 결과 데이터 수집 |
| `app.ontoolinput = fn` | 호스트 → 위젯 | Claude가 도구를 실행할 때 넣은 입력 인자값 수집 |
| `app.sendMessage({...})` | 위젯 → 호스트 | 사용자 대화 이력 타임라인에 명시적 메시지 전송 |
| `app.updateModelContext({...})` | 위젯 → 호스트 | 채팅창을 도배하지 않고, 백그라운드 인지 상태값만 유기적으로 동기화 |
| `app.callServerTool({name, arguments})` | 위젯 → 서버 | 서버 측에 마련된 다른 보조 도구 함수 직접 호출 |
| `app.openLink({url})` | 위젯 → 호스트 | 새 브라우저 창으로 주소 열기 (샌드박스로 인한 `window.open` 차단 대응용) |
| `app.getHostContext()` / `app.onhostcontextchanged` | 호스트 → 위젯 | 테마 상태, 호스트 CSS 연계 변수, 표시 크기, 디스플레이 모드, 입력 장치 사양 조회 |
| `app.requestDisplayMode({mode})` | 위젯 → 호스트 | 화면 표시 상태 변경 요청 (`inline`, `pip`, `fullscreen` 등) |
| `app.downloadFile({name, mimeType, content})` | 위젯 → 호스트 | 샌드박스 우회 방식의 바이너리 파일 다운로드 처리 (base64 전달) |
| `new App(info, caps, {autoResize: true})` | — | 위젯 높이를 그리고 있는 콘텐츠 높이에 맞추어 동적으로 가변 조정 |

`sendMessage`는 "사용자가 선택을 끝냈으니 Claude에게 알려 대화를 진행하자"는 시나리오의 표준 경로입니다. `updateModelContext`는 진행 중인 상태 정보를 Claude에게 상시 업데이트해 알려주고는 싶지만 사용자의 채팅 말풍선 영역을 어지럽히고 싶지 않을 때 사용합니다. `openLink`는 외부 주소로 링크를 아웃링크 처리할 때 **필수**로 써야 하는 우회 API입니다.

**위젯 샌드박스 내부에서 불가능한 작업들:**
- 호스트 모(parent) 화면의 DOM 트리 접근, 쿠키 정보 조회 및 브라우저 로컬 스토리지에 직접 접근하는 행위
- 도메인 간의 직접적인 외부 API `fetch()` 요청 호출 (CSP 정책으로 제한됨 — 필요한 통신은 `callServerTool` 우회 방식으로 해결)
- 직접 팝업 레이어를 띄우거나 페이지를 즉시 네비게이션으로 이동하는 행위 — `app.openLink`를 활용하십시오.
- 외부 정적 이미지 파일 주소를 이미지 태그로 직접 로드하는 행위 — 서버 단에서 미리 받아 base64 `data:` 이미지 규격으로 내려주십시오.

위젯은 **단일한 역할과 간결한 구조**를 유지하십시오. 피커는 조회 선택만 하고, 차트는 시각화만 그리게 하십시오. iframe 안에 거대한 서브 애플리케이션 전체를 몽땅 설계해 얹지 말고, 용도가 명확한 개별 도구와 이에 맞물린 단순한 위젯들로 파편화하여 연동하는 구조가 Claude가 활용하기 훨씬 좋습니다.

---

## 최소한의 피커 위젯 구현 예제 스캐폴딩

**종속성 라이브러리 추가:**

```bash
npm install @modelcontextprotocol/sdk @modelcontextprotocol/ext-apps zod express
```

**서버 로직 (`src/server.ts`):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { registerAppTool, registerAppResource, RESOURCE_MIME_TYPE }
  from "@modelcontextprotocol/ext-apps/server";
import express from "express";
import { readFileSync } from "node:fs";
import { createRequire } from "node:module";
import { z } from "zod";

const require = createRequire(import.meta.url);
const server = new McpServer({ name: "contact-picker", version: "1.0.0" });

// 브라우저 ext-apps SDK 모듈을 정적 HTML 내부에 인라인 치환하기 위한 준비 과정입니다.
// iframe CSP 정책상 외부 스크립트 로드가 제한되므로 이 빌드 처리는 필수 사항입니다.
const bundle = readFileSync(
  require.resolve("@modelcontextprotocol/ext-apps/app-with-deps"), "utf8",
).replace(/export\{([^}]+)\};?\s*$/, (_, body) =>
  "globalThis.ExtApps={" +
  body.split(",").map((p) => {
    const [local, exported] = p.split(" as ").map((s) => s.trim());
    return `${exported ?? local}:${local}`;
  }).join(",") + "};",
);
const pickerHtml = readFileSync("./widgets/picker.html", "utf8")
  .replace("/*__EXT_APPS_BUNDLE__*/", () => bundle);

registerAppTool(server, "pick_contact", {
  description: "Open an interactive contact picker. User selects one contact.",
  annotations: { title: "Pick Contact", readOnlyHint: true },
  inputSchema: { filter: z.string().optional().describe("Name/email prefix filter") },
  _meta: { ui: { resourceUri: "ui://widgets/picker.html" } },
}, async ({ filter }) => {
  const contacts = await db.contacts.search(filter ?? "");
  return { content: [{ type: "text", text: JSON.stringify(contacts) }] };
});

registerAppResource(server, "Contact Picker", "ui://widgets/picker.html", {},
  async () => ({
    contents: [{ uri: "ui://widgets/picker.html", mimeType: RESOURCE_MIME_TYPE, text: pickerHtml }],
  }),
);

const app = express();
app.use(express.json());
app.post("/mcp", async (req, res) => {
  const transport = new StreamableHTTPServerTransport({ sessionIdGenerator: undefined });
  res.on("close", () => transport.close());
  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});
app.listen(process.env.PORT ?? 3000);
```

만약 데스크톱 앱을 직접 다루거나 로컬 파일을 다루기 위해 로컬에서만 구동하는 위젯 앱을 만들고자 한다면, 통신 방식을 `StdioServerTransport`로 교체하고 `build-mcpb` skill 규칙에 맞춰 패키징하여 구성하십시오.

**위젯 파일 (`widgets/picker.html`):**

```html
<!doctype html>
<meta charset="utf-8" />
<style>
  body { font: 14px system-ui; margin: 0; }
  ul { list-style: none; padding: 0; margin: 0; max-height: 300px; overflow-y: auto; }
  li { padding: 10px 14px; cursor: pointer; border-bottom: 1px solid #eee; }
  li:hover { background: #f5f5f5; }
  .sub { color: #666; font-size: 12px; }
</style>
<ul id="list"></ul>
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "ContactPicker", version: "1.0.0" }, {});
  const ul = document.getElementById("list");

  app.ontoolresult = ({ content }) => {
    const contacts = JSON.parse(content[0].text);
    ul.innerHTML = "";
    for (const c of contacts) {
      const li = document.createElement("li");
      li.innerHTML = `<div>${c.name}</div><div class="sub">${c.email}</div>`;
      li.addEventListener("click", () => {
        app.sendMessage({
          role: "user",
          content: [{ type: "text", text: `Selected contact: ${c.id} (${c.name})` }],
        });
      });
      ul.append(li);
    }
  };

  await app.connect();
})();
</script>
```

다양한 형태의 UI 조각 템플릿들은 `references/widget-templates.md` 문서를 참고하십시오.

---

## 뼈아픈 리팩토링 예방을 위한 핵심 설계 팁

**One widget per tool.** Resist the urge to build one mega-widget that does everything. One tool → one focused widget → one clear result shape. Claude reasons about these far better.

**도구의 자연어 설명 부분에 '위젯 노출' 사실을 기재하십시오.** Claude는 오직 도구의 설명 구문만 읽고 무엇을 호출할지 결정합니다. 설명 부분에 "대화형 피커 창을 열어 사용자로부터 입력을 받습니다"라는 힌트 문구를 적어주어야만 Claude가 멋대로 ID 값을 추측해 텍스트 입력창으로 진행하지 않고 이 위젯 도구를 트리거하여 사용자에게 창을 띄워 줍니다.

**Widgets are optional at runtime.** Hosts that don't support the apps surface simply ignore `_meta.ui` and render the tool's text content normally. Since your tool handler already returns meaningful text/JSON (the widget's data), degradation is automatic — Claude sees the data directly instead of via the widget.

**Don't block on widget results for read-only tools.** A widget that just *displays* data (chart, preview) shouldn't require a user action to complete. Return the display widget *and* a text summary in the same result so Claude can continue reasoning without waiting.

**Layout-fork by item count, not by tool count.** If one use case is "show one result in detail" and another is "show many results side-by-side", don't make two tools — make one tool that accepts `items[]`, and let the widget pick a layout: `items.length === 1` → detail view, `> 1` → carousel. Keeps the server schema simple and lets Claude decide count naturally.

**Put Claude's reasoning in the payload.** A short `note` field on each item (why Claude picked it) rendered as a callout on the card gives users the reasoning inline with the choice. Mention this field in the tool description so Claude populates it.

**Normalize image shapes server-side.** If your data source returns images with wildly varying aspect ratios, rewrite to a predictable variant (e.g. square-bounded) *before* fetching for the data-URL inline. Then give the widget's image container a fixed `aspect-ratio` + `object-fit: contain` so everything sits centered.

**Follow host theme.** `app.getHostContext()?.theme` (after `connect()`) plus `app.onhostcontextchanged` for live updates. Toggle a `.dark` class on `<html>`, keep colors in CSS custom props with a `:root.dark {}` override block, set `color-scheme`. Disable `mix-blend-mode: multiply` in dark — it makes images vanish.

---

## 작동 확인 및 테스트 방법

**Claude Desktop** — current builds still require the `command`/`args` config shape (no native `"type": "http"`). Wrap with `mcp-remote` and force `http-only` transport so the SSE probe doesn't swallow widget-capability negotiation:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://localhost:3000/mcp",
               "--allow-http", "--transport", "http-only"]
    }
  }
}
```

Desktop caches UI resources aggressively. After editing widget HTML, **fully quit** (⌘Q / Alt+F4, not window-close) and relaunch to force a cold resource re-fetch.

**Headless JSON-RPC loop** — fast iteration without clicking through Desktop:

```bash
# test.jsonl — one JSON-RPC message per line
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"t","version":"0"}}}
{"jsonrpc":"2.0","method":"notifications/initialized"}
{"jsonrpc":"2.0","id":2,"method":"tools/list"}
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"your_tool","arguments":{...}}}

(cat test.jsonl; sleep 10) | npx mcp-remote http://localhost:3000/mcp --allow-http
```

The `sleep` keeps stdin open long enough to collect all responses. Parse the jsonl output with `jq` or a Python one-liner.

**Widget dev loop** — avoid the ⌘Q-relaunch cycle entirely by serving the inlined widget HTML at a plain GET route with a fake `ExtApps` shim that fires `ontoolresult` from a query param:

```ts
app.get("/widget-preview", (_req, res) => {
  const shim = `globalThis.ExtApps={applyHostStyleVariables:()=>{},App:class{
    constructor(){this.h={}} ontoolresult;onhostcontextchanged;
    async connect(){const p=new URLSearchParams(location.search).get("payload");
      if(p)this.ontoolresult?.({content:[{type:"text",text:p}]});}
    getHostContext(){return{theme:"light"}}
    sendMessage(m){console.log("sendMessage",m)} updateModelContext(){}
    callServerTool(){return Promise.resolve({content:[]})} openLink(){} downloadFile(){}
  }};`;
  res.type("html").send(widgetHtml.replace("/*__EXT_APPS_BUNDLE__*/", shim));
});
```

Open `http://localhost:3000/widget-preview?payload={"rows":[...]}` in a normal browser tab and iterate with ordinary devtools.

**Host fallback** — use a host without the apps surface (or MCP Inspector) and confirm the tool's text content degrades gracefully.

**CSP debugging** — open the iframe's own devtools console. CSP violations are the #1 reason widgets silently fail (blank rectangle, no error in the main console). See `references/iframe-sandbox.md`.

---

## 참고 문서 정보 목록

- `references/iframe-sandbox.md` — CSP 및 샌드박스 정책의 제약 수칙, 라이브러리 인라이닝 처리 요령, 이미지 인라인 치환 패턴 및 호스트 테마 설정 기법
- `references/widget-templates.md` — 바로 가져다 사용 가능한 피커 / 확인 대화창 / 진척 진계 / 차트 레이아웃 HTML 마크업 코드 세트
- `references/apps-sdk-messages.md` — `App` 클래스 API 상세 명세: 위젯과 호스트 간 통신 주기, 라이프사이클 관리 및 구버전 위젯 비활성 대응책
- `references/payload-budgeting.md` — 호스트별 연산 반환 크기 임계치, 불필요한 데이터 걸러내기 및 무거운 데이터의 `callServerTool` 처리 기법
- `references/abuse-protection.md` — 인증 없는 공개 서버의 남용 방지: Anthropic IP CIDR 매칭 분류, 프록시 신뢰 설정 기법 및 데이터 응답 캐싱 가이드
- `references/directory-checklist.md` — Claude 커넥터 공식 디렉토리 등록 제출 전 점검이 요구되는 최종 심사 통과 항목
