# Code Modernization Plugin

레거시 코드베이스(COBOL, 레거시 Java/C++/.NET, monolith 웹 애플리케이션)를 Claude에게 전달하면 다음과 같은 결과물을 얻을 수 있습니다: 임원 보고용 분석서, 인터랙티브 아키텍처 맵, 코드에서 발굴한 비즈니스 규칙, 추진위원회 승인용 현대화 설계 요약서(brief), 그리고 동작 일관성(behavior-equivalence) 테스트 하네스가 포함된 프로젝트 골격 또는 변환 완료된 신규 코드. 이를 통해 동작의 왜곡(drift)이 발생하지 않았음을 증명할 수 있습니다.

이 플러그인은 정해진 절차를 강제합니다. 현대화 프로젝트가 실패하는 주된 이유는 분석 단계를 건너뛰고 바로 코드를 변환하려 들거나, 동작 왜곡을 감지할 테스트 하네스 없이 코드를 배포하기 때문입니다:

```
preflight → assess → map → extract-rules → brief → (reimagine | transform | uplift) → harden
```

초기 조사 명령어들(`assess`, `map`, `extract-rules`)은 분석 아티팩트를 `analysis/<system>/` 경로에 작성합니다. `brief` 명령어는 이들을 종합하여 승인 관문(approval gate)을 형성합니다. 세 가지 빌드 명령어는 `modernized/<system>/` 경로에 코드를 작성하며, 각각 상이한 *방식*으로 가동됩니다. `brief`의 분석 결과에 따라 어떤 방식이 적절한지 권장해 줍니다:

- **`transform`** — 추출된 비즈니스 의도를 기반으로 한 이기종 스택 간 재작성 (예: COBOL → Java).
- **`reimagine`** — 새로운 아키텍처 상에서의 신규(greenfield) 재구축.
- **`uplift`** — 코드를 최대한 보존하고 버전 차이에 따른 델타만 수정하는 동일 스택 버전 업그레이드 (예: .NET Framework → .NET 8).

![AWS CardDemo의 인터랙티브 토폴로지 맵 — 도메인을 컨테이너로 표시하고, 모듈 크기는 코드 라인 수에 비례하며, 종속성 에지는 종류별로 색상화되고, 진입점은 링으로 둘러싸여 표시됨](assets/topology-viewer-screenshot.jpg)

## Install (설치 방법)

```
/plugin install code-modernization@claude-plugins-official
```

## Quickstart (빠른 시작)

각 명령어는 시스템 디렉터리 명칭인 `<system-dir>`을 인수로 받으며, 레거시 코드가 `legacy/<system-dir>/` 경로에 존재한다고 가정합니다. 분석 아티팩트는 `analysis/<system-dir>/`에 생성되고, 현대화된 신규 코드는 `modernized/<system-dir>/`에 작성됩니다. 만약 코드가 다른 경로에 존재한다면 심볼릭 링크를 생성하십시오: `mkdir -p legacy && ln -s /path/to/code legacy/billing`.

처음 세 명령어를 귀하의 코드베이스에 실행해 보십시오. 각각 독립적인 아티팩트를 생성하므로 언제든 멈추고 검토할 수 있습니다:

```bash
/modernize-preflight billing      # 내 환경이 실행 준비가 되었는가?
/modernize-assess billing         # 어떤 시스템을 다루고 있는가?
/modernize-map billing            # 구조 시각화 (인터랙티브 맵 실행)
```

그 다음 전체 경로를 실행합니다:

```bash
/modernize-extract-rules billing                              # 비즈니스 규칙 발굴 → 테스트 가능한 규칙 카드 생성
/modernize-brief billing java-spring                          # 추진위원회 승인용 계획서 작성 (인간 참여 승인 관문)
/modernize-transform billing interest-calc java-spring        # …또는 reimagine, 또는 uplift — 명령어 섹션 참조
/modernize-harden billing                                     # 가동 중인 레거시 시스템에 대한 보안 강화
/modernize-status billing                                     # 현재 진행 상황, 노후 파일 및 다음 단계 확인
```

## Commands (명령어 목록)

순서대로 실행하도록 권장되나 각각 독립적으로 작동하므로, 도중에 멈추고 검토한 뒤 재개할 수 있습니다.

- **`/modernize-preflight <system-dir> [target-stack]`** — 환경 준비 상태 검사. 소스 코드에서 알아낼 수 없는 5가지 질문(프로젝트 범위, 로컬 빌드 및 테스트 가능 여부, 특수한 빌드 인프라, 이전 시도 이력, 금지 구역 등)을 사용자에게 질문합니다. 그 후 레거시 기술 스택을 탐지하고, 분석 도구를 점검하며, CI/빌드 정의서를 읽어 시스템 빌드 방식을 파악하고, 실제 코드를 대상으로 툴체인 스모크 테스트를 실행합니다. 누락된 include 파일이나 배포 디스크립터(deployment descriptors)의 목록을 작성하고, 프로젝트의 **범위 경계(scope boundary)** — `<system-dir>`이 더 큰 저장소의 일부인지, 그리고 외부의 어떤 모듈이 이에 의존하는지 — 를 검사합니다. 명령어별로 Ready / Ready-with-gaps / Not-ready 판정을 포함하는 `PREFLIGHT.md` 파일을 생성합니다.

- **`/modernize-assess <system-dir>`** *(또는 `--portfolio <parent-dir>`)* — 시스템 인벤토리 조사: 프로그래밍 언어 구성, 복잡도, 기술 부채, 보안 태세 및 COCOMO 복잡도 지표 산출([참조 노트](#a-note-on-cocomo)). `ASSESSMENT.md` 및 `ARCHITECTURE.mmd` 파일을 생성합니다. `--portfolio` 옵션을 전달하면 모든 하위 디렉터리를 스캔하여 마이그레이션 순서 결정을 돕는 히트맵(`portfolio.html`)을 작성합니다.

- **`/modernize-map <system-dir>`** — 종속성 및 토폴로지 맵 작성: 콜 그래프(call graph), 데이터 계보(data lineage), 진입점(entry points), 그리고 사용자 페르소나별(예: 청구인, 감사인)로 추적한 2~4개의 비즈니스 흐름을 분석합니다. `topology.json` 파일과 **줌 확대가 가능한 인터랙티브 `TOPOLOGY.html`** 파일(코드 라인 수 기반 크기 조정, 에지 토글, 검색 및 페르소나별 흐름 시뮬레이션 지원) 및 문서용 소형 `.mmd` 다이어그램을 생성합니다.

- **`/modernize-extract-rules <system-dir> [module-pattern]`** — 레거시 코드에서 비즈니스 규칙(계산식, 유효성 검사, 적격성 검사, 상태 전이 등)을 발굴하여 `file:line` 인용 및 신뢰도 등급이 기재된 Given/When/Then 형식의 "Rule Cards (규칙 카드)"를 생성합니다. 결과물로 `BUSINESS_RULES.md`와 `DATA_OBJECTS.md`가 생성됩니다.

- **`/modernize-brief <system-dir> [target-stack]`** — 분석 단계에서 조사된 정보들을 종합하여 단계별 **Modernization Brief (현대화 설계 요약서)**를 작성합니다: 대상 아키텍처 설계, 단계별 실행 계획, 페르소나 시나리오, 동작 계약 및 승인 결재 란을 포함합니다. 분석 단계에서 생성된 아티팩트들을 읽어 들이며, **하나라도 누락된 경우 즉시 실행을 중단합니다**. 인간 참여형(human-in-the-loop) 승인 관문으로서 계획 모드(plan mode)로 진입합니다. 동일 스택 업리프트 마이그레이션의 경우 버전 델타에 따라 단계별 순서가 결정되므로 **델타 카탈로그** 파일도 필수적으로 요구됩니다. 빌드 실행 명령어들은 이 요약서를 읽어 들여 각 단계의 진입 기준을 제어 관문으로 사용하므로, 요약서를 수정하는 방식으로 빌드 실행 방향을 제어할 수 있습니다.

- **`/modernize-reimagine <system-dir> <target-vision>`** — 추출된 비즈니스 의도를 기반으로 새로운 설계를 적용하여 시스템을 처음부터 다시 구축(greenfield rebuild)합니다. 사양서를 도출하고, 대상 아키텍처를 설계 및 대립적으로 검토한 후, `modernized/<system>-reimagined/` 경로 하위에 실행 가능한 인수 테스트를 포함하는 신규 서비스 코드 골격을 생성합니다. 중간에 인간의 개입을 확인하는 2개의 체크포인트가 존재합니다.

- **`/modernize-transform <system-dir> <module> <target-stack>`** — 정교한 단일 모듈 재작성 (strangler-fig 패턴: 레거시 시스템이 가동되는 상태에서 특정 파트만 교체). 먼저 계획을 수립하고(승인 관문 거침), 특징 기술 테스트를 작성한 다음, 관용적인 대상 언어 코드로 구현한 뒤, 테스트를 실행하여 기능의 동등성을 입증합니다. `TRANSFORMATION_NOTES.md` 파일을 생성합니다.

- **`/modernize-uplift <system-dir> <source-version> <target-version> [project-pattern]`** — 동일 스택 내 버전 업그레이드 (예: `.NET Framework 4.8` → `.NET 8`, Spring Boot 2 → 3) — 전면 재작성 시 코드가 왜곡되기 쉬운 일반적인 케이스에 적합합니다. 코드의 뼈대를 최대한 유지한 상태로, 컴파일이 가능하고 기존과 완전히 동일하게 동작하도록 돕는 최소한의 코드 변경(diff)을 생성합니다. 이 과정은 **델타 카탈로그**(이 코드베이스가 실제로 마주치는 버전별 깨짐 변경 사항 목록)와 해당 생태계의 마이그레이션 도구들을 기반으로 수행됩니다. 동등성 입증은 로컬 환경에서 이전 런타임과 신규 런타임을 모두 실행할 수 있는 경우 양쪽에서 테스트 제품군을 실행하여 진행하며, 그렇지 않은 경우에는 `transform`과 마찬가지로 특징 기술 테스트(characterization tests)를 활용합니다. 업리프트 작업은 **파일럿 우선(pilot-first)** 방식으로 진행됩니다: 대표적인 단일 프로젝트를 세션 내에서 종단 간으로 마이그레이션하고 도출된 조리법을 `PLAYBOOK.md`에 먼저 기록한 뒤에 다른 작업을 진행합니다. 그 후 나머지 프로젝트들은 프로젝트당 하나의 에이전트가 할당되어 **서킷 브레이커(circuit breaker)가 작동하는 종속성 기반의 점진적 배치 방식으로 병렬 처리**됩니다. 결과물로 `DELTA_CATALOG.md`, `BASELINE.md`, `PLAYBOOK.md` 및 `UPLIFT_NOTES.md`가 생성됩니다. 카탈로그 검사 결과 코드의 대부분을 변경해야 하는 상황으로 판단되면 업리프트 대신 `transform` 명령어를 사용하도록 안내합니다.

- **`/modernize-harden <system-dir>`** — **레거시** 시스템에 대한 보안 강화 검사: OWASP/CWE 취약점, 종속성 CVE, 시크릿 유출, 인젝션 취약점 등을 진단합니다. 등급별로 분류된 `SECURITY_FINDINGS.md` 파일과 검토용 `security_remediation.patch` 패치 파일을 생성합니다. **`legacy/` 폴더의 코드를 직접 수정하지는 않으며**, 귀하가 직접 패치를 검토하고 반영해야 합니다. 마이그레이션이 진행되는 동안 운영 환경에서 레거시 시스템을 안전하게 계속 가동해야 할 때 유용합니다.

- **`/modernize-status <system-dir>`** — 읽기 전용 진행 상황 보고서: 생성된 아티팩트 목록, 파일 노후화 플래그, 시크릿 정보 관리 상태 검사 결과 및 다음에 실행하기에 가장 유용한 명령어를 제시합니다.

## Agents (에이전트 목록)

명령어에 의해 호출되거나 직접 가동되는 전문화된 하위 에이전트들입니다:

- **`legacy-analyst`** — 레거시 코드(COBOL, EJB, classic ASP 등)를 읽고 구조적 요약을 작성합니다. 묵시적 종속성 및 현대적 문법으로 포장된 절차형 레거시 코드("JOBOL")를 탐지합니다. *(assess, reimagine, uplift 명령어에서 사용)*
- **`business-rules-extractor`** — 소스 코드 인용구를 포함하여 절차형 코드로부터 도메인 규칙을 발굴합니다. *(extract-rules, reimagine 명령어에서 사용)*
- **`architecture-critic`** — 대상 설계 구조와 변환된 코드를 회의적인 관점에서 검토하여 오버엔지니어링을 차단합니다. *(reimagine, transform, uplift 명령어에서 사용)*
- **`security-auditor`** — 인증, 입력값 유효성 검사, 시크릿 유출, 종속성 CVE 등을 감사합니다. *(assess, harden 명령어에서 사용)*
- **`test-engineer`** — 레거시의 기존 동작을 완벽히 고정하는 특징 기술 및 동등성 테스트를 작성합니다. *(transform, uplift 명령어에서 사용)*
- **`version-delta-analyst`** — 동일 기술 스택의 두 버전 사이에서 이 코드베이스에 영향을 미치는 호환성 손상 변경 사항을 분석하고 생태계의 마이그레이션 도구 실행을 제어합니다. *(uplift 명령어에서 사용)*
- **`uplift-migrator`** — 이미 진행 중인 업리프트 작업에서 파일럿 플레이북을 적용하여 단일 프로젝트/모듈을 마이그레이션하고 실제 빌드를 검증합니다. 플레이북이 작성되지 않은 상태에서는 작업을 거부합니다. 쓰기 범위는 자체 프로젝트 디렉터리로 엄격하게 제한됩니다. *(uplift 명령어에서 사용)*
- **`scaffolder`** — 재구상된 시스템의 단일 서비스를 구축하며, 오직 자신에게 지정된 `modernized/.../<service>/` 디렉터리 내에만 코드를 생성합니다. *(reimagine 명령어에서 사용)*

## Recommended workspace setup (권장 워크스페이스 설정)

현대화 대상 프로젝트 하위에 `.claude/settings.json` 설정을 추가하면 핵심 불변 규칙 — `legacy/` 경로는 절대 건드리지 않고, `analysis/` 및 `modernized/` 경로만 자유롭게 편집하도록 강제할 수 있습니다:

```json
{
  "permissions": {
    "allow": ["Read(**)", "Write(analysis/**)", "Write(modernized/**)", "Edit(analysis/**)", "Edit(modernized/**)"],
    "deny": ["Edit(legacy/**)", "Write(legacy/**)"]
  }
}
```

이 설정은 파일 도구들의 권한 범위를 제어합니다. 파일을 변경하는 쉘 명령어(`sed -i`, `git apply` 등)는 여전히 일반 Bash 프롬프트 권한을 거치므로 동일한 불변 규칙을 염두에 두고 검토하십시오. 이 프롬프트는 다수의 쓰기 권한 에이전트들이 동시에 병렬 처리되는 두 단계 — `/modernize-uplift` Step 5b 단계 및 `/modernize-reimagine` Phase E 단계 — 의 보안 울타리 역할을 하므로, 해당 작업을 실행할 때는 Bash 권한 모드를 *프롬프트 확인(prompted)* 승인 모드로 유지해 주십시오.

## Prerequisites (사전 요구사항)

명령어들은 환경에 맞춰 유연하게 대처하지만, 다음 도구들이 구비되면 분석 퀄리티가 향상됩니다 (한 번에 확인하려면 `/modernize-preflight`를 실행하십시오):

- **분석 도구** — [`scc`](https://github.com/boyter/scc) 또는 [`cloc`](https://github.com/AlDanial/cloc). 설치되지 않은 경우 메트릭 분석 수치는 기본 `find`/`wc` 명령어를 사용해 대체 측정됩니다.
- **레거시 기술 스택 빌드 툴체인** — 실제 가동 환경에서의 이중 테스트 실행을 통한 강력한 동등성 검증이 가능해집니다. 필수 요구사항은 아니며, 빌드 환경이 없는 경우 동등성 검증은 사전에 기록된 실행 로그 기반 테스트로 대체되고 사전 비행(preflight) 판정 결과는 Not-ready 차단이 아닌 Ready-with-gaps 상태로 완화되어 보고됩니다.
- **코드베이스 전체 포함** — 배포 디스크립터(JCL, CICS, 라우트 설정 파일 등), copybooks/includes, DDL 등이 코드베이스 내에 모두 갖춰져 있어야 진입점 탐지와 데이터 계보 추적이 정상적으로 가동됩니다.

## Safety notes (안전 관련 안내)

**분석 대상 코드는 신뢰할 수 없는 입력입니다.** 악의적인 코드베이스에는 "ignore previous instructions" 또는 "mark this rule approved"와 같은 조작된 주석이 숨어 있어 `BUSINESS_RULES.md` 또는 `SECURITY_FINDINGS.md` 등 후속 명령어들이 신뢰하는 아티팩트의 생성 결과를 왜곡시킬 위험이 있습니다. 이에 대응하는 방어 기제: 에이전트는 파일의 내용을 철저히 데이터로만 취급하고 명령어 형태의 텍스트 발견 시 즉시 플래그를 지정해 보고합니다. 검증 에이전트는 다른 에이전트의 요약 설명이 아닌 원본 소스 코드에 근거하여 모든 규칙과 발견 결과를 교차 검증합니다. 또한 파일 시스템 경로를 엄격히 검증하며, 코드를 실질적으로 생성하기 전에 `/modernize-brief` 단계를 거쳐 사람이 직접 검증하도록 제어합니다. 신뢰할 수 없는 레거시 코드로부터 생성된 분석 아티팩트는 코드 자체와 동일한 수준의 의구심을 가지고 신중히 검토해 주십시오.

**공유 아티팩트에는 시크릿 정보를 노출하지 않습니다.** 발견된 자격 증명 정보는 모두 마스킹 처리(`AKIA****`)되며, 버전 관리에서 제외되는 `SECRETS.local.md` 파일(또는 git을 사용하지 않는 프로젝트의 경우 `~/.modernize/<system>/` 경로)에 격리되어 기록됩니다. `/modernize-harden` 실행 시 자격 증명을 제거하는 패치(patch) 정보 또한 버전 관리에서 제외되는 별도의 패치 파일로 관리됩니다. 격리된 보안 파일에 실질적인 원래 자격 증명 값을 포함시키려면 `--show-secrets` 옵션을 명시하여 실행하십시오. 만약 이 플러그인의 구버전을 실제 운영 프로젝트에 실행한 적이 있다면 `analysis/` 하위 아티팩트들이 커밋되어 형상 관리에 노출되었는지 확인하고 노출된 시크릿은 즉시 변경하십시오.

### A note on COCOMO (COCOMO 지표에 관하여)

`assess` 명령어가 코드 규모를 기초로 산출하는 COCOMO 수치는 **오직 여러 시스템 간의 상대적인 복잡도 및 규모를 비교하여 작업 순서를 설정하기 위한 인덱스로만 사용**되어야 하며, 실제 일정 예측이나 개발 비용 산정에 사용되어서는 안 됩니다. COCOMO 상수는 인간 개발팀의 생산성을 상정하고 설계된 지표이므로, AI 에이전트에 의한 현대화 작업 일정 측정에는 부적합하며 잘못된 결과로 이어집니다.

## Dynamic workflow orchestration (동적 워크플로우 오케스트레이션)

Workflow 도구를 포함하여 빌드된 Claude Code 환경에서 다음 5가지 명령어(`extract-rules`, `harden`, `assess --portfolio`, `reimagine`, `uplift`) run as scripted multi-agent orchestrations that fan out more agents for deeper coverage — looping until findings stabilize, and adversarially verifying each finding before it's written. `uplift`'s migration fan-out runs in dependency-aware escalating batches behind a per-batch **circuit breaker**, so a playbook that stops working is caught within a handful of agents and the spend stops until it is revised. They fall back to direct subagent fan-out on older builds automatically; no configuration needed. Invoking the slash command is the opt-in.

## License (라이선스)

Apache 2.0. 자세한 사항은 `LICENSE` 파일을 참조하십시오.
