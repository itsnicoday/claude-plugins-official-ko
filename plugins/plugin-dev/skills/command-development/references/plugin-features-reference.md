# 플러그인 전용 명령어 기능 참조서 (Plugin-Specific Command Features Reference)

이 참조서는 Claude Code 플러그인에 내장된 명령어 고유의 기능 및 패턴을 다룹니다.

## 목차

- [플러그인 명령어 감지](#plugin-command-discovery)
- [CLAUDE_PLUGIN_ROOT 환경 변수](#claude_plugin_root-환경-변수)
- [플러그인 명령어 패턴](#plugin-command-patterns)
- [플러그인 구성 요소와의 연동](#integration-with-plugin-components)
- [검증 패턴](#validation-patterns)

## 플러그인 명령어 감지 (Plugin Command Discovery)

### 자동 감지 (Auto-Discovery)

Claude Code는 다음 위치에서 플러그인의 명령어를 자동으로 감지합니다:

```
plugin-name/
├── commands/              # 자동 감지되는 명령어 디렉터리
│   ├── foo.md            # /foo (plugin:plugin-name)
│   └── bar.md            # /bar (plugin:plugin-name)
└── plugin.json           # 플러그인 매니페스트
```

**핵심 사항:**
- 플러그인 로드 시점에 명령어가 감지됩니다.
- 수동 등록이 필요하지 않습니다.
- 명령어는 `/help`에서 "(plugin:plugin-name)" 라벨과 함께 표시됩니다.
- 하위 디렉터리는 네임스페이스를 생성합니다.

### 네임스페이스가 지정된 플러그인 명령어 (Namespaced Plugin Commands)

논리적 그룹화를 위해 하위 디렉터리에 명령어를 구성합니다:

```
plugin-name/
└── commands/
    ├── review/
    │   ├── security.md    # /security (plugin:plugin-name:review)
    │   └── style.md       # /style (plugin:plugin-name:review)
    └── deploy/
        ├── staging.md     # /staging (plugin:plugin-name:deploy)
        └── prod.md        # /prod (plugin:plugin-name:deploy)
```

**네임스페이스 동작 방식:**
- 하위 디렉터리 이름이 네임스페이스가 됩니다.
- `/help`에서 "(plugin:plugin-name:namespace)" 형식으로 표시됩니다.
- 관련된 명령어들을 쉽게 정리하도록 돕습니다.
- 플러그인에 5개 이상의 명령어가 있을 때 사용합니다.

### 명령어 명명 규칙 (Command Naming Conventions)

**플러그인 명령어 이름은 다음과 같아야 합니다:**
1. 설명적이고 동작 지향적이어야 합니다.
2. 일반적인 명령어 이름과의 충돌을 피해야 합니다.
3. 여러 단어로 구성된 이름에는 하이픈을 사용해야 합니다.
4. 고유성을 위해 플러그인 이름을 접두사로 사용하는 것을 고려해야 합니다.

**예시:**
```
올바른 예시:
- /mylyn-sync          (플러그인 전용 접두사 사용)
- /analyze-performance (설명적인 액션)
- /docker-compose-up   (명확한 용도)

피해야 할 예시:
- /test               (일반적인 이름과 충돌 발생 가능)
- /run                (너무 범용적임)
- /do-stuff           (설명적이지 않음)
```

## CLAUDE_PLUGIN_ROOT 환경 변수

### 목적

`${CLAUDE_PLUGIN_ROOT}`는 플러그인 명령어 내에서 사용할 수 있는 특수한 환경 변수로, 플러그인 디렉터리의 절대 경로로 해상(resolve)됩니다.

**중요한 이유:**
- 플러그인 내부 경로의 이식성을 높여줍니다.
- 플러그인 내의 파일과 스크립트를 참조할 수 있게 합니다.
- 서로 다른 설치 환경에서도 문제없이 작동합니다.
- 다중 파일 플러그인 작업에 필수적입니다.

### 기본 사용법

플러그인 내부의 파일 참조 예시:

```markdown
---
description: Analyze using plugin script
allowed-tools: Bash(node:*), Read
---

Run analysis: !`node ${CLAUDE_PLUGIN_ROOT}/scripts/analyze.js`

Read template: @${CLAUDE_PLUGIN_ROOT}/templates/report.md
```

**다음과 같이 확장됩니다:**
```
Run analysis: !`node /path/to/plugins/plugin-name/scripts/analyze.js`

Read template: @/path/to/plugins/plugin-name/templates/report.md
```

### 일반적인 패턴

#### 1. 플러그인 스크립트 실행

```markdown
---
description: Run custom linter from plugin
allowed-tools: Bash(node:*)
---

Lint results: !`node ${CLAUDE_PLUGIN_ROOT}/bin/lint.js $1`

Review the linting output and suggest fixes.
```

#### 2. 설정 파일 로드

```markdown
---
description: Deploy using plugin configuration
allowed-tools: Read, Bash(*)
---

Configuration: @${CLAUDE_PLUGIN_ROOT}/config/deploy-config.json

Deploy application using the configuration above for $1 environment.
```

#### 3. 플러그인 리소스 접근

```markdown
---
description: Generate report from template
---

Use this template: @${CLAUDE_PLUGIN_ROOT}/templates/api-report.md

Generate a report for @$1 following the template format.
```

#### 4. 다단계 플러그인 워크플로우

```markdown
---
description: Complete plugin workflow
allowed-tools: Bash(*), Read
---

Step 1 - Prepare: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/prepare.sh $1`
Step 2 - Config: @${CLAUDE_PLUGIN_ROOT}/config/$1.json
Step 3 - Execute: !`${CLAUDE_PLUGIN_ROOT}/bin/execute $1`

Review results and report status.
```

### 권장 가이드 (Best Practices)

1. **항상 플러그인 내부 경로에 사용하세요:**
   ```markdown
   # 올바른 사용법
   @${CLAUDE_PLUGIN_ROOT}/templates/foo.md

   # 잘못된 사용법
   @./templates/foo.md  # 플러그인이 아닌 현재 작업 디렉터리를 기준으로 상대 경로를 잡음
   ```

2. **파일 존재 여부 확인:**
   ```markdown
   ---
   description: Use plugin config if exists
   allowed-tools: Bash(test:*), Read
   ---

   !`test -f ${CLAUDE_PLUGIN_ROOT}/config.json && echo "exists" || echo "missing"`

   If config exists, load it: @${CLAUDE_PLUGIN_ROOT}/config.json
   Otherwise, use defaults...
   ```

3. **플러그인 파일 구조 문서화:**
   ```markdown
   <!--
   Plugin structure:
   ${CLAUDE_PLUGIN_ROOT}/
   ├── scripts/analyze.js  (analysis script)
   ├── templates/          (report templates)
   └── config/             (configuration files)
   -->
   ```

4. **인자(arguments)와 결합하여 사용:**
   ```markdown
   Run: !`${CLAUDE_PLUGIN_ROOT}/bin/process.sh $1 $2`
   ```

### 문제 해결

**변수가 확장되지 않는 경우:**
- 명령어가 플러그인으로부터 올바르게 로드되었는지 확인하세요.
- bash 실행 권한이 허용되어 있는지 확인하세요.
- 구문이 정확히 `${CLAUDE_PLUGIN_ROOT}`인지 확인하세요.

**파일을 찾을 수 없음 에러:**
- 파일이 플러그인 디렉터리에 존재하는지 확인하세요.
- 플러그인 루트에 대해 파일 경로가 올바른지 확인하세요.
- 파일 권한이 읽기/실행을 허용하는지 확인하세요.

**경로에 공백이 포함된 경우:**
- Bash 명령어는 공백을 자동으로 처리합니다.
- 파일 참조는 경로에 공백이 있어도 작동합니다.
- 특별한 따옴표 처리가 필요하지 않습니다.

## 플러그인 명령어 패턴 (Plugin Command Patterns)

### 패턴 1: 설정 기반 명령어 (Configuration-Based Commands)

플러그인 고유의 설정을 로드하는 명령어:

```markdown
---
description: Deploy using plugin settings
allowed-tools: Read, Bash(*)
---

Load configuration: @${CLAUDE_PLUGIN_ROOT}/deploy-config.json

Deploy to $1 environment using:
1. Configuration settings above
2. Current git branch: !`git branch --show-current`
3. Application version: !`cat package.json | grep version`

Execute deployment and monitor progress.
```

**사용 시점:** 매번 실행할 때마다 일관된 설정이 필요한 명령어

### 패턴 2: 템플릿 기반 생성 (Template-Based Generation)

플러그인 템플릿을 사용하는 명령어:

```markdown
---
description: Generate documentation from template
argument-hint: [component-name]
---

Template: @${CLAUDE_PLUGIN_ROOT}/templates/component-docs.md

Generate documentation for $1 component following the template structure.
Include:
- Component purpose and usage
- API reference
- Examples
- Testing guidelines
```

**사용 시점:** 표준화된 결과물 생성

### 패턴 3: 다중 스크립트 워크플로우 (Multi-Script Workflow)

여러 플러그인 스크립트를 조정하는 명령어:

```markdown
---
description: Complete build and test workflow
allowed-tools: Bash(*)
---

Build: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh`
Validate: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh`
Test: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/test.sh`

Review all outputs and report:
1. Build status
2. Validation results
3. Test results
4. Recommended next steps
```

**사용 시점:** 여러 단계를 거치는 복잡한 플러그인 워크플로우

### 패턴 4: 환경 인지형 명령어 (Environment-Aware Commands)

환경에 맞게 적응하는 명령어:

```markdown
---
description: Deploy based on environment
argument-hint: [dev|staging|prod]
---

Environment config: @${CLAUDE_PLUGIN_ROOT}/config/$1.json

Environment check: !`echo "Deploying to: $1"`

Deploy application using $1 environment configuration.
Verify deployment and run smoke tests.
```

**사용 시점:** 환경마다 다르게 동작해야 하는 명령어

### 패턴 5: 플러그인 데이터 관리 (Plugin Data Management)

플러그인 전용 데이터를 관리하는 명령어:

```markdown
---
description: Save analysis results to plugin cache
allowed-tools: Bash(*), Read, Write
---

Cache directory: ${CLAUDE_PLUGIN_ROOT}/cache/

Analyze @$1 and save results to cache:
!`mkdir -p ${CLAUDE_PLUGIN_ROOT}/cache && date > ${CLAUDE_PLUGIN_ROOT}/cache/last-run.txt`

Store analysis for future reference and comparison.
```

**사용 시점:** 지속적인 데이터 저장이 필요한 명령어

## 플러그인 구성 요소와의 연동 (Integration with Plugin Components)

### 플러그인 에이전트 호출 (Invoking Plugin Agents)

명령어는 Task 도구를 사용하여 플러그인 에이전트를 트리거할 수 있습니다:

```markdown
---
description: Deep analysis using plugin agent
argument-hint: [file-path]
---

Initiate deep code analysis of @$1 using the code-analyzer agent.

The agent will:
1. Analyze code structure
2. Identify patterns
3. Suggest improvements
4. Generate detailed report

Note: This uses the Task tool to launch the plugin's code-analyzer agent.
```

**핵심 사항:**
- 에이전트는 플러그인의 `agents/` 디렉터리에 정의되어 있어야 합니다.
- Claude는 에이전트를 시작하기 위해 Task 도구를 자동으로 사용합니다.
- 에이전트는 동일한 플러그인 리소스에 접근할 수 있습니다.

### 플러그인 스킬 호출 (Invoking Plugin Skills)

명령어는 전문 지식을 얻기 위해 플러그인 스킬을 참조할 수 있습니다:

```markdown
---
description: API documentation with best practices
argument-hint: [api-file]
---

Document the API in @$1 following our API documentation standards.

Use the api-docs-standards skill to ensure documentation includes:
- Endpoint descriptions
- Parameter specifications
- Response formats
- Error codes
- Usage examples

Note: This leverages the plugin's api-docs-standards skill for consistency.
```

**핵심 사항:**
- 스킬은 플러그인의 `skills/` 디렉터리에 정의되어 있어야 합니다.
- Claude가 해당 스킬을 호출하도록 유도하기 위해 이름으로 스킬을 언급하세요.
- 스킬은 전문적인 도메인 지식을 제공합니다.

### 플러그인 훅과의 연계 (Coordinating with Plugin Hooks)

명령어는 플러그인 훅과 연동하여 작동하도록 설계할 수 있습니다:

```markdown
---
description: Commit with pre-commit validation
allowed-tools: Bash(git:*)
---

Stage changes: !\`git add $1\`

Commit changes: !\`git commit -m "$2"\`

Note: This commit will trigger the plugin's pre-commit hook for validation.
Review hook output for any issues.
```

**핵심 사항:**
- 훅은 이벤트 발생 시 자동으로 실행됩니다.
- 명령어는 훅을 위한 상태(state)를 준비할 수 있습니다.
- 명령어 파일에 훅과의 상호작용 내용을 기록해 둡니다.

### 다중 구성 요소 플러그인 명령어 (Multi-Component Plugin Commands)

여러 플러그인 구성 요소를 조정하는 명령어:

```markdown
---
description: Comprehensive code review workflow
argument-hint: [file-path]
---

File to review: @$1

Execute comprehensive review:

1. **Static Analysis** (via plugin scripts)
   !`node ${CLAUDE_PLUGIN_ROOT}/scripts/lint.js $1`

2. **Deep Review** (via plugin agent)
   Launch the code-reviewer agent for detailed analysis.

3. **Best Practices** (via plugin skill)
   Use the code-standards skill to ensure compliance.

4. **Documentation** (via plugin template)
   Template: @${CLAUDE_PLUGIN_ROOT}/templates/review-report.md

Generate final report combining all outputs.
```

**사용 시점:** 여러 플러그인 기능을 레버리지하는 복잡한 워크플로우

## 검증 패턴 (Validation Patterns)

### 입력값 검증 (Input Validation)

명령어는 처리하기 전에 입력값을 검증해야 합니다:

```markdown
---
description: Deploy to environment with validation
argument-hint: [environment]
---

Validate environment: !`echo "$1" | grep -E "^(dev|staging|prod)$" || echo "INVALID"`

$IF($1 in [dev, staging, prod],
  Deploy to $1 environment using validated configuration,
  ERROR: Invalid environment '$1'. Must be one of: dev, staging, prod
)
```

**검증 접근 방식:**
1. grep/test를 활용한 Bash 검증
2. 프롬프트 내의 인라인 검증
3. 스크립트 기반 검증

### 파일 존재 여부 확인 (File Existence Checks)

필수 파일이 존재하는지 검증합니다:

```markdown
---
description: Process configuration file
argument-hint: [config-file]
---

Check file: !`test -f $1 && echo "EXISTS" || echo "MISSING"`

Process configuration if file exists: @$1

If file doesn't exist, explain:
- Expected location
- 요구되는 파일 형식
- 생성 방법
```

### 필수 인자 (Required Arguments)

필수 인자가 제공되었는지 검증합니다:

```markdown
---
description: Create deployment with version
argument-hint: [environment] [version]
---

Validate inputs: !`test -n "$1" -a -n "$2" && echo "OK" || echo "MISSING"`

$IF($1 AND $2,
  Deploy version $2 to $1 environment,
  ERROR: Both environment and version required. Usage: /deploy [env] [version]
)
```

### 플러그인 리소스 검증 (Plugin Resource Validation)

플러그인 리소스가 사용 가능한지 검증합니다:

```markdown
---
description: Run analysis with plugin tools
allowed-tools: Bash(test:*)
---

Validate plugin setup:
- Config exists: !`test -f ${CLAUDE_PLUGIN_ROOT}/config.json && echo "✓" || echo "✗"`
- Scripts exist: !`test -d ${CLAUDE_PLUGIN_ROOT}/scripts && echo "✓" || echo "✗"`
- Tools available: !`test -x ${CLAUDE_PLUGIN_ROOT}/bin/analyze && echo "✓" || echo "✗"`

모든 검사를 통과하면 분석을 진행합니다.
통과하지 못한 경우, 누락된 구성 요소와 설치 단계를 보고합니다.
```

### 결과물 검증 (Output Validation)

명령어 실행 결과를 검증합니다:

```markdown
---
description: Build and validate output
allowed-tools: Bash(*)
---

Build: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh`

Validate output:
- Exit code: !`echo $?`
- Output exists: !`test -d dist && echo "✓" || echo "✗"`
- File count: !`find dist -type f | wc -l`

빌드 상태 및 검증 실패 내역을 보고합니다.
```

### 부드러운 에러 처리 (Graceful Error Handling)

도움이 되는 메시지와 함께 에러를 부드럽게 처리합니다:

```markdown
---
description: Process file with error handling
argument-hint: [file-path]
---

Try processing: !`node ${CLAUDE_PLUGIN_ROOT}/scripts/process.js $1 2>&1 || echo "ERROR: $?"`

If processing succeeded:
- 결과 보고
- 다음 단계 제안

If processing failed:
- 예상 원인 설명
- 트러블슈팅 단계 제공
- 대안적 접근 방식 제안
```

## 권장 가이드 요약 (Best Practices Summary)

### 플러그인 명령어 설계 가이드:

1. **모든 플러그인 내부 경로에 ${CLAUDE_PLUGIN_ROOT} 사용**
   - 스크립트, 템플릿, 설정, 리소스 등

2. **초기에 입력값 검증 진행**
   - 필수 인자 존재 여부 확인
   - 파일 존재 여부 확인
   - 인자 형식 검증

3. **플러그인 구조 문서화**
   - 필요한 파일 목록 설명
   - 스크립트 역할 문서화
   - 종속성 명확화

4. **플러그인 구성 요소와 연동**
   - 복잡한 작업에는 에이전트 참조
   - 전문 지식 활용에는 스킬 사용
   - 관련이 있는 경우 훅과 연계

5. **도움이 되는 에러 메시지 제공**
   - 무엇이 잘못되었는지 설명
   - 수정 방법 제안
   - 대안 제시

6. **예외 상황(Edge Cases) 처리**
   - 파일 누락
   - 유효하지 않은 인자
   - 스크립트 실행 실패
   - 종속성 누락

7. **명령어 목적 단순화**
   - 명령어당 명확한 목적 하나만 설정
   - 복잡한 로직은 스크립트에 위임
   - 다단계 워크플로우에는 에이전트 사용

8. **설치 환경별 테스트 진행**
   - 경로가 어디서든 작동하는지 검증
   - 다양한 인자로 테스트
   - 에러 케이스 검증

---

일반적인 명령어 개발 방법은 메인 SKILL.md 파일을 참고하세요.
명령어 예시는 examples/ 디렉터리를 참고하세요.
