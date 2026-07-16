# mcp-tunnels

Anthropic [**MCP 터널**](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/overview)을 통해 사설 네트워크 내에서 실행 중인 MCP 서버에 Claude를 연결하십시오. 인바운드 포트가 필요 없고, 외부 노출이 없으며, 오리진에 대한 IP 화이트리스팅도 필요하지 않습니다. 트래픽은 아웃바운드 전용 연결을 통해 흐릅니다.

> **연구 프리뷰.** MCP 터널은 업타임이나 지원 보장 없이 "있는 그대로" 제공되며 제3자 전송 프로바이더(Cloudflare)에 의존합니다. 민감한 정보를 전송하기 전에 [보안 모델](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/security)을 검토하십시오.

## 명령 (Commands)

### `/create-docker-mcp-tunnel [deployment-dir]`

수동으로 제공된 크리덴셜(로컬 테스트를 위한 가장 빠른 경로)과 Docker Compose를 사용하여 머신에서 MCP 터널 [**빠른 시작**](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/quickstart) 프로세스를 처음부터 끝까지 실행합니다. Claude 콘솔에서 사용자만이 직접 수행할 수 있는 단계를 안내하고, 그 외의 다른 모든 작업은 자동으로 대신 실행해 줍니다:

1. **사전 검사 (Preflight)** — Docker, Docker Compose, OpenSSL 및 아웃바운드 연결성을 확인합니다.
2. **터널 생성 (콘솔)** — 사용자가 터널을 생성하고 도메인을 복사합니다. 토큰은 채팅에 노출되지 않고 접근이 제한되고 gitignore 처리된 `.env` 파일에 기록됩니다.
3. **인증서** — OpenSSL을 사용해 터널에 정확히 필요한 확장자 사양을 갖춘 CA 및 서버 인증서를 생성합니다.
4. **CA 등록 (콘솔)** — 사용자가 `ca.crt`를 업로드하면 터널이 Active 상태가 됩니다.
5. **업스트림** — 검증 가능한 FastMCP 샘플 서버의 스캐폴딩을 작성하거나, 기존에 가지고 있는 MCP 서버를 연결합니다.
6. **프록시 설정 + Compose** — 다이제스트 핀(digest-pinned) 처리된 이미지 및 cloudflared 에이전트와 함께 `mcp-proxy.yaml` 및 `docker-compose.yaml`을 작성합니다.
7. **시작 및 검증** — 스택을 구동하고 프록시 및 터널 로그를 검사합니다.
8. **Claude에서 호출** — Managed Agents 및 Messages API에서 서버에 도달하는 방법을 보여줍니다.

또한 트러블슈팅 매트릭스(TLS 핸드셰이크 실패, `routes`가 맵이어야 하는 문제, `tls.key` 권한 문제, 설정 핫 리로드 미지원 함정, 업스트림 IP 검증)와 토큰 로테이션 및 인증서 갱신을 위한 운영 기초 지식을 포함하고 있습니다.

**사용법:**

```
/create-docker-mcp-tunnel
/create-docker-mcp-tunnel ~/work/my-tunnel
```

### CA 인증서를 다른 머신으로 복사하기 (Copying the CA certificate to another machine)

보통 브라우저의 콘솔에서 CA를 등록하게 되는데, 이 브라우저가 실행되는 머신은 스택이 작동 중인 머신과 다를 수 있습니다 (예: 터널은 원격 홈스페이스에서 실행되지만 `ca.crt` 업로드는 랩톱이나 개발 장비에서 수행하는 경우). 오직 **인증서** (`<deployment-dir>/data/ca.crt`, 약 1KB 크기의 PEM) 파일만 호스트 외부로 전달되어야 하며, `data/ca.key`나 `data/tls.key` 파일은 절대 외부로 유출되어서는 안 됩니다.

이처럼 작은 파일의 경우 가장 간단한 방법은 내용을 터널에 출력하여 콘솔의 인증서 입력 필드에 직접 붙여넣는 것입니다:

```bash
cat <deployment-dir>/data/ca.crt   # 기본 경로: ~/mcp-tunnel/data/ca.crt
```

`scp`를 사용해 파일로 복사하려면, 상대방 머신에 SSH 접속이 가능한 머신에서 명령을 실행하십시오 (`scp`는 두 개의 원격지 간에 직접 중계할 수 없습니다). 홈스페이스에서 개발 장비로 다운로드하는 경우 — 만약 `coder config-ssh`를 실행했다면 호스트는 `coder.<workspace>`가 됩니다:

```bash
scp coder.<workspace>:<deployment-dir>/data/ca.crt .
# generic form: scp <homespace-ssh-host>:~/mcp-tunnel/data/ca.crt .
```

또는 호스트에서 개발 장비로 접근할 수 있다면 호스트에서 직접 전송(push)하십시오:

```bash
scp <deployment-dir>/data/ca.crt <user>@<devbox-host>:~/
```

## 빌드되는 항목 (What gets built)

호스트에 구축되는 소규모 컨테이너 스택:

| 컨테이너 | 역할 |
|---|---|
| **mcp-proxy** | Anthropic의 프록시입니다. 사용자가 제어하는 인증서로 내부 TLS를 종단하고, 업스트림 IP를 검증하며, 호스트 이름별로 라우팅합니다. |
| **cloudflared** | 터널 에이전트입니다. Anthropic 터널 에지로만 아웃바운드 연결을 수행하며, 프록시의 네트워크 네임스페이스를 공유합니다. |
| **hello-mcp** *(선택)* | 아직 공개할 MCP 서버가 없는 경우에만 생성되는 FastMCP 샘플 서버입니다. |

구동 시 라우팅된 서버는 퍼블릭 포트에서 대기하는 서비스 없이 Claude에서 `https://<subdomain>.<your-tunnel-domain>/<path>` 주소로 도달 가능합니다.

## 요구 사항 (Requirements)

- Docker 및 Docker Compose
- OpenSSL 1.1.1 이상
- MCP 터널을 관리할 수 있는 권한을 가진 Claude Console 역할
- `api.anthropic.com:443` 및 7844 TCP/UDP 포트의 터널 에지에 대한 아웃바운드 액세스. 인바운드 포트는 열리지 않습니다.

## 범위 및 다음 단계 (Scope and next steps)

이 플러그인은 **수동 크리덴셜, 단일 호스트, 로컬 테스트** 경로를 대상으로 합니다. 강화된 단일 호스트 배포(비루트 권한, 읽기 전용 rootfs, 보안 성능 축소 적용), Kubernetes 배포 또는 [Workload Identity Federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)을 통한 프로그래밍 방식의 액세스에 대해서는 공식 배포 가이드를 참조하십시오:

[Docker Compose로 배포하기](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/deploy-compose) / [Helm으로 배포하기](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/deploy-helm).

## 작성자

Anthropic (support@anthropic.com)

## 라이선스

`LICENSE` 파일을 참조하십시오.
