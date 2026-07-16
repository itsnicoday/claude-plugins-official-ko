---
description: 현대화 워크플로우 상의 현재 위치 — 아티팩트 인벤토리, 노후화(staleness), 비밀 정보 위생(secrets hygiene), 다음 단계
argument-hint: <system-dir>
---

`$1`의 현대화 진행 상황을 한 화면으로 보고합니다. 이 명령은 읽기 전용이며, 조회만 하고 수정하지는 않습니다.

## 1 — 아티팩트 인벤토리

`analysis/$1/` 및 `modernized/$1*/`을 확인하고 테이블을 구성합니다 — 각 행은 워크플로우 단계를 나타내며 아티팩트의 존재 여부 및 수정 시간을 포함합니다:

| Stage | Artifacts |
|---|---|
| preflight | `PREFLIGHT.md` (Check 0 사람 답변과 Check 6 범위-경계 검색 결과가 있는지 기록) |
| assess | `ASSESSMENT.md`, `ARCHITECTURE.mmd` |
| map | `topology.json`, `TOPOLOGY.html`, `*.mmd`, `extract_topology.*` |
| extract-rules | `BUSINESS_RULES.md`, `DATA_OBJECTS.md` |
| brief | `MODERNIZATION_BRIEF.md` (승인 블록 서명 여부 기록) |
| harden | `SECURITY_FINDINGS.md`, `security_remediation.patch` |
| uplift | `DELTA_CATALOG.md`, `BASELINE.md`, `PLAYBOOK.md` (플레이북이 없음 = 파일럿이 아직 진행되지 않았음 — 팬아웃을 수행해서는 안 됨); `modernized/$1-uplifted/UPLIFT_NOTES.md` (단위별 기록: 대상 환경에서 빌드되는지? 베이스라인이 재현되었는지?) |
| transform | 각 `modernized/$1/<module>/` 디렉터리 — 테스트 존재 여부 및 `TRANSFORMATION_NOTES.md` 존재 여부 기록 |
| reimagine | `modernized/$1-reimagined/` — 서비스별 인수 테스트 및 `CLAUDE.md` 핸드오프 기록 (reimagine의 완료 표식이며, `TRANSFORMATION_NOTES.md`를 작성하지 **않음**) |

## 2 — 노후화(Staleness)

파생되어 나온 업스트림 아티팩트보다 오래된 아티팩트를 감지하여 플래그를 지정합니다:

- `MODERNIZATION_BRIEF.md`가 `ASSESSMENT.md`, `topology.json` 또는 `BUSINESS_RULES.md`보다 오래됨 → 브리프가 최신 탐색 결과를 반영하지 못하고 있음. `/modernize-brief` 재실행 권장.
- `MODERNIZATION_BRIEF.md`가 동일 스택 **Uplift** 계획에 대한 `DELTA_CATALOG.md`보다 오래되었거나 카탈로그가 전혀 없음 → 단계를 결정하는 버전 델타 이전에 (또는 버전 델타 없이) 단계 순서가 결정됨. `/modernize-brief` 재실행 권장.
- `TOPOLOGY.html`이 `topology.json`보다 오래됨 → `/modernize-map`에서 주입(injection) 단계를 재실행.
- 임의의 `TRANSFORMATION_NOTES.md`가 `BUSINESS_RULES.md`보다 오래됨 → 모듈이 최신 규칙 세트를 구현하지 않았을 수 있음. 해당하는 모듈 목록 표시.

## 3 — 비밀 정보 위생(Secrets hygiene)

- `analysis/.gitignore`가 존재하고 `SECRETS.local.md` / `*.local.patch`를 제외하는지 여부 (`git check-ignore` 명령어로 git 저장소 확인.)
- `SECRETS.local.md`가 존재하는 경우: 해당 파일이 추적되지 않으며(`git ls-files --error-unmatch` 실패 기대) 커밋된 적이 없는지(`git log --all --oneline -- <path>` 비어있음 기대) 확인합니다. 둘 중 하나라도 검증에 실패하면 두드러지게 표시하고 키 로테이션 및 히스토리 스크러빙(scrubbing)을 권장합니다.

## 4 — 종합 진단(Verdict)

마지막 세 줄로 구성:
- **Where you are** — 가장 최근에 완료된 단계와 대략적인 적용 범위 (예: "지도화 100% 완료, 14개 모듈 중 2개 변환 완료").
- **What's stale** — 노후된 파일 목록 또는 "없음(nothing)".
- **Next command** — 다음으로 실행해야 할 가장 유용한 단일 단계와 그 한 줄의 사유.
