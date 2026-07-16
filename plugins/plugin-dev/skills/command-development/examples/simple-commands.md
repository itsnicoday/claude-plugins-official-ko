# 간단한 명령어 예시 (Simple Command Examples)

일반적인 사용 사례를 위한 기본적인 슬래시 명령어 패턴입니다.

**중요:** 아래의 모든 예시는 사용자를 위한 메시지가 아니라 Claude를 위한 지침(에이전트 소비용)으로 작성되었습니다. 명령어는 사용자에게 어떤 일이 일어날지 알려주는 것이 아니라 Claude에게 무엇을 해야 하는지 알려줍니다.

## 예시 1: 코드 리뷰 명령어 (Code Review Command)

**파일:** `.claude/commands/review.md`

```markdown
---
description: Review code for quality and issues
allowed-tools: Read, Bash(git:*)
---

Review the code in this repository for:

1. **Code Quality:**
   - Readability and maintainability
   - Consistent style and formatting
   - Appropriate abstraction levels

2. **Potential Issues:**
   - Logic errors or bugs
   - Edge cases not handled
   - Performance concerns

3. **Best Practices:**
   - Design patterns used correctly
   - Error handling present
   - Documentation adequate

Provide specific feedback with file and line references.
```

**사용법:**
```
> /review
```

---

## 예시 2: 보안 리뷰 명령어 (Security Review Command)

**파일:** `.claude/commands/security-review.md`

```markdown
---
description: Review code for security vulnerabilities
allowed-tools: Read, Grep
model: sonnet
---

Perform comprehensive security review checking for:

**Common Vulnerabilities:**
- SQL injection risks
- Cross-site scripting (XSS)
- Authentication/authorization issues
- Insecure data handling
- Hardcoded secrets or credentials

**Security Best Practices:**
- Input validation present
- Output encoding correct
- Secure defaults used
- Error messages safe
- Logging appropriate (no sensitive data)

For each issue found:
- File and line number
- Severity (Critical/High/Medium/Low)
- Description of vulnerability
- Recommended fix

Prioritize issues by severity.
```

**사용법:**
```
> /security-review
```

---

## 예시 3: 파일 인수를 사용하는 테스트 명령어 (Test Command with File Argument)

**파일:** `.claude/commands/test-file.md`

```markdown
---
description: Run tests for specific file
argument-hint: [test-file]
allowed-tools: Bash(npm:*), Bash(jest:*)
---

Run tests for $1:

Test execution: !`npm test $1`

Analyze results:
- Tests passed/failed
- Code coverage
- Performance issues
- Flaky tests

If failures found, suggest fixes based on error messages.
```

**사용법:**
```
> /test-file src/utils/helpers.test.ts
```

---

## 예시 4: 문서 생성기 (Documentation Generator)

**파일:** `.claude/commands/document.md`

```markdown
---
description: Generate documentation for file
argument-hint: [source-file]
---

Generate comprehensive documentation for @$1

Include:

**Overview:**
- Purpose and responsibility
- Main functionality
- Dependencies

**API Documentation:**
- Function/method signatures
- Parameter descriptions with types
- Return values with types
- Exceptions/errors thrown

**Usage Examples:**
- Basic usage
- Common patterns
- Edge cases

**Implementation Notes:**
- Algorithm complexity
- Performance considerations
- Known limitations

Format as Markdown suitable for project documentation.
```

**사용법:**
```
> /document src/api/users.ts
```

---

## 예시 5: Git 상태 요약 (Git Status Summary)

**파일:** `.claude/commands/git-status.md`

```markdown
---
description: Summarize Git repository status
allowed-tools: Bash(git:*)
---

Repository Status Summary:

**Current Branch:** !`git branch --show-current`

**Status:** !`git status --short`

**Recent Commits:** !`git log --oneline -5`

**Remote Status:** !`git fetch && git status -sb`

Provide:
- Summary of changes
- Suggested next actions
- Any warnings or issues
```

**사용법:**
```
> /git-status
```

---

## 예시 6: 배포 명령어 (Deployment Command)

**파일:** `.claude/commands/deploy.md`

```markdown
---
description: Deploy to specified environment
argument-hint: [environment] [version]
allowed-tools: Bash(kubectl:*), Read
---

Deploy to $1 environment using version $2

**Pre-deployment Checks:**
1. Verify $1 configuration exists
2. Check version $2 is valid
3. Verify cluster accessibility: !`kubectl cluster-info`

**Deployment Steps:**
1. Update deployment manifest with version $2
2. Apply configuration to $1
3. Monitor rollout status
4. Verify pod health
5. Run smoke tests

**Rollback Plan:**
Document current version for rollback if issues occur.

Proceed with deployment? (yes/no)
```

**사용법:**
```
> /deploy staging v1.2.3
```

---

## 예시 7: 비교 명령어 (Comparison Command)

**파일:** `.claude/commands/compare-files.md`

```markdown
---
description: Compare two files
argument-hint: [file1] [file2]
---

Compare @$1 with @$2

**Analysis:**

1. **Differences:**
   - Lines added
   - Lines removed
   - Lines modified

2. **Functional Changes:**
   - Breaking changes
   - New features
   - Bug fixes
   - Refactoring

3. **Impact:**
   - Affected components
   - Required updates elsewhere
   - Migration requirements

4. **Recommendations:**
   - Code review focus areas
   - Testing requirements
   - Documentation updates needed

Present as structured comparison report.
```

**사용법:**
```
> /compare-files src/old-api.ts src/new-api.ts
```

---

## 예시 8: 빠른 수정 명령어 (Quick Fix Command)

**파일:** `.claude/commands/quick-fix.md`

```markdown
---
description: Quick fix for common issues
argument-hint: [issue-description]
model: haiku
---

Quickly fix: $ARGUMENTS

**Approach:**
1. Identify the issue
2. Find relevant code
3. Propose fix
4. Explain solution

Focus on:
- Simple, direct solution
- Minimal changes
- Following existing patterns
- No breaking changes

Provide code changes with file paths and line numbers.
```

**사용법:**
```
> /quick-fix button not responding to clicks
> /quick-fix typo in error message
```

---

## 예시 9: 연구 명령어 (Research Command)

**파일:** `.claude/commands/research.md`

```markdown
---
description: Research best practices for topic
argument-hint: [topic]
model: sonnet
---

Research best practices for: $ARGUMENTS

**Coverage:**

1. **Current State:**
   - How we currently handle this
   - Existing implementations

2. **Industry Standards:**
   - Common patterns
   - Recommended approaches
   - Tools and libraries

3. **Comparison:**
   - Our approach vs standards
   - Gaps or improvements needed
   - Migration considerations

4. **Recommendations:**
   - Concrete action items
   - Priority and effort estimates
   - Resources for implementation

Provide actionable guidance based on research.
```

**사용법:**
```
> /research error handling in async operations
> /research API authentication patterns
```

---

## 예시 10: 코드 설명 명령어 (Explain Code Command)

**파일:** `.claude/commands/explain.md`

```markdown
---
description: Explain how code works
argument-hint: [file-or-function]
---

Explain @$1 in detail

**Explanation Structure:**

1. **Overview:**
   - What it does
   - Why it exists
   - How it fits in system

2. **Step-by-Step:**
   - Line-by-line walkthrough
   - Key algorithms or logic
   - Important details

3. **Inputs and Outputs:**
   - Parameters and types
   - Return values
   - Side effects

4. **Edge Cases:**
   - Error handling
   - Special cases
   - Limitations

5. **Usage Examples:**
   - How to call it
   - Common patterns
   - Integration points

Explain at level appropriate for junior engineer.
```

**사용법:**
```
> /explain src/utils/cache.ts
> /explain AuthService.login
```

---

## 주요 패턴 (Key Patterns)

### 패턴 1: 읽기 전용 분석 (Read-Only Analysis)

```markdown
---
allowed-tools: Read, Grep
---

Analyze but don't modify...
```

**용도:** 코드 리뷰, 문서화, 분석

### 패턴 2: Git 작업 (Git Operations)

```markdown
---
allowed-tools: Bash(git:*)
---

!`git status`
Analyze and suggest...
```

**용도:** 저장소 상태, 커밋 분석

### 패턴 3: 단일 인수 (Single Argument)

```markdown
---
argument-hint: [target]
---

Process $1...
```

**용도:** 파일 작업, 특정 대상 작업

### 패턴 4: 다중 인수 (Multiple Arguments)

```markdown
---
argument-hint: [source] [target] [options]
---

Process $1 to $2 with $3...
```

**용도:** 워크플로우, 배포, 비교

### 패턴 5: 빠른 실행 (Fast Execution)

```markdown
---
model: haiku
---

Quick simple task...
```

**용도:** 간단하고 반복적인 명령어

### 패턴 6: 파일 비교 (File Comparison)

```markdown
Compare @$1 with @$2...
```

**용도:** Diff 분석, 마이그레이션 계획

### 패턴 7: 컨텍스트 수집 (Context Gathering)

```markdown
---
allowed-tools: Bash(git:*), Read
---

Context: !`git status`
Files: @file1 @file2

Analyze...
```

**용도:** 정보에 기반한 의사 결정

## 간단한 명령어를 작성하기 위한 팁 (Tips for Writing Simple Commands)

1. **기본부터 시작하기:** 단일 책임, 명확한 목적
2. **점진적으로 복잡성 추가하기:** 프론트매터 없이 시작하기
3. **점진적으로 테스트하기:** 각 기능이 작동하는지 확인
4. **설명적인 이름 사용하기:** 명령어 이름은 목적을 나타내야 함
5. **인수 문서화하기:** 항상 argument-hint 사용하기
6. **예시 제공하기:** 주석에 사용법 표시하기
7. **에러 처리하기:** 누락된 인수나 파일 고려하기
