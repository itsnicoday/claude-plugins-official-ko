# 에이전트 트리거링: 모범 사례 (Agent Triggering: Best Practices)

에이전트가 확실하게 디스패치(호출)되도록 트리거 설명을 작성하는 완전한 가이드입니다.

## 트리거 설명이 위치하는 곳

에이전트 파일에는 트리거링을 다루는 두 가지 위치가 있습니다:

1. **YAML frontmatter의 `description:` 필드.** 에이전트가 등록될 때마다 컨텍스트에 로드되며, 하네스(harness)가 언제 디스패치할지 결정하는 데 사용됩니다. 일반 텍스트 문장으로 작성하십시오.
2. **에이전트 본문의 "When to invoke" 섹션.** 에이전트가 실제로 호출될 때만 로드됩니다. 여기에 줄글 설명의 글머리 기호 목록 형식으로 구체적인 시나리오가 들어갑니다.

## 형식

### `description:` 필드

```
description: Use this agent when [conditions]. Typical triggers include [scenario 1 phrased as a prose noun phrase], [scenario 2], and [scenario 3]. See "When to invoke" in the agent body for worked scenarios.
```

규칙:
- YAML 스칼라 내의 단일 행 일반 텍스트 문장.
- 2~4개의 트리거 시나리오를 명사구로 지정.
- 본문의 "When to invoke" 섹션을 가리키는 포인터로 끝맺음.

### 본문의 "When to invoke" 섹션

```markdown
## When to invoke

[Two to four representative scenarios as prose bullets. Each describes the situation
in third person and what the agent should do.]

- **[Short scenario name].** [What the situation looks like — what just happened or what
  the user is asking for — and what the agent should do in response.]
- **[Short scenario name].** [Same.]
```

## 좋은 시나리오의 구조

### 시나리오 이름 (굵은 글씨의 리드)

**목적:** 상황 유형을 식별하는 짧은 명사구.

**좋은 예:**
- *기능이 도입된 후 사용자가 요청하는 리뷰 (User-requested review after a feature lands)*
- *새로 작성된 코드의 자발적 리뷰 (Proactive review of newly-written code)*
- *PR 전 정상성 확인 (Pre-PR sanity check)*
- *새로운 로직으로 업데이트된 PR (PR updated with new logic)*

**나쁜 예:**
- *일반적인 사용 (Normal usage)* (구체적이지 않음)
- *사용자가 도움을 필요로 함 (User needs help)* (모호함)

### 시나리오 본문 (리드 이후 부분)

**목적:** 어떤 일이 일어나고 에이전트가 무엇을 해야 하는지 줄글 및 3인칭으로 설명하며, 대화 인용구를 사용하지 않습니다.

**좋은 예:**
> 사용자가 방금 기능을 구현했고(종종 여러 파일에 걸쳐 있음) 모든 것이 괜찮아 보이는지 묻습니다. 최근 diff에 대한 리뷰를 실행하고 결과를 보고하십시오.

**나쁜 예 (대화 스크립트 형태 — 사용 금지):**
> ```
> user: "Can you check if everything looks good?"
> assistant: "I'll use the reviewer agent..."
> ```

나쁜 버전은 대화 턴(turn) 표시기 형태를 에이전트 파일에 섞어 씁니다. 시나리오는 줄글 형태의 상황 설명으로 유지하십시오.

## 다루어야 할 트리거 유형

다음 영역들을 아우르는 2~4개의 시나리오를 목표로 하십시오:

### 명시적 요청

사용자가 에이전트가 수행하는 작업을 직접 요청합니다.
- *사용자 요청 보안 검사 (User-requested security check).* 사용자가 최근 코드에 대한 보안 리뷰를 명시적으로 요청합니다.

### 자발적 트리거링

관련 작업이 완료된 후, 어시스턴트가 명시적인 요청 없이도 에이전트를 자발적으로 호출합니다.
- *데이터베이스 코드 작성 후 자발적 리뷰 (Proactive review after writing database code).* 어시스턴트가 방금 데이터베이스 액세스 코드를 작성했으며, 작업을 완료된 것으로 선언하기 전에 SQL 인젝션 및 기타 데이터베이스 계층의 리스크를 확인해야 합니다.

### 암시적 요청

사용자가 에이전트의 이름을 직접 부르지 않고 필요성을 암시합니다.
- *코드 명확성에 대한 불만 (Code-clarity complaint).* 사용자가 기존 코드가 혼란스럽거나 따라가기 어렵다고 설명합니다. 이를 가독성을 위한 리팩토링 요청으로 처리하십시오.

### 도구 사용 패턴

에이전트가 특정 도구 사용 패턴을 따라야 합니다.
- *테스트 수정 후 검증 (Post-test-edit verification).* 어시스턴트가 테스트 파일에 여러 수정을 가했습니다. 계속하기 전에 수정된 테스트가 여전히 품질 및 커버리지 표준을 충족하는지 검증하십시오.

## 다양한 표현 방식

동일한 의도가 흔히 여러 방식으로 표현된다면, 줄글로 이를 언급하십시오:

> **PR 전 정상성 확인 (Pre-PR sanity check).** 사용자가 풀 리퀘스트를 열려고 한다는 신호("PR을 열 준비가 됨", "여기서 끝난 것 같음", "배포하자" 등 임의의 표현 방식)를 보냅니다.

말 그대로 표현만 다른 거의 중복되는 시나리오를 여러 개 작성하지 말고, 다양한 표현들을 언급하는 하나의 줄글 시나리오로 합치십시오.

## 시나리오 개수는 몇 개가 적당할까요?

- **최소: 2개.** 보통 명시적 요청 1개 + 자발적 트리거 1개.
- **권장: 3~4개.** 명시적 요청, 자발적 트리거, 그리고 1개의 암시적 요청 또는 예외 사례.
- **최대: 5개.** 그 이상은 라우팅 신호를 추가하지 않으면서 본문만 비대하게 만듭니다.

## 실전 예시

### `description:` 필드의 줄글 트리거

```yaml
description: Use this agent when you need to review code. Typical triggers include user-requested review after a feature lands, proactive review of freshly-written code, and a pre-PR sanity check. See "When to invoke" in the agent body for worked scenarios.
```

### 본문의 상황 설명 시나리오

```markdown
## When to invoke

- **User-requested review.** The user asks for a review of recent changes (any phrasing). Run a review of the unstaged diff.
```

### 트리거 조건만 지정 — 출력 형식은 다른 곳에 지정

```markdown
- **Review.** The user asks for a review. Run the review and report findings as specified in the Output Format section.
```

## 템플릿 라이브러리

### 코드 리뷰 에이전트 (Code review agent)

```yaml
description: Use this agent when you need to review code for adherence to project guidelines and best practices. Typical triggers include the user asking for a review of a feature they just implemented, proactive review of newly-written code before declaring a task done, and a pre-PR sanity check. See "When to invoke" in the agent body.
```

```markdown
## When to invoke

- **User-requested review after a feature lands.** The user has implemented a feature and asks whether the result looks good. Review the recent diff and report findings.
- **Proactive review of newly-written code.** The assistant has just authored new code in response to a user request. Run a self-review before declaring the task done.
- **Pre-PR sanity check.** The user signals readiness to open a pull request. Review the full diff first.
```

### 테스트 생성 에이전트 (Test generation agent)

```yaml
description: Use this agent when you need to generate tests for code that lacks them. Typical triggers include the user explicitly asking for tests for a function or module, and the assistant proactively generating tests after writing new code that has no test coverage. See "When to invoke" in the agent body.
```

```markdown
## When to invoke

- **Explicit test request.** The user asks for tests covering a specific function, module, or feature. Generate a comprehensive test suite.
- **Proactive coverage after new code.** The assistant has just written new code with no accompanying tests. Generate tests before declaring the task done.
```

### 문서화 에이전트 (Documentation agent)

```yaml
description: Use this agent when you need to write or improve documentation for code, especially APIs. Typical triggers include the user asking for docs on a specific function or endpoint, and proactive documentation generation after the assistant adds new API surface. See "When to invoke" in the agent body.
```

```markdown
## When to invoke

- **Explicit doc request.** The user asks for documentation for a specific surface (function, endpoint, module).
- **Proactive docs for new API surface.** The assistant has just added new API endpoints or public functions without docstrings.
```

### 검증 에이전트 (Validation agent)

```yaml
description: Use this agent when you need to validate code before commit or merge. Typical triggers include the user signaling readiness to commit, and an explicit validation request. See "When to invoke" in the agent body.
```

```markdown
## When to invoke

- **Pre-commit validation.** The user signals readiness to commit. Run validation first and surface any issues.
- **Explicit validation request.** The user asks for the code to be validated.
```

## 트리거링 문제 디버깅

### 에이전트가 트리거되지 않음

확인 사항:
1. `description:` 줄글이 올바른 트리거 시나리오를 지명하고 있는지 확인하십시오.
2. 본문의 시나리오가 사용자가 사용하는 실제 표현 방식을 다루고 있는지 확인하십시오.
3. 라우팅 결정에서 더 구체적인 경쟁 에이전트가 우선권을 가져가고 있지 않은지 확인하십시오.

해결책: 본문의 시나리오를 추가하거나 확장하고, `description:`의 줄글 요약을 구체화하십시오.

### 에이전트가 너무 자주 트리거됨

확인 사항:
1. 트리거 시나리오가 너무 일반적이거나 다른 에이전트와 겹치는지 확인하십시오.
2. `description:`에 에이전트를 사용하지 말아야 할 때가 기재되어 있지 않은지 확인하십시오.

해결책: 시나리오 범위를 좁히고, 필요한 경우 `description:`에 "Do not invoke when..."(사용하지 말아야 할 경우...) 행을 추가하십시오.

### 에이전트가 잘못된 시나리오에서 트리거됨

확인 사항:
1. 본문의 시나리오가 에이전트의 실제 기능과 일치하는지 확인하십시오.

해결책: 에이전트가 실제로 수행하는 작업과 일치하도록 시나리오를 다시 작성하십시오.

## 모범 사례 요약

- `description:`은 트리거 시나리오의 짧은 요약을 포함한 단일 행 일반 텍스트로 유지하십시오.
- 상세 시나리오는 본문의 "When to invoke" 섹션에 줄글 글머리 기호 형태로 입력하십시오.
- 명시적 트리거링과 자발적 트리거링을 모두 다루십시오.
- 에이전트가 응답해야 하는 상황을 설명하십시오.
- 거의 중복되는 시나리오를 여러 개 만들기보다는 줄글 속에서 다양한 표현 방식("어떤 표현이든 — 'ready to ship', 'looks done'")을 언급하십시오.
- 트리거 시나리오는 출력 형식과 분리하여 유지하십시오.

## 결론

신뢰할 수 있는 트리거링은 에이전트가 응답해야 하는 상황에 대한 줄글 형식의 설명에서 비롯됩니다.
