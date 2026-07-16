# 페이로드 관리 (Payload budgeting)

호스트 환경은 도구 실행 결과로 전달되는 텍스트 크기에 상한을 둡니다. claude.ai 및 Claude 데스크톱 환경에서는 약 **150,000자**에서 절삭(truncation)이 일어나며, Claude Code 환경에서는 약 25k 토큰 근처에서 잘립니다. 도구 실행 결과가 이 한계치를 초과하면, 호스트는 원래의 JSON 데이터 대신 파일 포인터 문자열로 이를 치환해 버립니다.
이로 인해 위젯 측에서는 `ontoolresult` 이벤트 수신 시 정상적인 JSON이 아닌 값이 수집되어 `JSON.parse` 예외를 던지게 되고, 사용자는 크기 제한이 원인임을 모르는 채 *"Bad payload: SyntaxError: Unexpected token 'E'"* 형태의 알 수 없는 에러를 마주하게 됩니다.

## 오류 증상 → 원인 매칭

| 오류 증상 | 예상 원인 |
|---|---|
| 위젯 화면에 `content[0].text`에 대한 JSON 파싱 에러 노출 | 결과 데이터 크기가 호스트 임계치를 초과하여, 호스트가 파일 포인터 문자열로 데이터를 강제 치환함 |
| 조회 범위가 좁을 때는 작동하나, "모든 X 조회" 실행 시 에러 발생 | 행(row) 수 × 열(column) 수의 곱이 호스트 허용 크기를 초과함 |
| MCP 인스펙터에서는 잘 작동하지만, 데스크톱 버전 연동 시에는 깨짐 | 인스펙터는 크기 한계가 없지만, 데스크톱 앱에는 존재함 |

## 대응 전략 (Strategy)

스스로 페이로드 크기를 ~130KB 이내로 제한하고, 초과 시 순차적으로 데이터를 축소(degrade)하여 전달하십시오:

1. `JSON.stringify(rows).length`의 크기가 임계치보다 작다면 **전체 행 데이터를 전송**합니다.
2. 렌더링 스펙(spec)에서 참조하지 않는 **불필요한 열(column)들을 걸러냅니다**.
   스펙 파일의 `field: "..."` 지정 정보뿐만 아니라, 식(expression) 문자열 안의 `datum.X` / `datum['X']` 패턴까지 꼼꼼히 체크해 걸러내야 합니다 — 만약 스펙 내부의 `calculate` 변환(transform) 구문이 다른 이름으로 열 데이터를 매핑하고 있다면, 렌더링 스펙에는 변경된 엘리어스(alias)가 `field:`로 기재되어 있어도 원본 데이터 컬럼 정보인 `datum.X`가 실제 코드상에만 남아있을 수 있으므로 이를 지워버리면 위젯이 NaN 값을 표시하는 문제가 생깁니다.
3. 최후의 수단으로 **행 데이터 개수를 자르고(truncate)** 페이로드 안에 `{ truncated: N }` 속성을 포함시켜 위젯이 화면에 데이터가 생략되었음을 알리는 레이블을 표시할 수 있게 만듭니다.

```ts
const MAX = 130_000;
let out = rows;
if (JSON.stringify(out).length > MAX) {
  const keep = referencedFields(spec); // field: + datum.X reference 정보 파싱
  out = rows.map((r) => pick(r, keep));
  if (JSON.stringify(out).length > MAX) {
    const per = JSON.stringify(out[0] ?? {}).length || 1;
    out = out.slice(0, Math.floor(MAX / per));
  }
}
```

## 무거운 데이터 자원은 결과 페이로드가 아닌 `callServerTool`을 통해 요청

위젯 렌더링에 꼭 필요하지만 Claude가 대화 맥락에서 직접 인지할 필요는 없는 고용량 데이터(지도 지오메트리 정보, 이미지 바이트 스트림, 바이너리 객체 등)는, 위젯이 브라우저 상에 마운트(mount)된 후 전용으로 호출할 별도의 도구를 만들어 개별적으로 가져가도록 처리하십시오:

```js
const topo = await app.callServerTool({ name: "get-topojson", arguments: { level } });
```

사용자 대화 중 Claude의 도구 목록에 노출될 필요가 없는 이러한 헬퍼 도구에는 `_meta.ui.visibility: ["app"]` 메타데이터 설정을 부여하여 Claude가 해당 도구를 인지하지 못하게 처리해 주는 것이 깔끔합니다.
