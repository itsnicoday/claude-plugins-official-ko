# 고급 워크플로우 패턴 (Advanced Workflow Patterns)

복잡한 워크플로우를 처리하기 위한 다단계 명령어 시퀀스 및 조합 패턴들입니다.

## 개요

고급 워크플로우는 여러 명령어를 결합하고, 여러 실행 간의 상태(state)를 조정하며, 정교한 자동화 시퀀스를 생성합니다. 이러한 패턴들은 단순한 명령어 빌딩 블록으로부터 복잡한 기능을 구축할 수 있게 합니다.

## 다단계 명령어 패턴 (Multi-Step Command Patterns)

### 순차적 워크플로우 명령어 (Sequential Workflow Command)

사용자에게 다단계 프로세스를 가이드하는 명령어:

```markdown
---
description: Complete PR review workflow
argument-hint: [pr-number]
allowed-tools: Bash(gh:*), Read, Grep
---

# PR Review Workflow for #$1

## Step 1: Fetch PR Details
!`gh pr view $1 --json title,body,author,files`

## Step 2: Review Files
Files changed: !`gh pr diff $1 --name-only`

For each file:
- Check code quality
- Verify tests exist
- Review documentation

## Step 3: Run Checks
Test status: !`gh pr checks $1`

Verify:
- All tests passing
- No merge conflicts
- CI/CD successful

## Step 4: Provide Feedback

Summarize:
- Issues found (critical/minor)
- Suggestions for improvement
- Approval recommendation

Would you like to:
1. Approve PR
2. Request changes
3. Leave comments only

Reply with your choice and I'll help complete the action.
```

**주요 특징:**
- 가독성을 위한 순서가 매겨진 단계들
- 콘텍스트 확보를 위한 Bash 실행
- 사용자 입력을 위한 의사 결정 시점
- 다음 행동에 대한 제안

### 상태 유지형 워크플로우 (State-Carrying Workflow)

여러 실행 간에 상태를 유지하는 명령어:

```markdown
---
description: Initialize deployment workflow
allowed-tools: Write, Bash(git:*)
---

# Initialize Deployment

Creating deployment tracking file...

Current branch: !`git branch --show-current`
Latest commit: !`git log -1 --format=%H`

Deployment state saved to `.claude/deployment-state.local.md`:

\`\`\`markdown
---
initialized: true
branch: $(git branch --show-current)
commit: $(git log -1 --format=%H)
timestamp: $(date -u +%Y-%m-%dT%H:%M:%SZ)
status: initialized
---

# Deployment Tracking

Branch: $(git branch --show-current)
Started: $(date)

Next steps:
1. Run tests: /deploy-test
2. Build: /deploy-build
3. Deploy: /deploy-execute
\`\`\`

State saved. Run `/deploy-test` to continue.
```

**다음 명령어** (`/deploy-test`):
```markdown
---
description: Run deployment tests
allowed-tools: Read, Bash(npm:*)
---

Reading deployment state from `.claude/deployment-state.local.md`...

Running tests: !`npm test`

Updating state to 'tested'...

Tests complete. Run `/deploy-build` to continue.
```

**패턴 적용의 장점:**
- 명령어 전반에 걸친 지속적인 상태 관리
- 명확한 워크플로우 진행 상황 확인
- 안전한 체크포인트 설정
- 중단 시 재개 기능

### 조건부 워크플로우 분기 (Conditional Workflow Branching)

조건에 따라 다르게 조율되는 명령어:

```markdown
---
description: Smart deployment workflow
argument-hint: [environment]
allowed-tools: Bash(git:*), Bash(npm:*), Read
---

# Deploy to $1

## Pre-flight Checks

Branch: !`git branch --show-current`
Status: !`git status --short`

**Checking conditions:**

1. Branch status:
   - If main/master: Require approval
   - If feature branch: Warning about target
   - If hotfix: Fast-track process

2. Tests:
   !`npm test`
   - If tests fail: STOP - fix tests first
   - If tests pass: Continue

3. Environment:
   - If $1 = 'production': Extra validation
   - If $1 = 'staging': Standard process
   - If $1 = 'dev': Minimal checks

**Workflow decision:**
Based on above, proceeding with: [determined workflow]

[Conditional steps based on environment and status]

Ready to deploy? (yes/no)
```

## 명령어 조합 패턴 (Command Composition Patterns)

### 명령어 체이닝 (Command Chaining)

함께 동작하도록 설계된 명령어들:

```markdown
---
description: Prepare for code review
---

# Prepare Code Review

Running preparation sequence:

1. Format code: /format-code
2. Run linter: /lint-code
3. Run tests: /test-all
4. Generate coverage: /coverage-report
5. Create review summary: /review-summary

This is a meta-command. After completing each step above,
I'll compile results and prepare comprehensive review materials.

Starting sequence...
```

**개별 명령어**는 간단합니다:
- `/format-code` - 포맷팅만 수행
- `/lint-code` - 린트만 수행
- `/test-all` - 테스트만 실행

**조합 명령어**가 이들을 전반적으로 조정합니다.

### 파이프라인 패턴 (Pipeline Pattern)

이전 명령어의 출력 결과를 처리하는 명령어:

```markdown
---
description: Analyze test failures
---

# Analyze Test Failures

## Step 1: Get test results
(Run /test-all first if not done)

Reading test output...

## Step 2: Categorize failures
- Flaky tests (random failures)
- Consistent failures
- New failures vs existing

## Step 3: Prioritize
Rank by:
- Impact (critical path vs edge case)
- Frequency (always fails vs sometimes)
- Effort (quick fix vs major work)

## Step 4: Generate fix plan
For each failure:
- Root cause hypothesis
- Suggested fix approach
- Estimated effort

Would you like me to:
1. Fix highest priority failure
2. Generate detailed fix plans for all
3. Create GitHub issues for each
```

### 병렬 실행 패턴 (Parallel Execution Pattern)

여러 작업을 동시에 실행하고 제안하는 명령어:

```markdown
---
description: Run comprehensive validation
allowed-tools: Bash(*), Read
---

# Comprehensive Validation

Running validations in parallel...

Starting:
- Code quality checks
- Security scanning
- Dependency audit
- Performance profiling

This will take 2-3 minutes. I'll monitor all processes
and report when complete.

[Poll each process and report progress]

All validations complete. Summary:
- Quality: PASS (0 issues)
- Security: WARN (2 minor issues)
- Dependencies: PASS
- Performance: PASS (baseline met)

Details:
[Collated results from all checks]
```

## 워크플로우 상태 관리 (Workflow State Management)

### .local.md 파일 활용

플러그인 고유 파일에 워크플로우 상태를 저장합니다:

```markdown
.claude/plugin-name-workflow.local.md:

---
workflow: deployment
stage: testing
started: 2025-01-15T10:30:00Z
environment: staging
branch: feature/new-api
commit: abc123def
tests_passed: false
build_complete: false
---

# Deployment Workflow State

Current stage: Testing
Started: 2025-01-15 10:30 UTC

Completed steps:
- ✅ Validation
- ✅ Branch check
- ⏳ Testing (in progress)

Pending steps:
- Build
- Deploy
- Smoke tests
```

**명령어 내에서 상태 읽기:**

```markdown
---
description: Continue deployment workflow
allowed-tools: Read, Write
---

Reading workflow state from .claude/plugin-name-workflow.local.md...

Current stage: @.claude/plugin-name-workflow.local.md

[Parse YAML frontmatter to determine next step]

Next action based on state: [determined action]
```

### 워크플로우 복구 (Workflow Recovery)

중단된 워크플로우 처리:

```markdown
---
description: Resume deployment workflow
allowed-tools: Read
---

# Resume Deployment

Checking for interrupted workflow...

State file: @.claude/plugin-name-workflow.local.md

**Workflow found:**
- Started: [timestamp]
- Environment: [env]
- Last completed: [step]

**Recovery options:**
1. Resume from last step
2. Restart from beginning
3. Abort and clean up

Which would you like? (1/2/3)
```

## 워크플로우 조정 패턴 (Workflow Coordination Patterns)

### 명령어 간 통신 (Cross-Command Communication)

서로 신호를 주고받는 명령어들:

```markdown
---
description: Mark feature complete
allowed-tools: Write
---

# Mark Feature Complete

Writing completion marker...

Creating: .claude/feature-complete.flag

This signals other commands that feature is ready for:
- Integration testing (/integration-test will auto-detect)
- Documentation generation (/docs-generate will include)
- Release notes (/release-notes will add)

Feature marked complete.
```

**다른 명령어에서 플래그 확인:**

```markdown
---
description: Generate release notes
allowed-tools: Read, Bash(git:*)
---

Checking for completed features...

if [ -f .claude/feature-complete.flag ]; then
  Feature ready for release notes
fi

[Include in release notes]
```

### 워크플로우 락킹 (Workflow Locking)

워크플로우의 동시 실행 방지:

```markdown
---
description: Start deployment
allowed-tools: Read, Write, Bash
---

# Start Deployment

Checking for active deployments...

if [ -f .claude/deployment.lock ]; then
  ERROR: Deployment already in progress
  Started: [timestamp from lock file]

  Cannot start concurrent deployment.
  Wait for completion or run /deployment-abort

  Exit.
fi

Creating deployment lock...

Deployment started. Lock created.
[Proceed with deployment]
```

**락 해제:**

```markdown
---
description: Complete deployment
allowed-tools: Write, Bash
---

Deployment complete.

Removing deployment lock...
rm .claude/deployment.lock

Ready for next deployment.
```

## 고급 인자 처리 (Advanced Argument Handling)

### 기본값이 있는 선택적 인자

```markdown
---
description: Deploy with optional version
argument-hint: [environment] [version]
---

Environment: ${1:-staging}
Version: ${2:-latest}

Deploying ${2:-latest} to ${1:-staging}...

Note: Using defaults for missing arguments:
- Environment defaults to 'staging'
- Version defaults to 'latest'
```

### 인자 검증

```markdown
---
description: Deploy to validated environment
argument-hint: [environment]
---

Environment: $1

Validating environment...

valid_envs="dev staging production"
if ! echo "$valid_envs" | grep -w "$1" > /dev/null; then
  ERROR: Invalid environment '$1'
  Valid options: dev, staging, production
  Exit.
fi

Environment validated. Proceeding...
```

### 인자 변환

```markdown
---
description: Deploy with shorthand
argument-hint: [env-shorthand]
---

Input: $1

Expanding shorthand:
- d/dev → development
- s/stg → staging
- p/prod → production

case "$1" in
  d|dev) ENV="development";;
  s|stg) ENV="staging";;
  p|prod) ENV="production";;
  *) ENV="$1";;
esac

Deploying to: $ENV
```

## 워크플로우 내 에러 처리 (Workflow Handling)

### 부드러운 실패 처리 (Graceful Failure)

```markdown
---
description: Resilient deployment workflow
---

# Deployment Workflow

Running steps with error handling...

## Step 1: Tests
!`npm test`

if [ $? -ne 0 ]; then
  ERROR: Tests failed

  Options:
  1. Fix tests and retry
  2. Skip tests (NOT recommended)
  3. Abort deployment

  What would you like to do?

  [Wait for user input before continuing]
fi

## Step 2: Build
[Continue only if Step 1 succeeded]
```

### 실패 시 롤백 (Rollback on Failure)

```markdown
---
description: Deployment with rollback
---

# Deploy with Rollback

Saving current state for rollback...
Previous version: !`current-version.sh`

Deploying new version...

!`deploy.sh`

if [ $? -ne 0 ]; then
  DEPLOYMENT FAILED

  Initiating automatic rollback...
  !`rollback.sh`

  Rolled back to previous version.
  Check logs for failure details.
fi

Deployment complete.
```

### 체크포인트 복구 (Checkpoint Recovery)

```markdown
---
description: Workflow with checkpoints
---

# Multi-Stage Deployment

## Checkpoint 1: Validation
!`validate.sh`
echo "checkpoint:validation" >> .claude/deployment-checkpoints.log

## Checkpoint 2: Build
!`build.sh`
echo "checkpoint:build" >> .claude/deployment-checkpoints.log

## Checkpoint 3: Deploy
!`deploy.sh`
echo "checkpoint:deploy" >> .claude/deployment-checkpoints.log

If any step fails, resume with:
/deployment-resume [last-successful-checkpoint]
```

## 베스트 프랙티스

### 워크플로우 디자인:

1. **명확한 진행 흐름**: 단계별로 번호를 매기고, 현재 위치를 표시합니다.
2. **명시적인 상태 관리**: 암묵적인 상태에 의존하지 마세요.
3. **사용자 제어권 보장**: 의사 결정 시점을 제공합니다.
4. **에이전트 복구**: 실패를 유연하게 처리합니다.
5. **진행 상황 인디케이터**: 완료된 작업과 남은 작업을 명확히 보여줍니다.

### 명령어 조합:

1. **단일 책임 원칙**: 각 명령어는 단 하나의 일만 명확하게 잘 처리해야 합니다.
2. **조합 가능한 디자인**: 명령어들이 쉽게 묶여서 연동될 수 있어야 합니다.
3. **표준 인터페이스**: 입력/출력 포맷의 일관성을 유지합니다.
4. **느슨한 결합**: 명령어가 서로의 내부 로직에 강하게 종속되지 않도록 설계합니다.

### 상태 관리:

1. **지속적인 상태 관리**: .local.md 파일을 활용하세요.
2. **원자적 업데이트**: 상태 파일 전체를 원자적으로 작성합니다.
3. **상태 검증**: 상태 파일의 포맷 및 무결성을 검사합니다.
4. **정리**: 오래되어 쓸모없어진 상태 파일은 즉시 제거합니다.
5. **문서화**: 상태 파일 포맷에 대해 꼼꼼하게 문서로 기록해 둡니다.

### 에러 처리:

1. **Fail fast**: 에러는 가급적 초기에 감지하여 실패 처리합니다.
2. **명확한 메시지**: 무엇이 잘못되었는지 자세히 설명합니다.
3. **복구 옵션 제공**: 명확한 복구 단계와 다음 행동 지침을 제시합니다.
4. **상태 보존**: 복구를 위해 현재 상태 정보를 보존합니다.
5. **롤백 기능**: 이전 변경 사항을 되돌릴 수 있는 기능을 지원합니다.

## 예시: 종합 배포 워크플로우

### 초기화 명령어

```markdown
---
description: Initialize deployment
argument-hint: [environment]
allowed-tools: Write, Bash(git:*)
---

# Initialize Deployment to $1

Creating workflow state...

\`\`\`yaml
---
workflow: deployment
environment: $1
branch: !`git branch --show-current`
commit: !`git rev-parse HEAD`
stage: initialized
timestamp: !`date -u +%Y-%m-%dT%H:%M:%SZ`
---
\`\`\`

Written to .claude/deployment-state.local.md

Next: Run /deployment-validate
```

### 검증 명령어

```markdown
---
description: Validate deployment
allowed-tools: Read, Bash
---

Reading state: @.claude/deployment-state.local.md

Running validation...
- Branch check: PASS
- Tests: PASS
- Build: PASS

Updating state to 'validated'...

Next: Run /deployment-execute
```

### 실행 명령어

```markdown
---
description: Execute deployment
allowed-tools: Read, Bash, Write
---

Reading state: @.claude/deployment-state.local.md

Executing deployment to [environment]...

!`deploy.sh [environment]`

Deployment complete.
Updating state to 'completed'...

Cleanup: /deployment-cleanup
```

### 정리 명령어

```markdown
---
description: Clean up deployment
allowed-tools: Bash
---

Removing deployment state...
rm .claude/deployment-state.local.md

Deployment workflow complete.
```

이 종합 워크플로우 예시는 여러 명령어 전반에 걸친 상태 관리, 순차적 실행, 에러 처리, 그리고 명확한 관심사 분리(separation of concerns)를 잘 보여줍니다.
