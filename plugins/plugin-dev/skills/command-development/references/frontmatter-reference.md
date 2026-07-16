# 명령어 프론트매터 참조서 (Command Frontmatter Reference)

슬래시 명령어의 YAML 프론트매터 필드에 대한 종합 참조서입니다.

## 프론트매터 개요

YAML 프론트매터는 명령어 파일 시작 부분에 작성하는 선택적 메타데이터입니다:

```markdown
---
description: Brief description
allowed-tools: Read, Write
model: sonnet
argument-hint: [arg1] [arg2]
---

Command prompt content here...
```

모든 필드는 선택 사항입니다. 프론트매터 없이도 명령어가 정상 작동합니다.

## 필드 상세 사양

### description

**타입:** String
**필수 여부:** No
**기본값:** 명령어 프롬프트의 첫 번째 줄
**최대 길이:** `/help` 화면 표시를 위해 약 60자 권장

**목적:** 명령어의 역할을 설명하며, `/help` 출력 시 나타납니다.

**예시:**
```yaml
description: Review code for security issues
```
```yaml
description: Deploy to staging environment
```
```yaml
description: Generate API documentation
```

**권장 가이드:**
- 깔끔한 화면 표시를 위해 60자 미만으로 유지하세요.
- 동사(Review, Deploy, Generate 등)로 시작하세요.
- 명령어가 수행하는 작업을 구체적으로 나타내세요.
- 불필요한 "명령어" 또는 "슬래시 명령어" 같은 단어의 중복 사용은 피하세요.

**올바른 예시:**
- ✅ "Review PR for code quality and security"
- ✅ "Deploy application to specified environment"
- ✅ "Generate comprehensive API documentation"

**피해야 할 예시:**
- ❌ "This command reviews PRs" (불필요한 "This command" 포함)
- ❌ "Review" (너무 모호함)
- ❌ "A command that reviews pull requests for code quality, security issues, and best practices" (너무 긺)

### allowed-tools

**타입:** String 또는 String 배열
**필수 여부:** No
**기본값:** 대화 권한(conversation permissions)을 상속받음

**목적:** 명령어가 사용할 수 있는 도구를 제한하거나 지정합니다.

**형식:**

**단일 도구:**
```yaml
allowed-tools: Read
```

**여러 도구 (쉼표로 구분):**
```yaml
allowed-tools: Read, Write, Edit
```

**여러 도구 (배열 형태):**
```yaml
allowed-tools:
  - Read
  - Write
  - Bash(git:*)
```

**도구 패턴 (Tool Patterns):**

**특정 도구 지정:**
```yaml
allowed-tools: Read, Grep, Edit
```

**명령어 필터가 적용된 Bash:**
```yaml
allowed-tools: Bash(git:*)           # git 명령어만 허용
allowed-tools: Bash(npm:*)           # npm 명령어만 허용
allowed-tools: Bash(docker:*)        # docker 명령어만 허용
```

**모든 도구 허용 (권장하지 않음):**
```yaml
allowed-tools: "*"
```

**사용 시점:**

1. **보안:** 명령어를 안전한 작업으로만 제한
   ```yaml
   allowed-tools: Read, Grep  # 읽기 전용 명령어
   ```

2. **명확성:** 필요한 도구를 명확히 문서화
   ```yaml
   allowed-tools: Bash(git:*), Read
   ```

3. **Bash 실행:** bash 명령어 출력을 활성화
   ```yaml
   allowed-tools: Bash(git status:*), Bash(git diff:*)
   ```

**권장 가이드:**
- 가능한 한 가장 보수적으로(좁은 범위로) 제한하세요.
- Bash의 경우 명령어 필터를 사용하세요 (예: `*` 대신 `git:*`).
- 대화 권한과 달라야 하는 경우에만 지정하세요.
- 특정 도구가 필요한 이유를 문서로 남겨주세요.

### model

**타입:** String
**필수 여부:** No
**기본값:** 대화 설정을 상속받음
**지원 값:** `sonnet`, `opus`, `haiku`

**목적:** 명령어를 실행할 Claude 모델을 지정합니다.

**예시:**
```yaml
model: haiku    # 단순 작업에 빠르고 효율적임
```
```yaml
model: sonnet   # 균형 잡힌 성능 (기본값)
```
```yaml
model: opus     # 복잡한 작업에 최적의 능력 발휘
```

**사용 시점:**

**haiku 사용 권장 시점:**
- 단순하고 정형화된 명령어
- 빠른 실행이 필요한 경우
- 복잡도가 낮은 작업
- 자주 실행되는 경우

```yaml
---
description: Format code file
model: haiku
---
```

**sonnet 사용 권장 시점:**
- 표준 명령어 (기본값)
- 속도와 품질의 균형이 필요한 경우
- 대부분의 일반적인 사용 사례

```yaml
---
description: Review code changes
model: sonnet
---
```

**opus 사용 권장 시점:**
- 복잡한 분석 작업
- 아키텍처 의사 결정
- 심도 있는 코드 이해
- 대단히 중요한 작업

```yaml
---
description: Analyze system architecture
model: opus
---
```

**권장 가이드:**
- 특별한 이유가 없는 한 생략하세요.
- 속도를 위해 가능한 한 `haiku`를 사용하세요.
- `opus`는 정말 복잡한 작업에만 제한적으로 사용하세요.
- 적절한 균형을 찾기 위해 다양한 모델로 테스트해 보세요.

### argument-hint

**타입:** String
**필수 여부:** No
**기본값:** 없음

**목적:** 사용자와 자동 완성을 위해 예상되는 인자(arguments)를 문서화합니다.

**형식:**
```yaml
argument-hint: [arg1] [arg2] [optional-arg]
```

**예시:**

**단일 인자:**
```yaml
argument-hint: [pr-number]
```

**여러 필수 인자:**
```yaml
argument-hint: [environment] [version]
```

**선택적 인자:**
```yaml
argument-hint: [file-path] [options]
```

**설명적인 이름 사용:**
```yaml
argument-hint: [source-branch] [target-branch] [commit-message]
```

**권장 가이드:**
- 각 인자에 대해 대괄호 `[]`를 사용하세요.
- `arg1`, `arg2` 대신 설명적인 이름을 사용하세요.
- 설명 필드에서 필수 인자와 선택 인자를 구분해 표기하세요.
- 명령어의 위치 인자(positional arguments) 순서와 일치시키세요.
- 간결하면서도 명확하게 유지하세요.

**패턴별 예시:**

**단순한 명령어:**
```yaml
---
description: Fix issue by number
argument-hint: [issue-number]
---

Fix issue #$1...
```

**다중 인자:**
```yaml
---
description: Deploy to environment
argument-hint: [app-name] [environment] [version]
---

Deploy $1 to $2 using version $3...
```

**옵션 포함:**
```yaml
---
description: Run tests with options
argument-hint: [test-pattern] [options]
---

Run tests matching $1 with options: $2
```

### disable-model-invocation

**타입:** Boolean
**필수 여부:** No
**기본값:** false

**목적:** SlashCommand 도구가 프로그래밍 방식으로 명령어를 트리거하는 것을 방지합니다.

**예시:**
```yaml
disable-model-invocation: true
```

**사용 시점:**

1. **수동 실행 전용 명령어:** 사용자의 판단이 필요한 명령어
   ```yaml
   ---
   description: Approve deployment to production
   disable-model-invocation: true
   ---
   ```

2. **파괴적인 작업:** 되돌릴 수 없는 효과가 있는 명령어
   ```yaml
   ---
   description: Delete all test data
   disable-model-invocation: true
   ---
   ```

3. **인터랙티브 워크플로우:** 사용자 입력이 필요한 명령어
   ```yaml
   ---
   description: Walk through setup wizard
   disable-model-invocation: true
   ---
   ```

**기본 동작 (false):**
- SlashCommand 도구에서 명령어를 사용할 수 있음.
- Claude가 프로그래밍 방식으로 명령어를 호출할 수 있음.
- 사용자가 수동으로도 호출 가능.

**true로 설정된 경우:**
- 사용자가 직접 `/command`를 타이핑해서만 실행 가능.
- SlashCommand 도구에 노출되지 않음.
- 민감한 작업 수행 시 안전함.

**권장 가이드:**
- 신중하게 제한적으로 사용하세요 (Claude의 자율성을 제한하므로).
- 명령어 주석에 그 이유를 기록해 두세요.
- 항상 수동으로만 실행해야 한다면, 굳이 명령어로 존재해야 하는지 검토하세요.

## 전체 코드 예시

### 최소 구성 명령어

프론트매터가 필요 없는 경우:

```markdown
Review this code for common issues and suggest improvements.
```

### 단순한 명령어

설명만 기재된 경우:

```markdown
---
description: Review code for issues
---

Review this code for common issues and suggest improvements.
```

### 표준 명령어

설명 및 도구 지정:

```markdown
---
description: Review Git changes
allowed-tools: Bash(git:*), Read
---

Current changes: !`git diff --name-only`

Review each changed file for:
- Code quality
- Potential bugs
- Best practices
```

### 복잡한 명령어

모든 일반 필드 포함:

```markdown
---
description: Deploy application to environment
argument-hint: [app-name] [environment] [version]
allowed-tools: Bash(kubectl:*), Bash(helm:*), Read
model: sonnet
---

Deploy $1 to $2 environment using version $3

Pre-deployment checks:
- Verify $2 configuration
- Check cluster status: !`kubectl cluster-info`
- Validate version $3 exists

Proceed with deployment following deployment runbook.
```

### 수동 실행 전용 명령어

호출 제한 적용:

```markdown
---
description: Approve production deployment
argument-hint: [deployment-id]
disable-model-invocation: true
allowed-tools: Bash(gh:*)
---

<!--
MANUAL APPROVAL REQUIRED
This command requires human judgment and cannot be automated.
-->

Review deployment $1 for production approval:

Deployment details: !`gh api /deployments/$1`

Verify:
- All tests passed
- Security scan clean
- Stakeholder approval
- Rollback plan ready

Type "APPROVED" to confirm deployment.
```

## 검증 (Validation)

### 흔한 실수

**유효하지 않은 YAML 구문:**
```yaml
---
description: Missing quote
allowed-tools: Read, Write
model: sonnet
---  # ❌ 위 라인에서 닫는 따옴표가 누락됨
```

**해결책:** YAML 구문 검증 진행

**잘못된 도구 지정:**
```yaml
allowed-tools: Bash  # ❌ 명령어 필터 누락
```

**해결책:** `Bash(git:*)` 형식 사용

**유효하지 않은 모델 이름:**
```yaml
model: gpt4  # ❌ 유효한 Claude 모델이 아님
```

**해결책:** `sonnet`, `opus` 또는 `haiku` 사용

### 검증 체크리스트

명령어 커밋 전 확인 사항:
- [ ] YAML 구문이 유효한가 (에러가 없는가)
- [ ] 설명이 60자 미만인가
- [ ] allowed-tools가 올바른 형식을 사용했는가
- [ ] 모델이 지정된 경우 유효한 값인가
- [ ] argument-hint가 위치 인자(positional arguments)와 일치하는가
- [ ] disable-model-invocation이 적절하게 사용되었는가

## 권장 가이드 요약 (Best Practices Summary)

1. **최소한으로 시작:** 필요한 경우에만 프론트매터 추가
2. **인자 문서화:** 인자가 있는 경우 항상 argument-hint 사용
3. **도구 제한:** 가능한 한 가장 제한된 allowed-tools 사용
4. **적절한 모델 선택:** 속도는 haiku, 복잡한 작업은 opus 사용
5. **수동 전용은 신중히:** 꼭 필요한 경우에만 disable-model-invocation 사용
6. **명확한 설명:** `/help`에서 명령어를 쉽게 발견할 수 있도록 작성
7. **철저한 테스트:** 프론트매터가 예상대로 작동하는지 검증
