# MCP 서버 유형 심층 가이드 (MCP Server Types: Deep Dive)

Claude Code 플러그인에서 지원하는 모든 MCP 서버 유형에 대한 전체 참조 가이드.

## stdio (표준 입력/출력) (stdio (Standard Input/Output))

### 개요 (Overview)

표준 입력 및 출력(stdin/stdout)을 통해 통신하는 로컬 MCP 서버를 하위 프로세스로 실행합니다. 로컬 도구 개발, 맞춤형 서버 구동, NPM 패키지 구동 시 가장 적합한 선택입니다.

### 구성 (Configuration)

**기본 구성:**
```json
{
  "my-server": {
    "command": "npx",
    "args": ["-y", "my-mcp-server"]
  }
}
```

**환경 변수가 추가된 구성:**
```json
{
  "my-server": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/custom-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "API_KEY": "${MY_API_KEY}",
      "LOG_LEVEL": "debug",
      "DATABASE_URL": "${DB_URL}"
    }
  }
}
```

### 프로세스 수명 주기 (Process Lifecycle)

1. **시작**: Claude Code가 지정된 `command` 및 `args`를 기반으로 프로세스를 생성(spawn)합니다.
2. **통신**: stdin/stdout 채널을 통해 JSON-RPC 메시지를 교환합니다.
3. **수명 주기**: 프로세스는 전체 Claude Code 세션 동안 백그라운드에서 유지됩니다.
4. **종료**: Claude Code가 종료될 때 하위 프로세스도 함께 소멸됩니다.

### 사용 사례 (Use Cases)

**NPM 패키지 가동:**
```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
  }
}
```

**맞춤형 스크립트 실행:**
```json
{
  "custom": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-server.js",
    "args": ["--verbose"]
  }
}
```

**Python 서버 실행:**
```json
{
  "python-server": {
    "command": "python",
    "args": ["-m", "my_mcp_server"],
    "env": {
      "PYTHONUNBUFFERED": "1"
    }
  }
}
```

### 권장 사항 (Best Practices)

1. **절대 경로 또는 ${CLAUDE_PLUGIN_ROOT}를 사용하십시오.**
2. **Python 서버를 띄울 때는 PYTHONUNBUFFERED 환경 변수를 설정하십시오.**
3. **구성 정보는 stdin 대신 args나 env를 통해 전달하십시오.**
4. **서버 크래시 상황에 대처할 수 있도록 에러 처리를 우아하게 설계하십시오.**
5. **로그는 stdout 대신 반드시 stderr로 출력하십시오. (stdout은 MCP 프로토콜 통신용으로 사용됨)**

### 문제 해결 (Troubleshooting)

**서버 실행 실패:**
- 실행 명령어가 실제로 시스템에 존재하며 실행 권한이 있는지 확인합니다.
- 참조된 파일 경로의 정확성을 확인합니다.
- 권한(permissions) 구성을 확인합니다.
- `claude --debug` 로그를 확인합니다.

**통신 실패:**
- 서버가 stdin/stdout을 올바르게 활용하고 있는지 확인합니다.
- 디버그용 console.log나 임의의 print문 출력이 표준 출력을 오염시키고 있지 않은지 확인합니다.
- JSON-RPC 규격을 준수하고 있는지 검증합니다.

## SSE (Server-Sent Events)

### 개요 (Overview)

스트리밍 지원을 위해 서버 전송 이벤트(SSE) 기술을 활용하여 웹에 호스팅된 원격 MCP 서버에 HTTP로 연결합니다. 클라우드 서비스 연동 및 OAuth 인증 흐름 구성에 가장 적합합니다.

### 구성 (Configuration)

**기본 구성:**
```json
{
  "hosted-service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse"
  }
}
```

**맞춤형 헤더 추가 구성:**
```json
{
  "service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "X-API-Version": "v1",
      "X-Client-ID": "${CLIENT_ID}"
    }
  }
}
```

### 연결 수명 주기 (Connection Lifecycle)

1. **초기화**: 대상 URL 경로로 HTTP 연결이 생성됩니다.
2. **핸드셰이크**: MCP 프로토콜 협상이 일어납니다.
3. **스트리밍**: 서버가 SSE 채널을 통해 이벤트를 실시간 전달합니다.
4. **요청**: 클라이언트가 도구를 호출하기 위해 HTTP POST 요청을 보냅니다.
5. **재연결**: 연결 유실 시 자동으로 재연결을 시도합니다.

### 인증 (Authentication)

**OAuth (자동 처리):**
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

Claude Code가 OAuth 흐름을 완전히 대행합니다:
1. 최초 사용 시 사용자에게 인증 동의 창을 표시합니다.
2. 브라우저를 열어 OAuth 동의 흐름을 수행합니다.
3. 발급된 토큰을 보안 영역에 안전하게 보관합니다.
4. 백그라운드에서 주기적으로 토큰을 자동 갱신합니다.

**맞춤형 헤더 전달:**
```json
{
  "service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

### 사용 사례 (Use Cases)

**공식 클라우드 서비스:**
- Asana: `https://mcp.asana.com/sse`
- GitHub: `https://mcp.github.com/sse`
- 기타 외부 호스팅된 MCP 서버

**맞춤형 호스팅 서버:**
보유한 사내 MCP 서버를 HTTPS + SSE 환경으로 클라우드에 배포하여 플러그인과 연동합니다.

### 권장 사항 (Best Practices)

1. **보안을 위해 항상 HTTP 대신 HTTPS 프로토콜을 사용하십시오.**
2. **서버가 OAuth를 지원하는 경우 가급적 해당 흐름을 타도록 위임하십시오.**
3. **토큰 지정 시에는 항상 환경 변수 참조 서식을 활용하십시오.**
4. **일시적인 원격 연결 장애 상황에 우아하게 대응하도록 설계하십시오.**
5. **요구 스코프(OAuth Scopes) 정보를 README에 상세히 문서화하십시오.**

### 문제 해결 (Troubleshooting)

**연결 거부 (Connection refused):**
- URL 경로가 정확하며 도달 가능한 상태인지 확인합니다.
- HTTPS SSL 보안 인증서가 유효한지 확인합니다.
- 네트워크 방화벽 및 접근 제어 설정을 확인합니다.

**OAuth 실패:**
- 캐시된 기존 보안 토큰을 지웁니다.
- OAuth 스코프 구성을 다시 검사합니다.
- 리다이렉트 URL 구성을 검증합니다.
- 다시 한번 인증을 진행합니다.

## HTTP (REST API)

### 개요 (Overview)

표준 HTTP 요청을 통해 RESTful 설계 원칙을 준수하는 MCP 서버에 연결합니다. 토큰 기반 인증 및 무상태(stateless) 단순 요청/응답 패턴에 가장 어울리는 방식입니다.

### 구성 (Configuration)

**기본 구성:**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp"
  }
}
```

**인증 정보가 추가된 구성:**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "Content-Type": "application/json",
      "X-API-Version": "2024-01-01"
    }
  }
}
```

### 요청/응답 흐름 (Request/Response Flow)

1. **도구 탐색**: GET 요청을 발송하여 가용한 도구들의 사양을 확인합니다.
2. **도구 호출**: 도구 이름 및 매개변수를 담은 POST 요청을 보냅니다.
3. **응답**: 호출 결과 또는 오류 정보를 담은 JSON 데이터가 반환됩니다.
4. **무상태**: 각 요청이 서로 독립적인 세션 컨텍스트를 가집니다.

### 인증 (Authentication)

**토큰 기반 인증:**
```json
{
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

**API 키 인증:**
```json
{
  "headers": {
    "X-API-Key": "${API_KEY}"
  }
}
```

**기타 사용자 정의 인증:**
```json
{
  "headers": {
    "X-Auth-Token": "${AUTH_TOKEN}",
    "X-User-ID": "${USER_ID}"
  }
}
```

### 사용 사례 (Use Cases)

- REST API 백엔드 서비스 연동
- 사내 인트라넷 마이크로서비스 연동
- 서버리스(Serverless) 클라우드 함수 연동

### 권장 사항 (Best Practices)

1. **반드시 HTTPS 프로토콜을 사용하십시오.**
2. **인증 토큰은 환경 변수 바인딩 서식을 활용해 주입하십시오.**
3. **네트워크 일시 지연에 대처하기 위해 재시도(retry) 로직을 준비하십시오.**
4. **API 호출 속도 제한(Rate Limiting)을 준수하십시오.**
5. **클라이언트 및 서버 측에 적절한 제한 시간(timeout)을 설정하십시오.**

### 문제 해결 (Troubleshooting)

**HTTP 에러 코드:**
- 401: 인증 헤더 구성을 확인하십시오.
- 403: 호출 권한 및 역할을 검증하십시오.
- 429: API 호출 빈도를 낮추고 백오프(backoff) 알고리즘을 고려하십시오.
- 500: 연동 서버 측 내부 예외 로그를 확인하십시오.

**제한 시간 만료 (Timeout):**
- 필요한 경우 timeout 값을 늘려 잡으십시오.
- 연동 API 서버의 리소스 처리 성능을 모니터링하십시오.
- 도구 비즈니스 로직을 효율화하여 처리 소요 시간을 줄이십시오.

## WebSocket (실시간) (WebSocket (Real-time))

### 개요 (Overview)

실시간 양방향 통신을 위해 WebSocket 프로토콜로 MCP 서버에 연결합니다. 스트리밍 처리가 빈번하고 지연 시간(latency)을 낮춰야 하는 시스템에 적합합니다.

### 구성 (Configuration)

**기본 구성:**
```json
{
  "realtime": {
    "type": "ws",
    "url": "wss://mcp.example.com/ws"
  }
}
```

**인증 정보가 추가된 구성:**
```json
{
  "realtime": {
    "type": "ws",
    "url": "wss://mcp.example.com/ws",
    "headers": {
      "Authorization": "Bearer ${TOKEN}",
      "X-Client-ID": "${CLIENT_ID}"
    }
  }
}
```

### 연결 수명 주기 (Connection Lifecycle)

1. **핸드셰이크**: WebSocket 프로토콜 업그레이드 요청을 전송합니다.
2. **연결 수립**: 지속적이고 유지되는 양방향 통신 채널이 생성됩니다.
3. **데이터 교환**: 소켓 위에서 실시간 JSON-RPC 프레임이 송수신됩니다.
4. **하트비트**: 활성 상태를 모니터링하기 위한 ping-pong 프레임이 오고 갑니다.
5. **재연결**: 일시적 단절 발생 시 클라이언트가 자동으로 세션을 다시 맺습니다.

### 사용 사례 (Use Cases)

- 실시간 대용량 로그/출력 스트리밍
- 라이브 상태 업데이트 알림 서비스
- 지연 시간에 극도로 민감한 도구 호출
- 서버가 주도하는 클라이언트 대상 푸시 알림

### Best Practices

1. **보안이 통제되지 않은 일반 WS 대신 반드시 WSS(Secure WebSocket)를 채택하십시오.**
2. **좀비 커넥션을 예방하기 위해 하트비트(ping-pong)를 구현하십시오.**
3. **일시적 단절에 복구할 수 있는 안정적인 재연결 알고리즘을 준비하십시오.**
4. **연결 단절 시간 동안 전송 시도된 데이터를 메모리에 버퍼링했다가 순차적으로 전송하십시오.**
5. **커넥션 맺기 실패 시의 타임아웃 처리를 명확히 하십시오.**

### 문제 해결 (Troubleshooting)

**연결 유실 현상:**
- 재연결(reconnection) 핸들러의 유효성을 검사합니다.
- 로컬 및 중간 게이트웨이의 네트워크 상태를 확인합니다.
- 대상 웹 서버가 웹소켓 업그레이드를 정상 승인하는지 확인합니다.
- 방화벽이나 프록시 서버 설정에서 웹소켓 프로토콜 차단 여부를 검사합니다.

**메시지 유실 및 지연:**
- 메시지 수신 확인(acknowledgment) 설계를 검토합니다.
- 메시지 전송 순서 보장 핸들러를 검증합니다.
- 단절 상황에서 데이터를 유실하지 않도록 안정적인 로컬 버퍼 체계를 구성합니다.

## 유형별 비교 매트릭스 (Comparison Matrix)

| 구분 | stdio | SSE | HTTP | WebSocket |
|---------|-------|-----|------|-----------|
| **전송 방식** | 로컬 프로세스 파이프 | HTTP/SSE | HTTP | WebSocket |
| **통신 방향** | 양방향 | 서버→클라이언트 | 요청/응답 | 양방향 |
| **세션 상태** | 상태 보존 (Stateful) | 상태 보존 (Stateful) | 무상태 (Stateless) | 상태 보존 (Stateful) |
| **인증 방식** | 로컬 환경 변수 | OAuth/HTTP 헤더 | HTTP 헤더 | HTTP 헤더 |
| **주요 용도** | 로컬 유틸리티 도구 | 호스팅 클라우드 서비스 | REST API 서비스 | 실시간 스트리밍 |
| **지연 시간** | 가장 낮음 (Lowest) | 중간 (Medium) | 중간 (Medium) | 낮음 (Low) |
| **설정 난이도**| 매우 쉬움 | 보통 | 쉬움 | 보통 |
| **재연결 제어**| 하위 프로세스 재기동 | 자동 재연결 | 해당 없음 | 자동 재연결 |

## 적합한 유형 선택하기 (Choosing the Right Type)

**다음과 같은 경우 stdio 유형을 권장합니다:**
- 로컬 실행 바이너리 및 custom 서버를 띄울 때
- 지연 시간을 극도로 낮춰야 할 때
- 로컬 파일 시스템이나 사내 DB에 다이렉트로 접근해야 할 때
- 플러그인 빌드 패키지에 자체 실행 서버를 함께 패킹하여 동시 배포하고자 할 때

**다음과 같은 경우 SSE 유형을 권장합니다:**
- 외부 클라우드 서비스에 간접 연결할 때
- OAuth 위임 인증 기능이 유용할 때
- 공식적으로 공급되는 클라우드 연동 MCP 서버(Asana, GitHub)를 호출할 때
- 단절 시 즉각적인 자동 재연결이 유용할 때

**다음과 같은 경우 HTTP 유형을 권장합니다:**
- 일반적인 웹 기반 REST API와 직접 매핑할 때
- 복잡한 세션 유지 없이 단순 요청/응답 형식의 무상태 통신이 충분할 때
- API 토큰 기반 단순 헤더 인증으로 충분할 때

**다음과 같은 경우 WebSocket 유형을 권장합니다:**
- 실시간으로 갱신되는 대화식 스트리밍을 모니터링해야 할 때
- 양방향 푸시 모델을 활용해야 할 때
- HTTP 연결 대비 오버헤드를 줄여 저지연 처리를 지속해야 할 때

## 유형 간 마이그레이션 (Migration Between Types)

### stdio에서 SSE로 전환 (From stdio to SSE)

**이전 (로컬 프로세스 stdio 구동 방식):**
```json
{
  "local-server": {
    "command": "node",
    "args": ["server.js"]
  }
}
```

**이후 (클라우드 배포 후 SSE 연동 방식):**
```json
{
  "hosted-server": {
    "type": "sse",
    "url": "https://mcp.example.com/sse"
  }
}
```

### HTTP에서 WebSocket으로 전환 (From HTTP to WebSocket)

**이전 (REST API HTTP 폴링/요청 방식):**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp"
  }
}
```

**이후 (웹소켓 세션 유지 실시간 처리 방식):**
```json
{
  "realtime": {
    "type": "ws",
    "url": "wss://api.example.com/ws"
  }
}
```

이점: 실시간 정보 교환 활성화, 저지연 처리 지원, 양방향 푸시 설계 가능.

## 고급 구성 (Advanced Configuration)

### 다중 서버 구성 (Multiple Servers)

필요에 따라 복수의 다른 타입 서버를 동시 가동할 수 있습니다:

```json
{
  "local-db": {
    "command": "npx",
    "args": ["-y", "mcp-server-sqlite", "./data.db"]
  },
  "cloud-api": {
    "type": "sse",
    "url": "https://mcp.example.com/sse"
  },
  "internal-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

### 조건부 구성 (Conditional Configuration)

환경 변수를 주입하여 실행 대상 서버를 유연하게 바꿀 수 있습니다:

```json
{
  "api": {
    "type": "http",
    "url": "${API_URL}",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

각 환경별로 다른 변수 값을 할당하여 사용합니다:
- 개발 환경(Dev): `API_URL=http://localhost:8080/mcp`
- 운영 환경(Prod): `API_URL=https://api.production.com/mcp`

## 보안 고려 사항 (Security Considerations)

### Stdio 보안 (Stdio Security)

- 런타임에 주입되는 명령어의 경로를 엄격히 한정하십시오.
- 사용자 인터페이스로부터 온 임의의 명령줄 문자열을 검증 없이 그대로 기동 명령으로 전달하지 마십시오.
- 하위 자식 프로세스가 상위 쉘의 권한이나 무제한 환경 변수에 접근할 수 없도록 격리 강도를 높이십시오.
- 로컬 파일 시스템 내에서 접근할 수 있는 디렉토리를 최소 권한(sandbox) 수준으로 차단하십시오.

### 네트워크 보안 (Network Security)

- 실서비스 운영 환경에서는 절대로 일반 HTTP/WS를 기재하지 마십시오. (반드시 HTTPS/WSS 필수)
- 호스팅 서버의 유효한 SSL 인증서 상태를 주기적으로 체크하십시오.
- 편리함을 이유로 클라이언트의 SSL 인증서 신뢰성 검증 단계를 건너뛰지 마십시오.
- 클라이언트 측의 접근 토큰을 일반 텍스트 로그나 노출되기 쉬운 저장소에 보관하지 마십시오.

### 토큰 관리 (Token Management)

- 매니페스트 설정 JSON 파일 본문에 절대로 토큰 원본을 하드코딩하여 포함하지 마십시오.
- 시스템 환경 변수(`${API_TOKEN}`)를 경유하도록 조율하십시오.
- 만료 기간이 있는 토큰의 주기적인 교체(rotation) 일정을 관리하십시오.
- 가급적 동적인 OAuth 토큰 자동 갱신 흐름을 이용하십시오.
- 필요한 최소 수준의 접근 스코프(Scope)만 인가 받도록 소유 권한을 조율하십시오.

## 결론 (Conclusion)

요구 사항에 따라 아래의 연동 규격을 선택하십시오:
- **stdio**: 로컬 전용 툴링 가동용, 자사 특화 로컬 스크립트 실행용, NPM 패키지 가동용
- **SSE**: 클라우드에 호스팅된 공식 서비스 연동용 (OAuth 위임 연동 지원)
- **HTTP**: 전통적인 무상태 REST API 연동용
- **WebSocket**: 양방향 저지연 스트리밍 처리가 요구되는 연동 환경용

풍부하고 결함 없는 MCP 아키텍처 구축을 위해 사전에 충분한 예외 시나리오 테스트를 실행해 두십시오.
