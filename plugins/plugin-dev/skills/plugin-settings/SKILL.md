---
name: plugin-settings
description: This skill should be used when the user asks about "plugin settings", "store plugin configuration", "user-configurable plugin", ".local.md files", "plugin state files", "read YAML frontmatter", "per-project plugin settings", or wants to make plugin behavior configurable. Documents the .claude/plugin-name.local.md pattern for storing plugin-specific configuration with YAML frontmatter and markdown content.
version: 0.1.0
---

# Claude Code 플러그인을 위한 플러그인 설정 패턴 (Plugin Settings Pattern for Claude Code Plugins)

## 개요 (Overview)

플러그인은 프로젝트 디렉토리 내의 `.claude/plugin-name.local.md` 파일에 사용자 정의 설정 및 상태를 저장할 수 있습니다. 이 패턴은 구조화된 구성을 위해 YAML 프론트매터를 사용하고, 프롬프트나 추가 컨텍스트를 위해 마크다운 본문을 사용합니다.

**주요 특징:**
- 파일 위치: 프로젝트 루트의 `.claude/plugin-name.local.md`
- 구조: YAML 프론트매터 + 마크다운 본문
- 목적: 프로젝트별 플러그인 구성 및 상태 관리
- 사용법: 훅(hooks), 명령어(commands) 및 에이전트(agents)에서 읽기
- 수명 주기: 사용자 관리 대상 (git에 포함되지 않으며, `.gitignore`에 등록해야 함)

## 파일 구조 (File Structure)

### 기본 템플릿 (Basic Template)

```markdown
---
enabled: true
setting1: value1
setting2: value2
numeric_setting: 42
list_setting: ["item1", "item2"]
---

# Additional Context

This markdown body can contain:
- Task descriptions
- Additional instructions
- Prompts to feed back to Claude
- Documentation or notes
```

### 예시: 플러그인 상태 파일 (Example: Plugin State File)

**.claude/my-plugin.local.md:**
```markdown
---
enabled: true
strict_mode: false
max_retries: 3
notification_level: info
coordinator_session: team-leader
---

# Plugin Configuration

This plugin is configured for standard validation mode.
Contact @team-lead with questions.
```

## 설정 파일 읽기 (Reading Settings Files)

### 훅에서 읽기 (From Hooks (Bash Scripts))

**패턴: 파일 존재 여부 확인 및 프론트매터 파싱**

```bash
#!/bin/bash
set -euo pipefail

# Define state file path
# settings file path
STATE_FILE=".claude/my-plugin.local.md"

# Quick exit if file doesn't exist
if [[ ! -f "$STATE_FILE" ]]; then
  exit 0  # Plugin not configured, skip
fi

# Parse YAML frontmatter (between --- markers)
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$STATE_FILE")

# Extract individual fields
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//' | sed 's/^"\(.*\)"$/\1/')
STRICT_MODE=$(echo "$FRONTMATTER" | grep '^strict_mode:' | sed 's/strict_mode: *//' | sed 's/^"\(.*\)"$/\1/')

# Check if enabled
if [[ "$ENABLED" != "true" ]]; then
  exit 0  # Disabled
fi

# Use configuration in hook logic
if [[ "$STRICT_MODE" == "true" ]]; then
  # Apply strict validation
  # ...
fi
```

동작하는 완전한 예시는 `examples/read-settings-hook.sh`를 참조하십시오.

### 명령어에서 읽기 (From Commands)

명령어는 동작을 맞춤 정의하기 위해 설정 파일을 읽을 수 있습니다:

```markdown
---
description: Process data with plugin
allowed-tools: ["Read", "Bash"]
---

# Process Command

Steps:
1. Check if settings exist at `.claude/my-plugin.local.md`
2. Read configuration using Read tool
3. Parse YAML frontmatter to extract settings
4. Apply settings to processing logic
5. Execute with configured behavior
```

### 에이전트에서 읽기 (From Agents)

에이전트는 해당 지침에서 설정 파일을 참조할 수 있습니다:

```markdown
---
name: configured-agent
description: Agent that adapts to project settings
---

Check for plugin settings at `.claude/my-plugin.local.md`.
If present, parse YAML frontmatter and adapt behavior according to:
- enabled: Whether plugin is active
- mode: Processing mode (strict, standard, lenient)
- Additional configuration fields
```

## 파싱 기법 (Parsing Techniques)

### 프론트매터 추출 (Extract Frontmatter)

```bash
# Extract everything between --- markers
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")
```

### 개별 필드 읽기 (Read Individual Fields)

**문자열 필드:**
```bash
VALUE=$(echo "$FRONTMATTER" | grep '^field_name:' | sed 's/field_name: *//' | sed 's/^"\(.*\)"$/\1/')
```

**불리언 필드:**
```bash
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
# Compare: if [[ "$ENABLED" == "true" ]]; then
```

**숫자 필드:**
```bash
MAX=$(echo "$FRONTMATTER" | grep '^max_value:' | sed 's/max_value: *//')
# Use: if [[ $MAX -gt 100 ]]; then
```

### 마크다운 본문 읽기 (Read Markdown Body)

두 번째 `---` 이후의 본문 내용을 추출합니다:

```bash
# Get everything after closing ---
BODY=$(awk '/^---$/{i++; next} i>=2' "$FILE")
```

## 공통 패턴 (Common Patterns)

### 패턴 1: 일시적으로 활성화되는 훅 (Pattern 1: Temporarily Active Hooks)

설정 파일을 사용하여 훅 활성화를 제어합니다:

```bash
#!/bin/bash
STATE_FILE=".claude/security-scan.local.md"

# Quick exit if not configured
if [[ ! -f "$STATE_FILE" ]]; then
  exit 0
fi

# Read enabled flag
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$STATE_FILE")
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')

if [[ "$ENABLED" != "true" ]]; then
  exit 0  # Disabled
fi

# Run hook logic
# ...
```

**사용 사례:** `hooks.json`을 수정하지 않고 훅을 활성화/비활성화합니다 (재시작 필요).

### 패턴 2: 에이전트 상태 관리 (Pattern 2: Agent State Management)

에이전트 고유의 상태 및 구성을 저장합니다:

**.claude/multi-agent-swarm.local.md:**
```markdown
---
agent_name: auth-agent
task_number: 3.5
pr_number: 1234
coordinator_session: team-leader
enabled: true
dependencies: ["Task 3.4"]
---

# Task Assignment

Implement JWT authentication for the API.

**Success Criteria:**
- Authentication endpoints created
- Tests passing
- PR created and CI green
```

훅에서 읽어 에이전트들을 조정합니다:

```bash
AGENT_NAME=$(echo "$FRONTMATTER" | grep '^agent_name:' | sed 's/agent_name: *//')
COORDINATOR=$(echo "$FRONTMATTER" | grep '^coordinator_session:' | sed 's/coordinator_session: *//')

# Send notification to coordinator
tmux send-keys -t "$COORDINATOR" "Agent $AGENT_NAME completed task" Enter
```

### 패턴 3: 구성 구동형 동작 (Pattern 3: Configuration-Driven Behavior)

**.claude/my-plugin.local.md:**
```markdown
---
validation_level: strict
max_file_size: 1000000
allowed_extensions: [".js", ".ts", ".tsx"]
enable_logging: true
---

# Validation Configuration

Strict mode enabled for this project.
All writes validated against security policies.
```

훅이나 명령어에서 사용합니다:

```bash
LEVEL=$(echo "$FRONTMATTER" | grep '^validation_level:' | sed 's/validation_level: *//')

case "$LEVEL" in
  strict)
    # Apply strict validation
    ;;
  standard)
    # Apply standard validation
    ;;
  lenient)
    # Apply lenient validation
    ;;
esac
```

## 설정 파일 생성 (Creating Settings Files)

### 명령어에서 생성 (From Commands)

명령어가 설정 파일을 생성할 수 있습니다:

```markdown
# Setup Command

Steps:
1. Ask user for configuration preferences
2. Create `.claude/my-plugin.local.md` with YAML frontmatter
3. Set appropriate values based on user input
4. Inform user that settings are saved
5. Remind user to restart Claude Code for hooks to recognize changes
```

### 템플릿 생성 (Template Generation)

플러그인 README에서 템플릿을 제공합니다:

```markdown
## Configuration

Create `.claude/my-plugin.local.md` in your project:

\`\`\`markdown
---
enabled: true
mode: standard
max_retries: 3
---

# Plugin Configuration

Your settings are active.
\`\`\`

After creating or editing, restart Claude Code for changes to take effect.
```

## 권장 사항 (Best Practices)

### 파일 명명 규칙 (File Naming)

✅ **수행할 사항 (DO):**
- `.claude/plugin-name.local.md` 형식 사용
- 플러그인 이름과 정확히 일치시킴
- 사용자 로컬 파일임을 나타내기 위해 `.local.md` 접미사 사용

❌ **피해야 할 사항 (DON'T):**
- 다른 디렉토리 사용 (`.claude/`가 아닌 곳)
- 일관성 없는 이름 사용
- `.local` 없이 `.md`만 사용 (실수로 커밋될 위험 있음)

### Gitignore (Gitignore)

항상 `.gitignore`에 다음을 추가하십시오:

```gitignore
.claude/*.local.md
.claude/*.local.json
```

이를 플러그인 README에 문서화하십시오.

### 기본값 (Defaults)

설정 파일이 존재하지 않을 때 합리적인 기본값을 제공하십시오:

```bash
if [[ ! -f "$STATE_FILE" ]]; then
  # Use defaults
  ENABLED=true
  MODE=standard
else
  # Read from file
  # ...
fi
```

### 검증 (Validation)

설정 값을 검증하십시오:

```bash
MAX=$(echo "$FRONTMATTER" | grep '^max_value:' | sed 's/max_value: *//')

# Validate numeric range
if ! [[ "$MAX" =~ ^[0-9]+$ ]] || [[ $MAX -lt 1 ]] || [[ $MAX -gt 100 ]]; then
  echo "⚠️  Invalid max_value in settings (must be 1-100)" >&2
  MAX=10  # Use default
fi
```

### 재시작 요구 사항 (Restart Requirement)

**중요:** 설정 변경은 Claude Code 재시작이 필요합니다.

이를 README에 문서화하십시오:

```markdown
## Changing Settings

After editing `.claude/my-plugin.local.md`:
1. Save the file
2. Exit Claude Code
3. Restart: `claude` or `cc`
4. New settings will be loaded
```

훅은 세션 도중에 실시간으로 교체(hot-swap)될 수 없습니다.

## 보안 고려 사항 (Security Considerations)

### 사용자 입력 정제 (Sanitize User Input)

사용자 입력으로부터 설정 파일을 작성할 때:

```bash
# Escape quotes in user input
SAFE_VALUE=$(echo "$USER_INPUT" | sed 's/"/\\"/g')

# Write to file
cat > "$STATE_FILE" <<EOF
---
user_setting: "$SAFE_VALUE"
---
EOF
```

### 파일 경로 검증 (Validate File Paths)

설정에 파일 경로가 포함되어 있는 경우:

```bash
FILE_PATH=$(echo "$FRONTMATTER" | grep '^data_file:' | sed 's/data_file: *//')

# Check for path traversal
if [[ "$FILE_PATH" == *".."* ]]; then
  echo "⚠️  Invalid path in settings (path traversal)" >&2
  exit 2
fi
```

### 권한 (Permissions)

설정 파일은 다음과 같아야 합니다:
- 해당 사용자만 읽을 수 있어야 함 (`chmod 600`)
- git에 커밋되지 않아야 함
- 사용자 간에 공유되지 않아야 함

## 실제 사례 (Real-World Examples)

### multi-agent-swarm 플러그인 (multi-agent-swarm Plugin)

**.claude/multi-agent-swarm.local.md:**
```markdown
---
agent_name: auth-implementation
task_number: 3.5
pr_number: 1234
coordinator_session: team-leader
enabled: true
dependencies: ["Task 3.4"]
additional_instructions: Use JWT tokens, not sessions
---

# Task: Implement Authentication

Build JWT-based authentication for the REST API.
Coordinate with auth-agent on shared types.
```

**훅 사용 사례 (agent-stop-notification.sh):**
- 파일의 존재 여부 확인 (15-18행: 존재하지 않으면 빠른 종료)
- 프론트매터를 파싱하여 coordinator_session, agent_name, enabled 값을 가져옴
- 활성화되어 있으면 코디네이터에게 알림 전송
- `enabled: true/false`를 통해 신속한 활성화/비활성화 지원

### ralph-loop 플러그인 (ralph-loop Plugin)

**.claude/ralph-loop.local.md:**
```markdown
---
iteration: 1
max_iterations: 10
completion_promise: "All tests passing and build successful"
---

Fix all the linting errors in the project.
Make sure tests pass after each fix.
```

**훅 사용 사례 (stop-hook.sh):**
- 파일의 존재 여부 확인 (15-18행: 활성화 상태가 아니면 빠른 종료)
- iteration 횟수 및 max_iterations 값을 읽음
- 루프 종료를 위해 completion_promise 값을 추출
- 본문 내용을 다시 피드백할 프롬프트로 읽음
- 매 루프마다 iteration 횟수를 업데이트함

## 빠른 참조 (Quick Reference)

### 파일 위치 (File Location)

```
project-root/
└── .claude/
    └── plugin-name.local.md
```

### 프론트매터 파싱 (Frontmatter Parsing)

```bash
# Extract frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")

# Read field
VALUE=$(echo "$FRONTMATTER" | grep '^field:' | sed 's/field: *//' | sed 's/^"\(.*\)"$/\1/')
```

### 본문 파싱 (Body Parsing)

```bash
# Extract body (after second ---)
BODY=$(awk '/^---$/{i++; next} i>=2' "$FILE")
```

### 빠른 종료 패턴 (Quick Exit Pattern)

```bash
if [[ ! -f ".claude/my-plugin.local.md" ]]; then
  exit 0  # Not configured
fi
```

## 추가 리소스 (Additional Resources)

### 참조 파일

세부 구현 패턴:
- **`references/parsing-techniques.md`** - YAML 프론트매터 및 마크다운 본문을 파싱하는 전체 가이드
- **`references/real-world-examples.md`** - multi-agent-swarm 및 ralph-loop 구현 사례 상세 분석

### 예시 파일

`examples/` 내 작동 가능한 예시:
- **`read-settings-hook.sh`** - 설정을 읽고 사용하는 훅
- **`create-settings-command.md`** - 설정 파일을 생성하는 명령어
- **`example-settings.md`** - 템플릿 설정 파일

### 유틸리티 스크립트

`scripts/` 내 개발자용 도구:
- **`validate-settings.sh`** - 설정 파일 구조 검증
- **`parse-frontmatter.sh`** - 프론트매터 필드 추출

## 구현 워크플로우 (Implementation Workflow)

플러그인에 설정을 추가하는 절차:

1. 설정 스키마 설계 (필드, 타입, 기본값 결정)
2. 플러그인 문서에 템플릿 파일 작성
3. `.gitignore`에 `.claude/*.local.md` 항목 추가
4. 훅/명령어에 설정 파싱 기능 구현
5. 빠른 종료 패턴 적용 (파일 존재 확인, enabled 필드 확인)
6. 템플릿과 함께 플러그인 README에 설정 방법 기술
7. 설정 변경을 반영하려면 Claude Code를 재시작해야 함을 사용자에게 안내

설정 항목을 최대한 단순하게 유지하고, 설정 파일이 없을 경우 기본값이 잘 동작하도록 설계하십시오.
