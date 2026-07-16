# MCP 서버 인증 (Auth for MCP Servers)

인증은 로컬 서버가 더 간단함에도 불구하고 많은 개발자들이 결국 **원격(remote)** 서버 구성을 선택하게 되는 핵심적인 이유입니다. 인증 완료 후 리다이렉트해 올 실제 호스팅 엔드포인트가 존재할 때, OAuth 리다이렉션, 토큰 저장 및 갱신(refresh) 과정이 가장 원활하게 작동합니다.

## Claude 특화 인증 방식

Claude의 MCP 클라이언트는 특정 인증 유형 세트만을 지원하며, 스펙 상 호환되는 모든 흐름이 다 작동하는 것은 아닙니다. 상세 가이드라인: https://claude.com/docs/connectors/building/authentication

| 유형 | 참고 사항 |
|---|---|
| `oauth_dcr` | 지원됨. 디렉토리에 대규모로 등록하는 경우 CIMD나 Anthropic이 관리하는 크리덴셜 방식을 선호하십시오 — DCR은 매 연결 시마다 새로운 클라이언트를 동적으로 등록합니다. |
| `oauth_cimd` | 지원됨. 디렉토리 등록용 서버라면 DCR보다 이 방식을 강력히 권장합니다. |
| `oauth_anthropic_creds` | 파트너사가 Anthropic에 `client_id`/`client_secret`을 제공하고, 사용자의 동의 절차를 거치는 방식. 문의: `mcp-review@anthropic.com` |
| `custom_connection` | Snowflake 방식처럼 연결 시점에 사용자가 직접 URL 및 패스워드를 기재하는 방식. 문의: `mcp-review@anthropic.com` |
| `none` | 인증 없음. |

**지원하지 않는 방식:** 사용자가 직접 복사해서 붙여넣는 베어러 토큰(`static_bearer`), 사용자 동의 절차 없는 순수 기계 대 기계(machine-to-machine) 간의 `client_credentials` 발급 방식.

**리다이렉트(Callback) URL** (모든 플랫폼 공통): `https://claude.ai/api/mcp/auth_callback`

---

## 3가지 인증 방식 수준 (The three tiers)

### 수준 1: 인증 없음 / 고정형 API 키 방식

서버가 환경 변수에서 키 값을 읽습니다. 사용자는 최초 설정 시점에 한 번만 입력합니다. 끝.

```typescript
const apiKey = process.env.UPSTREAM_API_KEY;
if (!apiKey) throw new Error("UPSTREAM_API_KEY not set");
```

로컬 stdio, MCPB, 원격 서버 모두에 적용 가능합니다. 단순 조회 등만 필요한 상황이라면 이 단계에서 멈추셔도 됩니다.

### 수준 2: CIMD를 통한 OAuth 2.0 (스펙 2025-11-25 권장안)

**클라이언트 ID 메타데이터 문서(Client ID Metadata Document).** MCP 호스트는 자신의 클라이언트 메타데이터를 HTTPS URL 경로에 노출하고, 이 URL 자체를 `client_id`로 사용합니다. 여러분의 인증 서버는 이 URL에서 문서를 읽어 검증한 뒤 인증 코드(auth-code) 흐름을 진행합니다. 별도의 클라이언트 등록 엔드포인트를 열거나 클라이언트 등록 레코드를 보관할 필요가 없습니다.

2025-11-25 스펙 기준, CIMD는 SHOULD(권장) 수준으로 승격되었습니다. 여러분의 OAuth 인증 서버(AS) 메타데이터에 `client_id_metadata_document_supported: true` 속성을 명시하여 지원 여부를 알리십시오.

**서버 구현 요구사항:**

1. `/.well-known/oauth-authorization-server` 경로에서 `client_id_metadata_document_supported: true` 속성을 포함한 OAuth Authorization Server Metadata (RFC 8414) 제공
2. 위의 (1)을 가리키는 MCP 보호 리소스 메타데이터 문서 제공
3. 인증 요청 수신 시: `client_id`로 전달된 HTTPS URL에 접근해 클라이언트 메타데이터를 가져오고 유효성 검증을 거친 뒤 인증 진행
4. 들어오는 `/mcp` API 요청 헤더의 베어러(bearer) 토큰 검증

```
┌─────────┐  client_id=https://...  ┌──────────────┐   upstream OAuth   ┌──────────┐
│ MCP host│ ──────────────────────> │ Your MCP srv │ ─────────────────> │ Upstream │
│ (호스트) │ <─── bearer token ───── │  (MCP 서버)  │ <── access token ──│ (인증처) │
└─────────┘                         └──────────────┘                    └──────────┘
```

### 수준 3: 동적 클라이언트 등록(DCR)을 통한 OAuth 2.0

**하위 호환성 유지용 대체 수단** — 2025-11-25 스펙 기준, DCR은 MAY(선택) 수준으로 강등되었습니다. 호스트가 여러분이 정의한 `registration_endpoint`를 찾아 메타데이터를 POST하여 자신을 클라이언트로 등록하면, 서버는 생성된 `client_id`를 반환하고 이후 인증 코드 흐름을 밟습니다.

CIMD 방식으로 전환하지 않은 호스트 환경들을 지원해야 하는 경우 DCR을 구현하십시오. 기본적인 서버 역할은 CIMD와 유사하지만, `client_id` URL 정보를 즉석에서 읽는 대신 클라이언트 등록 내역을 보관할 등록 API 엔드포인트를 직접 구현 및 운영해야 하는 번거로움이 있습니다.

**클라이언트 우선순위 판단 순서:** 이미 수동 등록된 키 정보 → CIMD (AS가 지원함을 표명했을 경우) → DCR (AS에 등록 엔드포인트가 있을 경우) → 사용자에게 입력 요청.

---

## 자체 DCR/CIMD 연동을 지원하는 호스트 서비스

여러 MCP 전문 호스팅 플랫폼들은 이러한 복잡한 OAuth 인프라 구성을 대신 처리해 줍니다 — 개발자는 비즈니스 로직(도구)만 구현하고, 인증 서버 운영은 플랫폼이 대신 맡습니다. 플랫폼별 공식 문서를 참고하십시오. 특별히 선호하는 인프라 제약이 없는 사용자라면, 이러한 플랫폼들을 이용하는 것이 OAuth 인증이 적용된 서버를 가장 빠르게 확보하는 지름길입니다.

---

## 로컬 서버 환경에서의 OAuth 적용

로컬 stdio 방식의 서버도 OAuth 구현은 **가능합니다** (기본 웹 브라우저를 띄우고 localhost 포트로 리다이렉트를 잡아 토큰을 가로챈 뒤 OS 키체인에 저장하는 방식). 그러나 이 방식은 다음과 같이 취약합니다:

- 헤드리스(headless) 서버나 원격 개발 환경에서는 오작동합니다.
- 새로운 사용자가 유입될 때마다 매번 이 복잡한 인증 단계를 밟아야 합니다.
- 토큰을 일괄적으로 갱신하거나 강제 만료(revocation)시킬 수 있는 중앙 관리 수단이 없습니다.

따라서 OAuth가 반드시 필요한 환경이라면, 웬만하면 원격 HTTP 서버 구성을 고려하십시오. 그럼에도 꼭 로컬 실행 구조와 OAuth를 조합해야 한다면, `@modelcontextprotocol/sdk` 내의 localhost 리다이렉트 헬퍼를 활용하고 MCPB 패키징을 적용해 최소한 실행 환경의 예측 가능성이라도 확보하는 것이 좋습니다.

---

## 토큰 저장소 (Token storage)

| 배포 형태 | 권장 토큰 저장소 |
|---|---|
| 원격 무상태(stateless) 서버 | 별도 저장 안 함 — 호스트가 요청 시마다 베어러 토큰을 실어 보냄 |
| 원격 유상태(stateful) 서버 | MCP 세션 ID를 키로 하는 세션 저장소 (Redis 등) |
| MCPB / 로컬 실행 환경 | OS 보안 키체인 사용 (Node의 `keytar`, Python의 `keyring`). **절대 디스크에 평문 텍스트로 보관하지 말 것.** |

---

## 토큰 대상자 검증 (Token audience validation - 스펙 MUST 사항)

단순히 "이 토큰이 올바른 서명 형식을 가졌는가"만 확인하는 것으로는 불충분합니다. MCP 규격은 "이 토큰이 *오직 본 MCP 서버를 위해* 발행된 것인가"를 반드시 검증하도록 강제합니다 (RFC 8707 audience 사양). 비록 서명이 정상이라도 `api.other-service.com` 용으로 발급된 토큰이라면 이 서버에서 즉시 거부(reject) 처리해야 합니다.

**토큰 패스스루(Token passthrough)는 엄격히 금지됩니다.** 호스트가 전달한 토큰을 그대로 타 서비스 API 호출 헤더에 얹어서 포워딩하지 마십시오. 서버가 다른 외부 서비스를 호출해야 하는 경우, 전달받은 토큰을 서비스 교환 절차를 통해 바꾸거나 서버 자체에 등록된 크리덴셜 키를 별도로 사용해야 합니다.

---

## SDK 제공 헬퍼 도구 사용 — 직접 로직을 만들지 마십시오

`@modelcontextprotocol/sdk/server/auth` 패키지에서 다음 헬퍼들을 지원합니다:
- `mcpAuthRouter()` — OAuth 인증 서버가 제공해야 할 표준 API 엔드포인트 세트(metadata, authorize, token)를 지원하는 Express 라우터
- `bearerAuth` — 지정한 검증 조건에 부합하는 베어러 토큰인지를 확인하는 미들웨어
- `proxyProvider` — 외부 인증 제공자(IdP)로 인증을 대리 발송(forward)해 주는 도구

직접 인증 파이프라인을 작성하기 전에 이 기능들을 먼저 확인하십시오.
