# 명령어 문서화 패턴 (Command Documentation Patterns)

우수한 사용자 경험(UX)을 주면서 동시에 자체 문서화(self-documenting)가 가능하고 유지보수하기 쉬운 명령어를 작성하기 위한 전략입니다.

## 개요

문서화가 잘된 명령어는 사용, 유지보수 및 배포가 훨씬 더 수월합니다. 문서는 명령어 자체 내에 내장되어 있어야 하며, 이를 통해 사용자와 유지관리자가 즉시 접근할 수 있도록 해야 합니다.

## 자체 문서화 명령어 구조

### 종합 명령어 템플릿

```markdown
---
description: Clear, actionable description under 60 chars
argument-hint: [arg1] [arg2] [optional-arg]
allowed-tools: Read, Bash(git:*)
model: sonnet
---

<!--
COMMAND: command-name
VERSION: 1.0.0
AUTHOR: Team Name
LAST UPDATED: 2025-01-15

PURPOSE:
Detailed explanation of what this command does and why it exists.

USAGE:
  /command-name arg1 arg2

ARGUMENTS:
  arg1: Description of first argument (required)
  arg2: Description of second argument (optional, defaults to X)

EXAMPLES:
  /command-name feature-branch main
    → Compares feature-branch with main

  /command-name my-branch
    → Compares my-branch with current branch

REQUIREMENTS:
  - Git repository
  - Branch must exist
  - Permissions to read repository

RELATED COMMANDS:
  /other-command - Related functionality
  /another-command - Alternative approach

TROUBLESHOOTING:
  - If branch not found: Check branch name spelling
  - If permission denied: Check repository access

CHANGELOG:
  v1.0.0 (2025-01-15): Initial release
  v0.9.0 (2025-01-10): Beta version
-->

# Command Implementation

[Command prompt content here...]

[Explain what will happen...]

[Guide user through steps...]

[Provide clear output...]
```

### 문서화 주석 섹션

**PURPOSE**: 명령어가 존재하는 이유
- 해결하고자 하는 문제
- 사용 사례
- 사용 권장 상황 대 피해야 할 상황

**USAGE**: 기본 구문
- 명령어 호출 패턴
- 필수 인자 대 선택적 인자
- 기본값

**ARGUMENTS**: 상세 인자 문서화
- 각 인자별 상세 설명
- 타입 정보
- 유효한 값/범위
- 기본값

**EXAMPLES**: 구체적인 사용 예시
- 일반적인 사용 사례
- 예외 상황
- 예상 출력 결과

**REQUIREMENTS**: 필수 요건
- 종속성
- 권한
- 환경 설정

**RELATED COMMANDS**: 연관 명령어
- 유사한 명령어
- 보완적 명령어
- 대안적 접근 방식

**TROUBLESHOOTING**: 일반적인 문제 해결
- 알려진 문제
- 해결책
- 우회 방법

**CHANGELOG**: 버전 이력
- 시기별 변경 내용
- 중요 변경 사항(breaking changes) 강조
- 마이그레이션 안내

## 인라인 문서화 패턴 (In-Line Documentation Patterns)

### 주석 처리된 섹션

```markdown
---
description: Complex multi-step command
---

<!-- SECTION 1: VALIDATION -->
<!-- This section checks prerequisites before proceeding -->

Checking prerequisites...
- Git repository: !`git rev-parse --git-dir 2>/dev/null`
- Branch exists: [validation logic]

<!-- SECTION 2: ANALYSIS -->
<!-- Analyzes the differences between branches -->

Analyzing differences between $1 and $2...
[Analysis logic...]

<!-- SECTION 3: RECOMMENDATIONS -->
<!-- Provides actionable recommendations -->

Based on analysis, recommend:
[Recommendations...]

<!-- END: Next steps for user -->
```

### 인라인 설명

```markdown
---
description: Deployment command with inline docs
---

# Deploy to $1

## Pre-flight Checks

<!-- We check branch status to prevent deploying from wrong branch -->
Current branch: !`git branch --show-current`

<!-- Production deploys must come from main/master -->
if [ "$1" = "production" ] && [ "$(git branch --show-current)" != "main" ]; then
  ⚠️  WARNING: Not on main branch for production deploy
  This is unusual. Confirm this is intentional.
fi

<!-- Test status ensures we don't deploy broken code -->
Running tests: !`npm test`

  ✓ All checks passed

## Deployment

<!-- Actual deployment happens here -->
<!-- Uses blue-green strategy for zero-downtime -->
Deploying to $1 environment...
[Deployment steps...]

<!-- Post-deployment verification -->
Verifying deployment health...
[Health checks...]

Deployment complete!

## Next Steps

<!-- Guide user on what to do after deployment -->
1. Monitor logs: /logs $1
2. Run smoke tests: /smoke-test $1
3. Notify team: /notify-deployment $1
```

### 의사 결정 시점 문서화

```markdown
---
description: Interactive deployment command
---

# Interactive Deployment

## Configuration Review

Target: $1
Current version: !`cat version.txt`
New version: $2

<!-- DECISION POINT: User confirms configuration -->
<!-- This pause allows user to verify everything is correct -->
<!-- We can't automatically proceed because deployment is risky -->

Review the above configuration.

**Continue with deployment?**
- Reply "yes" to proceed
- Reply "no" to cancel
- Reply "edit" to modify configuration

[Await user input before continuing...]

<!-- After user confirms, we proceed with deployment -->
<!-- All subsequent steps are automated -->

Proceeding with deployment...
```

## 도움말 텍스트 패턴 (Help Text Patterns)

### 내장 도움말 명령어

```markdown
---
description: Main command with help
argument-hint: [subcommand] [args]
---

# Command Processor

if [ "$1" = "help" ] || [ "$1" = "--help" ] || [ "$1" = "-h" ]; then
  **Command Help**

  USAGE:
    /command [subcommand] [args]

  SUBCOMMANDS:
    init [name]       Initialize new configuration
    deploy [env]      Deploy to environment
    status            Show current status
    rollback          Rollback last deployment
    help              Show this help

  EXAMPLES:
    /command init my-project
    /command deploy staging
    /command status
    /command rollback

  For detailed help on a subcommand:
    /command [subcommand] --help

  Exit.
fi

[Regular command processing...]
```

### 콘텍스트 기반 도움말

```markdown
---
description: Context-aware command
argument-hint: [operation] [target]
---

# Context-Aware Operation

if [ -z "$1" ]; then
  **No operation specified**

  Available operations:
  - analyze: Analyze target for issues
  - fix: Apply automatic fixes
  - report: Generate detailed report

  Usage: /command [operation] [target]

  Examples:
    /command analyze src/
    /command fix src/app.js
    /command report

  Run /command help for more details.

  Exit.
fi

[Command continues if operation provided...]
```

## 에러 메시지 문서화

### 도움이 되는 에러 메시지

```markdown
---
description: Command with good error messages
---

# Validation Command

if [ -z "$1" ]; then
  ❌ ERROR: Missing required argument

  The 'file-path' argument is required.

  USAGE:
    /validate [file-path]

  EXAMPLE:
    /validate src/app.js

  Try again with a file path.

  Exit.
fi

if [ ! -f "$1" ]; then
  ❌ ERROR: File not found: $1

  The specified file does not exist or is not accessible.

  COMMON CAUSES:
  1. Typo in file path
  2. File was deleted or moved
  3. Insufficient permissions

  SUGGESTIONS:
  - Check spelling: $1
  - Verify file exists: ls -la $(dirname "$1")
  - Check permissions: ls -l "$1"

  Exit.
fi

[Command continues if validation passes...]
```

### 에러 복구 안내

```markdown
---
description: Command with recovery guidance
---

# Operation Command

Running operation...

!`risky-operation.sh`

if [ $? -ne 0 ]; then
  ❌ OPERATION FAILED

  The operation encountered an error and could not complete.

  WHAT HAPPENED:
  The risky-operation.sh script returned a non-zero exit code.

  WHAT THIS MEANS:
  - Changes may be partially applied
  - System may be in inconsistent state
  - Manual intervention may be needed

  RECOVERY STEPS:
  1. Check operation logs: cat /tmp/operation.log
  2. Verify system state: /check-state
  3. If needed, rollback: /rollback-operation
  4. Fix underlying issue
  5. Retry operation: /retry-operation

  NEED HELP?
  - Check troubleshooting guide: /help troubleshooting
  - Contact support with error code: ERR_OP_FAILED_001

  Exit.
fi
```

## 사용 예시 문서화

### 내장된 예시

```markdown
---
description: Command with embedded examples
---

# Feature Command

This command performs feature analysis with multiple options.

## Basic Usage

\`\`\`
/feature analyze src/
\`\`\`

Analyzes all files in src/ directory for feature usage.

## Advanced Usage

\`\`\`
/feature analyze src/ --detailed
\`\`\`

Provides detailed analysis including:
- Feature breakdown by file
- Usage patterns
- Optimization suggestions

## Use Cases

**Use Case 1: Quick overview**
\`\`\`
/feature analyze .
\`\`\`
Get high-level feature summary of entire project.

**Use Case 2: Specific directory**
\`\`\`
/feature analyze src/components
\`\`\`
Focus analysis on components directory only.

**Use Case 3: Comparison**
\`\`\`
/feature analyze src/ --compare baseline.json
\`\`\`
Compare current features against baseline.

---

Now processing your request...

[Command implementation...]
```

### 예시 중심 문서화

```markdown
---
description: Example-heavy command
---

# Transformation Command

## What This Does

Transforms data from one format to another.

## Examples First

### Example 1: JSON to YAML
**Input:** `data.json`
\`\`\`json
{"name": "test", "value": 42}
\`\`\`

**Command:** `/transform data.json yaml`

**Output:** `data.yaml`
\`\`\`yaml
name: test
value: 42
\`\`\`

### Example 2: CSV to JSON
**Input:** `data.csv`
\`\`\`csv
name,value
test,42
\`\`\`

**Command:** `/transform data.csv json`

**Output:** `data.json`
\`\`\`json
[{"name": "test", "value": "42"}]
\`\`\`

### Example 3: With Options
**Command:** `/transform data.json yaml --pretty --sort-keys`

**Result:** Formatted YAML with sorted keys

---

## Your Transformation

File: $1
Format: $2

[Perform transformation...]
```

## 유지보수용 문서화

### 버전 및 변경 이력

```markdown
<!--
VERSION: 2.1.0
LAST UPDATED: 2025-01-15
AUTHOR: DevOps Team

CHANGELOG:
  v2.1.0 (2025-01-15):
    - Added support for YAML configuration
    - Improved error messages
    - Fixed bug with special characters in arguments

  v2.0.0 (2025-01-01):
    - BREAKING: Changed argument order
    - BREAKING: Removed deprecated --old-flag
    - Added new validation checks
    - Migration guide: /migration-v2

  v1.5.0 (2024-12-15):
    - Added --verbose flag
    - Improved performance by 50%

  v1.0.0 (2024-12-01):
    - Initial stable release

MIGRATION NOTES:
  From v1.x to v2.0:
    Old: /command arg1 arg2 --old-flag
    New: /command arg2 arg1

  The --old-flag is removed. Use --new-flag instead.

DEPRECATION WARNINGS:
  - The --legacy-mode flag is deprecated as of v2.1.0
  - Will be removed in v3.0.0 (estimated 2025-06-01)
  - Use --modern-mode instead

KNOWN ISSUES:
  - #123: Slow performance with large files (workaround: use --stream flag)
  - #456: Special characters in Windows (fix planned for v2.2.0)
-->
```

### 유지보수 참고 사항

```markdown
<!--
MAINTENANCE NOTES:

CODE STRUCTURE:
  - Lines 1-50: Argument parsing and validation
  - Lines 51-100: Main processing logic
  - Lines 101-150: Output formatting
  - Lines 151-200: Error handling

DEPENDENCIES:
  - Requires git 2.x or later
  - Uses jq for JSON processing
  - Needs bash 4.0+ for associative arrays

PERFORMANCE:
  - Fast path for small inputs (< 1MB)
  - Streams large files to avoid memory issues
  - Caches results in /tmp for 1 hour

SECURITY CONSIDERATIONS:
  - Validates all inputs to prevent injection
  - Uses allowed-tools to limit Bash access
  - No credentials in command file

TESTING:
  - Unit tests: tests/command-test.sh
  - Integration tests: tests/integration/
  - Manual test checklist: tests/manual-checklist.md

FUTURE IMPROVEMENTS:
  - TODO: Add support for TOML format
  - TODO: Implement parallel processing
  - TODO: Add progress bar for large files

RELATED FILES:
  - lib/parser.sh: Shared parsing logic
  - lib/formatter.sh: Output formatting
  - config/defaults.yml: Default configuration
-->
```

## README 문서화

기능은 동반 README 파일을 가져야 합니다:

```markdown
# Command Name

Brief description of what the command does.

## Installation

This command is part of the [plugin-name] plugin.

Install with:
\`\`\`
/plugin install plugin-name
\`\`\`

## Usage

Basic usage:
\`\`\`
/command-name [arg1] [arg2]
\`\`\`

## Arguments

- `arg1`: Description (required)
- `arg2`: Description (optional, defaults to X)

## Examples

### Example 1: Basic Usage
\`\`\`
/command-name value1 value2
\`\`\`

Description of what happens.

### Example 2: Advanced Usage
\`\`\`
/command-name value1 --option
\`\`\`

Description of advanced feature.

## Configuration

Optional configuration file: `.claude/command-name.local.md`

\`\`\`markdown
---
default_arg: value
enable_feature: true
---
```

## Requirements

- Git 2.x or later
- jq (for JSON processing)
- Node.js 14+ (optional, for advanced features)

## Troubleshooting

### Issue: Command not found

**Solution:** Ensure plugin is installed and enabled.

### Issue: Permission denied

**Solution:** Check file permissions and allowed-tools setting.

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT License - See [LICENSE](LICENSE).

## Support

- Issues: https://github.com/user/plugin/issues
- Docs: https://docs.example.com
- Email: support@example.com
```

## 베스트 프랙티스

### 문서화 원칙:

1. **미래의 자신을 위해 작성하세요**: 세부 사항은 쉽게 잊혀진다고 가정합니다.
2. **설명보다 예시를 먼저 제시하세요**: 먼저 동작을 보여준 후 설명합니다.
3. **점진적 공개**: 기본 정보를 먼저 보여주고, 세부 사항을 선택적으로 제공합니다.
4. **최신 상태 유지**: 코드가 변경되면 문서를 함께 업데이트하세요.
5. **문서 테스트**: 예시가 실제로 정상 작동하는지 검증하세요.

### 문서화 위치:

1. **명령어 파일 내부**: 핵심 사용법, 예시, 인라인 설명
2. **README**: 설치, 설정, 문제 해결
3. **독립 문서**: 상세 가이드, 튜토리얼, API 참조서
4. **주석**: 유지보수자를 위한 구현 상세 정보

### 문서화 스타일:

1. **명확하고 간결하게**: 불필요한 수식어는 빼고 작문합니다.
2. **능동태 사용**: "명령어가 실행될 수 있습니다" 대신 "명령어를 실행하십시오"와 같이 기술합니다.
3. **일관된 용어 사용**: 본문 전반에 걸쳐 동일한 용어를 사용합니다.
4. **좋은 포맷팅**: 제목, 목록, 코드 블록을 적극 활용합니다.
5. **친숙함**: 독자가 초보자라고 가정하고 쉽게 작성합니다.

### 문서 유지보수:

1. **모든 것을 버전화하세요**: 언제 무엇이 바뀌었는지 추적합니다.
2. **부드러운 지원 종료**: 기능을 제거하기 전에 미리 경고를 제공합니다.
3. **마이그레이션 가이드**: 사용자의 업그레이드를 돕습니다.
4. **오래된 문서 아카이브**: 이전 버전 문서도 접근 가능한 상태로 유지합니다.
5. **정기적 검토**: 문서가 현실의 코드와 일치하는지 확인합니다.

## 문서화 체크리스트

명령어 배포 전 체크리스트:

- [ ] 프론트매터의 description 필드가 명확함
- [ ] argument-hint가 모든 인자를 잘 설명함
- [ ] 주석 내에 사용 예시가 기술되어 있음
- [ ] 일반적인 사용 사례들이 포함되어 있음
- [ ] 에러 메시지가 도움이 되도록 작성됨
- [ ] 요구 사항(requirements)이 성실히 기록됨
- [ ] 연관 명령어가 나열되어 있음
- [ ] 변경 이력(changelog)이 관리됨
- [ ] 버전 번호가 올바르게 업데이트됨
- [ ] README 파일이 새로 작성되거나 업데이트됨
- [ ] 예시 코드들이 실제로 잘 작동함
- [ ] 문제 해결(troubleshooting) 섹션이 완성됨

좋은 문서를 작성하면 명령어가 사용자 친화적인 도구로 완성되어, 지원 요청은 줄어들고 사용자 경험은 비약적으로 향상됩니다.
