---
name: mcp-integration
description: This skill should be used when the user asks to "add MCP server", "integrate MCP", "configure MCP in plugin", "use .mcp.json", "set up Model Context Protocol", "connect external service", mentions "${CLAUDE_PLUGIN_ROOT} with MCP", or discusses MCP server types (SSE, stdio, HTTP, WebSocket). Provides comprehensive guidance for integrating Model Context Protocol servers into Claude Code plugins for external tool and service integration.
version: 0.1.0
---

# Claude Code 플러그인을 위한 MCP 통합 (MCP Integration for Claude Code Plugins)

## 개요 (Overview)

Model Context Protocol (MCP)은 구조화된 도구 접근 권한을 제공하여 Claude Code 플러그인이 외부 서비스 및 API와 통합할 수 있도록 돕습니다. MCP 통합을 활용해 외부 서비스의 기능을 Claude Code 내부에서 도구처럼 사용할 수 있게 노출하십시오.

**핵심 기능:**
- 외부 서비스(데이터베이스, API, 파일 시스템 등) 연동
- 단일 서비스에서 10개 이상의 연관된 도구들을 묶어 공급
- OAuth 및 복잡한 인증 흐름 대행
- 자동화된 설정을 위해 플러그인 빌드 패키지에 MCP 서버 포함(bundle) 가능

## MCP 서버 구성 방식 (MCP Server Configuration Methods)

플러그인에 MCP 서버를 포함시키는 방식은 크게 두 가지로 나뉩니다:

### 방식 1: 전용 .mcp.json 파일 활용 (권장) (Method 1: Dedicated .mcp.json (Recommended))

플러그인 루트에 `.mcp.json` 파일을 새로 생성합니다:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

**이점:**
- 역할 분담(separation of concerns)이 명확함
- 유지보수가 훨씬 용이함
- 여러 대의 서버를 설정하고 연동하기에 가장 적절함

### 방식 2: plugin.json 내에 인라인 지정 (Method 2: Inline in plugin.json)

`plugin.json` 본문에 직접 `mcpServers` 필드를 추가합니다:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**이점:**
- 단일 구성 파일 형태로 관리가 용이함
- 단일 서버만 띄우는 아주 단순한 구성의 플러그인에 적합함

## MCP 서버 유형 (MCP Server Types)

### stdio (로컬 프로세스) (stdio (Local Process))

로컬 MCP 서버를 하위 프로세스로 기동합니다. 로컬 유틸리티 실행 및 커스텀 로컬 서버 구동에 적합합니다.

**구성:**
```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
    "env": {
      "LOG_LEVEL": "debug"
    }
  }
}
```

**사용 사례:**
- 로컬 파일 시스템 제어
- 로컬 데이터베이스 직접 연동
- 자체 제작 커스텀 MCP 서버
- NPM 패키지로 발행된 공식/비공식 MCP 서버

**프로세스 관리 모델:**
- Claude Code가 해당 프로세스를 직접 띄우고 라이프사이클을 추적합니다.
- stdin/stdout 파이프 통신 방식을 취합니다.
- Claude Code 종료 시 프로세스도 함께 종료됩니다.

### SSE (Server-Sent Events)

웹상에 호스팅된 외부 MCP 서버에 연결합니다 (OAuth 인증 흐름 지원). 클라우드 서비스 연동 시 가장 적합합니다.

**구성:**
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

**사용 사례:**
- 공식적으로 원격 호스팅된 MCP 서버 (Asana, GitHub 등) 연동
- MCP 규격 엔드포인트를 노출해 둔 클라우드 서비스 연동
- OAuth 기반 인증 흐름이 필요할 때
- 로컬에 별도의 구동 엔진 설치를 생략하고자 할 때

**인증 처리:**
- OAuth 인증 흐름을 Claude Code가 백그라운드에서 완전히 대행합니다.
- 최초 실행 시 사용자에게 브라우저 인증 승인 창을 표시합니다.
- 발급받은 토큰은 Claude Code 자체 시스템 보안 영역에서 관리됩니다.

### HTTP (REST API)

토큰 인증 방식을 사용하는 RESTful MCP 서버와 직접 연동합니다.

**구성:**
```json
{
  "api-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "X-Custom-Header": "value"
    }
  }
}
```

**사용 사례:**
- REST API 기반 MCP 백엔드 연동
- Bearer 토큰 및 API 키 기반 인증
- 사내 자체 API 연동 환경
- 무상태 단순 통신 구조

### WebSocket (실시간) (WebSocket (Real-time))

실시간 양방향 통신을 위해 WebSocket 기반 MCP 서버에 연결합니다.

**구성:**
```json
{
  "realtime-service": {
    "type": "ws",
    "url": "wss://mcp.example.com/ws",
    "headers": {
      "Authorization": "Bearer ${TOKEN}"
    }
  }
}
```

**사용 사례:**
- 실시간 데이터 스트리밍 연동
- 연결 상태 지속 유지 (Persistent Connection)
- 서버 측의 실시간 푸시 알림
- 저지연 극대화가 요구될 때

## 환경 변수 치환 (Environment Variable Expansion)

모든 MCP 설정 파트에서는 환경 변수 바인딩 서식을 활용해 동적으로 경로를 치환할 수 있습니다:

**${CLAUDE_PLUGIN_ROOT}** - 플러그인 실제 설치 디렉토리 (이식성 보장을 위해 사용 권장):
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-server"
}
```

**사용자 쉘 환경 변수** - 사용자의 실행 쉘 세션으로부터 온 변수 치환:
```json
{
  "env": {
    "API_KEY": "${MY_API_KEY}",
    "DATABASE_URL": "${DB_URL}"
  }
}
```

**권장 사항:** 런타임에 필요한 모든 환경 변수 정보는 플러그인 README 파일에 명확하게 문서화해 두십시오.

## MCP 도구 이름 정의 규칙 (MCP Tool Naming)

MCP 서버가 감지되면 제공하는 도구들은 자동으로 접두사가 붙은 이름 형태로 매핑됩니다:

**서식:** `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`

**예시:**
- 플러그인(Plugin): `asana`
- 서버(Server): `asana`
- 도구(Tool): `create_task`
- **시스템 인식 이름:** `mcp__plugin_asana_asana__asana_create_task`

### 명령어 내부에서 MCP 도구 사용하기 (Using MCP Tools in Commands)

명령어 설정 마크다운 파일의 YAML 프론트매터 블록에 사용할 외부 도구를 명시적으로 지정하십시오:

```markdown
---
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task",
  "mcp__plugin_asana_asana__asana_search_tasks"
]
---
```

**와일드카드 활용 (신중하게 사용):**
```markdown
---
allowed-tools: ["mcp__plugin_asana_asana__*"]
---
```

**보안 권장 사항:** 시스템 보안 및 권한 제어를 위해 와일드카드보다는 활용하고자 하는 개별 도구를 지명하여 안전하게 사전 허용(`allowed-tools`) 처리하십시오.

## 수명 주기 관리 (Lifecycle Management)

**자동 실행 규칙:**
- 플러그인이 로드 및 활성화될 때 연관된 MCP 서버가 자동으로 기동됩니다.
- 최초로 도구를 실행하기 전에 연결 통신 협상(connection handshake)이 진행됩니다.
- 구성을 변경한 경우, 반드시 Claude Code 터미널 세션을 재부팅해야만 변경된 내역이 기동 시점에 반영됩니다.

**라이프사이클 단계:**
1. 플러그인이 로드됩니다.
2. MCP 구성 설정이 파싱됩니다.
3. 로컬 프로세스를 구동(stdio)하거나 원격 세션 연결(SSE/HTTP/WS)을 맺습니다.
4. 가용한 도구들의 정보 스키마를 검색하고 등록합니다.
5. `mcp__plugin_...__...` 식별자 명칭으로 도구들을 즉시 사용할 수 있게 노출합니다.

**서버 상태 확인:**
`/mcp` 명령어를 터미널에서 수행하여 플러그인이 주입한 MCP 서버 상태를 확인할 수 있습니다.

## 인증 패턴 (Authentication Patterns)

### OAuth (SSE/HTTP 방식)

OAuth 2.0 흐름은 Claude Code에 의해 완전히 백그라운드 대행 처리됩니다:

```json
{
  "type": "sse",
  "url": "https://mcp.example.com/sse"
}
```

최초 도구 호출 시 브라우저 팝업이 활성화되어 사용자의 인증 동의를 받습니다. 추가적인 인증 설정을 기재할 필요가 없습니다.

### 토큰 기반 (Headers 방식)

환경 변수나 설정값을 HTTP 헤더에 담아 인증을 요청합니다:

```json
{
  "type": "http",
  "url": "https://api.example.com",
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

필요 환경 변수 목록은 README에 빠짐없이 기재합니다.

### 환경 변수 전달 (stdio 방식)

로컬 프로세스 구동 환경에 필요한 자격 정보를 주입합니다:

```json
{
  "command": "python",
  "args": ["-m", "my_mcp_server"],
  "env": {
    "DATABASE_URL": "${DB_URL}",
    "API_KEY": "${API_KEY}",
    "LOG_LEVEL": "info"
  }
}
```

## 통합 패턴 (Integration Patterns)

### 패턴 1: 단순 도구 래퍼 (Pattern 1: Simple Tool Wrapper)

사용자와의 대화를 통해 필요한 매개변수를 수집하고 도구를 연동 호출합니다:

```markdown
# Command: create-item.md
---
allowed-tools: ["mcp__plugin_name_server__create_item"]
---

Steps:
1. Gather item details from user
2. Use mcp__plugin_name_server__create_item
3. Confirm creation
```

**용도:** 실제 도구를 호출하기 전에 매개변수 유효성을 꼼꼼히 검사하고 전처리할 때 사용합니다.

### 패턴 2: 자율적 에이전트 (Pattern 2: Autonomous Agent)

에이전트가 주어진 역할을 완수하기 위해 자율적으로 도구들을 조합하여 태스크를 수행합니다:

```markdown
# Agent: data-analyzer.md

Analysis Process:
1. Query data via mcp__plugin_db_server__query
2. Process and analyze results
3. Generate insights report
```

**용도:** 중간 개입 없이 복잡한 다단계 호출 워크플로우를 대행 수행시킬 때 적절합니다.

### 패턴 3: 다중 서버 플러그인 (Pattern 3: Multi-Server Plugin)

단일 플러그인 아래에 이종의 복수 MCP 서비스를 복합 연동합니다:

```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  },
  "jira": {
    "type": "sse",
    "url": "https://mcp.jira.com/sse"
  }
}
```

**용도:** 서로 다른 외부 엔드포인트 간 데이터를 매핑하고 조율하는 시나리오에 활용합니다.

## 보안 권장 사항 (Security Best Practices)

### HTTPS/WSS 프로토콜 강제 (Use HTTPS/WSS)

웹 통신 구간 도킹 시에는 반드시 보안 프로토콜을 설정하십시오:

```json
✅ "url": "https://mcp.example.com/sse"
❌ "url": "http://mcp.example.com/sse"
```

### 토큰 관리 지침 (Token Management)

**권장 사항 (DO):**
- ✅ 토큰 바인딩을 위해 쉘 환경 변수 방식을 지향할 것
- ✅ 필요한 환경 변수 목록은 README에 빠짐없이 정리해 기재할 것
- ✅ 가용한 환경에서는 OAuth 위임 관리 방식을 적용할 것

**금지 사항 (DON'T):**
- ❌ 절대 매니페스트 본문에 토큰 문자열 자체를 노출하여 하드코딩하지 말 것
- ❌ 기밀 정보가 담긴 토큰을 소스 보관소(Git 등)에 커밋하지 말 것
- ❌ 공유 목적의 기술 문서 본문에 실제 토큰 예시를 작성하지 말 것

### 허용 권한 범위 제한 (Permission Scoping)

꼭 필요한 도구 식별자만 선별적으로 지명하십시오:

```markdown
✅ allowed-tools: [
  "mcp__plugin_api_server__read_data",
  "mcp__plugin_api_server__create_item"
]

❌ allowed-tools: ["mcp__plugin_api_server__*"]
```

## 에러 처리 (Error Handling)

### 연결 오류 대응 (Connection Failures)

외부 연동 대상 MCP 서버가 일시 단절되는 장애 상황을 고려하십시오:
- 명령어 지침 단계에 호출 실패 시의 대체 동작 가이드를 마련합니다.
- 사용자에게 단순히 에러 코드만 띄우지 않고, 친절하게 접속 장애 사유를 설명합니다.
- 등록된 엔드포인트 도메인 경로 및 포트 유효성을 재점검합니다.

### 도구 호출 실패 대응 (Tool Call Errors)

실제 도구 내부 연산 처리 실패 대응:
- 연동 툴로 인수를 쏘기 전에 매개변수 값 유효성 체크를 철저히 거칩니다.
- 에러 코드별 복구 방법을 사용자 가이드 형식으로 제공합니다.
- 과도한 호출로 인한 속도 제한(rate limits) 및 사용량 한도를 고려합니다.

### 구성 설정 오류 대응 (Configuration Errors)

인프라 구성 수준의 에러 방지:
- 개발 단계에서 기동 연결 테스트를 충분히 진행하십시오.
- JSON 파싱 오류 유무를 꼼꼼히 체크하십시오.
- 필수 환경 변수가 쉘 세션에 바인딩되어 있는지 누락 여부를 확인하십시오.

## 성능 고려 사항 (Performance Considerations)

### 지연 로딩 (Lazy Loading)

MCP 커넥션은 필요한 순간에 비로소 수립됩니다:
- 플러그인이 로드되는 시점에 모든 외부 연결이 맺어지는 것은 아닙니다.
- 실제 최초로 연동 도구 호출을 수행할 때 연결 수립 단계(connection handshake)를 탑니다.
- 세션 연결 풀(connection pool)은 시스템 내에서 효율적으로 자동 유지됩니다.

### 일괄 요청 처리 (Batching)

루프 연산을 돌며 반복 조회를 쏘는 설계를 피하고 일괄 질의 방식을 사용하십시오:

```
# 올바름 (Good): 단일 필터 쿼리로 조회 범위 제어
tasks = search_tasks(project="X", assignee="me", limit=50)

# 피해야 함 (Avoid): 여러 건에 대해 단건 상세 조회를 지속적으로 루프 발송
for id in task_ids:
    task = get_task(id)
```

## MCP 통합 검증 및 테스트 (Testing MCP Integration)

### 로컬 테스트 절차 (Local Testing)

1. 로컬 환경의 임시 `.mcp.json`에 서버 사양을 구성합니다.
2. 개발 중인 플러그인을 `.claude-plugin/` 로컬 설치 디렉토리에 배치합니다.
3. 터미널 세션 기동 후 `/mcp` 명령으로 대상 서버 목록 등록 상태를 검증합니다.
4. 해당 툴을 쏘는 커맨드를 가동해 봅니다.
5. 연결에 실패하거나 호출이 안 되는 원인을 확인하기 위해 `claude --debug` 출력을 모니터링합니다.

### 검증 체크리스트 (Validation Checklist)

- [ ] MCP 설정이 정상 JSON 문법 구조를 준수함
- [ ] 서버 주소가 유효하며 정상 도달 가능함
- [ ] 구동에 필요한 환경 변수 목록이 README 파일에 친절하게 설명됨
- [ ] `/mcp` 출력 목록 상에 대상 도구들이 누락 없이 표시됨
- [ ] OAuth나 토큰 기반 인증 단계 통과 상태가 정상임
- [ ] 명령어 동작 단계를 밟을 때 연동 도구가 부드럽게 호출됨
- [ ] 일시적 연동 실패 상황에 대한 장애 대응 메시지가 준비됨

## 디버깅 (Debugging)

### 디버그 로그 활성화 (Enable Debug Logging)

```bash
claude --debug
```

디버그 스트림 상에서 아래 정보를 눈여겨보십시오:
- MCP 서버 최초 연결 수립 시도 트레이스
- 도구 탐색 및 로딩 기록
- OAuth 팝업 및 핸드셰이크 진행 정보
- 도구 호출 중 발생한 오류 코드 내용

### 일반적인 이슈 (Common Issues)

**서버 연결 실패:**
- 도메인이나 IP 주소 기입 실수가 없는지 체크하십시오.
- 로컬 바이너리 기동(stdio)의 경우 실행 스크립트 경로가 유효한지 체크하십시오.
- 네트워크 방화벽이 차단하고 있지 않은지 망 상태를 점검하십시오.
- 주입한 인증 헤더 포맷이나 인가 수준이 올바른지 점검하십시오.

**도구 미표시/감지 불가:**
- MCP 연결 수립이 문제없이 통과되었는지 사전 확인합니다.
- 대소문자 구분을 포함해 호출하려는 명칭이 완벽하게 일치하는지 체크합니다.
- `/mcp` 목록에 대상 도구들이 올라왔는지 체크합니다.
- 기동 구성 설정 정보를 고친 후 세션을 리셋했는지 체크합니다.

**인증 오류 지속:**
- 기존에 받아둔 인증 토큰 정보가 만료된 것은 아닌지 세션을 비우고 초기화해 봅니다.
- 재승인 단계를 타도록 유도해 봅니다.
- 토큰의 접근 역할 등급이나 인가된 정보의 허용 범위 스코프를 재점검합니다.
- 필수 환경 변수 주입 유무를 재차 확인합니다.

## 빠른 참조 (Quick Reference)

### MCP 서버 유형 (MCP Server Types)

| 유형 | 통신 채널 | 최적 용도 | 주된 인증 수단 |
|------|-----------|----------|------|
| stdio | 로컬 프로세스 | 로컬 유틸리티 실행, 독자 커스텀 로컬 스크립트 | 환경 변수 주입 |
| SSE | HTTP | 호스팅 클라우드 서비스 API 연동 | OAuth 위임 |
| HTTP | REST | 기존 REST 백엔드 엔드포인트 연동 | Bearer 토큰, API Keys |
| ws | WebSocket | 실시간 푸시, 로그 스트리밍 모니터링 | Bearer 토큰 |

### 구성 체크리스트 (Configuration Checklist)

- [ ] 구동할 서버의 통신 유형 지정 (stdio/SSE/HTTP/ws)
- [ ] 필수 매개변수 선언 완료 (stdio면 command, 그 외면 url 경로 등)
- [ ] 필요한 인증 수단 마련
- [ ] 관련 환경 변수의 존재와 목적을 README 파일에 가이드
- [ ] HTTPS/WSS 보안 프로토콜 한정 사용
- [ ] 가급적 절대 경로 기입을 피하고 `${CLAUDE_PLUGIN_ROOT}` 치환 서식을 적용

### 권장 사항 (Best Practices)

**수행할 사항 (DO):**
- ✅ 이식성 보장을 위해 항상 `${CLAUDE_PLUGIN_ROOT}` 서식을 활용할 것
- ✅ 구동에 필수적인 외부 환경 변수를 친절하게 문서화할 것
- ✅ 항상 HTTPS/WSS 기반 보안 링크만 연결할 것
- ✅ 커맨드 파일 내에는 사용할 MCP 도구 식별자 목록을 꼼꼼하게 사전 지정(`allowed-tools`)할 것
- ✅ 릴리스 배포 전에 로컬 디렉토리에서 격리 테스트를 진행해 둘 것
- ✅ 연동 단절에 대처할 장애 대응 예외 처리를 마련해 둘 것

**피해야 할 사항 (DON'T):**
- ❌ 설치 경로 하드코딩
- ❌ 소스 코드 저장소에 API 기밀 토큰 커밋
- ❌ 안전성이 검증되지 않은 일반 HTTP/WS 링크 기재
- ❌ 와일드카드 기호(`*`)를 무분별하게 지정해 다 허용 처리하기
- ❌ 예외 처리 생략하기
- ❌ 설치 가이드를 누락하기

## 추가 리소스 (Additional Resources)

### 참조 파일

세부 연동 지식은 아래 파일을 참조하십시오:

- **`references/server-types.md`** - 개별 구동 서버 타입별 구성 심층 가이드
- **`references/authentication.md`** - OAuth를 포함한 유형별 상세 인증 패턴 가이드
- **`references/tool-usage.md`** - 명령어 및 에이전트 설계 시 도구 호출 방법 가이드

### 예시 구성 파일

`examples/` 내 작동 가능한 템플릿:

- **`stdio-server.json`** - 로컬 stdio 연동 템플릿
- **`sse-server.json`** - SSE 기반 OAuth 연동 템플릿
- **`http-server.json`** - REST API 토큰 헤더 연동 템플릿

### 외부 참조 문서

- **공식 MCP 가이드**: https://modelcontextprotocol.io/
- **Claude Code MCP 가이드**: https://docs.claude.com/en/docs/claude-code/mcp
- **MCP 개발 SDK 라이브러리**: `@modelcontextprotocol/sdk`
- **테스트**: 터미널 디버깅을 위해 `claude --debug` 및 `/mcp` 명령어 조합 활용

## 구현 워크플로우 (Implementation Workflow)

플러그인에 MCP 기능을 결합하는 개발 흐름:

1. 구현할 MCP 서버의 통신 연결 유형을 결정합니다 (stdio, SSE, HTTP, ws).
2. 플러그인 루트 경로에 `.mcp.json` 구성 설정 파일을 만듭니다.
3. 플러그인 내에 파일 경로나 실행 스크립트를 지정해야 하는 경우, 반드시 `${CLAUDE_PLUGIN_ROOT}` 변수를 사용합니다.
4. 구동을 위해 준비해야 할 환경 변수들의 이름과 목적을 README 파일에 문서화합니다.
5. 로컬 샌드박스 환경에서 `/mcp` 명령을 통해 인가된 도구 상태를 검증합니다.
6. 해당 도구를 호출하게 할 커맨드 설정 상에 `allowed-tools` 항목을 설정해 승인합니다.
7. 필요한 인증 절차(OAuth 또는 토큰 헤더 처리)의 정상 동작을 체크합니다.
8. 일시적인 원격 단절 등 장애 발생 시 예외 처리가 정상적으로 이루어지는지 체크합니다.
9. 플러그인 README 가이드에 최종적인 설치 및 작동 방법에 대해 친절하게 서술합니다.

일반적으로 커스텀/로컬 바이너리 연동 시에는 stdio 방식을 선택하고, 원격 기성 클라우드 연동 시에는 OAuth 동의 기능이 지원되는 SSE 방식을 채택하는 것이 가장 효과적입니다.
