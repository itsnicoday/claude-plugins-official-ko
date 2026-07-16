# 명령어를 위한 마켓플레이스 고려 사항 (Marketplace Considerations for Commands)

배포 및 마켓플레이스 등록 시 성공을 거둘 수 있는 명령어를 만들기 위한 가이드라인입니다.

## 개요

마켓플레이스를 통해 배포되는 명령어는 개인용 명령어와 달리 추가적인 고려가 필요합니다. 다양한 실행 환경에서도 원활히 작동해야 하고, 다양한 사용 사례를 처리할 수 있어야 하며, 생소한 사용자에게도 훌륭한 사용자 경험(UX)을 제공해야 합니다.

## 배포를 위한 설계

### 범용 호환성 (Universal Compatibility)

**크로스 플랫폼 고려 사항:**

```markdown
---
description: Cross-platform command
allowed-tools: Bash(*)
---

# Platform-Aware Command

Detecting platform...

case "$(uname)" in
  Darwin*)  PLATFORM="macOS" ;;
  Linux*)   PLATFORM="Linux" ;;
  MINGW*|MSYS*|CYGWIN*) PLATFORM="Windows" ;;
  *)        PLATFORM="Unknown" ;;
esac

Platform: $PLATFORM

<!-- Adjust behavior based on platform -->
if [ "$PLATFORM" = "Windows" ]; then
  # Windows-specific handling
  PATH_SEP="\\"
  NULL_DEVICE="NUL"
else
  # Unix-like handling
  PATH_SEP="/"
  NULL_DEVICE="/dev/null"
fi

[Platform-appropriate implementation...]
```

**플랫폼별로 국한된 명령어 사용 피하기:**

```markdown
<!-- 잘못된 사례: macOS 전용 -->
!`pbcopy < file.txt`

<!-- 올바른 사례: 플랫폼 감지 -->
if command -v pbcopy > /dev/null; then
  pbcopy < file.txt
elif command -v xclip > /dev/null; then
  xclip -selection clipboard < file.txt
elif command -v clip.exe > /dev/null; then
  cat file.txt | clip.exe
else
  echo "Clipboard not available on this platform"
fi
```

### 최소한의 종속성 (Minimal Dependencies)

**필수 도구 확인:**

```markdown
---
description: Dependency-aware command
allowed-tools: Bash(*)
---

# Check Dependencies

Required tools:
- git
- jq
- node

Checking availability...

MISSING_DEPS=""

for tool in git jq node; do
  if ! command -v $tool > /dev/null; then
    MISSING_DEPS="$MISSING_DEPS $tool"
  fi
done

if [ -n "$MISSING_DEPS" ]; then
  ❌ ERROR: Missing required dependencies:$MISSING_DEPS

  INSTALLATION:
  - git: https://git-scm.com/downloads
  - jq: https://stedolan.github.io/jq/download/
  - node: https://nodejs.org/

  Install missing tools and try again.

  Exit.
fi

✓ All dependencies available

[Continue with command...]
```

**선택적 종속성 문서화:**

```markdown
<!--
DEPENDENCIES:
  Required:
  - git 2.0+: Version control
  - jq 1.6+: JSON processing

  Optional:
  - gh: GitHub CLI (for PR operations)
  - docker: Container operations (for containerized tests)

  Feature availability depends on installed tools.
-->
```

### 유연한 기능 축소 (Graceful Degradation)

**누락된 기능 처리:**

```markdown
---
description: Feature-aware command
---

# Feature Detection

Detecting available features...

FEATURES=""

if command -v gh > /dev/null; then
  FEATURES="$FEATURES github"
fi

if command -v docker > /dev/null; then
  FEATURES="$FEATURES docker"
fi

Available features: $FEATURES

if echo "$FEATURES" | grep -q "github"; then
  # Full functionality with GitHub integration
  echo "✓ GitHub integration available"
else
  # Reduced functionality without GitHub
  echo "⚠ Limited functionality: GitHub CLI not installed"
  echo "  Install 'gh' for full features"
fi

[Adapt behavior based on available features...]
```

## 생소한 사용자를 위한 사용자 경험

### 명확한 온보딩 과정 (Clear Onboarding)

**최초 실행 경험:**

```markdown
---
description: Command with onboarding
allowed-tools: Read, Write
---

# First Run Check

if [ ! -f ".claude/command-initialized" ]; then
  **Welcome to Command Name!**

  This appears to be your first time using this command.

  WHAT THIS COMMAND DOES:
  [Brief explanation of purpose and benefits]

  QUICK START:
  1. Basic usage: /command [arg]
  2. For help: /command help
  3. Examples: /command examples

  SETUP:
  No additional setup required. You're ready to go!

  ✓ Initialization complete

  [Create initialization marker]

  Ready to proceed with your request...
fi

[Normal command execution...]
```

**점진적인 기능 발견:**

```markdown
---
description: Command with tips
---

# Command Execution

[Main functionality...]

---

💡 TIP: Did you know?

You can speed up this command with the --fast flag:
  /command --fast [args]

For more tips: /command tips
```

### 종합적인 에러 처리 (Comprehensive Error Handling)

**사용자의 실수 예측 및 대응:**

```markdown
---
description: Forgiving command
---

# User Input Handling

Argument: "$1"

<!-- Check for common typos -->
if [ "$1" = "hlep" ] || [ "$1" = "hepl" ]; then
  Did you mean: help?

  Showing help instead...
  [Display help]

  Exit.
fi

<!-- Suggest similar commands if not found -->
if [ "$1" != "valid-option1" ] && [ "$1" != "valid-option2" ]; then
  ❌ Unknown option: $1

  Did you mean:
  - valid-option1 (most similar)
  - valid-option2

  For all options: /command help

  Exit.
fi

[Command continues...]
```

**도움이 되는 진단 정보 제공:**

```markdown
---
description: Diagnostic command
---

# Operation Failed

The operation could not complete.

**Diagnostic Information:**

Environment:
- Platform: $(uname)
- Shell: $SHELL
- Working directory: $(pwd)
- Command: /command $@

Checking common issues:
- Git repository: $(git rev-parse --git-dir 2>&1)
- Write permissions: $(test -w . && echo "OK" || echo "DENIED")
- Required files: $(test -f config.yml && echo "Found" || echo "Missing")

This information helps debug the issue.

For support, include the above diagnostics.
```

## 배포 베스트 프랙티스

### 네임스페이스 인지 (Namespace Awareness)

**이름 충돌 방지:**

```markdown
---
description: Namespaced command
---

<!--
COMMAND NAME: plugin-name-command

This command is namespaced with the plugin name to avoid
conflicts with commands from other plugins.

Alternative naming approaches:
- Use plugin prefix: /plugin-command
- Use category: /category-command
- Use verb-noun: /verb-noun

Chosen approach: plugin-name prefix
Reasoning: Clearest ownership, least likely to conflict
-->

# Plugin Name Command

[Implementation...]
```

**명명 근거 문서화:**

```markdown
<!--
NAMING DECISION:

Command name: /deploy-app

Alternatives considered:
- /deploy: Too generic, likely conflicts
- /app-deploy: Less intuitive ordering
- /my-plugin-deploy: Too verbose

Final choice balances:
- Discoverability (clear purpose)
- Brevity (easy to type)
- Uniqueness (unlikely conflicts)
-->
```

### 설정 가능성 (Configurability)

**사용자 선호도 설정:**

```markdown
---
description: Configurable command
allowed-tools: Read
---

# Load User Configuration

Default configuration:
- verbose: false
- color: true
- max_results: 10

Checking for user config: .claude/plugin-name.local.md

if [ -f ".claude/plugin-name.local.md" ]; then
  # Parse YAML frontmatter for settings
  VERBOSE=$(grep "^verbose:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')
  COLOR=$(grep "^color:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')
  MAX_RESULTS=$(grep "^max_results:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')

  echo "✓ Using user configuration"
else
  echo "Using default configuration"
  echo "Create .claude/plugin-name.local.md to customize"
fi

[Use configuration in command...]
```

**합리적인 기본값 설정:**

```markdown
---
description: Command with smart defaults
---

# Smart Defaults

Configuration:
- Format: ${FORMAT:-json}  # Defaults to json
- Output: ${OUTPUT:-stdout}  # Defaults to stdout
- Verbose: ${VERBOSE:-false}  # Defaults to false

These defaults work for 80% of use cases.

Override with arguments:
  /command --format yaml --output file.txt --verbose

Or set in .claude/plugin-name.local.md:
\`\`\`yaml
---
format: yaml
output: custom.txt
verbose: true
---
\`\`\`
```

### 버전 호환성 (Version Compatibility)

**버전 검사:**

```markdown
---
description: Version-aware command
---

<!--
COMMAND VERSION: 2.1.0

COMPATIBILITY:
- Requires plugin version: >= 2.0.0
- Breaking changes from v1.x documented in MIGRATION.md

VERSION HISTORY:
- v2.1.0: Added --new-feature flag
- v2.0.0: BREAKING: Changed argument order
- v1.0.0: Initial release
-->

# Version Check

Command version: 2.1.0
Plugin version: [detect from plugin.json]

if [  "$PLUGIN_VERSION" < "2.0.0" ]; then
  ❌ ERROR: Incompatible plugin version

  This command requires plugin version >= 2.0.0
  Current version: $PLUGIN_VERSION

  Update plugin:
    /plugin update plugin-name

  Exit.
fi

✓ Version compatible

[Command continues...]
```

**지원 종료(Deprecation) 경고:**

```markdown
---
description: Command with deprecation warnings
---

# Deprecation Check

if [ "$1" = "--old-flag" ]; then
  ⚠️  DEPRECATION WARNING

  The --old-flag option is deprecated as of v2.0.0
  It will be removed in v3.0.0 (est. June 2025)

  Use instead: --new-flag

  Example:
    Old: /command --old-flag value
    New: /command --new-flag value

  See migration guide: /command migrate

  Continuing with deprecated behavior for now...
fi

[Handle both old and new flags during deprecation period...]
```

## 마켓플레이스 등록 양식

### 명령어 발견 용이성 (Command Discovery)

**설명적인 이름 사용:**

```markdown
---
description: Review pull request with security and quality checks
---

<!-- GOOD: Descriptive name and description -->
```

```markdown
---
description: Do the thing
---

<!-- BAD: Vague description -->
```

**검색 가능한 키워드 활용:**

```markdown
<!--
KEYWORDS: security, code-review, quality, validation, audit

These keywords help users discover this command when searching
for related functionality in the marketplace.
-->
```

### 시연용 예시 제공 (Showcase Examples)

**독자의 눈길을 끄는 시연 코드:**

```markdown
---
description: Advanced code analysis command
---

# Code Analysis Command

This command performs deep code analysis with actionable insights.

## Demo: Quick Security Audit

Try it now:
\`\`\`
/analyze-code src/ --security
\`\`\`

**What you'll get:**
- Security vulnerability detection
- Code quality metrics
- Performance bottleneck identification
- Actionable recommendations

**Sample output:**
\`\`\`
Security Analysis Results
=========================

🔴 Critical (2):
  - SQL injection risk in users.js:45
  - XSS vulnerability in display.js:23

🟡 Warnings (5):
  - Unvalidated input in api.js:67
  ...

Recommendations:
1. Fix critical issues immediately
2. Review warnings before next release
3. Run /analyze-code --fix for auto-fixes
\`\`\`

---

Ready to analyze your code...

[Command implementation...]
```

### 사용자 리뷰 및 피드백 (User Reviews and Feedback)

**피드백 수집 방법:**

```markdown
---
description: Command with feedback
---

# Command Complete

[Command results...]

---

**How was your experience?**

This helps improve the command for everyone.

Rate this command:
- 👍 Helpful
- 👎 Not helpful
- 🐛 Found a bug
- 💡 Have a suggestion

Reply with an emoji or:
- /command feedback

Your feedback matters!
```

**사용 분석 준비:**

```markdown
<!--
ANALYTICS NOTES:

Track for improvement:
- Most common arguments
- Failure rates
- Average execution time
- User satisfaction scores

Privacy-preserving:
- No personally identifiable information
- Aggregate statistics only
- User opt-out respected
-->
```

## 품질 표준 (Quality Standards)

### 전문적인 완성도 (Professional Polish)

**일관된 브랜딩:**

```markdown
---
description: Branded command
---

# ✨ Command Name

Part of the [Plugin Name] suite

[Command functionality...]

---

**Need Help?**
- Documentation: https://docs.example.com
- Support: support@example.com
- Community: https://community.example.com

Powered by Plugin Name v2.1.0
```

**디테일 완성도 높이기:**

```markdown
<!-- Details that matter -->

✓ Use proper emoji/symbols consistently
✓ Align output columns neatly
✓ Format numbers with thousands separators
✓ Use color/formatting appropriately
✓ Provide progress indicators
✓ Show estimated time remaining
✓ Confirm successful operations
```

### 신뢰성 (Reliability)

**멱등성 (Idempotency):**

```markdown
---
description: Idempotent command
---

# Safe Repeated Execution

Checking if operation already completed...

if [ -f ".claude/operation-completed.flag" ]; then
  ℹ️  Operation already completed

  Completed at: $(cat .claude/operation-completed.flag)

  To re-run:
  1. Remove flag: rm .claude/operation-completed.flag
  2. Run command again

  Otherwise, no action needed.

  Exit.
fi

Performing operation...

[Safe, repeatable operation...]

Marking complete...
echo "$(date)" > .claude/operation-completed.flag
```

**원자적 작업 (Atomic Operations):**

```markdown
---
description: Atomic command
---

# Atomic Operation

This operation is atomic - either fully succeeds or fully fails.

Creating temporary workspace...
TEMP_DIR=$(mktemp -d)

Performing changes in isolated environment...
[Make changes in $TEMP_DIR]

if [ $? -eq 0 ]; then
  ✓ Changes validated

  Applying changes atomically...
  mv $TEMP_DIR/* ./target/

  ✓ Operation complete
else
  ❌ Changes failed validation

  Rolling back...
  rm -rf $TEMP_DIR

  No changes applied. Safe to retry.
fi
```

## 배포용 테스트

### 배포 전 체크리스트

```markdown
<!--
PRE-RELEASE CHECKLIST:

Functionality:
- [ ] Works on macOS
- [ ] Works on Linux
- [ ] Works on Windows (WSL)
- [ ] All arguments tested
- [ ] Error cases handled
- [ ] Edge cases covered

User Experience:
- [ ] Clear description
- [ ] Helpful error messages
- [ ] Examples provided
- [ ] First-run experience good
- [ ] Documentation complete

Distribution:
- [ ] No hardcoded paths
- [ ] Dependencies documented
- [ ] Configuration options clear
- [ ] Version number set
- [ ] Changelog updated

Quality:
- [ ] No TODO comments
- [ ] No debug code
- [ ] Performance acceptable
- [ ] Security reviewed
- [ ] Privacy considered

Support:
- [ ] README complete
- [ ] Troubleshooting guide
- [ ] Support contact provided
- [ ] Feedback mechanism
- [ ] License specified
-->
```

### 베타 테스트 (Beta Testing)

**베타 릴리즈 방법:**

```markdown
---
description: Beta command (v0.9.0)
---

# 🧪 Beta Command

**This is a beta release**

Features may change based on feedback.

BETA STATUS:
- Version: 0.9.0
- Stability: Experimental
- Support: Limited
- Feedback: Encouraged

Known limitations:
- Performance not optimized
- Some edge cases not handled
- Documentation incomplete

Help improve this command:
- Report issues: /command report-issue
- Suggest features: /command suggest
- Join beta testers: /command join-beta

---

[Command implementation...]

---

**Thank you for beta testing!**

Your feedback helps make this command better.
```

## 유지보수 및 업데이트

### 업데이트 전략 (Update Strategy)

**버전 관리 방식:**

```markdown
<!--
VERSION STRATEGY:

Major (X.0.0): Breaking changes
- Document all breaking changes
- Provide migration guide
- Support old version briefly

Minor (x.Y.0): New features
- Backward compatible
- Announce new features
- Update examples

Patch (x.y.Z): Bug fixes
- No user-facing changes
- Update changelog
- Security fixes prioritized

Release schedule:
- Patches: As needed
- Minors: Monthly
- Majors: Annually or as needed
-->
```

**업데이트 알림:**

```markdown
---
description: Update-aware command
---

# Check for Updates

Current version: 2.1.0
Latest version: [check if available]

if [ "$CURRENT_VERSION" != "$LATEST_VERSION" ]; then
  📢 UPDATE AVAILABLE

  New version: $LATEST_VERSION
  Current: $CURRENT_VERSION

  What's new:
  - Feature improvements
  - Bug fixes
  - Performance enhancements

  Update with:
    /plugin update plugin-name

  Release notes: https://releases.example.com/v$LATEST_VERSION
fi

[Command continues...]
```

## 권장 가이드 요약 (Best Practices Summary)

### 배포 설계:

1. **범용성 (Universal)**: 다양한 플랫폼과 환경에서 작동
2. **자기 완결성 (Self-contained)**: 최소한의 종속성, 명확한 필요 조건
3. **유연성 (Graceful)**: 특정 기능이 없을 때 유연하게 기능을 축소하여 대응
4. **허용적 설계 (Forgiving)**: 사용자의 실수를 예측하고 적절히 처리
5. **도움이 됨 (Helpful)**: 명확한 에러 메시지, 좋은 기본값, 훌륭한 문서

### 마켓플레이스 성공 지침:

1. **발견 용이성 (Discoverable)**: 명확한 이름, 좋은 설명, 검색 가능한 키워드
2. **전문성 (Professional)**: 세련된 레이아웃, 일관된 브랜딩
3. **신뢰성 (Reliable)**: 철저하게 테스트 완료, 예외 상황 처리
4. **유지보수 용이성 (Maintainable)**: 버전 관리, 정기적인 업데이트, 지속적인 기술 지원
5. **사용자 중심 (User-focused)**: 훌륭한 UX, 피드백에 신속히 반응

### 품질 기준:

1. **완전함 (Complete)**: 완벽한 문서화, 모든 기능 작동
2. **테스트 완료 (Tested)**: 실제 작업 환경에서 테스트 완료, 예외 상황 처리
3. **보안 준수 (Secure)**: 보안 취약점 없음, 안전한 작업 수행
4. **성능 준수 (Performant)**: 수용 가능한 실행 속도, 효율적인 리소스 관리
5. **윤리 준수 (Ethical)**: 개인정보 존중, 사용자의 동의 획득

이러한 고려 사항들을 적용함으로써 명령어들은 마켓플레이스에 등록할 준비를 갖추게 되며, 다양한 사용 환경과 다양한 케이스의 사용자들을 모두 만족시킬 수 있을 것입니다.
