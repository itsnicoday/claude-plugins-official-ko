# 코드 맵 템플릿 (Code Map Template)

컴포넌트 관계, 데이터 흐름, 계층 다이어그램, 시스템 아키텍처 등 코드베이스 아키텍처를 시각화하고 피드백을 위해 대화형으로 댓글을 달 수 있는 플레이그라운드를 빌드할 때 이 템플릿을 사용하십시오.

## 레이아웃

```
+-------------------+----------------------------------+
|                   |                                  |
|  컨트롤 영역:     |  SVG 캔버스                      |
|  • 뷰 프리셋      |  (노드 + 연결선)                 |
|  • 계층 토글      |  줌 컨트롤 포함                  |
|  • 연결 유형 필터 |                                  |
|                   |  범례 (좌측 하단)                |
|  댓글 목록 (n):   |                                  |
|  • 삭제 버튼이    +----------------------------------+
|    포함된 사용자  |  프롬프트 출력                   |
|    댓글 목록      |  [ 프롬프트 복사 ]               |
+-------------------+----------------------------------+
```

코드 맵 플레이그라운드는 아키텍처 다이어그램을 표시하기 위해 SVG 캔버스를 사용합니다. 사용자가 컴포넌트를 클릭하여 댓글을 추가하면 생성된 프롬프트의 일부가 됩니다. 사용자는 계층 및 연결 유형 필터를 통해 시스템의 특정 부분에 집중할 수 있습니다.

## 코드 맵용 컨트롤 유형

| 결정 사항 | 컨트롤 유형 | 예시 |
|---|---|---|
| 시스템 뷰 | 프리셋 버튼 | Full System, Chat Flow, Data Flow, Agent System |
| 노출할 계층 | 체크박스 | Client, Server, SDK, Data, External |
| 연결 유형 | 색상 표시기가 있는 체크박스 | Data Flow (청색), Tool Calls (녹색), Events (적색) |
| 컴포넌트 피드백 | 클릭-댓글 입력 모달 | 피드백 작성을 위한 텍스트 영역(textarea)이 포함된 모달 노출 |
| 줌 배율 | +/−/초기화 버튼 | 세부 조정을 위한 SVG 크기 조절 |

## 캔버스 렌더링

동적으로 생성되는 노드와 경로가 포함된 `<svg>` 요소를 사용합니다. 주요 패턴:

- **노드(Nodes):** 제목과 부제목(파일 경로)이 포함된 둥근 모서리 사각형
- **연결선(Connections):** 화살표 마커가 있고 유형별로 스타일링된 곡선 경로 (베지에 곡선)
- **계층 구조화:** Y좌표 범위를 기준으로 노드 그룹화 (예: y: 30-80 = Client 계층, y: 130-180 = Server 계층)
- **클릭-댓글 작성:** 노드 클릭 → 모달 열기 → 댓글 저장 → 노드에 시각적 표시기 반영
- **필터링:** 계층별 노드 노출 토글, 유형별 연결선 노출 토글

```javascript
const nodes = [
  { id: 'api-client', label: 'API Client', subtitle: 'src/api/client.ts',
    x: 100, y: 50, w: 140, h: 45, layer: 'client', color: '#dbeafe' },
  // ...
];

const connections = [
  { from: 'api-client', to: 'server', type: 'data-flow', label: 'HTTP' },
  { from: 'server', to: 'db', type: 'data-flow' },
  // ...
];

function renderDiagram() {
  const visibleNodes = nodes.filter(n => state.layers[n.layer]);
  // Draw connections first (under nodes), then nodes
  connections.forEach(c => drawConnection(c));
  visibleNodes.forEach(n => drawNode(n));
}
```

## 연결 유형 및 스타일링

3~5가지 연결 유형에 대해 고유한 시각적 스타일을 정의합니다:

| 유형 | 색상 | 스타일 | 용도 |
|---|---|---|---|
| `data-flow` | 청색 (#3b82f6) | 실선 | 요청/응답, 데이터 전달 |
| `tool-call` | 녹색 (#10b981) | 파선 (6,3) | 함수 호출, API 호출 |
| `event` | 적색 (#ef4444) | 점선 (4,4) | 비동기 이벤트, 발행/구독(pub/sub) |
| `skill-invoke` | 주황색 (#f97316) | 점선 (8,4) | 플러그인/스킬 활성화 |
| `dependency` | 회색 (#6b7280) | 점선 (Dotted) | 임포트(import)/요구(require) 관계 |

화살표 머리를 그리기 위해 SVG 마커를 사용합니다:

```html
<marker id="arrowhead-blue" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
  <polygon points="0 0, 8 3, 0 6" fill="#3b82f6"/>
</marker>
```

## 댓글 시스템

코드 맵의 핵심적인 차별점은 클릭-댓글 작성 기능입니다:

1. **노드 클릭** → 컴포넌트 이름, 파일 경로, 텍스트 입력 영역이 포함된 모달 노출
2. **댓글 저장** → 댓글 목록에 추가하고, 해당 노드에 시각적 표시(색상 테두리)를 반영
3. **댓글 보기** → 사이드바 목록에 컴포넌트 이름, 댓글 미리보기, 삭제 버튼 노출
4. **댓글 삭제** → 목록에서 제거하고, 노드의 시각적 표시 업데이트 및 프롬프트 재생성

댓글에는 컴포넌트 컨텍스트가 포함되어야 합니다:

```javascript
state.comments.push({
  id: Date.now(),
  target: node.id,
  targetLabel: node.label,
  targetFile: node.subtitle,
  text: userInput
});
```

## 코드 맵용 프롬프트 출력

프롬프트는 시스템 컨텍스트와 사용자의 댓글을 결합합니다:

```
[visible layers] 계층에 집중한 [PROJECT NAME] 아키텍처입니다.

특정 컴포넌트에 대한 피드백:

**API Client** (src/api/client.ts):
여기 지수 백오프를 사용하는 재시도 로직을 추가하고 싶습니다.

**Database Manager** (src/db/manager.ts):
커넥션 풀링을 추가할 수 있을까요? 현재 구현에서는 요청당 새로운 커넥션을 생성합니다.

**Auth Middleware** (src/middleware/auth.ts):
JWT 토큰을 검증하고 사용자 컨텍스트를 추출해야 합니다.
```

사용자가 직접 추가한 댓글만 포함시키십시오. 전체 시스템을 표시하지 않는 경우 어떤 계층이 노출되어 있는지 언급하십시오.

## 실제 데이터 채우기

특정 코드베이스를 다루는 경우 다음을 미리 채워둡니다:

- **노드:** 실제 파일 경로가 포함된 15~25개의 핵심 컴포넌트
- **연결선:** 실제 임포트/호출에 기반한 20~40개의 관계
- **계층:** 논리적 그룹화 (UI, API, 비즈니스 로직, 데이터, 외부)
- **프리셋:** "Full System", "Frontend Only", "Backend Only", "Data Flow"

노드를 계층별로 가로 밴드 형태로 일정하게 배치하십시오.

## 계층 색상 팔레트 (라이트 테마)

| 계층 | 노드 채우기 색상 | 설명 |
|---|---|---|
| Client/UI | #dbeafe (blue-100) | React 컴포넌트, 훅, 페이지 |
| Server/API | #fef3c7 (amber-100) | Express 라우트, 미들웨어, 핸들러 |
| SDK/Core | #f3e8ff (purple-100) | 핵심 라이브러리, SDK 래퍼 |
| Agent/Logic | #dcfce7 (green-100) | 비즈니스 로직, 에이전트, 프로세서 |
| Data | #fce7f3 (pink-100) | 데이터베이스, 캐시, 스토리지 |
| External | #fbcfe8 (pink-200) | 서드파티 서비스, API |

## 예시 주제

- 코드베이스 아키텍처 탐색기 (모듈, 임포트, 데이터 흐름)
- 마이크로서비스 맵 (서비스, 큐, 데이터베이스, API 게이트웨이)
- React 컴포넌트 트리 (컴포넌트, 훅, 컨텍스트, 상태)
- API 아키텍처 (라우트, 미들웨어, 컨트롤러, 모델)
- 에이전트 시스템 (프롬프트, 도구, 스킬, 서브에이전트)
- 데이터 파이프라인 (소스, 변환, 싱크, 스케줄링)
- 플러그인/확장 아키텍처 (코어, 플러그인, 훅, 이벤트)
