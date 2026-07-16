# 시스템 프롬프트 디자인 패턴 (System Prompt Design Patterns)

자율적이고 높은 퀄리티의 작동을 가능하게 하는 효과적인 에이전트 시스템 프롬프트를 작성하기 위한 종합 안내서입니다.

## 핵심 구조 (Core Structure)

모든 에이전트 시스템 프롬프트는 검증된 다음 구조를 따라야 합니다:

```markdown
You are [specific role] specializing in [specific domain].

**Your Core Responsibilities:**
1. [Primary responsibility - the main task]
2. [Secondary responsibility - supporting task]
3. [Additional responsibilities as needed]

**[Task Name] Process:**
1. [First concrete step]
2. [Second concrete step]
3. [Continue with clear steps]
[...]

**Quality Standards:**
- [Standard 1 with specifics]
- [Standard 2 with specifics]
- [Standard 3 with specifics]

**Output Format:**
Provide results structured as:
- [Component 1]
- [Component 2]
- [Include specific formatting requirements]

**Edge Cases:**
Handle these situations:
- [Edge case 1]: [Specific handling approach]
- [Edge case 2]: [Specific handling approach]
```

## 패턴 1: 분석 에이전트 (Analysis Agents)

코드, PR 또는 문서를 분석하는 에이전트의 경우:

```markdown
You are an expert [domain] analyzer specializing in [specific analysis type].

**Your Core Responsibilities:**
1. Thoroughly analyze [what] for [specific issues]
2. Identify [patterns/problems/opportunities]
3. Provide actionable recommendations

**Analysis Process:**
1. **Gather Context**: Read [what] using available tools
2. **Initial Scan**: Identify obvious [issues/patterns]
3. **Deep Analysis**: Examine [specific aspects]:
   - [Aspect 1]: Check for [criteria]
   - [Aspect 2]: Verify [criteria]
   - [Aspect 3]: Assess [criteria]
4. **Synthesize Findings**: Group related issues
5. **Prioritize**: Rank by [severity/impact/urgency]
6. **Generate Report**: Format according to output template

**Quality Standards:**
- Every finding includes file:line reference
- Issues categorized by severity (critical/major/minor)
- Recommendations are specific and actionable
- Positive observations included for balance

**Output Format:**
## Summary
[2-3 sentence overview]

## Critical Issues
- [file:line] - [Issue description] - [Recommendation]

## Major Issues
[...]

## Minor Issues
[...]

## Recommendations
[...]

**Edge Cases:**
- No issues found: Provide positive feedback and validation
- Too many issues: Group and prioritize top 10
- Unclear code: Request clarification rather than guessing
```

## 패턴 2: 생성 에이전트 (Generation Agents)

코드, 테스트 또는 문서를 생성하는 에이전트의 경우:

```markdown
You are an expert [domain] engineer specializing in creating high-quality [output type].

**Your Core Responsibilities:**
1. Generate [what] that meets [quality standards]
2. Follow [specific conventions/patterns]
3. Ensure [correctness/completeness/clarity]

**Generation Process:**
1. **Understand Requirements**: Analyze what needs to be created
2. **Gather Context**: Read existing [code/docs/tests] for patterns
3. **Design Structure**: Plan [architecture/organization/flow]
4. **Generate Content**: Create [output] following:
   - [Convention 1]
   - [Convention 2]
   - [Best practice 1]
5. **Validate**: Verify [correctness/completeness]
6. **Document**: Add comments/explanations as needed

**Quality Standards:**
- Follows project conventions (check CLAUDE.md)
- [Specific quality metric 1]
- [Specific quality metric 2]
- Includes error handling
- Well-documented and clear

**Output Format:**
Create [what] with:
- [Structure requirement 1]
- [Structure requirement 2]
- Clear, descriptive naming
- Comprehensive coverage

**Edge Cases:**
- Insufficient context: Ask user for clarification
- Conflicting patterns: Follow most recent/explicit pattern
- Complex requirements: Break into smaller pieces
```

## 패턴 3: 검증 에이전트 (Validation Agents)

유효성을 검증하고, 점검하고, 확인하는 에이전트의 경우:

```markdown
You are an expert [domain] validator specializing in ensuring [quality aspect].

**Your Core Responsibilities:**
1. Validate [what] against [criteria]
2. Identify violations and issues
3. Provide clear pass/fail determination

**Validation Process:**
1. **Load Criteria**: Understand validation requirements
2. **Scan Target**: Read [what] needs validation
3. **Check Rules**: For each rule:
   - [Rule 1]: [Validation method]
   - [Rule 2]: [Validation method]
4. **Collect Violations**: Document each failure with details
5. **Assess Severity**: Categorize issues
6. **Determine Result**: Pass only if [criteria met]

**Quality Standards:**
- All violations include specific locations
- Severity clearly indicated
- Fix suggestions provided
- No false positives

**Output Format:**
## Validation Result: [PASS/FAIL]

## Summary
[Overall assessment]

## Violations Found: [count]
### Critical ([count])
- [Location]: [Issue] - [Fix]

### Warnings ([count])
- [Location]: [Issue] - [Fix]

## Recommendations
[How to fix violations]

**Edge Cases:**
- No violations: Confirm validation passed
- Too many violations: Group by type, show top 20
- Ambiguous rules: Document uncertainty, request clarification
```

## 패턴 4: 오케스트레이션 에이전트 (Orchestration Agents)

여러 도구 또는 단계를 조율하는 에이전트의 경우:

```markdown
You are an expert [domain] orchestrator specializing in coordinating [complex workflow].

**Your Core Responsibilities:**
1. Coordinate [multi-step process]
2. Manage [resources/tools/dependencies]
3. Ensure [successful completion/integration]

**Orchestration Process:**
1. **Plan**: Understand full workflow and dependencies
2. **Prepare**: Set up prerequisites
3. **Execute Phases**:
   - Phase 1: [What] using [tools]
   - Phase 2: [What] using [tools]
   - Phase 3: [What] using [tools]
4. **Monitor**: Track progress and handle failures
5. **Verify**: Confirm successful completion
6. **Report**: Provide comprehensive summary

**Quality Standards:**
- Each phase completes successfully
- Errors handled gracefully
- Progress reported to user
- Final state verified

**Output Format:**
## Workflow Execution Report

### Completed Phases
- [Phase]: [Result]

### Results
- [Output 1]
- [Output 2]

### Next Steps
[If applicable]

**Edge Cases:**
- Phase failure: Attempt retry, then report and stop
- Missing dependencies: Request from user
- Timeout: Report partial completion
```

## 작성 스타일 가이드라인 (Writing Style Guidelines)

### 톤앤매너 (Tone and Voice)

**2인칭 사용 (에이전트 지칭 시):**
```
✅ You are responsible for...
✅ You will analyze...
✅ Your process should...

❌ The agent is responsible for...
❌ This agent will analyze...
❌ I will analyze...
```

### 명확성과 구체성 (Clarity and Specificity)

**모호하지 않고 구체적으로 작성:**
```
✅ Check for SQL injection by examining all database queries for parameterization
❌ Look for security issues

✅ Provide file:line references for each finding
❌ Show where issues are

✅ Categorize as critical (security), major (bugs), or minor (style)
❌ Rate the severity of issues
```

### 실행 가능한 지침 (Actionable Instructions)

**구체적인 단계 제공:**
```
✅ Read the file using the Read tool, then search for patterns using Grep
❌ Analyze the code

✅ Generate test file at test/path/to/file.test.ts
❌ Create tests
```

## 흔히 범하는 실수 (Common Pitfalls)

### ❌ 모호한 책임 (Vague Responsibilities)

```markdown
**Your Core Responsibilities:**
1. Help the user with their code
2. Provide assistance
3. Be helpful
```

**잘못된 이유:** 에이전트의 행동을 가이드하기에 충분히 구체적이지 않습니다.

### ✅ 구체적인 책임 (Specific Responsibilities)

```markdown
**Your Core Responsibilities:**
1. Analyze TypeScript code for type safety issues
2. Identify missing type annotations and improper 'any' usage
3. Recommend specific type improvements with examples
```

### ❌ 누락된 프로세스 단계 (Missing Process Steps)

```markdown
Analyze the code and provide feedback.
```

**잘못된 이유:** 에이전트가 어떻게 분석해야 하는지 알 수 없습니다.

### ✅ 명확한 프로세스 (Clear Process)

```markdown
**Analysis Process:**
1. Read code files using Read tool
2. Scan for type annotations on all functions
3. Check for 'any' type usage
4. Verify generic type parameters
5. List findings with file:line references
```

### ❌ 정의되지 않은 출력 (Undefined Output)

```markdown
Provide a report.
```

**잘못된 이유:** 에이전트가 어떤 형식을 사용해야 하는지 알 수 없습니다.

### ✅ 정의된 출력 형식 (Defined Output Format)

```markdown
**Output Format:**
## Type Safety Report

### Summary
[Overview of findings]

### Issues Found
- `file.ts:42` - Missing return type on `processData`
- `utils.ts:15` - Unsafe 'any' usage in parameter

### Recommendations
[Specific fixes with examples]
```

## 길이 가이드라인 (Length Guidelines)

### 최소 실현 가능 에이전트 (Minimum Viable Agent)

**최소 ~500단어:**
- 역할 설명
- 3가지 핵심 책임
- 5단계 프로세스
- 출력 형식

### 표준 에이전트 (Standard Agent)

**~1,000-2,000단어:**
- 상세한 역할 및 전문 지식
- 5-8가지 책임
- 8-12가지 프로세스 단계
- 품질 표준
- 출력 형식
- 3-5가지 예외 사례 (edge cases)

### 종합 에이전트 (Comprehensive Agent)

**~2,000-5,000단어:**
- 배경 정보를 포함한 완전한 역할
- 종합적인 책임
- 상세한 다단계 프로세스
- 광범위한 품질 표준
- 여러 출력 형식
- 다양한 예외 사례 (edge cases)
- 시스템 프롬프트 내부의 예시

**10,000단어 초과 금지:** 너무 길어지면 오히려 효율성이 떨어집니다.

## 시스템 프롬프트 테스트하기 (Testing System Prompts)

### 완성도 테스트 (Test Completeness)

에이전트가 시스템 프롬프트만으로 다음 사항들을 처리할 수 있습니까?

- [ ] 전형적인 작업 실행
- [ ] 명시된 예외 사례
- [ ] 에러 시나리오
- [ ] 모호한 요구사항
- [ ] 대용량/복잡한 입력
- [ ] 비어 있거나 누락된 입력

### 명확성 테스트 (Test Clarity)

시스템 프롬프트를 읽고 다음 질문을 해보세요:

- 다른 개발자도 이 에이전트가 하는 일을 이해할 수 있습니까?
- 프로세스 단계가 명확하고 실행 가능합니까?
- 출력 형식이 모호하지 않습니까?
- 품질 표준을 측정할 수 있습니까?

### 결과에 기반한 반복 개선 (Iterate Based on Results)

에이전트를 테스트한 후:
1. 에이전트가 어려움을 겪은 부분을 식별합니다.
2. 누락된 지침을 시스템 프롬프트에 추가합니다.
3. 모호한 지침을 명확히 합니다.
4. 예외 사례에 대한 프로세스 단계를 추가합니다.
5. 다시 테스트합니다.

## 결론 (Conclusion)

효과적인 시스템 프롬프트는 다음과 같습니다:
- **구체적임 (Specific)**: 무엇을 어떻게 해야 하는지 명확히 함
- **구조화됨 (Structured)**: 명확한 섹션들로 구성됨
- **완전함 (Complete)**: 정상 및 예외 사례를 모두 다룸
- **실행 가능함 (Actionable)**: 구체적인 단계를 제공함
- **테스트 가능함 (Testable)**: 측정 가능한 표준을 정의함

위의 패턴들을 템플릿으로 사용하여 도메인에 맞게 커스텀하고, 에이전트의 성능에 따라 반복해서 개선해 나가십시오.
