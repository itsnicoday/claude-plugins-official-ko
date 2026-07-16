---
description: Docker Compose를 사용하여 로컬에 Anthropic MCP 터널을 가동함으로써 Claude가 사설 MCP 서버를 호출할 수 있도록 합니다 (수동 크리덴셜 빠른 시작).
argument-hint: "[deployment-dir] (default: ./mcp-tunnel)"
allowed-tools: [Bash, Read, Write, Edit, AskUserQuestion]
---

# Docker MCP 터널 생성 (Create a Docker MCP tunnel)

수동으로 제공된 크리덴셜과 Docker Compose(로컬 테스트를 위한 가장 빠른 경로)를 사용하여, 아무것도 없는 상태에서 Anthropic이 운영하는 터널을 통해 Claude가 사설 MCP 서버를 호출하기까지의 [**MCP 터널 빠른 시작**](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/quickstart) 프로세스를 처음부터 끝까지 실행합니다.

> MCP 터널은 **연구 프리뷰** 단계에 있습니다. 업타임이나 지원 보장 없이 "있는 그대로" 제공되며 제3자 전송(Cloudflare)에 의존합니다. [보안 모델](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/security)을 읽기 전에는 프로덕션 트래픽을 이 터널로 전송하지 마십시오.

귀하는 **귀하가 직접 실행하는 로컬 명령**과 **사용자만이 할 수 있는 콘솔 작업** (터널 생성, CA 업로드 등)이 혼합된 과정을 사용자에게 안내합니다. 주의 깊은 작업자로서 각 단계를 간략히 설명하고, 명령을 실행하고, 출력을 확인하며, 실패할 경우 명확한 진단 결과와 함께 멈추어야 합니다.

배포 디렉토리: 사용자가 경로를 전달한 경우 `$ARGUMENTS`를 사용하고, 그렇지 않으면 기본값인 `./mcp-tunnel`을 사용하십시오. 아래에서는 이를 `$DIR`로 참조합니다.

## 구축할 내용 (What you'll build)

사용자의 머신에 구축될 컨테이너 스택:

- **mcp-proxy** — Anthropic의 프록시입니다. 사용자가 제어하는 인증서를 사용하여 내부 TLS 핸드셰이크를 종단하고, 업스트림 IP를 검증하며, 호스트 이름별로 라우팅합니다.
- **cloudflared** — 터널 에이전트입니다. Anthropic 터널 에지에 대해 아웃바운드 전용 연결을 수행하며, 프록시의 네트워크 네임스페이스를 공유합니다.
- **hello-mcp** *(선택)* — 사용자가 아직 공개할 자체 MCP 서버가 없는 경우에만 생성되는 FastMCP 샘플 서버입니다.

스택이 구동되면 라우팅된 서버는 퍼블릭 포트에서 대기하는 서비스 없이 Claude에서 `https://<subdomain>.<tunnel-domain>/<path>` 주소로 도달 가능합니다.

## 0단계 — 사전 검사 (Preflight)

더 진행하기 전에 아래 명령을 실행하고 누락된 항목이 있는지 보고하십시오:

```bash
docker --version && docker compose version && openssl version
```

- Docker 및 Docker Compose가 필수적입니다. `openssl` 1.1.1 이상이 필요합니다 (아래 명령들은 1.1.1 이상에서 제공되는 `-addext` 옵션을 사용합니다).
- 호스트가 `api.anthropic.com:443` 및 7844 TCP/UDP 포트의 터널 에지 주소 대역(`198.41.192.0/19`, `2606:4700:a0::/44`)에 대한 **아웃바운드** 액세스 권한이 있는지 확인하십시오. 인바운드 포트는 열리지 않습니다.

만약 `docker compose` (v2)를 사용할 수 없지만 `docker-compose` (v1)이 있다면, 이를 사용하고 사용자에게 안내하십시오. compose 파일은 v2와 호환됩니다.

## 1단계 — 터널 생성 (콘솔 — 사용자 작업)

사용자에게 [Claude Console](https://console.anthropic.com)에서 다음을 수행하도록 요청하십시오 (참조: [터널 생성](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/console#create-a-tunnel)):

1. 사이드바 → **Manage → MCP tunnels** → **New tunnel**. 터널의 이름을 지정합니다.
2. **Set up programmatic access** 옵션을 **off** 상태로 둡니다. 이 빠른 시작 가이드는 수동으로 제공하는 크리덴셜을 사용합니다.
3. 생성된 터널을 엽니다. **Connection** 섹션에서 다음 두 값을 복사합니다:
   - **Domain** — `abcd1234.tunnel.anthropic.com` 과 같은 형태입니다.
   - **Token** — 눈 모양 아이콘을 클릭한 뒤 복사합니다.

그 후 AskUserQuestion이나 직접적인 질문을 통해 사용자에게 **Domain**을 물어보십시오. **대화창에 Token을 붙여넣으라고 요청하지 마십시오.** 토큰은 아웃바운드 터널 연결을 인증하는 일종의 비밀번호이므로 대화 기록에 남지 않아야 합니다. 대신 귀하가 `$DIR/.env` 파일을 생성할 것이며, 사용자가 직접 해당 파일에 토큰을 붙여넣어야 한다고 안내하거나(3단계), compose를 실행할 쉘에서 `export TUNNEL_TOKEN='eyJ...'`로 환경변수를 내보내도록 지시하십시오.

아래 단계를 위해 도메인을 `TUNNEL_DOMAIN`으로 기록해 둡니다.

## 2단계 — 배포 디렉토리

```bash
mkdir -p "$DIR"/{config,data}
cd "$DIR"
```

## 3단계 — 크리덴셜 파일

`$DIR/.env` 파일을 생성합니다 (compose가 자동으로 로드하며, 쉘의 `export`와 달리 재부팅 후에도 유지됩니다). `TUNNEL_DOMAIN`은 귀하가 직접 작성하되, 비밀 토큰 부분은 자리표시자(placeholder)로 남겨두어 **사용자**가 직접 작성하도록 하십시오:

```
TUNNEL_DOMAIN=<the domain from step 1>
TUNNEL_TOKEN=PASTE_TUNNEL_TOKEN_HERE
```

그 후 파일 접근 권한을 제한하고 절대 git에 커밋되지 않도록 조치하십시오:

```bash
chmod 600 "$DIR/.env"
printf '.env\ndata/\n' > "$DIR/.gitignore"
```

잠시 멈추고 사용자에게 `PASTE_TUNNEL_TOKEN_HERE`를 실제 토큰으로 교체하도록 요청하십시오 (정확한 파일 경로를 알려주십시오). 값을 출력하지 않고 설정 여부를 확인하십시오:

```bash
cd "$DIR" && grep -q '^TUNNEL_TOKEN=eyJ' .env && echo "token looks set" || echo "token NOT set — edit .env"
```

이 쉘의 openssl/config 단계를 위해 로드합니다:

```bash
cd "$DIR" && set -a && . ./.env && set +a && echo "domain: $TUNNEL_DOMAIN"
```

## 4단계 — CA 및 서버 인증서 생성

프록시는 사용자가 제어하는 CA가 서명한 인증서를 사용하여 내부 TLS 핸드셰이크를 종단합니다. 둘 다 생성합니다 (Linux/macOS 기준 예시. [빠른 시작](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/quickstart) 문서에 Windows PowerShell 변형도 존재하므로 사용자가 Windows 환경인 경우 이를 제공하십시오):

```bash
cd "$DIR"

openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout data/ca.key -out data/ca.crt \
  -days 3650 -subj "/CN=mcp-tunnel-ca" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign" \
  -addext "subjectKeyIdentifier=hash"

cat > data/tls.ext <<EOF
subjectAltName = DNS:${TUNNEL_DOMAIN},DNS:*.${TUNNEL_DOMAIN}
authorityKeyIdentifier = keyid,issuer
extendedKeyUsage = serverAuth
EOF

openssl req -newkey rsa:2048 -nodes \
  -keyout data/tls.key -out /tmp/server.csr \
  -subj "/CN=${TUNNEL_DOMAIN}"
openssl x509 -req -in /tmp/server.csr \
  -CA data/ca.crt -CAkey data/ca.key -CAcreateserial \
  -out data/tls.crt -days 90 -extfile data/tls.ext

chmod 644 data/tls.key
```

이 플래그들을 사용하는 이유: 명시적인 `-addext` 확장 옵션은 배포판의 `openssl.cnf` 기본값에 구애받지 않고 터널의 [인증서 요구사항](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/reference#certificate-requirements)을 CA가 충족하도록 만듭니다. `-extfile` 옵션(OpenSSL 3.0 이상에서만 제공되는 `-copy_extensions` 대신 사용)은 OpenSSL 1.1.x 버전에서도 작동을 보장하며 프록시가 요구하는 `AuthorityKeyIdentifier`를 추가합니다. `chmod 644 data/tls.key` 명령어는 **필수**입니다. openssl은 기본적으로 키 권한을 `0600`으로 작성하지만, 프록시 컨테이너는 루트가 아닌 사용자로 실행되므로 이 키 파일을 읽을 수 있어야 합니다.

`data/tls.key` 및 `data/ca.key`는 민감한 데이터입니다. 이 파일들이 위치한 `data/` 디렉토리는 3단계에서 작성한 `.gitignore`에 의해 이미 제외되었습니다.

## 5단계 — CA 등록 (콘솔 — 사용자 작업)

터널 상세 페이지에서 사용자에게 **Certificates** → **Add certificate** 위치로 스크롤하여 (참조: [CA 인증서 추가](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)), `$DIR/data/ca.crt` 파일을 업로드하도록 요청하십시오 (또는 파일 내용을 복사해서 붙여넣도록 `cat data/ca.crt`로 출력해 주십시오). 인증서가 등록되면 터널 상태가 **Active**로 전환됩니다. 이 작업이 완료되기 전까지는 에이전트 선택기에 터널이 표시되지 않습니다.

사용자가 터널 상태가 **Active**로 표시됨을 확인할 때까지 대기한 후 다음 단계를 진행하십시오.

## 6단계 — 업스트림 MCP 서버 선택

사용자에게 물어보십시오 (AskUserQuestion):

- **"이미 MCP 서버를 가지고 있습니다"** — 접근 가능한 주소를 `scheme://host:port` 형식으로 획득합니다 (포트 번호 필수, 경로 생략 — 프록시는 설정 로드 시 업스트림 값에 지정된 경로를 거부합니다). 이는 프록시 컨테이너에서 도달 가능해야 하며 RFC1918 사설 IP 대역(`10/8`, `172.16/12`, `192.168/16`)으로 해석되어야 합니다. 프록시는 기본적으로 퍼블릭 및 루프백 업스트림 주소를 거부합니다 (SSRF 방지). 만약 Compose 서비스로 실행 중이라면, 네트워크를 공유할 수 있도록 compose 파일에 추가하십시오. 호스트 상에서 실행 중인 경우 트러블슈팅("호스트 프로세스")을 참조하십시오. 사용자와 함께 라우팅 서브도메인(예: `wiki`)을 선택하십시오.
- **"샘플 서버를 사용하겠습니다"** — 아래의 FastMCP `hello-server`를 Compose 서비스인 `hello-mcp`로 스캐폴딩하고 라우팅 서브도메인을 `echo`로 설정합니다.

### 샘플 서버 (선택된 경우에만 해당)

`$DIR/hello_server.py` 파일을 작성합니다:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("hello-server", host="0.0.0.0", port=9000)


@mcp.tool()
def hello(name: str = "world") -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"


if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

## 7단계 — 프록시 설정

`$DIR/config/mcp-proxy.yaml` 파일을 작성합니다. `tunnel_domain`은 **필수** 사항입니다 (프록시는 유입되는 호스트 이름에서 이 도메인을 떼어내어 `routes`에서 서브도메인을 찾습니다). `routes`는 서브도메인 → 업스트림 URL의 **평면 맵(flat map)** 구조이며, 리스트가 *아닙니다*:

```yaml
listen_addr: ":8080"
log_level: info
tunnel_domain: <TUNNEL_DOMAIN>
tls:
  cert_file: /data/tls.crt
  key_file: /data/tls.key
routes:
  echo: http://hello-mcp:9000
```

실제 `TUNNEL_DOMAIN`을 대입하십시오. 사용자가 자체 서버를 가져온 경우 `routes:` 블록을 사용자가 선택한 서브도메인 → 업스트림 주소(예: `wiki: http://wiki-mcp.internal:8080`)로 교체하십시오. 여러 라우트를 유지할 수 있습니다.

## 8단계 — Compose 파일

`$DIR/docker-compose.yaml` 파일을 작성합니다. 이미지는 다이제스트(digest)로 핀 고정되어 있습니다:

```yaml
services:
  mcp-proxy:
    image: us-docker.pkg.dev/anthropic-public-registry/images/mcp-proxy@sha256:6b9adedbf2763143ec72f106ecaf0ce7fd3294e89b208f54a1db97a33d14c5ba
    command: ["-config", "/etc/mcp-proxy/config.yaml"]
    volumes:
      - ./config/mcp-proxy.yaml:/etc/mcp-proxy/config.yaml:ro
      - ./data:/data:ro
    restart: unless-stopped

  cloudflared:
    image: cloudflare/cloudflared@sha256:6b599ca3e974349ead3286d178da61d291961182ec3fe9c505e1dd02c8ac31b0
    command: tunnel --no-autoupdate run --url http://localhost:8080
    environment:
      - TUNNEL_TOKEN
    network_mode: "service:mcp-proxy"
    restart: unless-stopped
```

수동 흐름에서는 `--url http://localhost:8080`이 **필수적**입니다. 서버 측으로 푸시되는 인그레스 규칙이 없으므로, 이 옵션이 없으면 cloudflared가 모든 요청에 대해 503 오류를 반환합니다. `network_mode: "service:mcp-proxy"`는 프록시의 네트워크 네임스페이스를 공유하여 `localhost:8080`으로 접근할 수 있게 합니다. `environment: - TUNNEL_TOKEN` (값 미지정)은 `.env`로부터 변수를 그대로 상속하여 전달합니다.

샘플 서버를 선택한 경우 서비스를 덧붙입을 수 있습니다:

```yaml
  hello-mcp:
    image: python:3.13-slim
    working_dir: /app
    volumes:
      - ./hello_server.py:/app/hello_server.py:ro
    command: sh -c "pip install --quiet mcp && python hello_server.py"
    restart: unless-stopped
```

사용자가 자체 서버를 제공하며 그것이 컨테이너화되어 있는 경우, 프록시와 Compose 네트워크를 공유할 수 있도록 해당 서비스를 여기에 추가하십시오.

(보안 강화가 적용된 단일 호스트 배포 — 비루트 사용자, 읽기 전용 rootfs, `cap_drop: ALL`, `no-new-privileges` 등을 적용하려면 사용자에게 [Docker Compose로 배포하기](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/deploy-compose) 문서를 참조하도록 안내하십시오. 이 빠른 시작 문서는 신속한 로컬 테스트를 위해 설정을 최소한으로 유지합니다.)

## 9단계 — 시작 및 검증

```bash
cd "$DIR" && docker compose up -d
sleep 5
docker compose logs mcp-proxy | grep -i "route configured"
docker compose logs cloudflared | grep -i "Registered tunnel connection"
```

라우트마다 한 줄의 `route configured` 메시지와 **4개**의 `Registered tunnel connection` 메시지가 출력되어야 합니다. 컨테이너 구동에는 몇 초가 소요됩니다. 로그 검색 결과가 비어 있더라도 즉시 실패로 단정하지 말고 다시 로그 조회를 실행해 보십시오. 계속 비어 있다면 트러블슈팅 섹션을 참조하십시오.

## 10단계 — Claude에서 호출하기

사용자에게 다음 두 옵션을 모두 설명해 주십시오:

**Managed Agents (콘솔):** **Managed Agents → Sessions** → 새 세션 → 에이전트 선택기 **Create new agent** → **+ MCP Server** → 해당 터널 선택 → **Subdomain** = 설정된 라우트 이름 (`echo`), **Path** = `mcp` (FastMCP `streamable-http`는 `/mcp` 경로에서 서비스됩니다). 그 후 다음과 같이 요청하십시오: *"Use the hello tool to greet tunnel."* — 도구 호출과 그 결과가 예상대로 반환되어야 합니다.

**Messages API:** 호스트는 `<subdomain>.<tunnel-domain>` 형태이며, 경로는 업스트림이 제공하는 경로(FastMCP의 경우 `/mcp`)가 됩니다. 터널이 생성된 워크스페이스의 API 키를 사용하십시오.

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: mcp-client-2025-11-20" \
  -d "{
    \"model\": \"claude-opus-4-7\",
    \"max_tokens\": 1024,
    \"mcp_servers\": [{\"type\": \"url\", \"name\": \"echo\", \"url\": \"https://echo.${TUNNEL_DOMAIN}/mcp\"}],
    \"tools\": [{\"type\": \"mcp_toolset\", \"mcp_server_name\": \"echo\"}],
    \"messages\": [{\"role\": \"user\", \"content\": \"call hello with name=tunnel\"}]
  }"
```

터널은 암호화된 트래픽을 전달하지만 업스트림에 대한 **인증은 수행하지 않습니다.** 만약 업스트림 MCP 서버 자체의 별도 인증이 필요한 경우, 다른 일반적인 MCP 서버와 마찬가지로 사용자가 직접 인증 정보를 제공해야 합니다.

## 트러블슈팅 (다음 순서대로 진단)

| 증상 | 원인 | 해결책 |
|---|---|---|
| 호출자가 HTTP 500을 확인하며, cloudflared 로그에 `No ingress rules were defined` 가 표시됨 | cloudflared에 지정된 로컬 대상이 없음 | `--url http://localhost:8080` 및 `network_mode: "service:mcp-proxy"`가 모두 정의되어 있는지 확인하고 `docker compose up -d`를 실행하십시오. |
| 프록시가 `cannot unmarshal !!seq into map[string]string` 오류로 종료됨 | `routes`가 YAML 리스트 형식으로 작성됨 | 객체의 리스트가 아닌 `routes: { 이름: http://host:port }` 형식을 사용하십시오. |
| 프록시가 `open /data/tls.key: permission denied` 오류로 종료됨 | 키 권한이 `0600`이며 프록시가 non-root 사용자로 실행됨 | `chmod 644 data/tls.key`를 실행하십시오. |
| 프록시 로그에 `no route for host`가 기록됨 (호출자는 `502 No route configured for host` 수신) | `tunnel_domain`이 누락되었거나 잘못 지정됨 | 터널 상세 페이지에 표시된 정확한 도메인으로 설정하고 **프록시를 재시작**하십시오 (다음 행 참조). |
| 설정을 편집했으나 아무것도 바뀌지 않음 | 프록시가 `config.yaml`을 핫 리로드하지 않음 (`tls.cert_file`만 지원) | `docker compose restart mcp-proxy`를 실행하십시오. 파일 내용이 변경되더라도 `up -d`만으로는 컨테이너가 다시 생성되지 않습니다. |
| `tls handshake failed ... unknown certificate authority` | 이 터널에 CA가 등록되지 않았거나 해지됨 | 콘솔에서 `data/ca.crt`를 다시 업로드하십시오 (5단계). |
| `tls handshake failed ... bad certificate` | 서버 인증서의 SAN이 `*.<tunnel-domain>`과 다르거나 만료됨 | 올바른 `TUNNEL_DOMAIN`을 반영하여 서버 인증서를 재생성하십시오 (4단계). |
| `IP validation failed: <ip> is not a private address` | 업스트림이 RFC1918 사설 대역 외부 주소(예: `127.0.0.1`, 공인 IP)로 확인됨 | 프록시 네트워크 상에서 업스트림을 Compose 서비스로 실행하거나, 의도적으로 `upstream.allowed_ips` 범위를 좁히십시오 (로컬 테스트 목적 외에 `0.0.0.0/0`은 지양). |
| `dial tcp ...: connect: connection refused` (대상: `host.docker.internal`) | rootless Docker 환경이 호스트의 네트워크 네임스페이스에 연결할 수 없음 | MCP 서버를 호스트 프로세스가 아닌 Compose 서비스로 실행하십시오. |
| HTTP 502 오류 발생, 프록시 로그에 `request started` 가 없음 | cloudflared 등록이 아직 완료되지 않았거나 롤링 업데이트 중임 | 4개의 `Registered tunnel connection` 로그가 표시될 때까지 대기한 후 재시도하십시오. |
| 에이전트 **+ MCP Server** 선택기에서 터널을 찾을 수 없음 | 활성화된 인증서가 없거나 워크스페이스가 일치하지 않음 | CA 인증서를 등록하고 (5단계), 터널이 생성된 워크스페이스에서 세션을 여십시오. |
| `curl https://<proxy>:8080` 실행 시 `wrong version number` 로 실패함 | 정상적인 동작입니다. 리스너는 일반 텍스트 WS이며 TLS는 WS 스트림 내부에 래핑되어 있습니다. | 프록시를 직접 curl로 검사하지 말고, Managed Agent 또는 Messages API를 통해 검증하십시오. |

`docker compose logs cloudflared` (토큰/에지 도달성) 및 `docker compose logs mcp-proxy` (설정/인증서/라우팅) 로그는 핵심 진단 도구입니다. 아웃바운드 연결을 먼저 점검하고, 그 후 내부 TLS 핸드셰이크와 업스트림 라우팅 순서로 점검하십시오. 추가 사례는 [트러블슈팅](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/troubleshooting) 문서를 참조하십시오.

## 운영 유의사항 (간략히 설명하며, 임의로 실행하지 마십시오)

- **토큰 로테이션 (Token rotation):** 콘솔 → **Rotate token**을 누르면 즉시 이전 토큰이 무효화됩니다. `.env` 파일의 `TUNNEL_TOKEN`을 업데이트하고 `docker compose up -d cloudflared`를 실행하십시오.
- **인증서 갱신 (Cert renewal):** 서버 인증서의 유효 기한은 90일입니다. 동일한 CA(등록된 CA는 바뀌지 않음)를 사용하여 재서명하고 `data/tls.crt` 파일을 교체하십시오. 프록시가 이를 폴링하여 새로고침하므로 프록시 재시작은 불필요합니다.
- **설정 변경 시에는 항상** `docker compose restart mcp-proxy`를 실행해야 합니다.

## 요약 (Wrap up)

요약 제시: 배포 디렉토리, 설정된 라우트 정보, 터널 도메인, 그리고 Claude가 서버에 도달하는 정확한 URL 주소. 사용자에게 토큰이 `$DIR/.env` 파일 내에 평문으로 존재하며 중요 비밀값(chmod 600 설정 및 gitignore 처리됨)임을 다시 한번 인지시키고, 본 설정은 연구 프리뷰 및 로컬 테스트용이므로 보안 강화 배포나 프로그래밍 방식 접근을 위해서는 [Docker Compose로 배포하기](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/deploy-compose) / [Helm으로 배포하기](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/deploy-helm) 문서를 참고하도록 안내하십시오.
