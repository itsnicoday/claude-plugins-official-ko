# PR Review Toolkit

코드 주석, 테스트 커버리지, 에러 처리, 타입 설계, 코드 품질, 코드 단순화 등 철저한 풀 리퀘스트(pull request) 리뷰를 위한 전문 에이전트의 종합 컬렉션입니다.

## Overview (개요)

이 플러그인은 코드 품질의 특정 측면에 각각 집중하는 6개의 전문 리뷰 에이전트를 번들로 제공합니다. 각 에이전트를 개별적으로 사용하여 대상 리뷰를 진행하거나, 함께 사용하여 종합적인 PR 분석을 수행할 수 있습니다.

## Agents (에이전트 목록)

### 1. comment-analyzer
**초점**: 코드 주석의 정확성 및 유지보수성

**분석 대상:**
- 실제 코드 대비 주석의 정확성
- 문서화 완성도
- 주석의 노후화 및 기술 부채
- 잘못된 정보나 오래된 주석

**사용 시기:**
- 문서를 추가한 후
- 주석 변경사항이 포함된 PR을 완료하기 전
- 기존 주석을 검토할 때

**트리거 문구:**
```
"Check if the comments are accurate"
"Review the documentation I added"
"Analyze comments for technical debt"
```

### 2. pr-test-analyzer
**초점**: 테스트 커버리지 품질 및 완성도

**분석 대상:**
- 동작(Behavioral) 커버리지 vs 라인(Line) 커버리지
- 테스트 커버리지의 심각한 격차
- 테스트 품질 및 복원력
- 에지 케이스(Edge cases) 및 에러 조건

**사용 시기:**
- PR을 생성한 후
- 새로운 기능을 추가할 때
- 테스트의 철저함을 검증하고자 할 때

**트리거 문구:**
```
"Check if the tests are thorough"
"Review test coverage for this PR"
"Are there any critical test gaps?"
```

### 3. silent-failure-hunter
**초점**: 에러 처리 및 사일런트 실패(Silent failures)

**분석 대상:**
- catch 블록의 사일런트 실패
- 불충분한 에러 처리
- 부적절한 대체(Fallback) 동작
- 누락된 에러 로깅

**사용 시기:**
- 에러 처리를 구현한 후
- try/catch 블록을 검토할 때
- 에러 처리가 포함된 PR을 완료하기 전

**트리거 문구:**
```
"Review the error handling"
"Check for silent failures"
"Analyze catch blocks in this PR"
```

### 4. type-design-analyzer
**초점**: 타입 설계 품질 및 불변성(Invariants)

**분석 대상:**
- 타입 캡슐화 (1-10점 점수 부여)
- 불변성 표현 (1-10점 점수 부여)
- 타입 유용성 (1-10점 점수 부여)
- 불변성 강제 (1-10점 점수 부여)

**사용 시기:**
- 새로운 타입을 도입할 때
- 데이터 모델이 포함된 PR 생성 시
- 타입 설계를 리팩토링할 때

**트리거 문구:**
```
"Review the UserAccount type design"
"Analyze type design in this PR"
"Check if this type has strong invariants"
```

### 5. code-reviewer
**초점**: 프로젝트 가이드라인 준수 여부를 확인하는 일반 코드 리뷰

**분석 대상:**
- CLAUDE.md 준수 여부
- 스타일 위반 사항
- 버그 감지
- 코드 품질 이슈

**사용 시기:**
- 코드를 작성하거나 수정한 후
- 변경사항을 커밋하기 전
- 풀 리퀘스트를 생성하기 전

**트리거 문구:**
```
"Review my recent changes"
"Check if everything looks good"
"Review this code before I commit"
```

### 6. code-simplifier
**초점**: 코드 단순화 및 리팩토링

**분석 대상:**
- 코드의 명확성 및 가독성
- 불필요한 복잡성 및 중첩(Nesting)
- 중복 코드 및 추상화
- 프로젝트 표준과의 일관성
- 지나치게 간결하거나 까다로운 코드

**사용 시기:**
- 코드를 작성하거나 수정한 후
- 코드 리뷰를 통과한 후
- 코드는 작동하지만 복잡하게 느껴질 때

**트리거 문구:**
```
"Simplify this code"
"Make this clearer"
"Refine this implementation"
```

**참고**: 이 에이전트는 코드 구조와 유지보수성을 개선하면서 기능은 그대로 보존합니다.

## Usage Patterns (사용 패턴)

### 개별 에이전트 사용

에이전트의 초점 영역에 맞는 질문을 하면 Claude가 적절한 에이전트를 자동으로 트리거합니다:

```
"Can you check if the tests cover all edge cases?"
→ pr-test-analyzer 트리거

"Review the error handling in the API client"
→ silent-failure-hunter 트리거

"I've added documentation - is it accurate?"
→ comment-analyzer 트리거
```

### 종합 PR 리뷰

철저한 PR 리뷰를 원한다면 여러 측면을 함께 요청하세요:

```
"I'm ready to create this PR. Please:
1. Review test coverage
2. Check for silent failures
3. Verify code comments are accurate
4. Review any new types
5. General code review"
```

이렇게 하면 관련된 모든 에이전트가 작동하여 PR의 다양한 측면을 분석합니다.

### 선제적 리뷰 (Proactive Review)

Claude는 콘텍스트에 따라 다음 에이전트들을 선제적으로 활용할 수 있습니다:

- **코드 작성 후** → code-reviewer
- **문서 추가 후** → comment-analyzer
- **PR 생성 전** → 상황에 맞는 여러 에이전트
- **타입 추가 후** → type-design-analyzer

## Installation (설치 방법)

개인 마켓플레이스에서 설치할 수 있습니다:

```bash
/plugins
# "pr-review-toolkit" 찾기
# Install 선택
```

또는 필요한 경우 설정 파일에 수동으로 추가하세요.

## Agent Details (에이전트 세부 정보)

### 신뢰도 점수 (Confidence Scoring)

에이전트는 분석 결과에 대해 신뢰도 점수를 제공합니다:

**comment-analyzer**: 정확성 검사에서 높은 신뢰도로 이슈를 식별합니다.

**pr-test-analyzer**: 테스트 격차를 1-10점으로 평가합니다 (10 = 심각, 반드시 추가해야 함).

**silent-failure-hunter**: 에러 처리 이슈의 심각도를 표시합니다.

**type-design-analyzer**: 4가지 차원을 1-10점 척도로 평가합니다.

**code-reviewer**: 이슈에 0-100점의 점수를 부여합니다 (91-100 = 심각).

**code-simplifier**: 복잡성을 식별하고 단순화 방안을 제시합니다.

### 출력 형식 (Output Formats)

모든 에이전트는 다음과 같이 구조화되고 즉시 실행 가능한 분석 결과를 제공합니다:
- 명확한 이슈 식별
- 구체적인 파일 및 라인 참조
- 문제가 되는 이유에 대한 설명
- 개선을 위한 제안
- 심각도에 따른 우선순위 지정

## Best Practices (권장 문서)

### 각 에이전트의 권장 사용 시기

**커밋하기 전:**
- code-reviewer (일반 품질 검사)
- silent-failure-hunter (에러 처리가 변경된 경우)

**PR 생성 전:**
- pr-test-analyzer (테스트 커버리지 검사)
- comment-analyzer (주석을 추가하거나 수정한 경우)
- type-design-analyzer (타입을 추가하거나 수정한 경우)
- code-reviewer (최종 점검)

**리뷰 통과 후:**
- code-simplifier (명확성 및 유지보수성 향상)

**PR 리뷰 진행 중:**
- 제기된 특정 우려 사항에 맞는 에이전트 실행
- 수정 후 특정 부분만 재리뷰 수행

### 여러 에이전트 실행하기

여러 에이전트를 병렬 또는 순차적으로 실행하도록 요청할 수 있습니다:

**병렬 실행** (더 빠름):
```
"Run pr-test-analyzer and comment-analyzer in parallel"
```

**순차 실행** (한 결과가 다른 결과에 영향을 미치는 경우):
```
"First review test coverage, then check code quality"
```

## Tips (팁)

- **구체적으로 요청하세요**: 집중적인 리뷰를 위해 특정 에이전트를 지정하세요.
- **선제적으로 사용하세요**: PR을 생성한 후가 아니라 생성하기 전에 실행하세요.
- **심각한 이슈부터 해결하세요**: 에이전트는 발견한 이슈의 우선순위를 정해 줍니다.
- **반복 검증하세요**: 수정 사항을 반영한 후 다시 실행하여 검증하세요.
- **과도하게 사용하지 마세요**: 전체 코드베이스가 아닌 변경된 코드 위주로 집중하세요.

## Troubleshooting (문제 해결)

### 에이전트가 작동하지 않는 경우

**증상**: 리뷰를 요청했으나 에이전트가 실행되지 않음

**해결책**:
- 요청을 더 구체적으로 작성하세요.
- 에이전트 유형을 명시적으로 언급하세요.
- 구체적인 우려 사항(예: "test coverage")을 언급하세요.

### 에이전트가 잘못된 파일을 분석하는 경우

**증상**: 에이전트가 너무 많은 파일을 분석하거나 엉뚱한 파일을 리뷰함

**해결책**:
- 집중해야 할 파일을 명시적으로 지정하세요.
- PR 번호나 브랜치를 참조하세요.
- "recent changes" 또는 "git diff"를 언급하세요.

## Integration with Workflow (워크플로우 통합)

이 플러그인은 다음 플러그인과 함께 사용하면 좋습니다:
- **build-validator**: 리뷰 전에 빌드/테스트를 실행합니다.
- **프로젝트 전용 에이전트**: 커스텀 에이전트와 결합하여 사용합니다.

**추천 워크플로우:**
1. 코드 작성 → **code-reviewer**
2. 이슈 수정 → **silent-failure-hunter** (에러 처리인 경우)
3. 테스트 추가 → **pr-test-analyzer**
4. 문서화 → **comment-analyzer**
5. 리뷰 통과 → **code-simplifier** (다듬기)
6. PR 생성

## Contributing (기여하기)

이슈를 발견했거나 제안 사항이 있으신가요? 이 에이전트들은 다음 경로에서 관리됩니다:
- 사용자 에이전트: `~/.claude/agents/`
- 프로젝트 에이전트: claude-cli-internal 내의 `.claude/agents/`

## License (라이선스)

MIT

## Author (작성자)

Daisy (daisy@anthropic.com)

---

**빠른 시작**: 리뷰를 요청하기만 하면 적합한 에이전트가 자동으로 트리거됩니다!
