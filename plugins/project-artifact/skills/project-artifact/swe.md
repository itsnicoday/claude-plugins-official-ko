# project-artifact — 소프트웨어 (워크스트림 = PR)

워크스트림이 PR인 경우에도 `SKILL.md`의 모든 내용이 그대로 적용됩니다. 기본 템플릿과 실질적으로 다른 유일한 부분은 **X.Y 번호 지정 규칙**입니다. 이 파일의 나머지 내용은 PR 상태를 가져오는 방법, PR별 작성 프래그먼트, 그리고 중량급 프로젝트를 위한 *선택적* 메뉴에 대해 다룹니다.

**PR 번호를 X.Y로 지정합니다.** `X`는 PR이 이전 단계에 의해 차단(block)되는 경우 증가시키고, 한 단계 내에서 병렬로 반영(land)될 수 있는 PR인 경우 `Y`를 사용합니다 (`2.0`은 단계 1의 모든 PR이 머지되어야 함. `1.1`과 `1.2`는 `1.0`과 동시에 진행될 수 있음). 이 번호가 의존성 순서를 나타내므로 별도로 DAG를 그리지 마십시오.

**실시간 상태 수집 — 항상 설정에 지정된 저장소/작성자(author)/브랜치 접두사(branch-prefix)로부터 실시간으로 가져옵니다** (첫 빌드이고 아직 설정이 없는 경우: 현재 디렉터리의 저장소, 현재 `gh` 사용자를 작성자로 사용하고, 프로젝트 브랜치들이 실제로 사용하는 브랜치 접두사를 사용합니다. 이 정보들은 이후 설정 파일에 기록됩니다). 열려 있는 PR은 번호 기준 중복을 제거한, 작성자 쿼리와 브랜치 접두사 쿼리의 합집합입니다 (봇이나 팀원이 프로젝트 브랜치에 생성한 PR도 캡처됨):

```bash
gh pr list --repo <repo> --state open --author <author> \
  --json number,title,url,headRefName,isDraft,mergeable,reviewDecision,reviewRequests --limit 100
gh pr list --repo <repo> --state open --search "head:<prefix>" \
  --json number,title,url,headRefName,isDraft,mergeable,reviewDecision,reviewRequests --limit 100
```

최근 머지된 항목들(`--state merged --json number,title,url,mergedAt --limit 40`)은 완료(done) 행을 구성합니다. 완전히 머지된 단계는 개별적으로 다 나열하지 않고 하나의 요약 행("N개의 PR, 모두 머지됨")으로 축소(collapse)합니다. 행을 차지할 만한 열린 PR당 다음을 포함합니다:

- **CI**: `gh pr checks <n> --repo <repo> --required`는 통과 관문(gating) 상태입니다. 단순 자문용 봇 실패는 차단 요소가 아니므로, 조치가 필요한 경우에만 언급하십시오.
- **해결되지 않은 리뷰 스레드 (Unresolved review threads)**: GraphQL만 사용 — REST 방식은 해결된 스레드에도 여전히 상위 댓글이 남아 있어 잘못 계산하기 때문입니다. `repository.pullRequest.reviewThreads(first:100){nodes{isResolved}}`에서 `isResolved: false`의 개수를 셉니다.
- 아래에 PR별 세부 내용을 작성하는 PR의 경우: 반영 내용/검증 설명을 위해 `gh pr view <n> --json body`를 사용하고, 커밋 테이블을 표시할 경우 `git log --oneline <base>..<branch>`를 사용합니다.

프로젝트의 브랜치 / PR 제목 컨벤션 (예: 브랜치 `<user>/abc-12-...` 또는 제목의 `(ABC-12)`) 및 트래커의 마일스톤을 통해 **PR을 워크스트림에 매핑**합니다. 확실히 매칭되지 않는 PR은 임의의 워크스트림에 넣지 말고 그 판단 근거와 함께 미분류 행(catch-all row)에 넣으십시오.

설계 문서 / 사양서: 문서를 대체하지 말고 요약 및 링크를 연결하십시오. 그것이 `claude.ai/code/artifact/...` 페이지인 경우 WebFetch를 사용합니다 (`SKILL.md` "기존 아티팩트 페이지 읽기" 참조). 기능 플래그(build flag)가 적용되는 변경사항인 경우: 저장소의 기능 플래그 시스템에서 찾아 상태 배너에 기입합니다.

**상태 블록 필드** (`SKILL.md` "아티팩트 새로 고침"의 `artifact-state` JSON): PR 기반 프로젝트의 경우, `workstreams` 배열은 PR당 하나의 엔트리를 가지며 `{"repo", "number", "workstream", "draft", "ci", "unresolved", "state"}` 형태를 띱니다. 이는 다음 새로 고침 시 이전 텍스트를 다시 읽지 않고도 머지됨 / 신규 / CI 상태 전환 / 리뷰 스레드 변화를 감지하여 보고하기에 충분한 정보입니다. 다음 렌더링 결과와의 비교(diff)가 깔끔하게 수행되도록 이 키들을 정확히 유지하십시오. 브랜치 이름이나 PR 제목에서 유도된 값들은 신뢰할 수 없는 마크업입니다. JSON 내부에서 `<`를 `\u003c`로 작성하고 화면에 표시되는 셀에서는 엔티티 인코딩을 수행하십시오 (`SKILL.md` "신선도와 신뢰성" 참조).

**PR별 세부 내용 작성:** PR이 워크스트림 테이블의 한 행 이상의 가치가 있는 경우, 테이블 아래에 이 내용을 붙여넣습니다 (템플릿 CSS에 `.pill.*` 클래스가 정의되어 있습니다. 여기서 사용되는 필은 다음과 같습니다: `in review` = `now`, `merged`/`tested ✓`/`verified ✓` = `done`):

```html
<hr>
<h2>PR 1.0 — <a href="#">#NNNNN</a> · short title <span class="pill now">in review</span></h2>
<h3>반영 내용</h3>
<table><tr><th style="width:140px">영역</th><th></th></tr><tr><td>CLI</td><td>...</td></tr></table>
<h3>검증</h3>
<p>이 PR이 어떻게 검증되었는지 작성합니다 — 테스트, 예외/공격 시나리오 워크플로우, 실제 빌드 대상 수동 실행, 통과 관문(gating) 검사 등.</p>
<details><summary>확인된 버그 발견 사항 (이 PR에서 수정됨)</summary>
<table><tr><th>#</th><th>버그</th><th>수정안</th></tr><tr><td>1</td><td>...</td><td>...</td></tr></table></details>
<h3>커밋</h3>
<p class="meta">위에서 아래로: 기능 개발(feat) → 안정화 단계(hardening) → 다듬기(polish) → 필수 검사(gating) → 린트(lint).</p>
<table><tr><th style="width:110px">SHA</th><th></th></tr><tr><td><code>abc1234567</code></td><td><b>feat(...):</b> ...</td></tr></table>
<h3>파일</h3>
<pre><code>path/to/file.go   — 파일의 역할</code></pre>
```

(제안 단계이며 열려 있는 PR이 없습니까? Workstreams 탭은 `next` 필과 함께 *계획된* X.Y 시퀀스를 보여주며, PR별 세부 정보 부분은 SHA를 임의로 지어내지 않고 "아직 커밋이 없습니다 — 브랜치가 생성되면 채워집니다"라고 표기합니다.)

**중량급 프로젝트를 위한 선택사항 — 불필요한 것은 생략하십시오.** 엄격한 불변조건(invariants)이 있는 마이그레이션의 경우 "성공 기준(Success criteria)"을 "요구사항(Requirements)"으로 변경하고, 필수 항목(must-have)과 권장 사항(nice-to-have)을 분리하며, 각각에 대해 검증 가능한 검사 방법(정적 검사: "이 diff가 비어 있음", 동적 검사: "플래그를 켠 상태로 X를 실행하고 Y가 변화 없이 유지되는지 관찰")을 부여할 수 있습니다. 또한 **Architecture(아키텍처)** 탭(프로토콜, 토폴로지, 파일 단위 구성, 명시적으로 구분된 신뢰 경계(trust boundaries)), **Findings & fixes(발견 사항 및 수정안)** 탭(검토/예외 시나리오 검증 결과인 `번호 · 버그 · 수정안`, 이전 주기의 버그는 `<details>` 내에 표기), 그리고 **Rollout & rollback(배포 및 롤백)** 탭(점진적 배포 단계, 지표 및 대역폭 임계치, 롤백 절차, "50% 배포 중 오류 발생 시" 매뉴얼, "완료" 기준 정의)을 추가할 수 있습니다. 이 중 어느 것도 필수가 아닙니다 — 소프트웨어에도 동일하게 적용되는 "실제 내용이 있는 경우에만 탭 추가" 규칙입니다. 본문 전반에 걸쳐 PR 설명 수준의 쉬운 일상 언어로 설명하십시오.
