# 커넥터 디렉토리 제출 전 점검 목록 (Connector-directory submission checklist)

원격 MCP app을 Claude 커넥터 디렉토리에 제출하기 전의 사전 점검 리스트입니다. 각 항목은 실제 심사 시 합격/탈락을 결정하는 핵심 기준이 됩니다.

| 영역 | 세부 요구사항 |
|---|---|
| **인증 (Auth)** | OAuth (DCR 또는 CIMD 방식) 또는 **`none`** (인증 없음)으로 구성되어야 합니다. 고정된 static bearer 토큰 방식은 개별 사설 배포 환경에서만 허용되며 공용 디렉토리 등록은 불가능합니다. 공공 데이터 제공 서버인 경우 인증 없음 모드도 승인 대상이 될 수 있으며, 이 경우 업스트림 API 키는 서버 측에서 안전하게 관리해야 합니다. |
| **도구 주석 (Annotations)** | 모든 도구는 `annotations.title`을 설정하고 관련 힌트들을 지정해야 합니다: 정보 조회/검색용 도구에는 `readOnlyHint: true`, 쓰기용 도구에는 `destructiveHint` / `idempotentHint` 주석, 외부 시스템과 통신하는 경우에는 `openWorldHint: true`를 지정합니다. |
| **도구 이름** | 64자 이내의 길이로 snake_case 또는 kebab-case 규칙을 준수해야 합니다. |
| **위젯 레이아웃** | 인라인 높이가 500px 이하여야 하며, 내부 중첩 스크롤 컨테이너 구성을 지양하고, 터치 영역 크기는 최소 44pt 이상이어야 하며, 두 테마 모두에 대해 WCAG-AA 명도 대비 규격을 준수해야 합니다. |
| **테마 연동** | `html, body { background: transparent }` 속성 지정, `<meta name="color-scheme" content="light dark">` 메타 태그 적용, `applyHostStyleVariables`를 이용한 호스트 CSS 변수 상속 반영을 준수해야 합니다. |
| **외부 링크 연동** | `app.openLink` 함수를 사용하십시오. 연동 대상 주소 도메인(예: `https://api.example.com`)은 커넥터 설정 내의 *Allowed link URIs* 목록에 등록해야 사용자가 이동 버튼 클릭 시 확인 창 팝업을 건너뛰게 할 수 있습니다. |
| **도우미 도구 필터링** | 위젯 화면용 도우미 도구(지도 탑포지션 또는 이미지 청크 로더 등)에는 `_meta.ui.visibility: ["app"]` 속성을 부여하여 Claude의 일반 도구 추천 목록에 노출되지 않게 처리해야 합니다. |
| **스크린샷** | 3~5장의 PNG 포맷 이미지 파일이 필요하며, 가로 폭은 1000px 이상이어야 하고, 화면 주변의 프롬프트 텍스트 등을 잘라내고 순수 앱 렌더링 화면만 캡처된 형태여야 합니다. |

인증 없는 공개 엔드포인트에 대한 레이트 리밋 제어 및 IP 분기 분류 설계에 대해서는 `abuse-protection.md` 문서를 참고하십시오.
