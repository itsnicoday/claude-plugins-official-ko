---
name: command-development
description: This skill should be used when the user asks to "create a slash command", "add a command", "write a custom command", "define command arguments", "use command frontmatter", "organize commands", "create command with file references", "interactive command", "use AskUserQuestion in command", or needs guidance on slash command structure, YAML frontmatter fields, dynamic arguments, bash execution in commands, user interaction patterns, or command development best practices for Claude Code.
version: 0.2.0
---

# Claude Code를 위한 명령어 개발 (Command Development for Claude Code)

> **참고:** `.claude/commands/` 디렉토리는 레거시 형식입니다. 새로운 스킬의 경우 `.claude/skills/<name>/SKILL.md` 디렉토리 형식을 사용하세요. 두 형식 모두 동일하게 로드되며, 유일한 차이점은 파일 레이아웃입니다. 선호되는 형식은 `skill-development` 스킬을 참조하세요.

## 개요 (Overview)

슬래시 명령어는 Claude가 대화식 세션 중에 실행하는 마크다운 파일로 정의된 자주 사용되는 프롬프트입니다. 명령어 구조, 프론트매터 옵션 및 동적 기능을 이해하면 강력하고 재사용 가능한 워크플로우를 생성할 수 있습니다.

**주요 개념:**

- 명령어를 위한 마크다운 파일 형식
- 구성을 위한 YAML 프론트매터
- 동적 인수 및 파일 참조
- 컨텍스트를 위한 Bash 실행
- 명령어 구성 및 네임스페이스 지정

## 명령어 기초 (Command Basics)

### 슬래시 명령어란 무엇인가요? (What is a Slash Command?)

슬래시 명령어는 호출 시 Claude가 실행하는 프롬프트가 포함된 마크다운 파일입니다. 명령어는 다음을 제공합니다:

- **재사용성**: 한 번 정의하고 반복해서 사용 가능
- **일관성**: 공통 워크플로우 표준화
- **공유**: 팀 또는 프로젝트 간에 배포
- **효율성**: 복잡한 프롬프트에 신속하게 접근

### 중요: 명령어는 Claude를 위한 지침입니다 (Critical: Commands are Instructions FOR Claude)

**명령어는 인간이 아닌 에이전트(Claude)가 읽을 목적으로 작성됩니다.**

사용자가 `/command-name`을 호출하면 명령어 내용이 Claude의 지침이 됩니다. 명령어를 사용자에게 전달할 메시지가 아니라, Claude가 수행해야 할 작업에 대한 지시 사항으로 작성하세요.

**올바른 접근 방식 (Claude를 위한 지침):**

```markdown
Review this code for security vulnerabilities including:

- SQL injection
- XSS attacks
- Authentication issues

Provide specific line numbers and severity ratings.
```

**잘못된 접근 방식 (사용자에게 전달하는 메시지):**

```markdown
This command will review your code for security issues.
You'll receive a report with vulnerability details.
```

첫 번째 예시는 Claude에게 무엇을 해야 하는지 알려줍니다. 두 번째 예시는 사용자에게 어떤 일이 일어날지 알려주지만 Claude에게는 지시하지 않습니다. 항상 첫 번째 접근 방식을 사용하세요.

### 명령어 위치 (Command Locations)

**프로젝트 명령어** (팀과 공유):

- 위치: `.claude/commands/`
- 범위: 특정 프로젝트에서 사용 가능
- 라벨: `/help`에서 "(project)"로 표시됨
- 용도: 팀 워크플로우, 프로젝트 전용 작업

**개인 명령어** (모든 곳에서 사용 가능):

- 위치: `~/.claude/commands/`
- 범위: 모든 프로젝트에서 사용 가능
- 라벨: `/help`에서 "(user)"로 표시됨
- 용도: 개인 워크플로우, 크로스 프로젝트 유틸리티

**플러그인 명령어** (플러그인과 함께 번들로 제공):

- 위치: `plugin-name/commands/`
- 범위: 플러그인이 설치되었을 때 사용 가능
- 라벨: `/help`에서 "(plugin-name)"로 표시됨
- 용도: 플러그인 전용 기능

## 파일 형식 (File Format)

### 기본 구조 (Basic Structure)

명령어는 `.md` 확장자를 가진 마크다운 파일입니다:

```
.claude/commands/
├── review.md           # /review 명령어
├── test.md             # /test 명령어
└── deploy.md           # /deploy 명령어
```

**간단한 명령어:**

```markdown
Review this code for security vulnerabilities including:

- SQL injection
- XSS attacks
- Authentication bypass
- Insecure data handling
```

기본 명령어에는 프론트매터가 필요하지 않습니다.

### YAML 프론트매터 사용 (With YAML Frontmatter)

YAML 프론트매터를 사용하여 구성을 추가합니다:

```markdown
---
description: Review code for security issues
allowed-tools: Read, Grep, Bash(git:*)
model: sonnet
---

Review this code for security vulnerabilities...
```

## YAML 프론트매터 필드 (YAML Frontmatter Fields)

### description

**목적:** `/help`에 표시되는 간단한 설명
**타입:** String
**기본값:** 명령어 프롬프트의 첫 번째 줄

```yaml
---
description: Review pull request for code quality
---
```

**권장 사항:** 명확하고 실행 가능한 설명 (60자 미만)

### allowed-tools

**목적:** 명령어가 사용할 수 있는 도구 지정
**타입:** String 또는 Array
**기본값:** 대화에서 상속됨

```yaml
---
allowed-tools: Read, Write, Edit, Bash(git:*)
---
```

**패턴:**

- `Read, Write, Edit` - 특정 도구
- `Bash(git:*)` - git 명령어만 사용하는 Bash
- `*` - 모든 도구 (거의 필요하지 않음)

**용도:** 명령어가 특정 도구에 대한 접근 권한을 필요로 할 때 사용

### model

**목적:** 명령어 실행을 위한 모델 지정
**타입:** String (sonnet, opus, haiku)
**기본값:** 대화에서 상속됨

```yaml
---
model: haiku
---
```

**사용 사례:**

- `haiku` - 빠르고 간단한 명령어
- `sonnet` - 표준 워크플로우
- `opus` - 복잡한 분석

### argument-hint

**목적:** 자동 완성을 위해 예상되는 인수를 문서화
**타입:** String
**기본값:** 없음

```yaml
---
argument-hint: [pr-number] [priority] [assignee]
---
```

**장점:**

- 사용자가 명령어 인수를 이해하는 데 도움을 줌
- 명령어 검색 가능성 개선
- 명령어 인터페이스 문서화

### disable-model-invocation

**목적:** SlashCommand 도구가 프로그래밍 방식으로 명령어를 호출하지 못하도록 방지
**타입:** Boolean
**기본값:** false

```yaml
---
disable-model-invocation: true
---
```

**용도:** 명령어가 수동으로만 호출되어야 할 때 사용

## 동적 인수 (Dynamic Arguments)

### $ARGUMENTS 사용하기 (Using $ARGUMENTS)

모든 인수를 단일 문자열로 캡처합니다:

```markdown
---
description: Fix issue by number
argument-hint: [issue-number]
---

Fix issue #$ARGUMENTS following our coding standards and best practices.
```

**사용법:**

```
> /fix-issue 123
> /fix-issue 456
```

**확장된 결과:**

```
Fix issue #123 following our coding standards...
Fix issue #456 following our coding standards...
```

### 위치 인수 사용하기 (Using Positional Arguments)

`$1`, `$2`, `$3` 등으로 개별 인수를 캡처합니다:

```markdown
---
description: Review PR with priority and assignee
argument-hint: [pr-number] [priority] [assignee]
---

Review pull request #$1 with priority level $2.
After review, assign to $3 for follow-up.
```

**사용법:**

```
> /review-pr 123 high alice
```

**확장된 결과:**

```
Review pull request #123 with priority level high.
After review, assign to alice for follow-up.
```

### 인수 결합하기 (Combining Arguments)

위치 인수와 나머지 인수를 혼합합니다:

```markdown
Deploy $1 to $2 environment with options: $3
```

**사용법:**

```
> /deploy api staging --force --skip-tests
```

**확장된 결과:**

```
Deploy api to staging environment with options: --force --skip-tests
```

## 파일 참조 (File References)

### @ 구문 사용하기 (Using @ Syntax)

명령어에 파일 내용을 포함합니다:

```markdown
---
description: Review specific file
argument-hint: [file-path]
---

Review @$1 for:

- Code quality
- Best practices
- Potential bugs
```

**사용법:**

```
> /review-file src/api/users.ts
```

**효과:** Claude는 명령어를 처리하기 전에 `src/api/users.ts`를 읽습니다.

### 다중 파일 참조 (Multiple File References)

여러 파일을 참조합니다:

```markdown
Compare @src/old-version.js with @src/new-version.js

Identify:

- Breaking changes
- New features
- Bug fixes
```

### 정적 파일 참조 (Static File References)

인수 없이 알려진 파일을 참조합니다:

```markdown
Review @package.json and @tsconfig.json for consistency

Ensure:

- TypeScript version matches
- Dependencies are aligned
- Build configuration is correct
```

## 명령어 내 Bash 실행 (Bash Execution in Commands)

명령어는 Claude가 명령어를 처리하기 전에 동적으로 컨텍스트를 수집하기 위해 인라인으로 bash 명령어를 실행할 수 있습니다. 이는 저장소 상태, 환경 정보 또는 프로젝트별 컨텍스트를 포함하는 데 유용합니다.

**사용 시기:**

- 동적 컨텍스트(git status, 환경 변수 등) 포함
- 프로젝트/저장소 상태 수집
- 컨텍스트 인식 워크플로우 빌드

**구현 세부 정보:**
전체 구문, 예시 및 권장 사항은 `references/plugin-features-reference.md`에서 bash 실행 섹션을 참조하세요. 이 참조에는 실행 문제를 방지하기 위한 정확한 구문과 여러 작동 예시가 포함되어 있습니다.

## 명령어 구성 (Command Organization)

### 플랫 구조 (Flat Structure)

소규모 명령어 세트를 위한 간단한 구성:

```
.claude/commands/
├── build.md
├── test.md
├── deploy.md
├── review.md
└── docs.md
```

**사용 시기:** 명령어 개수가 5~15개이며 명확한 카테고리가 없을 때

### 네임스페이스 구조 (Namespaced Structure)

하위 디렉토리에 명령어를 구성합니다:

```
.claude/commands/
├── ci/
│   ├── build.md        # /build (project:ci)
│   ├── test.md         # /test (project:ci)
│   └── lint.md         # /lint (project:ci)
├── git/
│   ├── commit.md       # /commit (project:git)
│   └── pr.md           # /pr (project:git)
└── docs/
    ├── generate.md     # /generate (project:docs)
    └── publish.md      # /publish (project:docs)
```

**장점:**

- 카테고리별 논리적 그룹화
- `/help`에 네임스페이스 표시됨
- 관련된 명령어 검색이 쉬워짐

**사용 시기:** 명령어 개수가 15개 이상이며 명확한 카테고리가 있을 때

## 권장 사항 (Best Practices)

### 명령어 설계 (Command Design)

1. **단일 책임:** 하나의 명령어는 하나의 작업만 수행
2. **명확한 설명:** `/help`에서 직관적으로 이해할 수 있는 설명
3. **명시적 의존성:** 필요한 경우 `allowed-tools` 사용
4. **인수 문서화:** 항상 `argument-hint` 제공
5. **일관된 네이밍:** 동사-명사 패턴 사용 (review-pr, fix-issue)

### 인수 처리 (Argument Handling)

1. **인수 검증:** 프롬프트에서 필수 인수가 있는지 확인
2. **기본값 제공:** 인수가 누락되었을 때 기본값 제안
3. **문서 형식:** 예상되는 인수 형식 설명
4. **예외 케이스 처리:** 누락되었거나 잘못된 인수 고려

```markdown
---
argument-hint: [pr-number]
---

$IF($1,
Review PR #$1,
Please provide a PR number. Usage: /review-pr [number]
)
```

### 파일 참조 (File References)

1. **명시적 경로:** 명확한 파일 경로 사용
2. **존재 확인:** 누락된 파일을 정상적으로 처리
3. **상대 경로:** 프로젝트 상대 경로 사용
4. **Glob 지원:** 패턴에 Glob 도구 사용 고려

### Bash 명령어 (Bash Commands)

1. **범위 제한:** `Bash(*)` 대신 `Bash(git:*)` 사용
2. **안전한 명령어:** 파괴적인 작업 방지
3. **에러 처리:** 명령어 실패 고려
4. **빠른 실행 유지:** 실행 시간이 긴 명령어는 호출을 지연시킴

### 문서화 (Documentation)

1. **주석 추가:** 복잡한 로직 설명
2. **예시 제공:** 주석에 사용법 표시
3. **요구 사항 나열:** 종속성 문서화
4. **명령어 버전 관리:** 주요 변경 사항 기록

```markdown
---
description: Deploy application to environment
argument-hint: [environment] [version]
---

<!--
Usage: /deploy [staging|production] [version]
Requires: AWS credentials configured
Example: /deploy staging v1.2.3
-->

Deploy application to $1 environment using version $2...
```

## 공통 패턴 (Common Patterns)

### 리뷰 패턴 (Review Pattern)

```markdown
---
description: Review code changes
allowed-tools: Read, Bash(git:*)
---

Files changed: !`git diff --name-only`

Review each file for:

1. Code quality and style
2. Potential bugs or issues
3. Test coverage
4. Documentation needs

Provide specific feedback for each file.
```

### 테스트 패턴 (Testing Pattern)

```markdown
---
description: Run tests for specific file
argument-hint: [test-file]
allowed-tools: Bash(npm:*)
---

Run tests: !`npm test $1`

Analyze results and suggest fixes for failures.
```

### 문서화 패턴 (Documentation Pattern)

```markdown
---
description: Generate documentation for file
argument-hint: [source-file]
---

Generate comprehensive documentation for @$1 including:

- Function/class descriptions
- Parameter documentation
- Return value descriptions
- Usage examples
- Edge cases and errors
```

### 워크플로우 패턴 (Workflow Pattern)

```markdown
---
description: Complete PR workflow
argument-hint: [pr-number]
allowed-tools: Bash(gh:*), Read
---

PR #$1 Workflow:

1. Fetch PR: !`gh pr view $1`
2. Review changes
3. Run checks
4. Approve or request changes
```

## 문제 해결 (Troubleshooting)

**명령어가 나타나지 않음:**

- 파일이 올바른 디렉토리에 있는지 확인
- `.md` 확장자가 있는지 검증
- 유효한 마크다운 형식인지 확인
- Claude Code 재시작

**인수가 작동하지 않음:**

- `$1`, `$2` 구문이 올바른지 검증
- `argument-hint`가 사용법과 일치하는지 확인
- 불필요한 공백이 없는지 확인

**Bash 실행 실패:**

- `allowed-tools`에 Bash가 포함되어 있는지 확인
- 백틱 안의 명령어 구문 확인
- 터미널에서 명령어를 먼저 테스트
- 필요한 권한 확인

**파일 참조가 작동하지 않음:**

- `@` 구문이 올바른지 검증
- 파일 경로가 유효한지 확인
- Read 도구가 허용되었는지 확인
- 절대 경로 또는 프로젝트 상대 경로 사용

## 플러그인 전용 기능 (Plugin-Specific Features)

### CLAUDE_PLUGIN_ROOT 변수 (CLAUDE_PLUGIN_ROOT Variable)

플러그인 명령어는 플러그인의 절대 경로로 해석되는 환경 변수인 `${CLAUDE_PLUGIN_ROOT}`에 접근할 수 있습니다.

**목적:**

- 이식 가능한 방식으로 플러그인 파일 참조
- 플러그인 스크립트 실행
- 플러그인 구성 로드
- 플러그인 템플릿 사용

**기본 사용법:**

```markdown
---
description: Analyze using plugin script
allowed-tools: Bash(node:*)
---

Run analysis: !`node ${CLAUDE_PLUGIN_ROOT}/scripts/analyze.js $1`

Review results and report findings.
```

**공통 패턴:**

```markdown
# Execute plugin script

!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/script.sh`

# Load plugin configuration

@${CLAUDE_PLUGIN_ROOT}/config/settings.json

# Use plugin template

@${CLAUDE_PLUGIN_ROOT}/templates/report.md

# Access plugin resources

@${CLAUDE_PLUGIN_ROOT}/docs/reference.md
```

**사용해야 하는 이유:**

- 모든 설치 환경에서 작동
- 시스템 간 이식성 보장
- 하드코딩된 경로가 필요 없음
- 다중 파일 플러그인에 필수적

### 플러그인 명령어 구성 (Plugin Command Organization)

플러그인 명령어는 `commands/` 디렉토리에서 자동으로 검색됩니다:

```
plugin-name/
├── commands/
│   ├── foo.md              # /foo (plugin:plugin-name)
│   ├── bar.md              # /bar (plugin:plugin-name)
│   └── utils/
│       └── helper.md       # /helper (plugin:plugin-name:utils)
└── plugin.json
```

**네임스페이스 장점:**

- 논리적 명령어 그룹화
- `/help` 출력에 표시됨
- 이름 충돌 방지
- 관련된 명령어 구성

**명명 규칙:**

- 설명적인 동작 이름 사용
- 제네릭 이름(test, run) 피하기
- 플러그인 전용 접두사 고려
- 단어가 여러 개인 경우 하이픈 사용

### 플러그인 명령어 패턴 (Plugin Command Patterns)

**구성 기반 패턴:**

```markdown
---
description: Deploy using plugin configuration
argument-hint: [environment]
allowed-tools: Read, Bash(*)
---

Load configuration: @${CLAUDE_PLUGIN_ROOT}/config/$1-deploy.json

Deploy to $1 using configuration settings.
Monitor deployment and report status.
```

**템플릿 기반 패턴:**

```markdown
---
description: Generate docs from template
argument-hint: [component]
---

Template: @${CLAUDE_PLUGIN_ROOT}/templates/docs.md

Generate documentation for $1 following template structure.
```

**다중 스크립트 패턴:**

```markdown
---
description: Complete build workflow
allowed-tools: Bash(*)
---

Build: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh`
Test: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/test.sh`
Package: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/package.sh`

Review outputs and report workflow status.
```

**자세한 패턴은 `references/plugin-features-reference.md`를 참조하세요.**

## 플러그인 컴포넌트와의 통합 (Integration with Plugin Components)

명령어는 강력한 워크플로우를 위해 다른 플러그인 컴포넌트와 통합할 수 있습니다.

### 에이전트 통합 (Agent Integration)

복잡한 작업을 위해 플러그인 에이전트를 시작합니다:

```markdown
---
description: Deep code review
argument-hint: [file-path]
---

Initiate comprehensive review of @$1 using the code-reviewer agent.

The agent will analyze:

- Code structure
- Security issues
- Performance
- Best practices

Agent uses plugin resources:

- ${CLAUDE_PLUGIN_ROOT}/config/rules.json
- ${CLAUDE_PLUGIN_ROOT}/checklists/review.md
```

**주요 사항:**

- 에이전트는 `plugin/agents/` 디렉토리에 존재해야 합니다.
- Claude는 Task 도구를 사용하여 에이전트를 실행합니다.
- 에이전트 기능 문서화
- 에이전트가 사용하는 플러그인 리소스 참조

### 스킬 통합 (Skill Integration)

전문 지식을 위해 플러그인 스킬을 활용합니다:

```markdown
---
description: Document API with standards
argument-hint: [api-file]
---

Document API in @$1 following plugin standards.

Use the api-docs-standards skill to ensure:

- Complete endpoint documentation
- Consistent formatting
- Example quality
- Error documentation

Generate production-ready API docs.
```

**주요 사항:**

- 스킬은 `plugin/skills/` 디렉토리에 존재해야 합니다.
- 호출을 트리거하기 위해 스킬 이름 언급
- 스킬 목적 문서화
- 스킬이 제공하는 기능 설명

### 훅 조정 (Hook Coordination)

플러그인 훅과 함께 작동하는 명령어를 설계합니다:

- 명령어는 훅이 처리할 수 있는 상태를 준비할 수 있습니다.
- 훅은 도구 이벤트 시 자동으로 실행됩니다.
- 명령어는 예상되는 훅의 동작을 문서화해야 합니다.
- Claude에게 훅 출력의 해석 방법을 안내합니다.

훅과 조정하는 명령어 예시는 `references/plugin-features-reference.md`를 참조하세요.

### 다중 컴포넌트 워크플로우 (Multi-Component Workflows)

에이전트, 스킬 및 스크립트를 결합합니다:

```markdown
---
description: Comprehensive review workflow
argument-hint: [file]
allowed-tools: Bash(node:*), Read
---

Target: @$1

Phase 1 - Static Analysis:
!`node ${CLAUDE_PLUGIN_ROOT}/scripts/lint.js $1`

Phase 2 - Deep Review:
Launch code-reviewer agent for detailed analysis.

Phase 3 - Standards Check:
Use coding-standards skill for validation.

Phase 4 - Report:
Template: @${CLAUDE_PLUGIN_ROOT}/templates/review.md

Compile findings into report following template.
```

**사용 시기:**

- 복잡한 다단계 워크플로우
- 여러 플러그인 기능 활용
- 세분화된 분석 요구
- 구조화된 출력 필요

## 검증 패턴 (Validation Patterns)

명령어는 처리하기 전에 입력 및 리소스를 검증해야 합니다.

### 인수 검증 (Argument Validation)

```markdown
---
description: Deploy with validation
argument-hint: [environment]
---

Validate environment: !`echo "$1" | grep -E "^(dev|staging|prod)$" || echo "INVALID"`

If $1 is valid environment:
Deploy to $1
Otherwise:
Explain valid environments: dev, staging, prod
Show usage: /deploy [environment]
```

### 파일 존재 여부 확인 (File Existence Checks)

```markdown
---
description: Process configuration
argument-hint: [config-file]
---

Check file exists: !`test -f $1 && echo "EXISTS" || echo "MISSING"`

If file exists:
Process configuration: @$1
Otherwise:
Explain where to place config file
Show expected format
Provide example configuration
```

### 플러그인 리소스 검증 (Plugin Resource Validation)

```markdown
---
description: Run plugin analyzer
allowed-tools: Bash(test:*)
---

Validate plugin setup:

- Script: !`test -x ${CLAUDE_PLUGIN_ROOT}/bin/analyze && echo "✓" || echo "✗"`
- Config: !`test -f ${CLAUDE_PLUGIN_ROOT}/config.json && echo "✓" || echo "✗"`

If all checks pass, run analysis.
Otherwise, report missing components.
```

### 에러 처리 (Error Handling)

```markdown
---
description: Build with error handling
allowed-tools: Bash(*)
---

Execute build: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh 2>&1 || echo "BUILD_FAILED"`

If build succeeded:
Report success and output location
If build failed:
Analyze error output
Suggest likely causes
Provide troubleshooting steps
```

**권장 사항:**

- 명령어 초기에 유효성 검사 수행
- 유용한 에러 메시지 제공
- 조치 사항 제안
- 에러 상황을 우아하게 처리

---

자세한 프론트매터 필드 사양은 `references/frontmatter-reference.md`를 참조하세요.
플러그인 전용 기능 및 패턴은 `references/plugin-features-reference.md`를 참조하세요.
명령어 패턴 예시는 `examples/` 디렉토리를 참조하세요.
