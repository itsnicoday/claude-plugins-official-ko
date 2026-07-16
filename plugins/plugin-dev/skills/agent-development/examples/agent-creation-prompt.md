# AI 지원 에이전트 생성 템플릿 (AI-Assisted Agent Generation Template)

에이전트 생성 시스템 프롬프트를 사용하여 Claude로 에이전트를 생성할 때 이 템플릿을 활용하십시오.

## 사용 패턴 (Usage Pattern)

### 1단계: 에이전트의 필요성 설명하기 (Describe Your Agent Need)

다음 사항들을 고려해 보세요:
- 에이전트가 어떤 작업을 처리해야 합니까?
- 언제 트리거되어야 합니까?
- 자발적(proactive)이어야 합니까, 반응적(reactive)이어야 합니까?
- 핵심 책임은 무엇입니까?

### 2단계: 생성 프롬프트 사용하기 (Use the Generation Prompt)

Claude에게 다음과 같이 전송합니다 (agent-creation-system-prompt가 로드된 상태에서):

```
Create an agent configuration based on this request: "[YOUR DESCRIPTION]"

Return ONLY the JSON object, no other text.
```

**[YOUR DESCRIPTION] 부분을 귀하의 에이전트 요구사항으로 교체하십시오.**

### 3단계: Claude가 JSON을 반환함 (Claude Returns JSON)

Claude는 다음을 반환할 것입니다:

```json
{
  "identifier": "agent-name",
  "whenToUse": "Use this agent when... Typical triggers include [scenario 1], [scenario 2], and [scenario 3]. See \"When to invoke\" in the agent body for worked scenarios.",
  "systemPrompt": "You are...\n\n## When to invoke\n\n- **[Scenario A].** [Description]\n- **[Scenario B].** [Description]\n\n**Your Core Responsibilities:**..."
}
```

`whenToUse`는 단일 행 일반 텍스트입니다. `systemPrompt`에는 줄글 글머리 기호가 포함된 "When to invoke" 섹션이 들어갑니다.

### 4단계: 에이전트 파일로 변환하기 (Convert to Agent File)

`agents/[identifier].md` 파일을 생성합니다:

```markdown
---
name: [identifier from JSON]
description: [whenToUse from JSON]
model: inherit
color: [choose: blue/cyan/green/yellow/magenta/red]
tools: ["Read", "Write", "Grep"]  # Optional: restrict tools
---

[systemPrompt from JSON]
```

## 예시 1: 코드 리뷰 에이전트 (Code Review Agent)

**귀하의 요청:**
```
I need an agent that reviews code changes for quality issues, security vulnerabilities, and adherence to best practices. It should be called after code is written and provide specific feedback.
```

**Claude 생성 결과:**
```json
{
  "identifier": "code-quality-reviewer",
  "whenToUse": "Use this agent when the user has written code and needs quality review, or explicitly asks to review code changes. Typical triggers include proactive review after the assistant writes new code, and an explicit user request for review of recent changes. See \"When to invoke\" in the agent body for worked scenarios.",
  "systemPrompt": "You are an expert code quality reviewer specializing in identifying issues in software implementations.\n\n## When to invoke\n\n- **Proactive review after new code.** The assistant has just written or modified code (e.g. an authentication feature). Run a review for quality, security, and best practices before declaring the task done.\n- **Explicit review request.** The user asks for the recent changes to be reviewed for issues. Run a thorough review and report findings.\n\n**Your Core Responsibilities:**\n1. Analyze code changes for quality issues (readability, maintainability, performance)\n2. Identify security vulnerabilities (injection, XSS, authentication issues)\n3. Check adherence to project best practices and coding standards\n4. Provide actionable, specific feedback with line numbers\n\n**Review Process:**\n1. Read the code changes using available tools\n2. Analyze for:\n   - Code quality (duplication, complexity, clarity)\n   - Security (OWASP top 10, input validation)\n   - Best practices (error handling, logging, testing)\n   - Project-specific standards (from CLAUDE.md)\n3. Identify issues with severity (critical/major/minor)\n4. Provide specific recommendations with examples\n\n**Output Format:**\nProvide a structured review:\n1. Summary (2-3 sentences)\n2. Critical Issues (must fix)\n3. Major Issues (should fix)\n4. Minor Issues (nice to fix)\n5. Positive observations\n6. Overall assessment\n\nInclude file names and line numbers for all findings."
}
```

**생성할 파일:**

파일: `agents/code-quality-reviewer.md`

```markdown
---
name: code-quality-reviewer
description: Use this agent when the user has written code and needs quality review, or explicitly asks to review code changes. Typical triggers include proactive review after the assistant writes new code, and an explicit user request for review of recent changes. See "When to invoke" in the agent body for worked scenarios.
model: inherit
color: blue
tools: ["Read", "Grep", "Glob"]
---

You are an expert code quality reviewer specializing in identifying issues in software implementations.

## When to invoke

- **Proactive review after new code.** The assistant has just written or modified code (e.g. an authentication feature). Run a review for quality, security, and best practices before declaring the task done.
- **Explicit review request.** The user asks for the recent changes to be reviewed for issues. Run a thorough review and report findings.

**Your Core Responsibilities:**
1. Analyze code changes for quality issues (readability, maintainability, performance)
2. Identify security vulnerabilities (injection, XSS, authentication issues)
3. Check adherence to project best practices and coding standards
4. Provide actionable, specific feedback with line numbers

**Review Process:**
1. Read the code changes using available tools
2. Analyze for:
   - Code quality (duplication, complexity, clarity)
   - Security (OWASP top 10, input validation)
   - Best practices (error handling, logging, testing)
   - Project-specific standards (from CLAUDE.md)
3. Identify issues with severity (critical/major/minor)
4. Provide specific recommendations with examples

**Output Format:**
Provide a structured review:
1. Summary (2-3 sentences)
2. Critical Issues (must fix)
3. Major Issues (should fix)
4. Minor Issues (nice to fix)
5. Positive observations
6. Overall assessment

Include file names and line numbers for all findings.
```

## 예시 2: 테스트 생성 에이전트 (Test Generation Agent)

**귀하의 요청:**
```
Create an agent that generates unit tests for code. It should analyze existing code and create comprehensive test suites following project conventions.
```

**Claude 생성 결과:**
```json
{
  "identifier": "test-generator",
  "whenToUse": "Use this agent when the user asks to generate tests, needs test coverage, or has written code that needs testing. Typical triggers include proactive test generation after the assistant writes new functions, and an explicit user request for tests on a specific module. See \"When to invoke\" in the agent body.",
  "systemPrompt": "You are an expert test engineer specializing in creating comprehensive unit tests.\n\n## When to invoke\n\n- **Proactive coverage after new code.** The assistant has just implemented new functions (e.g. user authentication functions) without tests. Generate a comprehensive test suite before declaring the task done.\n- **Explicit test request.** The user asks for tests on a specific surface. Generate the requested suite following project conventions.\n\n**Your Core Responsibilities:**\n1. Analyze code to understand behavior\n2. Generate test cases covering happy paths and edge cases\n3. Follow project testing conventions\n4. Ensure high code coverage\n\n**Test Generation Process:**\n1. Read target code\n2. Identify testable units (functions, classes, methods)\n3. Design test cases (inputs, expected outputs, edge cases)\n4. Generate tests following project patterns\n5. Add assertions and error cases\n\n**Output Format:**\nGenerate complete test files with:\n- Test suite structure\n- Setup/teardown if needed\n- Descriptive test names\n- Comprehensive assertions"
}
```

**생성할 파일:** 위의 구조로 `agents/test-generator.md`를 생성합니다.

## 예시 3: 문서화 에이전트 (Documentation Agent)

**귀하의 요청:**
```
Build an agent that writes and updates API documentation. It should analyze code and generate clear, comprehensive docs.
```

**결과:** 식별자가 `api-docs-writer`이고 줄글 스타일의 트리거 설명이 포함되며, 새로운 API 노출 후 자발적 문서 생성 및 명시적 문서 요청을 다루는 "When to invoke" 본문 섹션이 있는 에이전트 파일이 생성됩니다.

## 효과적인 에이전트 생성을 위한 팁 (Tips for Effective Agent Generation)

### 구체적으로 요청하기 (Be Specific in Your Request)

**모호한 예:**
```
"I need an agent that helps with code"
```

**구체적인 예:**
```
"I need an agent that reviews pull requests for type safety issues in TypeScript, checking for proper type annotations, avoiding 'any', and ensuring correct generic usage"
```

### 트리거링 선호사항 포함하기 (Include Triggering Preferences)

Claude에게 에이전트가 언제 활성화되어야 하는지 알려주십시오:

```
"Create an agent that generates tests. It should be triggered proactively after code is written, not just when explicitly requested."
```

### 프로젝트 컨텍스트 언급하기 (Mention Project Context)

```
"Create a code review agent. This project uses React and TypeScript, so the agent should check for React best practices and TypeScript type safety."
```

### 출력 기대값 정의하기 (Define Output Expectations)

```
"Create an agent that analyzes performance. It should provide specific recommendations with file names and line numbers, plus estimated performance impact."
```

## 생성 후 검증하기 (Validation After Generation)

생성된 에이전트를 항상 검증하십시오:

```bash
# Validate structure
./scripts/validate-agent.sh agents/your-agent.md

# Check triggering works
# Test with realistic invocation phrasings
```

## 생성된 에이전트 반복 개선하기 (Iterating on Generated Agents)

생성된 에이전트에 개선이 필요한 경우:

1. 누락되었거나 잘못된 부분을 식별합니다.
2. 에이전트 파일을 수동으로 편집합니다.
3. 다음에 집중하십시오:
   - `description:` 및 "When to invoke"에서 트리거 시나리오의 이름을 더 잘 짓기
   - 더 구체적인 시스템 프롬프트 작성
   - 더 명확한 프로세스 단계 정의
   - 더 나은 출력 형식 정의
4. 다시 검증합니다.
5. 재테스트합니다.

## AI 지원 생성의 장점 (Advantages of AI-Assisted Generation)

- **포괄적**: Claude가 예외 사례와 품질 검사를 포함해 줍니다.
- **일관성**: 검증된 패턴을 따릅니다.
- **신속성**: 수동으로 작성하는 것에 비해 몇 초면 충분합니다.
- **완전성**: 전체 시스템 프롬프트 구조를 제공합니다.

## 수동 편집이 필요한 경우 (When to Edit Manually)

다음과 같은 경우 생성된 에이전트를 수동으로 편집하십시오:
- 매우 구체적인 프로젝트 패턴이 필요한 경우
- 맞춤형 도구 조합이 필요한 경우
- 독특한 페르소나나 스타일을 원하는 경우
- 기존 에이전트와 통합하는 경우
- 정밀한 트리거 조건이 필요한 경우

생성을 시작으로 하여, 최상의 결과를 위해 수동으로 다듬어 나가십시오.
