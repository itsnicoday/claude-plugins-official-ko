# mcp-server-dev

Claude와 원활하게 연동되는 MCP 서버를 설계하고 구축하기 위한 skill 모음입니다.

## 포함된 기능

전체 빌드 경로를 구성하는 세 가지 skill이 포함되어 있습니다:

| Skill | 용도 |
|---|---|
| **`build-mcp-server`** | 진입점. 요구사항을 파악하고 배포 모델(원격 HTTP / MCPB / 로컬 stdio) 및 도구 디자인 패턴을 선택한 뒤 특화된 skill로 라우팅합니다. |
| **`build-mcp-app`** | 채팅창 내에 렌더링되는 대화형 UI 위젯(폼, 피커, 확인 대화창 등)을 추가합니다. 원격 서버 및 MCPB 번들에서 작동합니다. |
| **`build-mcpb`** | Node/Python 설치 없이도 사용할 수 있도록 로컬 stdio 서버를 런타임과 함께 패키징합니다. 로컬 머신에 직접 접근해야 하는 서버에 적합합니다. |

## 동작 방식

`build-mcp-server`가 진입점 역할을 합니다. 어떤 환경에 연결하는지, 누가 사용하는지, 실행 영역의 규모가 어느 정도인지, 채팅 내 UI가 필요한지 등을 질문합니다. 답변을 바탕으로 다음 네 가지 경로 중 하나를 추천합니다:

- **원격 streamable-HTTP** (클라우드 API를 래핑하는 경우 기본 추천 사항) — 인라인으로 스캐폴딩 생성
- **MCP app** — `build-mcp-app`으로 전환
- **MCPB** — `build-mcpb`로 전환
- **로컬 stdio 프로토타입** — MCPB 업그레이드 안내와 함께 인라인으로 스캐폴딩 생성

각 skill은 메인 지침에 다 담지 못한 핵심 참조 파일들을 제공합니다: 인증 흐름(DCR/CIMD), 도구 설명 작성법, 위젯 템플릿, 매니페스트 스키마, 보안 강화 지침 등.

## 사용법

Claude에게 "help me build an MCP server"라고 요청하면 진입점 skill이 자동으로 트리거됩니다. 또는 직접 호출할 수도 있습니다:

```
/mcp-server-dev:build-mcp-server
```
