---
name: playground
description: 대화형 HTML 플레이그라운드를 생성합니다. 이는 사용자가 컨트롤을 통해 시각적으로 무언가를 설정하고 실시간 미리보기를 확인하며 프롬프트를 복사할 수 있는 독립형 단일 파일 탐색기입니다. 사용자가 특정 주제에 대한 플레이그라운드, 탐색기 또는 대화형 도구 제작을 요청할 때 사용하십시오.
---

# 플레이그라운드 빌더 (Playground Builder)

플레이그라운드는 한쪽에 대화형 컨트롤, 다른 쪽에 실시간 미리보기, 그리고 복사 버튼이 있는 프롬프트 출력이 하단에 배치된 독립형 HTML 파일입니다. 사용자는 컨트롤을 조정하고 시각적으로 탐색한 뒤, 생성된 프롬프트를 복사하여 Claude에 다시 입력합니다.

## 이 스킬을 사용하는 시점

사용자가 특정 주제에 대한 대화형 플레이그라운드, 탐색기 또는 시각적 도구를 요청할 때 사용하십시오. 특히 입력 공간이 방대하고 시각적이거나 구조적이어서 단순 텍스트로 표현하기 어려운 경우에 적합합니다.

## 이 스킬을 사용하는 방법

1. 사용자의 요청에서 **플레이그라운드 유형**을 식별합니다.
2. `templates/`에서 일치하는 템플릿을 로드합니다:
   - `templates/design-playground.md` — 시각적 디자인 결정 (컴포넌트, 레이아웃, 여백, 색상, 타이포그래피)
   - `templates/data-explorer.md` — 데이터 및 쿼리 빌딩 (SQL, API, 파이프라인, 정규표현식)
   - `templates/concept-map.md` — 학습 및 탐색 (개념 맵, 지식 격차, 범위 매핑)
   - `templates/document-critique.md` — 문서 검토 (승인/거부/의견 작성 워크플로우를 포함한 제안 사항)
   - `templates/diff-review.md` — 코드 리뷰 (라인별 댓글 작성이 포함된 git diff, 커밋, PR)
   - `templates/code-map.md` — 코드베이스 아키텍처 (컴포넌트 관계, 데이터 흐름, 계층 다이어그램)
3. **템플릿을 따라** 플레이그라운드를 빌드합니다. 해당 주제가 어떠한 템플릿에도 딱 맞지 않는 경우, 가장 유사한 것을 골라 조정하여 사용하십시오.
4. **브라우저에서 열기.** HTML 파일을 작성한 후, `open <filename>.html`을 실행하여 사용자의 기본 브라우저에서 실행하십시오.

## 핵심 요구 사항 (모든 플레이그라운드 공통)

- **단일 HTML 파일.** 모든 CSS와 JS를 파일 내에 인라인으로 포함시킵니다. 외부 종속성은 허용되지 않습니다.
- **실시간 미리보기.** 컨트롤이 변경될 때마다 즉시 업데이트되어야 합니다. "적용(Apply)" 버튼은 필요 없습니다.
- **프롬프트 출력.** 단순 값 나열이 아닌 자연어 형태여야 합니다. 기본값이 아닌 선택 항목만 기재하십시오. 플레이그라운드를 보지 않고도 조치를 취할 수 있을 만큼 충분한 컨텍스트를 제공하며, 실시간으로 업데이트되어야 합니다.
- **복사 버튼.** 클립보드 복사 기능 및 "복사됨(Copied!)"이라는 짤막한 피드백을 제공합니다.
- **적절한 기본값 + 프리셋.** 처음 로드될 때부터 보기 좋아야 합니다. 모든 컨트롤을 유기적인 조합으로 한 번에 변경할 수 있는 이름 있는 프리셋 3~5개를 포함하십시오.
- **다크 테마.** UI에는 시스템 글꼴을, 코드나 값에는 고정폭(monospace) 글꼴을 사용합니다. 프레임워크 요소(chrome)는 최소화합니다.

## 상태 관리 패턴

단일 상태(state) 객체를 유지하십시오. 모든 컨트롤은 상태를 수정하고, 모든 렌더러는 상태를 조회합니다.

```javascript
const state = { /* all configurable values */ };

function updateAll() {
  renderPreview(); // update the visual
  updatePrompt();  // rebuild the prompt text
}
// Every control calls updateAll() on change
```

## 프롬프트 출력 패턴

```javascript
function updatePrompt() {
  const parts = [];

  // Only mention non-default values
  if (state.borderRadius !== DEFAULTS.borderRadius) {
    parts.push(`border-radius of ${state.borderRadius}px`);
  }

  // Use qualitative language alongside numbers
  if (state.shadowBlur > 16) parts.push('a pronounced shadow');
  else if (state.shadowBlur > 0) parts.push('a subtle shadow');

  prompt.textContent = `Update the card to use ${parts.join(', ')}.`;
}
```

## 피해야 할 흔한 실수

- 프롬프트 출력이 단순한 값 나열인 경우 → 자연스러운 명령문 형태로 작성하십시오.
- 너무 많은 컨트롤이 한꺼번에 표시되는 경우 → 관심사별로 그룹화하고, 고급 설정은 접을 수 있는 섹션에 숨기십시오.
- 미리보기가 즉각 업데이트되지 않는 경우 → 모든 컨트롤 변경 시 반드시 즉각적인 재렌더링이 발생해야 합니다.
- 기본값이나 프리셋이 없는 경우 → 로드 시 빈 화면이 나타나거나 화면이 깨질 수 있습니다.
- 외부 종속성이 있는 경우 → CDN이 다운되면 플레이그라운드가 정상 작동하지 않습니다.
- 프롬프트에 컨텍스트가 부족한 경우 → 플레이그라운드 없이도 조치를 취할 수 있도록 충분한 정보를 포함하십시오.
