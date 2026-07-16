# 위젯 템플릿 (Widget Templates)

자주 사용되는 위젯 형태의 최소한의 HTML 스캐폴딩 코드 모음입니다. 복사해서 알맞게 채워 넣고 배포하십시오.

모든 템플릿은 빌드 시점에 `@modelcontextprotocol/ext-apps` 패키지에서 제공하는 `App` 클래스를 인라인 코드로 포함합니다. iframe의 CSP 보안 정책이 CDN 스크립트 호출을 원천 차단하기 때문입니다. 또한 프레임워크를 전혀 사용하지 않는 순수 JS 형태로 설계되었습니다. 위젯의 크기는 보통 크지 않기 때문에, React/Vue 등을 올려 발생하는 하이드레이션(hydration) 연산 비용을 지불할 가치가 크지 않습니다.

---

## 위젯 HTML 서빙 방법 (Serving widget HTML)

위젯은 정적 HTML 파일 구조로 이루어져 있으며, 단 하나의 플레이스홀더 주석 `/*__EXT_APPS_BUNDLE__*/`을 포함합니다. 서버 구동 시점에 이 주석 영역은 `ExtApps` 객체를 전역 노출하도록 다시 쓰여진 `ext-apps/app-with-deps` 번들 스크립트 문자열로 직접 치환됩니다.

```typescript
import { readFileSync } from "node:fs";
import { createRequire } from "node:module";
import { registerAppResource, RESOURCE_MIME_TYPE } from "@modelcontextprotocol/ext-apps/server";

const require = createRequire(import.meta.url);

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

registerAppResource(server, "Picker", "ui://widgets/picker.html", {},
  async () => ({
    contents: [{ uri: "ui://widgets/picker.html", mimeType: RESOURCE_MIME_TYPE, text: pickerHtml }],
  }),
);
```

이 번들링 연산은 서버 기동 시 한 번만(또는 빌드 타임에) 수행하면 됩니다. 확보한 `bundle` 문자열을 모든 위젯 템플릿에 재사용하십시오.

---

## 피커 위젯 (Picker — 단일 항목 선택 목록)

```html
<!doctype html>
<meta charset="utf-8" />
<style>
  body { font: 14px system-ui; margin: 0; }
  ul { list-style: none; padding: 0; margin: 0; max-height: 280px; overflow-y: auto; }
  li { padding: 10px 14px; cursor: pointer; border-bottom: 1px solid #eee; }
  li:hover { background: #f5f5f5; }
  .sub { color: #666; font-size: 12px; }
</style>
<ul id="list"></ul>
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "Picker", version: "1.0.0" }, {});
  const ul = document.getElementById("list");

  app.ontoolresult = ({ content }) => {
    const { items } = JSON.parse(content[0].text);
    ul.innerHTML = "";
    for (const it of items) {
      const li = document.createElement("li");
      li.innerHTML = `<div>${it.label}</div><div class="sub">${it.sub ?? ""}</div>`;
      li.addEventListener("click", () => {
        app.sendMessage({
          role: "user",
          content: [{ type: "text", text: `Selected: ${it.id}` }],
        });
      });
      ul.append(li);
    }
  };

  await app.connect();
})();
</script>
```

**도구 반환 형식:** `{ content: [{ type: "text", text: JSON.stringify({ items: [{ id, label, sub? }] }) }] }`

---

## 확인 대화창 (Confirm dialog)

```html
<!doctype html>
<meta charset="utf-8" />
<style>
  body { font: 14px system-ui; margin: 16px; }
  .actions { display: flex; gap: 8px; margin-top: 16px; }
  button { padding: 8px 16px; cursor: pointer; }
  .danger { background: #d33; color: white; border: none; }
</style>
<p id="msg"></p>
<div class="actions">
  <button id="cancel">Cancel</button>
  <button id="confirm" class="danger">Confirm</button>
</div>
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "Confirm", version: "1.0.0" }, {});

  app.ontoolresult = ({ content }) => {
    const { message, confirmLabel } = JSON.parse(content[0].text);
    document.getElementById("msg").textContent = message;
    if (confirmLabel) document.getElementById("confirm").textContent = confirmLabel;
  };

  await app.connect();

  document.getElementById("confirm").addEventListener("click", () => {
    app.sendMessage({ role: "user", content: [{ type: "text", text: "Confirmed." }] });
  });
  document.getElementById("cancel").addEventListener("click", () => {
    app.sendMessage({ role: "user", content: [{ type: "text", text: "Cancelled." }] });
  });
})();
</script>
```

**도구 반환 형식:** `{ content: [{ type: "text", text: JSON.stringify({ message, confirmLabel? }) }] }`

**참고:** 단순 승인/확인 시나리오에서는 위젯을 그리는 것보다 **elicitation** 기능을 이용하는 것이 훨씬 간편합니다 — `../build-mcp-server/references/elicitation.md` 참고. UI 커스텀 스타일링이 꼭 필요하거나 네이티브 입력 폼 규격을 벗어난 정교한 정보 표기가 필요할 때만 이 위젯 구조를 쓰십시오.

---

## 진척도 표시 위젯 (Progress — 장시간 연산용)

```html
<!doctype html>
<meta charset="utf-8" />
<style>
  body { font: 14px system-ui; margin: 16px; }
  .bar { height: 8px; background: #eee; border-radius: 4px; overflow: hidden; }
  .fill { height: 100%; background: #2a7; transition: width 200ms; }
</style>
<p id="label">Starting…</p>
<div class="bar"><div id="fill" class="fill" style="width:0%"></div></div>
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "Progress", version: "1.0.0" }, {});
  const label = document.getElementById("label");
  const fill = document.getElementById("fill");

  // The tool result fires when the job completes — intermediate updates
  // arrive via the same handler if the server streams them
  app.ontoolresult = ({ content }) => {
    const state = JSON.parse(content[0].text);
    if (state.progress !== undefined) {
      label.textContent = state.message ?? `${state.progress}/${state.total}`;
      fill.style.width = `${(state.progress / state.total) * 100}%`;
    }
    if (state.done) {
      label.textContent = "Complete";
      fill.style.width = "100%";
    }
  };

  await app.connect();
})();
</script>
```

서버 측에서 진척 상태 전송은 `extra.sendNotification({ method: "notifications/progress", ... })` 명령 채널을 사용해 처리합니다 — `apps-sdk-messages.md` 참고.

---

## 출력 전용 위젯 (Display-only — 차트 / 미리보기 패널)

출력 목적의 위젯은 `sendMessage` 등을 호출하지 않고, 데이터를 렌더링한 채 그대로 대기합니다. 도구 응답값은 위젯 시각화 정보 **이외에** Claude가 계속 대화 맥락을 읽고 인프런스할 수 있도록 설명 요약 문자열을 텍스트 형태로 함께 내려주어야 합니다:

```typescript
registerAppTool(server, "show_chart", {
  description: "Render a revenue chart",
  inputSchema: { range: z.enum(["week", "month", "year"]) },
  _meta: { ui: { resourceUri: "ui://widgets/chart.html" } },
}, async ({ range }) => {
  const data = await fetchRevenue(range);
  return {
    content: [{
      type: "text",
      text: `Revenue is up ${data.change}% over the ${range}. Chart rendered.\n\n` +
            JSON.stringify(data.points),
    }],
  };
});
```

```html
<!doctype html>
<meta charset="utf-8" />
<style>body { font: 14px system-ui; margin: 12px; }</style>
<canvas id="chart" width="400" height="200"></canvas>
<script type="module">
/*__EXT_APPS_BUNDLE__*/
const { App } = globalThis.ExtApps;
(async () => {
  const app = new App({ name: "Chart", version: "1.0.0" }, {});

  app.ontoolresult = ({ content }) => {
    // Parse the JSON points from the text content (after the summary line)
    const text = content[0].text;
    const jsonStart = text.indexOf("\n\n") + 2;
    const points = JSON.parse(text.slice(jsonStart));
    drawChart(document.getElementById("chart"), points);
  };

  await app.connect();

  function drawChart(canvas, points) { /* ... */ }
})();
</script>
```

---

## 캐러셀 위젯 (Carousel — 선택 버튼이 달린 가로 스크롤 카드 레이아웃)

여러 개의 카드 목록 정보(추천 상품 목록, 상세 검색 결과 등)를 슬라이딩 레일 형태로 노출할 때 사용하는 서식입니다. 사용성 테스트 결과 다음 설계안들이 권장됩니다:

- **가로 스크롤 제어용 좌우 버튼 제거** — 요즘 모바일/데스크톱 사용자들은 자연스러운 제스처 스크롤에 익숙합니다. `scroll-snap-type` 스타일은 렌더링 첫 프레임에서 카드 정렬이 픽셀 단위로 미세하게 밀리는 오작동이 발생하기 쉬우므로, 렌더링 완료 후 `scrollLeft = 0` 스크립트 실행으로 초기화하는 방식을 권장합니다.
- **아이템 개수에 따른 분기 대응** — 카드 개수가 1개인 경우(`items.length === 1`) 단독 와이드 상세 레이아웃을 표시하고, 여러 개인 경우 가로 스크롤 캐러셀 레이아웃으로 동적 분기 처리하는 것이 좋습니다. 도구 스케마는 가볍게 플랫 구조로 설계하고 분기 로직은 위젯의 JS가 처리하게 만드십시오.
- **각 카드마다 Claude가 추천한 사유 명시** — 카탈로그 데이터 이외에 `note` 속성 필드를 두어 Claude가 왜 이 상품을 골라 추천했는지 근거 요약문을 카드에 작게 함께 렌더링해주면 사용자 경험이 매우 훌륭해집니다.
- **`updateModelContext`를 활용한 은밀한 상태 변경** — 장바구니에 아이템을 담는 등의 동작은 대화창에 말풍선을 뿌려 도배하지 말고, 무음으로 Claude의 인지 상태값만 갱신해 주십시오. `sendMessage` 통신 채널은 "주문서 작성 완료", "최종 확인"과 같이 대화 맥락의 완전한 턴 전환 시점에만 아껴 쓰십시오.
- **외부 이동은 무조건 `app.openLink` 이용** — 샌드박스로 인해 `window.open`이나 `<a target="_blank">` 링크는 차단됩니다.

```html
<style>
  .rail { display: flex; gap: 10px; overflow-x: auto; padding: 12px; scrollbar-width: none; }
  .rail::-webkit-scrollbar { display: none; }
  .card { flex: 0 0 220px; border: 1px solid #ddd; border-radius: 6px; padding: 10px; }
  .thumb-box { aspect-ratio: 1 / 1; display: grid; place-items: center; background: #f7f8f8; }
  .thumb { max-width: 100%; max-height: 100%; object-fit: contain; }
  .note { font-size: 12px; color: #666; border-left: 3px solid orange; padding: 2px 8px; margin: 8px 0; }
</style>
<div class="rail" id="rail"></div>
```

**이미지 처리 요령:** iframe의 CSP 보안 정책 상 외부 `img-src` 통신이 차단됩니다. 이미지 썸네일 자원은 서버 단에서 먼저 받아 긁어온 뒤, 도구 결과 페이로드 문자열 안에 base64 인라인 `data:` 이미지 URL 스키마 포맷으로 직접 치환해 위젯에 내려주어야 정상적으로 렌더링됩니다. 대체 수단으로 이미지 태그에 `referrerpolicy="no-referrer"` 설정을 함께 부여하는 기법도 고려해 두십시오.
