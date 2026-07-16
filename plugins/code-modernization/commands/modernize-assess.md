---
description: 레거시 시스템에 대한 완전한 분석 및 포트폴리오 분석 — 인벤토리, 복잡도, 부채, 상대적 규모
argument-hint: <system-dir> [--show-secrets] | --portfolio <parent-dir>
---

**모드 선택.** `$ARGUMENTS`가 `--portfolio`로 시작하는 경우 그 다음에 오는 디렉터리에 대해 **포트폴리오 모드(Portfolio mode)**를 실행합니다. 그렇지 않은 경우 시스템 디렉터리에 대해 **단일 시스템 모드(Single-system mode)**를 실행합니다. 플래그는 위치와 무관하게 파싱됩니다. `--show-secrets`는 시스템 디렉터리 앞이나 뒤에 올 수 있으며, 시스템 디렉터리는 플래그가 아닌 첫 번째 토큰입니다.

---

# 포트폴리오 모드 (`--portfolio <parent-dir>`)

상위 디렉터리의 모든 직속 하위 디렉터리를 훑어보고 운영 위원회가 다년 계획의 순서를 결정하는 데 사용할 수 있는 히트맵을 생성합니다.

**권장 — 워크플로우 오케스트레이션(Workflow orchestration).** 이번 세션에서 **Workflow 도구**를 사용할 수 있는 경우(이 명령 호출이 실행 권한 부여임), 직속 하위 디렉터리를 먼저 나열한 뒤 — 워크플로우 스크립트는 파일 시스템에 직접 접근할 수 없으므로 — 시스템당 하나의 분석 에이전트를 독립적으로 실행합니다:

```bash
ls -d <parent-dir>/*/ | xargs -n1 basename   # bare subdir names, not paths
```

```
Workflow({
  scriptPath: "${CLAUDE_PLUGIN_ROOT}/workflows/portfolio-assess.js",
  args: { parentDir: "<parent-dir>", systems: ["<sub1>", "<sub2>", ...] }
})
```

시스템당 하나의 에이전트가 실행됩니다. (30개 시스템이 있는 경우 = 30개 에이전트 — 실행 전에 사용자에게 개수를 알리십시오. 런타임은 동시성 한계에 따라 대기열에 추가합니다.) 각 에이전트는 구조화된 메트릭 행을 반환하며 워크플로우는 코드 상에서 균일하게 COCOMO-II를 계산하므로 모든 행이 동일한 공식을 사용합니다. 반환 시 `rows`(`unmeasured`에 있는 항목의 경우 "측정되지 않음" 마커 행 포함)를 Step P4 히트맵에 렌더링하고, 순서 결정 권장사항을 직접 추가한 뒤 Step P1~P3을 건너뜁니다. 스윕 과정이 매우 긴 경우 워크플로우의 `runId`를 기록하십시오. 스윕 도중 세션이 종료되면 `resumeFromRunId`를 사용하여 다시 시작할 수 있고, 이미 완료된 시스템은 캐시에서 즉시 결과를 반환합니다.

**대체 방안** (Workflow 도구가 없는 경우): 시스템별로 Step P1~P3을 직접 실행한 뒤 P4를 실행합니다.

## Step P1 — 시스템별 메트릭

각 하위 디렉터리 `<sys>`에 대해:

```bash
cloc --quiet --csv <parent>/<sys>          # LOC by language
lizard -s cyclomatic_complexity <parent>/<sys> 2>/dev/null | tail -1
```

`cloc`/`lizard`가 설치되어 있지 않은 경우 `scc <parent>/<sys>` (LOC + 복잡도) 또는 확장자별로 그룹화한 `find` + `wc -l`로 대체하고, 파일당 결정 키워드(decision keywords)의 개수를 세어 복잡도를 추정합니다. 사용한 도구를 기록하십시오.

수집 항목: 총 SLOC, 주요 사용 언어, 파일 개수, 평균 및 최대 순환 복잡도(CCN). 의존성 최신성(dependency freshness)의 경우, 매니페스트(`package.json`, `pom.xml`, `*.csproj`, `requirements*.txt`, 카피북 디렉터리)를 찾아 생성 시기 및 고정된 버전(pinned-version) 개수를 기록합니다.

## Step P2 — COCOMO-II 복잡도 인덱스

시스템별 COCOMO-II 기본 수치를 계산합니다: `2.94 × (KSLOC)^1.10` (명목상 규모 인자 기준). 단순한 추측이 아닌 신뢰할 수 있는 정보가 되도록 공식과 입력값을 보여줍니다.

**이를 단지 시스템의 순위를 매기고 순서를 결정하기 위한 상대적 복잡도/규모 인덱스로만 사용하십시오.** 숫자가 클수록 더 크고 복잡한 시스템군을 의미합니다. **이는 현대화 일정이나 비용이 아닙니다.** COCOMO의 M/M(인월) 수치는 전통적인 사람이 작업하는 팀의 생산성을 가정합니다. 에이전트 기반 현대화는 이러한 생산성 곡선을 따르지 않으므로, 작업 소요 시간이나 비용으로 제시(또는 변환)하지 마십시오. 열 이름을 "인월(person-months)"이 아닌 인덱스로 라벨링하고, 날짜나 기간을 절대 연결하지 마십시오.

## Step P3 — 문서화 커버리지

각 시스템에 대해 헤더 주석 블록이 있는 소스 파일과 없는 소스 파일의 개수를 세고, 존재하는 아키텍처 문서(`README`, `docs/`, ADR)를 나열합니다. 커버리지 비율(%)과 가장 문서화가 안 된 하위 시스템을 보고합니다.

## Step P4 — 히트맵 렌더링

`analysis/portfolio.html`을 작성합니다 (어두운 `#1e1e1e` 배경, `#d4d4d4` 텍스트, `#cc785c` 악센트, system-ui 글꼴, 모든 CSS는 인라인으로 처리). 시스템당 하나의 행으로 구성하며 열은 다음과 같습니다: **System · Lang · KSLOC · Files · Mean CCN · Max CCN · Dep Freshness · Doc Coverage % · Complexity (COCOMO index) · Risk**. 인덱스 및 리스크(Risk) 셀은 색상 등급(녹색→황색→적색)을 지정합니다. 테이블 아래에는 어떤 시스템을 먼저 진행해야 하는지와 그 이유에 대해 2~3문장의 순서 결정 권장사항을 작성합니다.

작업을 멈춥니다. 사용자에게 `analysis/portfolio.html`을 열어보라고 안내합니다.

---

# 단일 시스템 모드

`legacy/$1`에 대한 완전한 **현대화 평가(modernization assessment)**를 수행합니다.

이는 탐색 단계입니다. 목표는 엔지니어링 부사장이 예산 회의에 가져갈 수 있도록 사실에 기반한 의사결정 브리프를 작성하는 것입니다. 다음 순서로 작업하십시오:

## Step 1 — 정량적 인벤토리

다음을 실행하고 출력을 보여줍니다:
```bash
scc legacy/$1
```
그런 다음 `scc --by-file -s complexity legacy/$1 | head -25`를 실행하여 복잡도가 가장 높은 파일을 식별합니다. scc의 COCOMO 수치를 **오직 상대적 복잡도/규모 인덱스로만 사용**하십시오. 그리고 **scc의 "Estimated Schedule Effort" 및 달러 기준 비용 행은 무시하십시오.** 이는 사람이 작업하는 팀의 일정과 예산을 반영한 것이므로 에이전트 기반 현대화에는 유효하지 않습니다 (Step 6의 일정 미지정 참고사항 확인).

`scc`가 설치되어 있지 않은 경우 다음 순서로 대체합니다:
1. `cloc legacy/$1`을 실행하여 LOC 테이블을 얻은 후, 직접 COCOMO-II 인덱스를 계산합니다: `2.94 × (KSLOC)^1.10` (명목상 규모 인자 기준). 입력값을 보여줍니다.
2. `cloc`마저 없는 경우, LOC를 구하기 위해 확장자별로 그룹화한 `find` + `wc -l`을 사용하고, 결정 키워드(COBOL의 경우 `IF`/`EVALUATE`/`WHEN`/`PERFORM`, C 계열 언어의 경우 `if`/`for`/`while`/`case`/`catch`)를 세어 파일 복잡도 순위를 매깁니다. 위와 동일하게 KSLOC로부터 COCOMO를 계산합니다.

수치가 재현 가능하도록 평가서에 어떤 도구를 사용했는지 기록합니다.

## Step 2 — 기술 핑거프린트

파일 증거를 바탕으로 다음을 식별합니다:
- 사용 중인 언어, 프레임워크 및 런타임 버전
- 빌드 시스템 및 의존성 매니페스트 위치
- 데이터 저장소 (스키마, 카피북, DDL, ORM 설정)
- 통합 지점 (큐, API, 배치 인터페이스, 화면 맵)
- 테스트 존재 여부 및 대략적인 커버리지 신호

## Step 3 — 병렬 심층 분석

3개의 하위 에이전트를 **병렬로** 실행합니다:

1. **legacy-analyst** — "legacy/$1의 구조적 지도를 작성합니다: 5~12개의 주요 기능 도메인(선택 사항이거나 기능 플래그로 제한된 하위 시스템은 하나의 우산 아래로 그룹화), 각 도메인에 속한 소스 파일, 그리고 그들의 상호 의존성(제어 흐름 + 공유 데이터)을 정리합니다. 마크다운 테이블 + 도메인 레벨 의존성의 Mermaid `graph TD`를 반환합니다. 클러스터링을 위해 `subgraph`를 사용하고 엣지는 약 40개 이하로 제한합니다. 저장소 상대 경로 파일 위치를 인용하십시오. 대롱거리는 참조(정의되었으나 소스가 없거나 미사용)를 플래그로 표시합니다."

2. **legacy-analyst** — "legacy/$1에서 기술 부채를 식별합니다: 데드 코드, 더 이상 사용되지 않는 API, 복사-붙여넣기 중복, 신 객체/프로그램(god objects/programs), 예외 처리 누락, 하드코딩된 설정. 수정 가치가 높은 순으로 상위 10개의 발견 항목을 반환하며, 각각 file:line 증거를 포함합니다. 증거에 자격 증명 값이 포함된 경우 비밀 처리 규칙에 따라 마스킹하십시오 — 절대로 그대로 인용하지 마십시오."

3. **security-auditor** — "legacy/$1의 보안 취약점을 스캔합니다: 주입(injection), 인증 취약점, 하드코딩된 비밀 정보, 취약한 의존성, 입력값 검증 누락. 결과를 CWE 태그가 붙은 테이블 형태로 file:line 증거 및 심각도와 함께 반환합니다. 발견된 모든 자격 증명 값은 비밀 처리 규칙에 따라 마스킹하십시오 — file:line 정보 및 2~4글자의 마스킹된 미리보기만 제공하고, 값 자체는 절대로 노출하지 마십시오."

세 에이전트가 모두 완료될 때까지 기다립니다. 발견된 결과를 종합합니다.

## Step 4 — 프로덕션 런타임 오버레이 (선택 사항)

만약 프로덕션 텔레메트리(telemetry) — observability/APM MCP 서버, 배치 작업 로그, 또는 사용자가 제공할 수 있는 런타임 내보내기 데이터 — 가 제공되는 경우, 시스템의 주요 작업/트랜잭션(예: `legacy/$1/jcl/` 아래의 JCL 멤버, 예약된 배치, 주요 API 경로)에 대한 p50/p95/p99 경과 시간(wall-clock)을 수집합니다. 이를 통해 다음을 수행합니다:

- Step 3의 각 기능 도메인에 프로덕션 경과 시간 비용 및 **p99 편차(variance)** (p99/p50 ratio) 태그를 지정합니다.
- 가장 높은 편차를 보이는 도메인을 가장 높은 운영 리스크로 지정합니다 — 이는 정적 분석 의견이 아닌 텔레메트리 기반 분석입니다.

평가서에 작은 **Runtime Profile** 테이블(Job/Route · Domain · p50 · p95 · p99 · p99/p50)을 포함합니다. 제공되는 텔레메트리가 없으면 이 단계를 건너뛰고 평가서에 누락 사실을 기록합니다.

## Step 5 — 문서 갭 분석

코드가 실제로 *동작하는 방식*과 README/문서/주역석이 *설명하는 방식*을 비교합니다. 신입 엔지니어에게 설명이 필요할 만한 것 중 문서화되지 않은 상위 5개의 동작 또는 하위 시스템을 나열합니다.

## Step 6 — 평가서 작성

**비밀 정보 격리를 최우선으로 합니다.** 평가서는 공유 및 커밋되므로, 발견된 자격 증명 값이 절대 포함되어서는 안 됩니다. 만약 security-auditor가 하드코딩된 자증명을 발견한 경우:

1. `analysis/.gitignore`가 존재하고 `SECRETS.local.md` 및 `*.local.patch` 라인을 포함하는지 확인합니다 (필요에 따라 생성하거나 추가합니다. 패치 패턴은 `/modernize-harden`에서 사용되며, 지금 두 가지를 모두 작성해 두면 첫 단계부터 제외 설정이 완료됩니다). 프로젝트가 git 저장소인 경우 `git check-ignore -q analysis/$1/SECRETS.local.md`로 확인합니다 — 검증이 통과할 때까지는 발견 사항을 작성하지 마십시오. 만약 **git 저장소가 아닌 경우** (`.svn`/`.hg`/`CVS`도 확인하십시오 — `.gitignore`는 다른 VCS에서 보호 기능을 하지 못함), `--show-secrets`를 거부하고 `SECRETS.local.md`를 프로젝트 트리가 아닌 `~/.modernize/$1/`에 기록한 뒤 사용자에게 위치와 사유를 안내합니다.
2. `SECRETS.local.md`를 작성합니다: 자격 증명당 하나의 행 — 마스킹된 미리보기, `file:line`, 자격 증명 유형, 접근 권한 부여 대상, 프로덕션/테스트 추정, 로테이션 권장사항. 사용자가 `--show-secrets`를 명시적으로 전달한 경우에만 여기에 원본 값 열을 추가합니다. 이는 이 파일에만 해당하며 ASSESSMENT.md에는 절대로 추가하지 않습니다.
3. 마스킹은 어느 에이전트가 발견 사항을 생성했든 관계없이 **ASSESSMENT.md의 모든 섹션**에 적용됩니다. 기술 부채(Technical Debt) 섹션에서 하드코딩된 설정을 인용할 때도 보안 취약점(Security Findings)과 동일한 마스킹 규칙을 따릅니다. 보안 취약점 섹션에는 다음과 같이 한 줄의 포인터를 추가합니다: "자격 증명 인벤토리는 SECRETS.local.md에 포함되어 있습니다 (gitignored 항목이며, 외부 공유 금지)."

다음 섹션들을 포함하는 `analysis/$1/ASSESSMENT.md`를 생성합니다:
- **Executive Summary** (3-4문장: 어떤 시스템인지, 규모, 위험 수준, 핵심 권장사항)
- **System Inventory** (scc 테이블 및 기술 핑거프린트)
- **Architecture-at-a-Glance** (도메인 테이블. 아키텍처 다이어그램 참조 포함)
- **Production Runtime Profile** (Step 4의 런타임 테이블, 편차가 가장 높은 도메인 강조 표시 또는 "제공되는 텔레메트리 없음")
- **Technical Debt** (순위가 매겨진 상위 10개 항목)
- **Security Findings** (CWE 테이블)
- **Documentation Gaps** (상위 5개 항목)
- **Relative Scale** (COCOMO-II 인덱스 + KSLOC를 복잡도/규모 신호로 사용하여 이 시스템을 다른 시스템과 비교 분석함. **일정이 아님:** 이는 상대적 크기 척도이며 현대화에 얼마나 걸릴지 또는 비용이 얼마나 들지에 대한 예측이 아님을 분명히 명시하십시오. 이는 전통적인 인적 개발 팀의 생산성을 가정한 것이며 에이전트 기반 전환에는 해당하지 않습니다. 인월(person-months), 일정, 비용, 날짜를 표시하지 마십시오.)
- **Recommended Modernization Pattern** (Rehost / Replatform / Refactor / Rearchitect / Rebuild / Replace 중 하나 — 한 단락의 근거 제시 및 다음과 같이 분기되는 실행 명령: **Replatform / Refactor-in-place 동일 스택 버전 업그레이드 → `/modernize-uplift`**, Rearchitect/크로스 스택 → `/modernize-transform`, Rebuild → `/modernize-reimagine`)

또한 legacy-analyst가 작성한 Mermaid 도메인 의존성 다이어그램을 포함하는 `analysis/$1/ARCHITECTURE.mmd`를 생성합니다.

## Step 7 — 결과 제시

사용자에게 평가서가 준비되었음을 알리고 다음 명령을 권장합니다:
`glow -p analysis/$1/ASSESSMENT.md`
