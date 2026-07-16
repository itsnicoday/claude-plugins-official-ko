# MCP 인증 패턴 (MCP Authentication Patterns)

Claude Code 플러그인의 MCP 서버용 인증 방식 전체 가이드.

## 개요 (Overview)

MCP 서버는 서버 유형 및 서비스 요구 사항에 따라 여러 인증 방식을 지원합니다. 사용 사례와 보안 요구 사항에 가장 잘 맞는 방식을 선택하십시오.

## OAuth (자동 처리) (OAuth (Automatic))

### 작동 방식 (How It Works)

Claude Code는 SSE 및 HTTP 서버에 대한 완전한 OAuth 2.0 흐름을 자동으로 처리합니다:

1. 사용자가 MCP 도구 사용을 시도합니다.
2. Claude Code가 인증이 필요함을 감지합니다.
3. OAuth 동의를 위한 브라우저를 엽니다.
4. 사용자가 브라우저에서 승인합니다.
5. 토큰이 Claude Code에 의해 안전하게 저장됩니다.
6. 자동으로 토큰을 갱신(refresh)합니다.

### 구성 (Configuration)

```json
{
  "service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse"
  }
}
```

추가적인 인증 구성이 필요하지 않습니다! Claude Code가 모든 것을 처리합니다.

### 지원되는 서비스 (Supported Services)

**OAuth가 활성화된 것으로 알려진 MCP 서버:**
- Asana: `https://mcp.asana.com/sse`
- GitHub (가용 시)
- Google 서비스 (가용 시)
- 맞춤형 OAuth 서버

### OAuth 스코프 (OAuth Scopes)

OAuth 스코프는 MCP 서버에 의해 결정됩니다. 사용자는 동의 흐름을 거치는 동안 필요한 스코프를 확인할 수 있습니다.

**README에 필요한 스코프를 문서화하십시오:**
```markdown
## Authentication

This plugin requires the following Asana permissions:
- Read tasks and projects
- Create and update tasks
- Access workspace data
```

### 토큰 저장 (Token Storage)

토큰은 Claude Code에 의해 안전하게 보관됩니다:
- 플러그인이 직접 접근할 수 없음
- 미사용 시 암호화되어 저장됨
- 자동으로 토큰이 갱신됨
- 로그아웃 시 삭제됨

### OAuth 문제 해결 (Troubleshooting OAuth)

**인증 루프 현상:**
- 캐시된 토큰을 지웁니다 (로그아웃 후 다시 로그인).
- OAuth 리다이렉트 URL을 확인합니다.
- 서버의 OAuth 구성을 검증합니다.

**스코프 관련 문제:**
- 사용자가 새로운 스코프에 대해 재인증을 해야 할 수 있습니다.
- 필요한 스코프에 대해 서버 문서를 확인하십시오.

**토큰 만료:**
- Claude Code가 자동으로 갱신합니다.
- 갱신에 실패하면 재인증을 요청합니다.

## 토큰 기반 인증 (Token-Based Authentication)

### Bearer 토큰 (Bearer Tokens)

HTTP 및 WebSocket 서버에서 가장 흔하게 쓰입니다.

**구성:**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

**환경 변수 설정:**
```bash
export API_TOKEN="your-secret-token-here"
```

### API 키 (API Keys)

Bearer 토큰 대신 주로 전용 맞춤형 헤더에 실어 전송합니다.

**구성:**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "X-API-Key": "${API_KEY}",
      "X-API-Secret": "${API_SECRET}"
    }
  }
}
```

### 맞춤형 헤더 (Custom Headers)

서비스에 따라 고유한 인증용 맞춤형 헤더를 요구할 수 있습니다.

**구성:**
```json
{
  "service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "X-Auth-Token": "${AUTH_TOKEN}",
      "X-User-ID": "${USER_ID}",
      "X-Tenant-ID": "${TENANT_ID}"
    }
  }
}
```

### 토큰 요구 사항 문서화 (Documenting Token Requirements)

항상 README 파일에 이 정보를 명시하십시오:

```markdown
## Setup

### Required Environment Variables

Set these environment variables before using the plugin:

```bash
export API_TOKEN="your-token-here"
export API_SECRET="your-secret-here"
```

### Obtaining Tokens

1. Visit https://api.example.com/tokens
2. Create a new API token
3. Copy the token and secret
4. Set environment variables as shown above

### Token Permissions

The API token needs the following permissions:
- Read access to resources
- Write access for creating items
- Delete access (optional, for cleanup operations)
```

## 환경 변수 인증 (stdio 방식) (Environment Variable Authentication (stdio))

### 서버에 크리덴셜 전달 (Passing Credentials to Server)

stdio 서버의 경우, 환경 변수를 통해 서버 프로세스에 직접 신원 정보를 주입합니다:

```json
{
  "database": {
    "command": "python",
    "args": ["-m", "mcp_server_db"],
    "env": {
      "DATABASE_URL": "${DATABASE_URL}",
      "DB_USER": "${DB_USER}",
      "DB_PASSWORD": "${DB_PASSWORD}"
    }
  }
}
```

### 사용자 환경 변수 (User Environment Variables)

```bash
# User sets these in their shell
export DATABASE_URL="postgresql://localhost/mydb"
export DB_USER="myuser"
export DB_PASSWORD="mypassword"
```

### 문서화 템플릿 (Documentation Template)

```markdown
## Database Configuration

Set these environment variables:

```bash
export DATABASE_URL="postgresql://host:port/database"
export DB_USER="username"
export DB_PASSWORD="password"
```

Or create a `.env` file (add to `.gitignore`):

```
DATABASE_URL=postgresql://localhost:5432/mydb
DB_USER=myuser
DB_PASSWORD=mypassword
```

Load with: `source .env` or `export $(cat .env | xargs)`
```

## 동적 헤더 (Dynamic Headers)

### 헤더 헬퍼 스크립트 (Headers Helper Script)

주기적으로 갱신되거나 만료 기간이 있는 토큰의 경우, 외부 헬퍼 스크립트를 연동하여 헤더를 주입합니다:

```json
{
  "api": {
    "type": "sse",
    "url": "https://api.example.com",
    "headersHelper": "${CLAUDE_PLUGIN_ROOT}/scripts/get-headers.sh"
  }
}
```

**스크립트 (get-headers.sh):**
```bash
#!/bin/bash
# Generate dynamic authentication headers

# Fetch fresh token
TOKEN=$(get-fresh-token-from-somewhere)

# Output JSON headers
cat <<EOF
{
  "Authorization": "Bearer $TOKEN",
  "X-Timestamp": "$(date -Iseconds)"
}
EOF
```

### 동적 헤더 사용 사례 (Use Cases for Dynamic Headers)

- 갱신이 빈번하게 일어나는 단기 토큰
- HMAC 서명이 포함된 서명 검증 토큰
- 타임스탬프 기반 인증 방식
- 동적인 테넌트/워크스페이스 선택이 필요할 때

## 보안 권장 사항 (Security Best Practices)

### 권장 사항 (DO)

✅ **환경 변수 사용:**
```json
{
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

✅ **README에 필수 변수 문서화**

✅ **항상 HTTPS/WSS 사용**

✅ **토큰 주기적 갱신(rotation) 구현**

✅ **파일이 아닌 임시 환경 변수에 토큰 안전하게 저장**

✅ **지원되는 경우 가급적 OAuth 연동**

### 금지 사항 (DON'T)

❌ **토큰 하드코딩:**
```json
{
  "headers": {
    "Authorization": "Bearer sk-abc123..."  // NEVER!
  }
}
```

❌ **Git 저장소에 토큰 노출**

❌ **공개 문서나 가이드에 실제 토큰 기재**

❌ **암호화되지 않은 HTTP 프로토콜 사용**

❌ **플러그인 파일 자체에 물리적으로 토큰 저장**

❌ **보안 비밀이나 토큰 헤더 값을 로그 파일에 기록**

## 멀티 테넌시 패턴 (Multi-Tenancy Patterns)

### 워크스페이스/테넌트 선택 (Workspace/Tenant Selection)

**환경 변수 지정 방식:**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "X-Workspace-ID": "${WORKSPACE_ID}"
    }
  }
}
```

**URL 경로 지정 방식:**
```json
{
  "api": {
    "type": "http",
    "url": "https://${TENANT_ID}.api.example.com/mcp"
  }
}
```

### 사용자별 구성 (Per-User Configuration)

사용자가 자신의 워크스페이스를 환경 변수로 개별 설정합니다:

```bash
export WORKSPACE_ID="my-workspace-123"
export TENANT_ID="my-company"
```

## 인증 관련 문제 해결 (Authentication Troubleshooting)

### 일반적인 이슈 (Common Issues)

**401 Unauthorized:**
- 토큰이 올바르게 설정되었는지 확인
- 토큰이 만료되지 않았는지 확인
- 토큰이 필요한 권한을 가졌는지 확인
- 헤더 서식 형식이 올바른지 확인

**403 Forbidden:**
- 토큰은 유효하나 해당 연산에 대한 권한이 없음
- 스코프/권한을 확인
- 워크스페이스/테넌트 ID를 확인
- 관리자의 승인이 필요할 수 있음

**토큰 검색 실패:**
```bash
# Check environment variable is set
echo $API_TOKEN

# If empty, set it
export API_TOKEN="your-token"
```

**잘못된 토큰 서식:**
```json
// 올바름 (Correct)
"Authorization": "Bearer sk-abc123"

// 잘못됨 (Wrong)
"Authorization": "sk-abc123"
```

### 인증 과정 디버깅 (Debugging Authentication)

**디버그 모드 활성화:**
```bash
claude --debug
```

디버그 로그 중에서 다음 사항을 찾아보십시오:
- 인증 헤더 값 (민감한 데이터는 숨김 처리됨)
- OAuth 인증 절차 진행 로그
- 토큰 갱신 시도 로그
- 발생한 인증 에러 메시지

**격리 테스트 실행:**
```bash
# Test HTTP endpoint
curl -H "Authorization: Bearer $API_TOKEN" \
     https://api.example.com/mcp/health

# Should return 200 OK
```

## 마이그레이션 패턴 (Migration Patterns)

### 하드코딩에서 환경 변수 방식으로 전환 (From Hardcoded to Environment Variables)

**이전:**
```json
{
  "headers": {
    "Authorization": "Bearer sk-hardcoded-token"
  }
}
```

**이후:**
```json
{
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

**마이그레이션 절차:**
1. 플러그인 README 파일에 새로운 환경 변수 정보 추가
2. 매니페스트 구성 정보를 `${VAR}` 형식을 사용하도록 변경
3. 실제 변수 값을 설정한 후 동작 테스트
4. 설정 내의 하드코딩된 원본 문자열 제거
5. 변경 사항 커밋

### Basic 인증에서 OAuth로 전환 (From Basic Auth to OAuth)

**이전:**
```json
{
  "headers": {
    "Authorization": "Basic ${BASE64_CREDENTIALS}"
  }
}
```

**이후:**
```json
{
  "type": "sse",
  "url": "https://mcp.example.com/sse"
}
```

**이점:**
- 대폭 향상된 보안 통제
- 번거로운 크리덴셜 관리 절차 생략
- 자동으로 백그라운드 토큰 갱신 처리
- 세분화된 스코프 권한 통제

## 고급 인증 (Advanced Authentication)

### 상호 TLS (mTLS) (Mutual TLS (mTLS))

일부 기업용 보안 환경에서는 클라이언트 인증서(client certificates)를 요구합니다.

**MCP 구성에서 mTLS를 직접 설정하는 것은 불가능합니다.**

**해결 방법:** mTLS 핸드셰이크를 전담 처리하는 stdio 래퍼 서버를 활용해 간접 연동합니다:

```json
{
  "secure-api": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/mtls-wrapper",
    "args": ["--cert", "${CLIENT_CERT}", "--key", "${CLIENT_KEY}"],
    "env": {
      "API_URL": "https://secure.example.com"
    }
  }
}
```

### JWT 토큰 (JWT Tokens)

헤더 헬퍼 스크립트를 사용하여 JWT 토큰을 동적으로 생성 및 주입합니다:

```bash
#!/bin/bash
# generate-jwt.sh

# Generate JWT (using library or API call)
JWT=$(generate-jwt-token)

echo "{\"Authorization\": \"Bearer $JWT\"}"
```

```json
{
  "headersHelper": "${CLAUDE_PLUGIN_ROOT}/scripts/generate-jwt.sh"
}
```

### HMAC 서명 (HMAC Signatures)

API 요청 서명 검증이 필요할 때:

```bash
#!/bin/bash
# generate-hmac.sh

TIMESTAMP=$(date -Iseconds)
SIGNATURE=$(echo -n "$TIMESTAMP" | openssl dgst -sha256 -hmac "$SECRET_KEY" | cut -d' ' -f2)

cat <<EOF
{
  "X-Timestamp": "$TIMESTAMP",
  "X-Signature": "$SIGNATURE",
  "X-API-Key": "$API_KEY"
}
EOF
```

## 권장 사항 요약 (Best Practices Summary)

### 플러그인 개발자 가이드 (For Plugin Developers)

1. 서비스가 지원하는 경우 **가급적 OAuth 연동을 권장**합니다.
2. 토큰 전달 시 반드시 **환경 변수 지정 형식**을 활용합니다.
3. 필수 변수 목록은 README에 **빠짐없이 기재**합니다.
4. 명확하고 친절한 설치 및 설정 예시 가이드를 제공합니다.
5. 크리덴셜 정보를 절대로 **코드로 커밋하지 않습니다**.
6. 반드시 **HTTPS/WSS 보안 프로토콜만 사용**합니다.
7. 인증 절차에 대해 충분한 동작 검증을 거칩니다.

### 플러그인 사용자 가이드 (For Plugin Users)

1. 플러그인을 활성화하기 전에 필수 **환경 변수들을 쉘에 설정**합니다.
2. 발급받은 토큰 정보가 외부에 노출되지 않도록 각별히 유의합니다.
3. 주기적으로 토큰을 **새로 갱신(rotation)**합니다.
4. 개발 환경과 프로덕션 환경의 토큰을 철저히 분리하여 사용합니다.
5. 설정용 `.env` 파일 등을 Git에 커밋하는 실수를 방지합니다.
6. 승인하기 전에 OAuth 스코프가 과도하게 넓지 않은지 꼼꼼히 확인합니다.

## 결론 (Conclusion)

MCP 서버 요구 사항에 부합하는 인증 체계를 구성하십시오:
- **OAuth**: 클라우드 서비스 연동용 (사용자 편의성이 가장 높음)
- **Bearer 토큰**: API 서비스 연동용
- **환경 변수**: stdio 프로세스 연동용
- **동적 헤더**: 복잡한 조건부 인증 제어용

항상 시스템 보안을 우선시하며, 사용자를 위해 명확하고 친절한 설정 가이드 문서를 기재해 두십시오.
