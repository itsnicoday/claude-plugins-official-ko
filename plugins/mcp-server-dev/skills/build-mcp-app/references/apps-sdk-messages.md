# ext-apps 메시지 규격 — 위젯 ↔ 호스트 ↔ 서버 (ext-apps messaging — widget ↔ host ↔ server)

`@modelcontextprotocol/ext-apps` 패키지는 브라우저단용 `App` 클래스와 서버단용 `registerAppTool` / `registerAppResource` 헬퍼 함수들을 제공합니다. 메시지 교환은 양방향으로 유지되며 영속적입니다.

## 객체 초기화 (Construction)

```js
const app = new App(
  { name: "MyWidget", version: "1.0.0" },
  {},                       // capabilities 설정
  { autoResize: true },     // options 설정
);
```

`autoResize: true` 설정을 부여하면 `ResizeObserver` 인프라를 연결하여 위젯 높이가 변경될 때마다 `ui/notifications/size-changed` 이벤트를 발행해 호스트 iframe 영역의 세로 폭을 콘텐츠 크기에 동적으로 밀착시킵니다. 이 옵션을 켜지 않으면 프레임 높이가 고정되어 대량의 데이터 렌더링 시 하단 영역이 잘리게 되므로, 데이터 조회 결과에 따라 렌더링 세로 길이가 변하는 모든 위젯에는 반드시 적용해 주십시오.

---

## 위젯 → 호스트 전송 메시지 (Widget → Host)

### `app.sendMessage({ role, content })`

사용자의 대화 흐름 속에 눈에 보이는 일반 텍스트 메시지를 삽입합니다. 이 방식으로 위젯 내 사용자 동작이 대화의 흐름으로 매핑됩니다.

```js
app.sendMessage({
  role: "user",
  content: [{ type: "text", text: "User selected order #1234" }],
});
```

메시지가 채팅창에 출력되고 Claude가 그에 반응해 답변을 내어놓습니다. 위젯이 사용자의 의사를 대변해 이야기하는 형식이므로 항상 `role: "user"`를 지정하십시오.

### `app.updateModelContext({ content })`

**채팅창에 출력되지 않고** 백그라운드에서 조용히 Claude의 상황 판단 컨텍스트 정보만 교체합니다. 채팅 말풍선을 띄울 정도는 아니지만 알아야 할 부차 정보나 상태가 변경되었을 때 유용합니다.

```js
app.updateModelContext({
  content: [{ type: "text", text: "Currently viewing: orders from last 30 days" }],
});
```

### `app.callServerTool({ name, arguments })`

Claude의 해석 단계를 거치지 않고, MCP 서버 내의 도구 함수를 위젯이 다이렉트로 직접 호출하여 결과를 리턴 받습니다.

```js
const result = await app.callServerTool({
  name: "fetch_order_details",
  arguments: { orderId: "1234" },
});
```

페이징 데이터 불러오기, 상세내역 추가 조회, 실시간 화면 갱신 등 Claude의 직접적인 인프런스 추론이 개입될 필요가 없는 영역에 적극 활용하십시오.

### `app.openLink({ url })`

호스트의 중재를 거쳐 새 웹 브라우저 탭으로 주소를 엽니다. iframe 샌드박스 정책 상 `window.open()` 및 `<a target="_blank">` 구문 작동이 차단되므로, 외부 이동 주소 링크의 렌더링에는 **반드시** 이 헬퍼 함수를 호출해야 합니다.

```js
await app.openLink({ url: "https://example.com/cart" });
```

HTML 코드에 들어있는 앵커 태그에 바인딩하려면 아래와 같이 이벤트를 가로채 처리하십시오:

```js
card.querySelector("a").addEventListener("click", (e) => {
  e.preventDefault();
  app.openLink({ url: e.currentTarget.href });
});
```

### `app.downloadFile({ name, mimeType, content })`

호스트 중재 방식의 파일 내려받기 기능입니다 (샌드박스로 인해 직접 `<a download>`를 실행하는 방식은 차단됨). `content` 파라미터에는 base64 형태의 문자열 값을 실어 보냅니다.

```js
const csv = rows.map((r) => Object.values(r).join(",")).join("\n");
app.downloadFile({
  name: "export.csv",
  mimeType: "text/csv",
  content: btoa(unescape(encodeURIComponent(csv))),
});
```

### `app.requestDisplayMode({ mode })`

호스트 애플리케이션 측에 위젯 표시 모드를 `"inline"`, `"pip"`, 또는 `"fullscreen"` 모드로 변경해 줄 것을 요청합니다. 실행 전에 먼저 `getHostContext().availableDisplayModes` 정보를 훑어 지원 목록에 없는 레이아웃 옵션인 경우에는 관련 제어 버튼을 사전에 비활성화 처리하십시오. 호스트는 해당 요청에 반응해 새롭게 바뀐 `displayMode`와 `containerDimensions` 규격을 담아 `onhostcontextchanged` 이벤트를 발생시키며, 위젯은 이에 대응해 리렌더링 처리를 수행하면 됩니다.

```js
if (app.getHostContext()?.availableDisplayModes?.includes("fullscreen")) {
  expandBtn.hidden = false;
  expandBtn.onclick = () => app.requestDisplayMode({ mode: "fullscreen" });
}
```

---

## 호스트 → 위젯 수신 메시지 (Host → Widget)

### `app.ontoolresult = ({ content }) => {...}`

도구 핸들러의 연산 반환 데이터가 위젯으로 전달되어 올 때 발생합니다. 위젯으로 실질적인 데이터가 유입되는 가장 메인 파이프라인입니다.

```js
app.ontoolresult = ({ content }) => {
  const data = JSON.parse(content[0].text);
  renderUI(data);
};
```

**반드시 `await app.connect()`를 호출하기 전에 이 핸들러를 정의해 두십시오** — 커넥션 수립 즉시 도구 결과가 유입되어 들어올 수 있습니다.

### `app.ontoolinput = ({ arguments }) => {...}`

Claude가 도구를 실행하며 전달했던 원본 매개변수 정보를 실어 보냅니다. 위젯이 자신이 무슨 키워드로 조회된 결과인지 표시하거나 매칭 텍스트를 하이라이트하는 기능 등에 활용할 수 있습니다.

### `app.ontoolinputpartial = ({ arguments }) => {...}` / `app.ontoolcancelled = () => {...}`

`ontoolinputpartial`은 Claude가 도구 매개변수를 실시간으로 스트리밍하는 도중에 발행되므로, 최종 데이터가 오기 전 화면에 스켈레톤 UI("준비 중: <제목>...") 같은 로딩 상태를 표시할 때 사용합니다. 호출이 중단되면 `ontoolcancelled` 이벤트가 발행되므로 표시하던 스켈레톤 처리를 걷어냅니다.

### `app.getHostContext()` / `app.onhostcontextchanged = (ctx) => {...}`

호스트의 뷰 및 스타일 맥락 정보에 접근하고 감지합니다. `getHostContext()` 호출은 커넥션 연결 수립(`connect()`) **이후에** 수행해야 합니다. 사용자가 브라우저 다크 모드를 토글하거나 윈도우 창 크기를 늘릴 때의 실시간 동기화를 위해 리스너를 바인딩하십시오.

| 컨텍스트 필드 | 사용 목적 |
|---|---|
| `theme` | `"light"` 또는 `"dark"` 값 전달 — 루트 요소의 다크 모드 클래스 토글 등에 사용 |
| `styles.variables` | 호스트 환경의 기본 CSS 스타일 토큰 정보 — `applyHostStyleVariables()`를 호출해 넘기면 위젯 색상/폰트 룩을 호스트 테두리 스타일과 손쉽게 일치시킬 수 있습니다. |
| `displayMode` / `availableDisplayModes` | 현재 위젯 표시 상태 및 `requestDisplayMode` 요청이 수용 가능한 모드 일람 |
| `containerDimensions.{maxHeight,width}` | 하드코딩된 px 고정값 대신 이 영역 규격에 맞추어 레이아웃 크기를 가변 처리 |
| `deviceCapabilities.touch` | 모바일 터치 장비인 경우 호버(hover) 인터페이스를 탭(`pointerdown`) 인터페이스로 대처 |
| `safeAreaInsets` | 기기 상단 노치나 작문 창 영역 하단과의 겹침 방지용 패딩 값 |

```js
const applyTheme = (t) =>
  document.documentElement.classList.toggle("dark", t === "dark");

app.onhostcontextchanged = (ctx) => applyTheme(ctx.theme);
await app.connect();
applyTheme(app.getHostContext()?.theme);
```

CSS 변수 정의 시 `:root.dark {}` 오버라이드 블록과 `color-scheme: light | dark` 브라우저 지침 설정을 함께 명시하면 네이티브 폼 컨트롤 양식의 색상 톤까지 호스트 디자인에 맞추어 유기적으로 변합니다.

---

## 서버 → 위젯 통신 (진행 표시)

실행 시간이 긴 백그라운드 작업인 경우, 진행 진척도 알림을 발행하십시오. 클라이언트는 요청 프로토콜 헤더 `_meta`에 `progressToken` 값을 실어 주며, 서버는 이 키를 대상으로 상태를 발신합니다.

```typescript
// 도구 핸들러 함수 내부
async ({ query }, extra) => {
  const token = extra._meta?.progressToken;
  for (let i = 0; i < steps.length; i++) {
    if (token !== undefined) {
      await extra.sendNotification({
        method: "notifications/progress",
        params: { progressToken: token, progress: i, total: steps.length, message: steps[i].name },
      });
    }
    await steps[i].run();
  }
  return { content: [{ type: "text", text: "Complete" }] };
}
```

인자 분해 대입 시 `{ notify }`를 뽑아내지 마십시오 — `extra`는 `RequestHandlerExtra` 객체 사양이며, 알림 제어는 `sendNotification` 메서드로 바로 처리합니다.

---

## 라이프사이클 (Lifecycle)

1. Claude가 `_meta.ui.resourceUri` 속성이 선언된 특정 도구를 실행합니다.
2. 호스트가 리소스 주소의 정적 파일(HTML)을 읽어와 **완전히 새로운 iframe**을 임시 마운트합니다.
3. 위젯 자바스크립트 코드가 기동하여 수신 리스너들을 등록하고 `await app.connect()`를 진행합니다.
4. 호스트가 도구의 최종 반환 값 데이터를 밀어 넣어 `ontoolresult` 이벤트가 발행됩니다.
5. 위젯이 받은 데이터를 그리고 사용자는 위젯 화면과 상호작용합니다.
6. 위젯은 액션에 맞춰 `sendMessage` / `updateModelContext` / `callServerTool` 등을 요청합니다.
7. 대화 이력 뷰포트 내에 해당 iframe은 그대로 보존되어 박제됩니다. **해당 도구가 새로 호출될 때마다 완전히 별개의 새 iframe 인스턴스가 그 하단에 새로 생성되어 마운트됩니다.**

인터페이스에 명시적인 "제출 및 닫기" 개념은 없습니다 — 각 프레임 인스턴스는 한 번 올라가면 그대로 영구 활성 상태로 남되, 동일 도구 실행 시 과거 인스턴스가 재활용되지 않고 매번 새로 로드됩니다.

### 구버전 인스턴스의 비활성 처리 (Supersession)

과거 이력의 iframe들이 여전히 화면에 렌더링된 채 살아있기 때문에, 사용자가 실수로 위로 스크롤하여 구버전 위젯 내부 버튼을 누르면 새로운 데이터 전송 트랜잭션이 인입되어 꼬일 수 있습니다. `BroadcastChannel`을 연결해 가장 최근에 뜬 위젯 외의 나머지 구버전 프레임은 비활성 처리하는 패턴을 권장합니다:

```js
let superseded = false;
const seq = Date.now() + Math.random();
const bc = new BroadcastChannel("my-widget");
bc.onmessage = (e) => {
  if (e.data?.seq > seq) {
    superseded = true;
    document.body.classList.add("superseded"); // CSS: opacity:.45; pointer-events:none 지정
  }
};
bc.postMessage({ seq });

// 데이터 전송 보호용 게이트 래퍼 함수:
function safeSend(msg) {
  if (!superseded) app.sendMessage(msg);
}
```

---

## 샌드박스 및 보안 콘텐츠 정책(CSP) 유의점

위젯이 구동되는 iframe 환경은 HTML `sandbox` 강력 보호 속성과 극도로 엄격히 한정된 CSP(Content-Security-Policy) 보호막 아래 기동합니다. 실제 구현 관점에서는 외부의 어떠한 주소에서도 자원을 불러올 수 없음을 의미하므로, 위젯 리소스 파일은 완벽히 독립적으로 패키징(self-contained)되어 있어야 합니다.

| 오류 현상 | 원인 | 대응책 |
|---|---|---|
| 위젯 캔버스가 빈 하얀 사각형으로만 뜨고 렌더링이 안 됨 | ext-apps 라이브러리를 CDN import 주소로 호출하다 차단당함 (SDK 내부에서 추가 자원 요청 실패) | `ext-apps/app-with-deps` 패키지 파일을 자바스크립트 코드 내에 **인라인으로 직접 포함(inlining)** 시킵니다 — `iframe-sandbox.md` 참고 |
| 화면 렌더링은 잘 되는데 버튼 스크립트 실행만 안 됨 | HTML 코드 안의 인라인 이벤트 바인딩이 CSP에 의해 차단됨 | 마크업 HTML 파일에 `onclick="..."` 속성을 적지 말고, JS 코드 안에서 `addEventListener`를 통해 등록하십시오. |
| `eval` 또는 `new Function` 구문 에러 발생 | CSP script-src 문자열 평가 제한 | eval 함수 계열은 절대 쓰면 안 됩니다. 구조화된 문자열 파싱에는 `JSON.parse`를 쓰십시오. |
| 외부 서버 API 주소로 직접 `fetch()` 요청을 보내면 실패함 | Cross-origin 도메인 접근 차단 | 서버와의 통신이 필요하다면 반드시 `app.callServerTool()` 채널로 우회 통신하십시오. |
| 외부 CSS 리소스 로드 실패 | `style-src` 정책 제한 | 스타일 정보는 마크업 파일 내부 `<style>` 태그 안에 직접 작성하십시오. |
| 폰트가 깨지거나 대체 폰트로만 렌더링됨 | `font-src` 정책 제한 | 기기 로컬 시스템 폰트명들을 지정해 사용하십시오 (`font: 14px system-ui` 등) |
| 외부 이미지 `<img src="http://...">`가 차단되어 표시 안 됨 | CSP `img-src` 통신 제한 및 리퍼러 확인 우회 차단 | 이미지 바이트 데이터를 서버 단에서 먼저 받아 긁어온 뒤, 도구 반환값 안에 인라인 base64 `data:` 이미지 URL 스키마 포맷으로 직접 치환해 넘기십시오. |
| `window.open()`을 실행해도 아무런 반응이 없음 | 샌드박스 설정에 `allow-popups` 권한이 누락됨 | 호스트 중재 헬퍼인 `app.openLink({url})`를 호출하십시오. |
| `<a target="_blank">` 링크가 작동하지 않음 | 위와 같음 | 클릭 이벤트를 받아 `preventDefault()`로 브라우저 동작을 차단하고 `app.openLink`로 쏘십시오. |
| 수정한 HTML/CSS 코드가 Desktop 클라이언트 환경에서 반영이 안 됨 | 데스크톱 앱 내부적으로 UI 리소스를 강하게 캐싱함 | 단순히 화면을 닫았다 열지 말고, 데스크톱 앱 자체를 완전히 끄고(⌘Q) 재실행하십시오. |

연동 과정에서 화면이 제대로 작동하지 않을 때는, 메인 개발자 도구 콘솔창이 아닌 **해당 iframe의 독립 개발자 도구 콘솔**을 열어 CSP 차단 에러 로그를 확인해 보면 원인을 바로 짚어낼 수 있습니다. 독립 패키징 빌드 패턴에 대해서는 `iframe-sandbox.md` 문서를 참고하십시오.
