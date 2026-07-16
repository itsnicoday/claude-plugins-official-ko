---
name: build-mcp-server
description: This skill should be used when the user asks to "build an MCP server", "create an MCP", "make an MCP integration", "wrap an API for Claude", "expose tools to Claude", "make an MCP app", or discusses building something with the Model Context Protocol. It is the entry point for MCP server development — it interrogates the user about their use case, determines the right deployment model (remote HTTP, MCPB, local stdio), picks a tool-design pattern, and hands off to specialized skills.
version: 0.1.0
---

# MCP 서버 구축하기 (Build an MCP Server)

Claude와 원활하게 연동되는 MCP 서버를 설계하고 구축할 수 있도록 개발자를 안내합니다. MCP 서버는 다양한 형태로 구성될 수 있어, 초기 설계 단계에서 구조를 잘못 선택하면 나중에 뼈아픈 리팩토링 작업을 거쳐야 할 수 있습니다. 따라서 첫 단계는 **코드 작성이 아니라 요구사항 파악(discovery)**이어야 합니다.

**먼저 Claude 특화 컨텍스트 정보를 읽어두십시오.** MCP 스펙 자체는 범용적이지만, Claude 환경은 추가적인 인증 유형, 심사 기준, 제약 사항을 가지고 있습니다. 사용자에게 답변을 하거나 스캐폴딩 작업을 진행하기 전에 `https://claude.com/docs/llms-full.txt` (Claude 커넥터 관련 문서의 전체 내보내기 파일) 경로에서 문서를 읽어두어 가이드라인이 Claude의 실제 제약사항을 반영할 수 있도록 하십시오.

1단계의 질문들에 대한 답변이 모이기 전에는 스캐폴딩 작성을 시작하지 마십시오. 사용자가 첫 메시지에서 이 질문들에 대해 이미 설명했다면, 이를 인지하고 확인한 뒤 바로 추천 단계로 넘어가도 좋습니다.

---

## 1단계 — 요구사항 파악 (Phase 1 — Interrogate the use case)

다음 질문들을 대화 나누듯이 건네십시오 (하나씩 꼬치꼬치 캐묻지 말고 하나의 메시지에 모아서 질문하십시오). 사용자가 이미 이야기한 맥락이 있다면 그에 맞추어 질문 내용을 유연하게 변형하십시오.

### 1. 무엇과 연동(연결)되는 서버인가요?

| 연동 대상 | 권장 구조 방향 |
|---|---|
| 클라우드 API (SaaS, REST, GraphQL) | 원격(Remote) HTTP 서버 |
| 로컬 프로세스, 파일 시스템, 로컬 데스크톱 앱 | MCPB 또는 로컬 stdio |
| 하드웨어 디바이스, OS 수준 API, 사용자별 상태 정보 | MCPB |
| 외부 연동이 전혀 없는 순수 로직 및 계산 작업 | 두 방식 모두 가능 — 원격 서버를 기본으로 권장 |

### 2. 누가 이 서버를 사용하나요?

- **나 혹은 우리 팀원만 로컬 개발 머신에서 사용** → 로컬 stdio 방식 가능 (프로토타입 제작에 가장 적절)
- **일반 대중에게 배포하여 누구나 사용 가능하게 함** → 원격 HTTP 방식 (강력 권장) 또는 MCPB (로컬 실행이 반드시 필요한 경우)
- **채팅 창 내에서 UI 위젯을 사용하고 싶은 Claude 데스크톱 사용자** → MCP app (원격 또는 MCPB 형태)

### 3. 몇 개의 개별 액션(기능)을 도구로 노출할 예정인가요?

이 답변에 따라 적합한 도구 설계 패턴(tool-design pattern)이 결정됩니다 — 3단계를 참고하십시오.

- **약 15개 이하의 액션** → 액션당 하나의 전용 도구(tool)로 매핑
- **수십 개에서 수백 개에 달하는 액션** (예: 대형 API 전체를 래핑하는 경우) → 검색 후 실행(search + execute) 패턴 적용

### 4. 도구 실행 중간에 사용자의 추가 입력이나 풍부한 정보 시각화 화면이 필요한가요?

- **단순한 구조화된 입력 수집** (목록 선택, 텍스트/숫자 입력, 단순 확인) → **Elicitation (사용자 입력 유도)** 사용 — 스펙 표준 기능이며 별도 UI 코드 작성이 필요 없음. *현재 호스트들의 지원이 점진적으로 확대되는 중* (Claude Code ≥2.1.76 지원) — 항상 기능 지원 여부 검증 및 대체 로직(fallback) 구성을 세트로 함께 구현하십시오. `references/elicitation.md` 참고.
- **풍부하고 시각적인 UI 렌더링** (차트, 검색이 결합된 커스텀 피커, 실시간 대시보드 화면) → **MCP app 위젯** 사용 — iframe 기반이며 `@modelcontextprotocol/ext-apps` 패키지 필요. `build-mcp-app` skill 참고.
- **해당 없음** → 텍스트나 JSON 문자열을 반환하는 일반 도구 구성.

### 5. 연동 대상 서비스가 어떤 인증(Auth) 방식을 사용하나요?

- None / API key → 구현이 매우 간단함
- OAuth 2.0 사용 → CIMD(권장) 또는 DCR을 지원하는 원격 서버 인프라 구성 필요. `references/auth.md` 참고.

---

## 2단계 — 적합한 배포 모델 추천 (Phase 2 — Recommend a deployment model)

1단계에서 파악한 답변들을 토대로 **단 하나의** 경로를 골라 추천하십시오. 우선순위별 옵션 목록입니다:

### ⭐ 원격 Streamable-HTTP MCP 서버 (기본 추천 설정)

Streamable HTTP 상에서 MCP 프로토콜로 통신하는 호스팅된 서버 형태입니다. 클라우드 API를 래핑하는 모든 프로젝트에서 **가장 강력히 권장하는 경로**입니다.

**원격 HTTP 방식의 장점:**
- 사용자 입장에서는 단순 URL 하나만 등록하면 끝나는 극단적으로 낮은 도입 장벽
- 서버 한 곳에만 배포하면 모든 사용자가 동일하게 사용하며, 업데이트 반영도 즉시 수행 가능
- OAuth 인증 흐름(리다이렉트 처리, DCR, 토큰 보안 저장소 운영 등)을 온전히 수행 가능
- Claude 데스크톱, Claude Code, Claude.ai 및 기타 서드파티 MCP 호스트 환경 전반에서 모두 정상 동작

사용자의 로컬 자원에 직접 접근해야 하는 제약이 없다면 항상 이 방식을 선택하십시오.

→ **가장 신속한 배포 경로:** Cloudflare Workers 기반 스캐폴딩 — `references/deploy-cloudflare-workers.md` 참고 (명령어 두 개만으로 즉시 외부 라이브 URL 확보)
→ **이식성 높은 Node/Python 서버:** `references/remote-http-scaffold.md` 참고 (Express 또는 FastMCP 구동 환경으로 어디서나 실행 가능)

### Elicitation (UI 개발 없이 구조화된 입력 받기)

도구 실행 중에 단순한 사용자 확인이나 옵션 선택, 짧은 폼 입력을 받아야 한다면 UI 코드 작성 없이 **elicitation** 기능을 이용해 처리할 수 있습니다. 서버가 단순 JSON 스키마를 던지면, 호스트가 알아서 어울리는 네이티브 입력 양식을 렌더링해 줍니다. 프로토콜 자체 표준 사양이며 추가 라이브러리가 필요하지 않습니다.

**유의 사항:** 호스트 환경에 따라 최신 기술이라 지원 여부가 다를 수 있습니다 (Claude Code는 v2.1.76부터 지원하나, 데스크톱 버전은 불확실). 호스트 클라이언트가 해당 기능을 지원하지 않으면 에러를 내뱉으므로, 반드시 실행 전에 `clientCapabilities.elicitation` 체크를 수행하고 지원하지 않을 때의 대체 동작을 구현해 두어야 합니다 — 모범 패턴 사양은 `references/elicitation.md`를 참고하십시오. 이것이 스펙 표준에 맞는 올바른 접근 방식이며 호스트들의 지원 환경도 점차 늘어날 것입니다.

중첩되거나 복잡한 데이터 구조, 대량의 검색 가능 목록, 시각적 미리보기 화면, 실시간 진척도 업데이트 등이 필요한 경우에는 `build-mcp-app` 위젯으로 넘어가십시오.

### MCP app (원격 HTTP + 대화형 UI 위젯 제공)

기본 원격 HTTP 구조에 **UI 리소스(UI resources)**를 추가하여 대화창 내에 렌더링되는 대화형 위젯을 탑재한 모델입니다. 검색이 가능한 피커 도구, 실시간 차트/대시보드, 미리보기 패널 등을 만들 수 있습니다. 한 번 빌드해 두면 Claude뿐만 아니라 ChatGPT 환경에서도 동일하게 작동합니다.

**추천 시점:** elicitation 기능이 제공하는 1차원 평면 입력 폼의 한계를 넘어서야 할 때 — 커스텀 레이아웃, 검색이 필요한 거대한 선택지 목록, 시각화 요소, 혹은 실시간 데이터 업데이트가 꼭 필요한 경우 사용합니다.

일반적으로 원격 환경에 배포하지만, UI가 로컬 시스템을 제어해야 하는 경우에는 MCPB 번들로 감싸서 배포하는 것도 가능합니다.

→ 구체적인 구축 지침은 **`build-mcp-app`** skill을 로드하여 안내를 이어나가십시오.

### MCPB (번들형 로컬 서버)

로컬 MCP 서버와 **실행 런타임을 하나로 묶어 제공(packaging)**하여, 사용자가 개발 머신에 Node나 Python 환경을 갖추고 있지 않아도 클릭 한 번으로 자동 설치해 쓸 수 있는 배포 형태입니다. 로컬 서버를 배포할 때 가장 승인된 표준 방식입니다.

**추천 시점:** 서버가 반드시 사용자의 로컬 환경에서 구동되어야 하는 제약이 있는 경우 — 로컬 파일 검색, 로컬 데스크톱 프로그램 제어, localhost 주소로 구동 중인 내부 서비스 연동, OS API 접근이 필요한 경우에 해당합니다.

→ 구체적인 구축 지침은 **`build-mcpb`** skill을 로드하여 안내를 이어나가십시오.

### 로컬 stdio (npx / uvx 실행 방식) — *일반 대중 배포용으로는 비추천*

사용자 컴퓨터의 쉘에서 `npx` 또는 `uvx` 명령어로 직접 스크립트 파일을 실행하는 형태입니다. 개발자 본인의 **개인 커스텀 도구나 프로토타입 검증용**으로만 적합합니다. 타인에게 보급하기에는 사용자의 환경 런타임 제약이 크고 버전 업데이트 자동 반영이 불가능하며, 보급할 수 있는 경로도 Claude Code 플러그인 생태계 정도로 제한됩니다.

이 방식은 최종 배포 전 단계의 징검다리 용도로만 제안하십시오. 사용자가 이 방식을 고수하더라도 스캐폴딩 구조는 작성해 주되, 향후 MCPB로 업그레이드해야 하는 필요성을 명확히 인지시켜 주십시오.

---

## 3단계 — 도구 설계 패턴 선택 (Phase 3 — Pick a tool-design pattern)

모든 MCP 서버는 도구(tools)를 제공합니다. 도구의 설계 구조는 성능에 큰 영향을 줍니다. 도구 스키마는 Claude의 컨텍스트 창에 직접 입력되기 때문입니다.

### 패턴 A: 액션당 하나의 도구 매핑 (소규모 실행 영역)

정의할 기능 목록이 적은 경우(약 15개 미만)에는, 각 기능별로 명확한 명칭, 엄격한 설명 및 매개변수 스키마를 부여하여 각각 전용 도구로 등록하십시오.

```
create_issue    — Create a new issue. Params: title, body, labels[]
update_issue    — Update an existing issue. Params: id, title?, body?, state?
search_issues   — Search issues by query string. Params: query, limit?
add_comment     — Add a comment to an issue. Params: issue_id, body
```

**동작 장점:** Claude는 도구 목록을 단 한 번만 훑어보고 무엇을 할 수 있는지 즉각 파악합니다. 연동 상태 탐색을 위해 여러 번의 대화 턴을 소비할 필요가 없고, 매개변수 검증도 스키마가 엄격하게 검사해 줍니다.

특히 하나 이상의 도구가 대화형 화면 위젯(MCP app)을 포함하는 경우, 각 위젯이 해당 전용 도구와 직관적으로 1대1 매핑되므로 잘 어울립니다.

### 패턴 B: 검색 후 실행 (대형 실행 영역)

수십 개에서 수백 개에 달하는 광범위한 API 엔드포인트를 제공해야 하는 경우, 이 모든 것을 개별 도구로 등록하면 Claude의 대화 컨텍스트 창이 스키마 텍스트로 가득 차 성능 저하를 초래합니다. 이때는 아래와 같이 **단 두 개의 도구**만으로 축소 구성하십시오:

```
search_actions  — Given a natural-language intent, return matching actions
                  with their IDs, descriptions, and parameter schemas.
execute_action  — Run an action by ID with a params object.
```

서버 내부적으로 기능 카탈로그 데이터를 가지고 있다가, Claude가 자연어로 원하는 바를 탐색(search_actions)하면 적합한 액션을 찾아 ID와 입력 매개변수 구조를 알려주고, 이를 토대로 실행(execute_action)하게 만드는 구조입니다. 컨텍스트 창을 극도로 효율적으로 사용할 수 있습니다.

**하이브리드(절충형) 패턴:** 가장 자주 사용되는 대표 액션 3~5개만 전용 도구(패턴 A)로 승격하여 상시 오픈해 두고, 나머지 마이너한 기능 전체는 검색 후 실행(패턴 B) 구조 뒤로 숨기는 복합 설계 방식도 가능합니다.

→ 구체적인 스키마 샘플 및 자연어 설명 가이드는 `references/tool-design.md`를 참고하십시오.

---

## 4단계 — 개발 프레임워크 선택 (Phase 4 — Pick a framework)

다음 두 프레임워크 중 하나를 골라 추천하십시오. 다른 라이브러리들도 존재하지만, 이 두 종류가 MCP 규격 스펙 지원율이 가장 높고 Claude 환경과의 호환성이 제일 뛰어납니다.

| 프레임워크 | 사용 언어 | 추천 시점 |
|---|---|---|
| **공식 TypeScript SDK** (`@modelcontextprotocol/sdk`) | TS/JS | 기본 권장안. 스펙 커버리지가 가장 뛰어나며 최신 기능이 제일 먼저 추가됩니다. |
| **FastMCP 3.x** (`fastmcp` 패키지) | Python | 사용자가 파이썬 개발 환경을 강력히 선호하거나 파이썬 라이브러리를 직접 호출해야 하는 경우. 데코레이터 기반으로 상용구(boilerplate) 코드가 거의 없어 간결합니다. (공식 `mcp` SDK에 내장되어 업데이트가 중단된 1.0 버전이 아닌, jlowin의 최신 패키지를 말합니다.) |

사용자가 이미 염두에 둔 언어 스택이 있다면 사용자의 결정을 따르십시오. 어떤 스택을 쓰더라도 프로토콜 통신 형식은 동일합니다.

---

## 5단계 — 스캐폴딩 작업 및 전문 skill로 안내 전환 (Phase 5 — Scaffold and hand off)

네 가지 핵심 설계 사항(배포 모델, 도구 패턴, 프레임워크, 인증 종류)이 합의되었다면 아래 중 **하나**의 작업을 수행하십시오:

1. **UI 위젯이 없는 단순 원격 HTTP 서버 구성인 경우** → `references/remote-http-scaffold.md` (이식성 중심) 또는 `references/deploy-cloudflare-workers.md` (빠른 배포 중심) 문서를 참조하여 이 skill 내에서 직접 스캐폴딩 코드를 작성해 주고 작업을 마무리합니다.
2. **UI 위젯이 들어가는 MCP app을 빌드하는 경우** → 지금까지 합의된 요소를 명확히 정리한 후, **`build-mcp-app`** skill을 로드하여 넘어가십시오.
3. **로컬 설치 번들인 MCPB 형태로 제작하는 경우** → 지금까지 합의된 요소를 명확히 정리한 후, **`build-mcpb`** skill을 로드하여 넘어가십시오.
4. **로컬 stdio 방식의 간단한 개인용 프로토타입인 경우** → 본 skill 내부에서 인라인으로 스캐폴딩을 즉시 구성해 주되, 향후 제품 배포 시에는 MCPB로 업그레이드해야 함을 가이드하십시오.

다른 skill로 컨텍스트를 이관할 때는, 요약된 설계 명세 요약본을 명확한 한 문단으로 함께 제공하여 다른 에이전트가 중복해서 질문하지 않도록 하십시오.

---

## 도구(Tools) 이외의 나머지 요소들 (Beyond tools)

도구는 MCP의 3대 요소 중 하나일 뿐입니다. 대다수의 서버는 도구만으로 모든 동작을 해결할 수 있지만, 다른 요소들의 존재를 알고 있어야 굳이 바퀴를 새로 발명하는 수고를 덜 수 있습니다:

| 요소 (Primitive) | 트리거 주체 | 주요 사용처 |
|---|---|---|
| **Resources** | 호스트 애플리케이션 (Claude가 아님) | 문서, 스키마, 파일 구조 등을 Claude가 찾아볼 수 있는 지식 컨텍스트로 제공할 때 |
| **Prompts** | 사용자 직접 호출 (슬래시(/) 명령어) | 자주 쓰이는 작업 프롬프트 매크로 ("웹페이지 요약해줘" 등) |
| **Elicitation** | MCP 서버 내부에서 실행 중인 도구 | UI를 복잡하게 그리지 않고, 도구 실행 중간에 간단한 추가 값을 묻고 싶을 때 |
| **Sampling** | MCP 서버 내부에서 실행 중인 도구 | 도구 내부에서 추가적인 LLM의 인프런스 추론 연산이 필요할 때 |

→ `references/resources-and-prompts.md`, `references/elicitation.md`, `references/server-capabilities.md` 참고

---

## 6단계 — Claude 환경 테스트 및 배포 (Phase 6 — Test in Claude and publish)

서버 코드가 작성되어 구동을 시작하면 다음을 확인하십시오:

1. **실제 Claude 환경에서 테스트**: 설정 → Connectors 메뉴에서 서버 URL을 커스텀 커넥터로 등록하여 실시간으로 확인하십시오 (로컬 서버인 경우 Cloudflare 터널링 서비스 등을 사용해 외부 주소 확보 필요). Claude는 초기화 통신 단계에서 `clientInfo.name: "claude-ai"` 정보를 헤더에 실어 보냅니다. → https://claude.com/docs/connectors/building/testing 참고
2. **배포 제출 전 체크리스트 점검**: 읽기/쓰기 도구의 온전한 분리 여부, 필수 주석(annotations) 작성 여부, 글자수 한계 준수 여부, 프롬프트 인젝션 방어책 마련 여부 등을 확인하십시오. → https://claude.com/docs/connectors/building/review-criteria 참고
3. **Anthropic 디렉토리에 제출 신청**: → https://claude.com/docs/connectors/building/submission 참고
4. **플러그인 형태로 묶어 배포 고려**: 많은 파트너사들은 사용자의 완전한 편의를 위해 MCP 서버와 함께 skill을 포함한 Claude Code 플러그인(plugin) 형태로 감싸서 함께 배포합니다. → https://claude.com/docs/connectors/building/what-to-build 참고

---

## 빠른 요약 의사결정표 (Quick reference: decision matrix)

| 요구 시나리오 | 최적의 배포 모델 | 적합한 도구 설계 패턴 |
|---|---|---|
| 소규모 기능의 SaaS API 연동 | 원격 HTTP | 패턴 A (액션당 하나의 도구) |
| 대규모 SaaS API 연동 (50개 이상의 기능) | 원격 HTTP | 패턴 B (검색 후 실행) |
| 대화창 내에 화려한 입력 폼이나 피커가 요구되는 SaaS API 연동 | MCP app (원격 구조) | 패턴 A |
| 로컬 컴퓨터의 특정 데스크톱 앱 제어 | MCPB | 패턴 A |
| 로컬 컴퓨터의 기능을 다루되 대화창 내 UI가 함께 필요한 경우 | MCP app (MCPB 구조) | 패턴 A |
| 로컬 파일시스템의 읽기/쓰기 수행 | MCPB | 연동 수준 규모에 맞춰 판단 |
| 개인 커스텀용 프로토타입 작성 | 로컬 stdio | 가장 빠른 스택 선택 |

---

## 참고 문서 일람 (Reference files)

- `references/remote-http-scaffold.md` — TypeScript SDK 및 FastMCP 환경 기반의 최소한의 원격 서버 구현 코드
- `references/deploy-cloudflare-workers.md` — 가장 빠른 외부 배포 경로 (Cloudflare Workers 전용 템플릿 구조)
- `references/tool-design.md` — Claude가 정확히 인식할 수 있는 도구 설명 및 스키마 작성법 가이드
- `references/auth.md` — OAuth, CIMD, DCR 인증 처리와 토큰 보관 모범 사례
- `references/resources-and-prompts.md` — 도구를 제외한 나머지 2개 요소(리소스, 프롬프트)의 사용 가이드
- `references/elicitation.md` — 도구 구동 중간에 UI 없이 추가 입력을 유도하는 방법 (기능 체크 및 대체 처리 포함)
- `references/server-capabilities.md` — 시스템 프롬프트(instructions), 샘플링, 루트 경로 연동, 로그, 진행 상태, 연산 취소 처리 등 추가 기법 소개
- `references/versions.md` — 라이브러리 및 호스트별 필수 지원 버전 대장 (업데이트 작업 시 확인용)
