# security-guidance

Claude가 생성한 코드에 대한 보안 리뷰 플러그인입니다. 다음 3단계 계층으로 작동합니다:

1. **Pattern warnings (패턴 경고)** — 알려진 위험한 약 25가지 패턴(`yaml.load`, `torch.load(weights_only=False)`, 신뢰할 수 없는 데이터에 대한 `pickle.load`, raw `innerHTML`, 하드코딩된 시크릿 등)에 대해 `Edit`/`Write` 수행 시 정규식 기반의 인스턴트 미리 알림을 보냅니다.
2. **LLM diff review (LLM diff 리뷰)** — Claude가 턴을 마칠 때 플러그인이 변경 사항(diff)을 빠른 LLM 호출(기본값: Opus 4.7)로 보내 리뷰를 수행하고, 심각도가 높은 보안 취약점 분석 결과를 Claude에게 피드백하여 사용자가 응답을 보기 전에 이를 수정할 수 있도록 합니다.
3. **Agentic commit review (에이전트식 커밋 리뷰)** — `git commit` 시 SDK 기반의 리뷰어가 관련 파일들(`Read`/`Grep`/`Glob`)을 읽고 코드베이스 전체의 데이터 흐름을 추적하여, 단순 패턴 매칭으로는 놓치기 쉬운 다중 파일 보안 취약점(IDOR, 인증 우회, 다중 파일 SSRF 등)을 감지합니다.

발견되는 취약점에는 인젝션, XSS, SSRF, 하드코딩된 시크릿, IDOR, 인증 우회, 안전하지 않은 역직렬화, 경로 탐색(path traversal) 등 일반적인 웹 취약점 분류가 포함됩니다.

## Install (설치 방법)

```
/plugin install security-guidance@claude-plugins-official
```

마켓플레이스 플러그인은 Claude Code에 기본적으로 활성화되어 배포되므로 CLI 자체가 설치되어 있다면 별도의 설정이 필요하지 않습니다.

## Prerequisites (사전 요구사항)

- Claude Code CLI ≥ v2.1.144
- `PATH` 환경 변수에 등록된 Python 3.8+ (`python3`, `python` 또는 `py -3` 중 플러그인이 작동하는 첫 번째 버전을 자동으로 선택함)
- 작동 가능한 API 경로 (구독 서비스, API 키 또는 제3자 제공업체 설정)

## Configuration (설정)

모든 설정은 환경 변수를 통해 이루어집니다. 기본 동작을 실행하는 데 필수적인 환경 변수는 없습니다.

### Selecting a model (모델 선택)

```bash
# 1P / gateway: 표준 모델 ID
SECURITY_REVIEW_MODEL=claude-opus-4-7   # 기본값

# Bedrock: inference-profile ID 사용
SECURITY_REVIEW_MODEL=us.anthropic.claude-opus-4-7

# Vertex: Vertex 날짜 태그 형식 사용
SECURITY_REVIEW_MODEL=claude-opus-4-7@20260218
```

`SECURITY_REVIEW_MODEL`은 LLM diff 리뷰를 제어합니다. `SG_AGENTIC_MODEL` (동일한 구문)은 에이전트식 커밋 리뷰어를 제어하며, 기본적으로 동일한 모델로 지정됩니다.

### Enabling/disabling layers (단계 활성화/비활성화)

| 환경 변수 | 기본값 | 기능 |
|---|---|---|
| `SECURITY_GUIDANCE_DISABLE=1` | 미설정 | 킬 스위치(Kill switch) — 플러그인 전체를 비활성화합니다. |
| `ENABLE_PATTERN_RULES=0` | 활성화 | 1단계(정규식 패턴 경고)를 비활성화합니다. |
| `ENABLE_CODE_SECURITY_REVIEW=0` | 활성화 | 모든 LLM 리뷰(Stop hook + commit/push)를 비활성화합니다. |
| `ENABLE_STOP_REVIEW=0` | 활성화 | Stop-hook diff 리뷰만 비활성화하고 commit/push 리뷰는 유지합니다. 다른 에이전트가 작업자의 턴 사이에 HEAD를 이동시킬 수 있는 멀티 에이전트 / 공유 작업 트리 환경에서 유용합니다. |
| `ENABLE_COMMIT_REVIEW=0` | 활성화 | 3단계(에이전트식 커밋 리뷰)를 비활성화합니다. |

### Higher-recall mode (높은 재현율 모드)

```bash
SG_DUAL_OR=on   # 기본값은 off
```

두 개의 리뷰 호출을 병렬로 실행하고 그 결과를 병합합니다. 당사 테스트에 따르면 API 비용이 약 2배로 증가하지만 취약점을 몇 퍼센트 더 감지할 수 있습니다. 대부분의 사용자에게는 필요하지 않습니다.

## Org-specific policies (조직 전용 정책)

다음 중 원하는 위치에 `claude-security-guidance.md` 파일을 생성하여 규칙을 추가할 수 있습니다:

- `~/.claude/claude-security-guidance.md` — 사용자 전역 규칙
- `<project>/.claude/claude-security-guidance.md` — 프로젝트 규칙 (버전 관리에 커밋하도록 권장)
- `<project>/.claude/claude-security-guidance.local.md` — 로컬 덮어쓰기 규칙 (`.gitignore`에 추가하도록 권장)

세 파일 모두 로드되어 사용자 → 프로젝트 → 프로젝트 로컬 순서로 LLM diff 리뷰 프롬프트 뒤에 연결되어 적용됩니다. 결합된 크기가 8 KB 프롬프트 한도를 초과하면 뒷부분이 잘리므로 전역 규칙이 유지되고 프로젝트 로컬 규칙이 가장 먼저 잘립니다. 에이전트식 커밋 리뷰어(3단계)는 현재 이 파일을 읽지 않습니다. 작성 예시:

```markdown
# Acme security rules

- `customers` 또는 `orders` 테이블에 대한 모든 SELECT 쿼리는 반드시 `db.replica`를 거쳐야 하며,
  절대 `db.primary`를 직접 조회해서는 안 됩니다. primary는 쓰기 작업 전용입니다.
- 백그라운드 작업은 사용자 콘텍스트 인증 토큰을 사용해서는 안 됩니다. 대신
  `jobs.get_service_account()`를 통해 서비스 계정 자격 증명을 획득해야 합니다.
- 사용자 제어 기능이 있는 `url`을 사용하는 `requests.get(url)` 호출 시에는
  `acme.net.safe_request`에 정의된 SSRF 허용 목록 래퍼를 거쳐야 합니다.
```

내장 규칙만으로도 일반적인 웹 취약점 클래스들을 처리할 수 있으므로, `claude-security-guidance.md` 파일은 모델이 스스로 유추할 수 없는 귀하의 코드베이스에 특화된 규칙을 정의할 때 사용하십시오.

## Privacy and data handling (개인정보 보호 및 데이터 처리)

이 플러그인은 리뷰를 수행하기 위해 모델 엔드포인트로 데이터를 전송합니다. 구체적으로, Stop-hook diff 리뷰가 실행될 때마다 변경된 파일 경로, diff 덩어리(hunks) 및 diff에 포함된 관련 파일 내용이 전송됩니다. 에이전트식 커밋 리뷰 실행 시에는 이에 더해 리뷰어가 데이터 흐름을 추적하는 동안 `Read`/`Grep`/`Glob`을 통해 수집한 파일 내용도 함께 전송됩니다. 작성한 `claude-security-guidance.md` 내용(전역, 프로젝트, 로컬)은 매 리뷰 프롬프트 뒤에 추가되어 전송되므로 보안 시크릿(비밀 정보)을 기록하지 마십시오.

데이터가 전송되는 엔드포인트는 Claude Code 설정에 따라 결정됩니다:
- **기본값 (Anthropic API / 구독 서비스):** `api.anthropic.com`으로 전송되며 Anthropic의 [상업 약관(Commercial Terms)](https://www.anthropic.com/legal/commercial-terms) 및 [개인정보 처리방침(Privacy Policy)](https://www.anthropic.com/legal/privacy)에 따라 처리됩니다.
- **LLM 게이트웨이** (`ANTHROPIC_BASE_URL` 설정 시): 설정된 게이트웨이 URL로 전송됩니다. 게이트웨이 운영자의 약관이 적용됩니다.
- **제3자 제공업체** (Bedrock / Vertex / Foundry / Mantle): 설정한 제공업체 엔드포인트로 전송됩니다. 해당 제공업체(예: AWS / GCP / Azure)의 데이터 처리 약관이 적용됩니다.

플러그인은 자체 디버그 로그를 `~/.claude/security/log.txt`에 기록합니다 (`SECURITY_GUIDANCE_DEBUG_LOG` 변수로 변경 가능). 이 로그에는 파일 내용 전체나 모델 프롬프트는 제외된 채 diffstate 메타데이터와 탐지 카테고리만 포함되며, 파일 크기가 1 MB에 도달하면 순환(rotate)됩니다. 어떤 정보도 외부로 업로드되지 않습니다.

## Limitations (제한 사항)

이 도구는 개발 보조 수단이며 완전성을 보장하지는 않습니다. 발견된 분석 결과는 제안 사항으로 처리해야 하며, 사람에 의한 코드 리뷰, SAST/DAST, 종속성 스캔 또는 모의 침투 테스트를 완전히 대체할 수 없습니다. 리뷰어가 취약점을 놓치거나 오탐(false positive)을 발생시킬 수 있으며 코드베이스, 프로그래밍 언어, 모델 버전에 따라 동작이 다를 수 있습니다. **어떠한 품질 보증도 제공되지 않으며** 사용 시 Anthropic의 [상업 약관(Commercial Terms)](https://www.anthropic.com/legal/commercial-terms)이 적용됩니다.

## Troubleshooting (문제 해결)

**플러그인이 실행되지 않는 것처럼 보임** — 디버그 로그에 `~/.claude/claude-security-guidance.md` 로드 기록이나 훅 활동이 나타나는지 확인하십시오. Claude Code를 `--debug-file /tmp/claude/debug.txt` 옵션과 함께 실행하고 `security_reminder_hook`을 검색(grep)해보십시오. 또한 플러그인은 자체 로그를 `~/.claude/security/log.txt`에 기록합니다.

**리뷰 결과가 아무것도 나오지 않음** — API 경로가 정상적으로 동작하는지 확인하십시오. 제3자 제공업체(3P)를 사용하는 경우 `SECURITY_REVIEW_MODEL`이 단순히 `claude-opus-4-7`이 아닌 해당 제공업체 전용 ID로 지정되었는지 확인하십시오. LLM 게이트웨이를 사용하는 경우, 게이트웨이 로그에서 플러그인으로부터 오는 `POST /v1/messages` 트래픽이 있는지 확인하십시오.

**오탐이 너무 많음** — `SECURITY_REVIEW_MODEL`을 더 저렴한 모델(`claude-sonnet-4-6`)로 변경하여 테스트해 보십시오. 분석의 정밀도가 최우선인 경우 Opus 4.7을 유지하십시오.

**특정 결과를 탐지에서 제외하고 싶음** — 해당 라인에 왜 안전한지 설명하는 주석을 남기십시오. LLM 리뷰어는 인라인으로 기록된 소명을 예외 처리로 간주합니다. 시스템 전체에 걸친 지속적인 예외 처리는 `claude-security-guidance.md` 파일에 기록하십시오.

## Reporting issues (이슈 제보)

[security-guidance 플러그인 저장소](https://github.com/anthropics/claude-code/issues)에 다음 정보를 포함하여 이슈를 생성해 주십시오:
- Claude Code CLI 버전 (`claude --version`)
- Provider setup (1P / Bedrock / Vertex / LLM 게이트웨이 등)
- 취약점이 재현되는 최소 크기의 diff
- `~/.claude/security/log.txt` 파일의 관련 로그 섹션
