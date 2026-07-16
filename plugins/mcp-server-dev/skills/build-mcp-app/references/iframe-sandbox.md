# Iframe 샌드박스 제약 조건 (Iframe sandbox constraints)

MCP-app 위젯은 호스트(Claude Desktop, claude.ai) 내부의 보안 샌드박스가 적용된 `<iframe>` 안에서 실행됩니다. 샌드박스 속성과 CSP(콘텐츠 보안 정책) 설정은 위젯이 수행할 수 있는 동작을 엄격하게 통제합니다.
아래에 열려 있는 모든 오류 현상들은, 아래 제시된 조치를 적용하기 전까지 아무런 에러 메시지 없이 빈 흰색 화면으로만 표시되는 고질적인 원인들입니다 — 에러 로그는 호스트 창 콘솔이 아닌 오직 iframe 자체 개발자 도구 콘솔에만 출력됩니다.

---

## 오류 증상 → 대응 방법 일람

| 오류 증상 | 근본 원인 | 해결 방책 |
|---|---|---|
| 위젯이 빈 사각형으로만 뜨고 아무 반응이 없음 | CSP `script-src` 정책이 esm.sh를 경유해 연쇄 호출되는 `@modelcontextprotocol/sdk` 종속성 다운로드를 차단함 | `ext-apps/app-with-deps` 번들 파일을 HTML 소스에 **인라인으로 직접 포함**시킵니다. |
| `window.open()`을 호출해도 아무런 창이 뜨지 않음 | 샌드박스 옵션에 `allow-popups` 권한이 부여되지 않음 | 호스트 제공 API인 `app.openLink({ url })` 함수를 호출하십시오. |
| `<a target="_blank">` 태그를 클릭해도 아무 반응이 없음 | 위와 같음 | 클릭 이벤트 발생 시 `e.preventDefault()`로 가로채고 `app.openLink({ url })`를 실행하십시오. |
| 외부 이미지 `<img src="http://...">` 로드 실패 | CSP `img-src` 정책 및 리퍼러 확인 보안 정책에 의해 로드 차단 | 서버 단에서 이미지를 미리 내려받아, 도구 실행 결과 페이로드 문자열 안에 base64 `data:` 이미지 URL 포맷으로 직접 변환하여 전달하십시오. |
| 서버 코드를 재시작하고 수정해도 위젯 화면이 갱신되지 않음 | 호스트 프로그램이 과거 UI 자원을 메모리에 강하게 캐싱하고 있음 | 호스트 프로그램 자체를 완전히 종료(⌘Q / Alt+F4)한 뒤 재실행하십시오. |
| Top-level `await` 구문 실행 시 에러 발생 | 과거 버전의 iframe 런타임 호환성 한계 | 모듈 실행 영역 전체를 비동기 즉시 실행 함수(async IIFE)로 한 단계 감싸 실행하십시오. |

---

## ext-apps 번들 인라인 적용 방법 (Inlining the ext-apps bundle)

`@modelcontextprotocol/ext-apps` 패키지는 브라우저용으로 완전하게 패키징된 빌드 결과물(약 300KB)을 `app-with-deps` 모듈 경로로 제공합니다. 이는 `export{…}` 구문으로 마치는 경량 ESM 코드 형태이므로, 이를 인라인 `<script type="module">` 블록 내부에서 사용하려면 빌드 단계(혹은 서버 실행 시)에 아래와 같이 export 구문을 전역 assignment 형태로 바꾸어 주어야 합니다:

```ts
import { readFileSync } from "node:fs";
import { createRequire } from "node:module";
const require = createRequire(import.meta.url);

const bundle = readFileSync(
  require.resolve("@modelcontextprotocol/ext-apps/app-with-deps"),
  "utf8",
).replace(/export\{([^}]+)\};?\s*$/, (_, body) =>
  "globalThis.ExtApps={" +
  body.split(",").map((pair) => {
    const [local, exported] = pair.split(" as ").map((s) => s.trim());
    return `${exported ?? local}:${local}`;
  }).join(",") + "};",
);

const widgetHtml = readFileSync("./widgets/widget.html", "utf8")
  .replace("/*__EXT_APPS_BUNDLE__*/", () => bundle);
```

위젯 측 코드 작성법:

```html
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "…", version: "…" }, {});
  // …
})();
</script>
```

치환 시 단순 문자열이 아닌 `() => bundle`과 같이 콜백 함수 형태로 대체 처리 인자를 전달해야 합니다 — `String.replace` 함수는 치환할 문자열 내의 `$…` 문자를 특수 치환 시퀀스로 오해해 파싱할 우려가 있는데, 미니파이된 번들 자바스크립트 코드 내에는 이러한 달러($) 기호가 매우 많이 섞여 있기 때문입니다.

---

## 외부 이동 링크 (Outbound links)

```js
// ✗ 차단당하는 방식
window.open(url, "_blank");
// ✗ 차단당하는 방식
<a href="…" target="_blank">…</a>

// ✓ 호스트 중재를 거친 정상적인 실행 방식
await app.openLink({ url });
```

앵커 태그 클릭 이벤트를 직접 가로채십시오:

```js
el.addEventListener("click", (e) => {
  e.preventDefault();
  app.openLink({ url: el.href });
});
```

---

## 외부 이미지 리소스 (External images)

기본적으로 설정된 CSP `img-src` 정책 및 각 CDN 서버들의 레퍼러 확인 정책으로 인해 `<img src="https://external-cdn/…">` 형태의 직접 호출은 모두 차단됩니다. 서버 단 도구 핸들러 함수 안에서 미리 프록시 조회하여 인라인 처리해 보내십시오:

```ts
async function toDataUrl(url: string): Promise<string | undefined> {
  try {
    const res = await fetch(url, { signal: AbortSignal.timeout(5000) });
    if (!res.ok) return undefined;
    const buf = Buffer.from(await res.arrayBuffer());
    const mime = res.headers.get("content-type") ?? "image/jpeg";
    return `data:${mime};base64,${buf.toString("base64")}`;
  } catch {
    return undefined;
  }
}

// 도구 핸들러 함수 내부
const inlined = await Promise.all(
  items.map(async (it) =>
    it.thumb ? { ...it, thumb: await toDataUrl(it.thumb) ?? it.thumb } : it,
  ),
);
```

인라인으로 치환되지 않고 주소 그대로 살아남은 이미지가 있을 때의 최종 방어책으로 `<img>` 태그에 `referrerpolicy="no-referrer"` 속성을 함께 명시해 주십시오.

---

## 테마 및 호스트 스타일 상속 (Theme & host styles)

호스트 프로그램은 iframe을 자신의 카드 디자인 틀 안에 얹어 보여줍니다 — 뒷배경을 **투명(transparent)**하게 설정하고 호스트의 CSS 변수 명세를 상속 받아 사용하여 위젯이 다크/라이트 모드 및 다양한 호스트 환경 전반에서 자연스럽게 스며들도록 디자인하십시오.

```html
<meta name="color-scheme" content="light dark" />
```

```css
:root {
  --ink:  var(--color-text-primary,   #0f1111);
  --sub:  var(--color-text-secondary, #5a6270);
  --line: var(--color-border-default, #e3e6ea);
}
html, body { background: transparent; color: var(--ink); }
:root.dark .thumb { mix-blend-mode: normal; } /* multiply 효과 사용 시 다크 모드에서 이미지가 검게 묻혀 안 보일 수 있음 */
```

```js
const { App, applyHostStyleVariables } = globalThis.ExtApps;

function applyHostContext(ctx) {
  document.documentElement.classList.toggle("dark", ctx?.theme === "dark");
  if (ctx?.styles?.variables) applyHostStyleVariables(ctx.styles.variables);
}
app.onhostcontextchanged = applyHostContext;
await app.connect();
applyHostContext(app.getHostContext());
```

`applyHostStyleVariables` 함수는 호스트 환경이 지정한 `--color-*` / `--font-*` / `--border-radius-*` 등의 CSS 변수 토큰들을 위젯의 `:root` 객체 위에 그대로 복제하여 입혀 줍니다. 위 CSS 코드 내에 지정한 헥사(#)값들은 이러한 정보를 지원하지 않는 이전 버전 호스트 환경을 위한 예외용 디폴트(fallback) 설정입니다.

---

## 디버깅 방법 (Debugging)

iframe은 호스트와 별개의 독립된 자바스크립트 콘솔 창을 갖습니다. Claude Desktop 환경에서는 개발자 도구를 띄운 뒤 (View → Toggle Developer Tools), Console 탭의 좌측 상단에 위치한 컨텍스트 타겟 드롭다운 메뉴를 "top"에서 해당 위젯의 iframe 이름으로 직접 전환하십시오. CSP 정책 위반 로그, 잡히지 않은 예외(uncaught exceptions), ESM 모듈 호출 실패 내역 등이 모두 이곳에만 찍히며, 호스트 기본 콘솔창에는 아무 흔적도 남지 않기 때문에 매우 중요한 팁입니다.
