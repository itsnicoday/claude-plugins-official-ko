---
description: "전문 에이전트를 사용한 종합 PR 리뷰"
argument-hint: "[review-aspects]"
allowed-tools: ["Bash", "Glob", "Grep", "Read", "Task"]
---

# Comprehensive PR Review

전문 에이전트를 여러 개 실행하여 종합적인 풀 리퀘스트 리뷰를 진행하며, 각 에이전트는 코드 품질의 서로 다른 측면에 초점을 맞춥니다.

**Review Aspects (선택 항목):** "$ARGUMENTS"

## Review Workflow:

1. **Determine Review Scope (리뷰 범위 결정)**
   - git 상태를 확인하여 변경된 파일을 식별합니다.
   - 인수를 파싱하여 사용자가 특정 리뷰 영역을 요청했는지 확인합니다.
   - 기본값: 적용 가능한 모든 리뷰를 실행합니다.

2. **Available Review Aspects (사용 가능한 리뷰 영역):**

   - **comments** - 코드 주석의 정확성 및 유지보수성 분석
   - **tests** - 테스트 커버리지 품질 및 완성도 검토
   - **errors** - 에러 처리에서 사일런트 실패 검사
   - **types** - 타입 설계 및 불변성 분석 (새로운 타입이 추가된 경우)
   - **code** - 프로젝트 가이드라인 준수 여부를 확인하는 일반 코드 리뷰
   - **simplify** - 명확성과 유지보수성을 위해 코드 단순화
   - **all** - 적용 가능한 모든 리뷰 실행 (기본값)

3. **Identify Changed Files (변경된 파일 식별)**
   - `git diff --name-only`를 실행하여 수정된 파일을 확인합니다.
   - PR이 이미 존재하는지 확인합니다: `gh pr view`
   - 파일 유형을 식별하고 어떤 리뷰가 적용되는지 파악합니다.

4. **Determine Applicable Reviews (적용 가능한 리뷰 결정)**

   변경 사항에 기반한 기준:
   - **항상 적용**: code-reviewer (일반 품질 검사)
   - **테스트 파일이 변경된 경우**: pr-test-analyzer
   - **주석/문서가 추가된 경우**: comment-analyzer
   - **에러 처리가 변경된 경우**: silent-failure-hunter
   - **타입이 추가/수정된 경우**: type-design-analyzer
   - **리뷰 통과 후**: code-simplifier (코드 다듬기 및 정제)

5. **Launch Review Agents (리뷰 에이전트 실행)**

   **순차적 접근 방식** (한 번에 하나씩):
   - 이해하고 조치하기가 더 쉽습니다.
   - 다음 에이전트가 시작되기 전에 각 보고서가 완료됩니다.
   - 대화형 리뷰에 유용합니다.

   **병렬적 접근 방식** (사용자가 요청 가능):
   - 모든 에이전트를 동시에 실행합니다.
   - 종합 리뷰를 빠르게 처리할 때 좋습니다.
   - 결과가 동시에 반환됩니다.

6. **Aggregate Results (결과 집계)**

   에이전트 작업 완료 후 다음과 같이 요약합니다:
   - **Critical Issues** (머지 전에 반드시 수정해야 하는 핵심 이슈)
   - **Important Issues** (수정해야 하는 중요 이슈)
   - **Suggestions** (권장 사항)
   - **Positive Observations** (좋은 부분)

7. **Provide Action Plan (실행 계획 제공)**

   발견된 문제점 정리:
   ```markdown
   # PR Review Summary

   ## Critical Issues (X found)
   - [agent-name]: Issue description [file:line]

   ## Important Issues (X found)
   - [agent-name]: Issue description [file:line]

   ## Suggestions (X found)
   - [agent-name]: Suggestion [file:line]

   ## Strengths
   - What's well-done in this PR

   ## Recommended Action
   1. Fix critical issues first
   2. Address important issues
   3. Consider suggestions
   4. Re-run review after fixes
   ```

## Usage Examples:

**전체 리뷰 (기본값):**
```
/pr-review-toolkit:review-pr
```

**특정 측면만 리뷰:**
```
/pr-review-toolkit:review-pr tests errors
# 테스트 커버리지와 에러 처리만 검토

/pr-review-toolkit:review-pr comments
# 코드 주석만 검토

/pr-review-toolkit:review-pr simplify
# 리뷰 통과 후 코드를 단순화함
```

**병렬 리뷰:**
```
/pr-review-toolkit:review-pr all parallel
# 모든 에이전트를 병렬로 실행
```

## Agent Descriptions:

**comment-analyzer**:
- 코드 대비 주석의 정확성 검증
- 주석 노후화 식별
- 문서화 완성도 검사

**pr-test-analyzer**:
- 동작 테스트 커버리지 검토
- 심각한 테스트 격차 식별
- 테스트 품질 평가

**silent-failure-hunter**:
- 사일런트 실패 탐지
- catch 블록 검토
- 에러 로깅 확인

**type-design-analyzer**:
- 타입 캡슐화 분석
- 불변성 표현 검토
- 타입 설계 품질 평가

**code-reviewer**:
- CLAUDE.md 준수 여부 확인
- 버그 및 이슈 감지
- 일반 코드 품질 검토

**code-simplifier**:
- 복잡한 코드 단순화
- 명확성 및 가독성 개선
- 프로젝트 표준 적용
- 기능 보존

## Tips:

- **Run early (일찍 실행하세요)**: PR 생성 후가 아니라 생성하기 전에 실행하십시오.
- **Focus on changes (변경 사항에 집중하세요)**: 에이전트는 기본적으로 git diff를 분석합니다.
- **Address critical first (심각한 문제부터 해결하세요)**: 우선순위가 낮은 이슈보다 높은 이슈를 먼저 해결하십시오.
- **Re-run after fixes (수정 후 재실행하세요)**: 이슈가 해결되었는지 검증하십시오.
- **Use specific reviews (특정 리뷰를 지정해 사용하세요)**: 우려되는 사항이 확실할 때는 특정 영역을 타겟팅하십시오.

## Workflow Integration:

**Before committing:**
```
1. Write code
2. Run: /pr-review-toolkit:review-pr code errors
3. Fix any critical issues
4. Commit
```

**Before creating PR:**
```
1. Stage all changes
2. Run: /pr-review-toolkit:review-pr all
3. Address all critical and important issues
4. Run specific reviews again to verify
5. Create PR
```

**After PR feedback:**
```
1. Make requested changes
2. Run targeted reviews based on feedback
3. Verify issues are resolved
4. Push updates
```

## Notes:

- 에이전트는 자율적으로 실행되며 세부 보고서를 반환합니다.
- 각 에이전트는 심층 분석을 위해 자체 전문 영역에 집중합니다.
- 분석 결과는 구체적인 file:line 참조를 포함하여 즉시 실행 가능합니다.
- 에이전트는 작업 복잡도에 적합한 모델을 사용합니다.
- 모든 에이전트는 `/agents` 목록에서 확인할 수 있습니다.
