# 실제 플러그인 설정 사례 (Real-World Plugin Settings Examples)

프로덕션 플러그인이 `.claude/plugin-name.local.md` 패턴을 어떻게 사용하는지에 대한 세부 분석.

## multi-agent-swarm 플러그인 (multi-agent-swarm Plugin)

### 설정 파일 구조 (Settings File Structure)

**.claude/multi-agent-swarm.local.md:**

```markdown
---
agent_name: auth-implementation
task_number: 3.5
pr_number: 1234
coordinator_session: team-leader
enabled: true
dependencies: ["Task 3.4"]
additional_instructions: "Use JWT tokens, not sessions"
---

# Task: Implement Authentication

Build JWT-based authentication for the REST API.

## Requirements
- JWT token generation and validation
- Refresh token flow
- Secure password hashing

## Success Criteria
- Auth endpoints implemented
- Tests passing (100% coverage)
- PR created and CI green
- Documentation updated

## Coordination
Depends on Task 3.4 (user model).
Report status to 'team-leader' session.
```

### 사용 방법 (How It's Used)

**파일:** `hooks/agent-stop-notification.sh`

**목적:** 에이전트가 유휴 상태가 되었을 때 코디네이터에게 알림 전송

**구현:**

```bash
#!/bin/bash
set -euo pipefail

SWARM_STATE_FILE=".claude/multi-agent-swarm.local.md"

# Quick exit if no swarm active
if [[ ! -f "$SWARM_STATE_FILE" ]]; then
  exit 0
fi

# Parse frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$SWARM_STATE_FILE")

# Extract configuration
COORDINATOR_SESSION=$(echo "$FRONTMATTER" | grep '^coordinator_session:' | sed 's/coordinator_session: *//' | sed 's/^"\(.*\)"$/\1/')
AGENT_NAME=$(echo "$FRONTMATTER" | grep '^agent_name:' | sed 's/agent_name: *//' | sed 's/^"\(.*\)"$/\1/')
TASK_NUMBER=$(echo "$FRONTMATTER" | grep '^task_number:' | sed 's/task_number: *//' | sed 's/^"\(.*\)"$/\1/')
PR_NUMBER=$(echo "$FRONTMATTER" | grep '^pr_number:' | sed 's/pr_number: *//' | sed 's/^"\(.*\)"$/\1/')
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')

# Check if enabled
if [[ "$ENABLED" != "true" ]]; then
  exit 0
fi

# Send notification to coordinator
NOTIFICATION="🤖 Agent ${AGENT_NAME} (Task ${TASK_NUMBER}, PR #${PR_NUMBER}) is idle."

if tmux has-session -t "$COORDINATOR_SESSION" 2>/dev/null; then
  tmux send-keys -t "$COORDINATOR_SESSION" "$NOTIFICATION" Enter
  sleep 0.5
  tmux send-keys -t "$COORDINATOR_SESSION" Enter
fi

exit 0
```

**주요 패턴:**
1. **빠른 종료** (7-9행): 파일이 없으면 즉시 종료
2. **필드 추출** (11-17행): 각 프론트매터 필드 파싱
3. **활성화 검사** (19-21행): enabled 플래그 존중
4. **설정 기준 작업** (23-29행): coordinator_session을 사용하여 알림 전송

### 생성 (Creation)

**파일:** `commands/launch-swarm.md`

설정 파일은 스웜이 시작될 때 다음과 같이 생성됩니다:

```bash
cat > "$WORKTREE_PATH/.claude/multi-agent-swarm.local.md" <<EOF
---
agent_name: $AGENT_NAME
task_number: $TASK_ID
pr_number: TBD
coordinator_session: $COORDINATOR_SESSION
enabled: true
dependencies: [$DEPENDENCIES]
additional_instructions: "$EXTRA_INSTRUCTIONS"
---

# Task: $TASK_DESCRIPTION

$TASK_DETAILS
EOF
```

### 업데이트 (Updates)

PR 생성 후 업데이트된 PR 번호:

```bash
# Update pr_number field
sed "s/^pr_number: .*/pr_number: $PR_NUM/" \
  ".claude/multi-agent-swarm.local.md" > temp.md
mv temp.md ".claude/multi-agent-swarm.local.md"
```

## ralph-loop 플러그인 (ralph-loop Plugin)

### 설정 파일 구조 (Settings File Structure)

**.claude/ralph-loop.local.md:**

```markdown
---
iteration: 1
max_iterations: 10
completion_promise: "All tests passing and build successful"
started_at: "2025-01-15T14:30:00Z"
---

Fix all the linting errors in the project.
Make sure tests pass after each fix.
Document any changes needed in CLAUDE.md.
```

### 사용 방법 (How It's Used)

**파일:** `hooks/stop-hook.sh`

**목적:** 세션 종료를 차단하고 Claude의 출력을 다시 입력으로 루프백

**구현:**

```bash
#!/bin/bash
set -euo pipefail

RALPH_STATE_FILE=".claude/ralph-loop.local.md"

# Quick exit if no active loop
if [[ ! -f "$RALPH_STATE_FILE" ]]; then
  exit 0
fi

# Parse frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$RALPH_STATE_FILE")

# Extract configuration
ITERATION=$(echo "$FRONTMATTER" | grep '^iteration:' | sed 's/iteration: *//')
MAX_ITERATIONS=$(echo "$FRONTMATTER" | grep '^max_iterations:' | sed 's/max_iterations: *//')
COMPLETION_PROMISE=$(echo "$FRONTMATTER" | grep '^completion_promise:' | sed 's/completion_promise: *//' | sed 's/^"\(.*\)"$/\1/')

# Check max iterations
if [[ $MAX_ITERATIONS -gt 0 ]] && [[ $ITERATION -ge $MAX_ITERATIONS ]]; then
  echo "🛑 Ralph loop: Max iterations ($MAX_ITERATIONS) reached."
  rm "$RALPH_STATE_FILE"
  exit 0
fi

# Get transcript and check for completion promise
TRANSCRIPT_PATH=$(echo "$HOOK_INPUT" | jq -r '.transcript_path')
LAST_OUTPUT=$(grep '"role":"assistant"' "$TRANSCRIPT_PATH" | tail -1 | jq -r '.message.content | map(select(.type == "text")) | map(.text) | join("\n")')

# Check for completion
if [[ "$COMPLETION_PROMISE" != "null" ]] && [[ -n "$COMPLETION_PROMISE" ]]; then
  PROMISE_TEXT=$(echo "$LAST_OUTPUT" | perl -0777 -pe 's/.*?<promise>(.*?)<\/promise>.*/$1/s; s/^\s+|\s+$//g')

  if [[ "$PROMISE_TEXT" = "$COMPLETION_PROMISE" ]]; then
    echo "✅ Ralph loop: Detected completion"
    rm "$RALPH_STATE_FILE"
    exit 0
  fi
fi

# Continue loop - increment iteration
NEXT_ITERATION=$((ITERATION + 1))

# Extract prompt from markdown body
PROMPT_TEXT=$(awk '/^---$/{i++; next} i>=2' "$RALPH_STATE_FILE")

# Update iteration counter
TEMP_FILE="${RALPH_STATE_FILE}.tmp.$$"
sed "s/^iteration: .*/iteration: $NEXT_ITERATION/" "$RALPH_STATE_FILE" > "$TEMP_FILE"
mv "$TEMP_FILE" "$RALPH_STATE_FILE"

# Block exit and feed prompt back
jq -n \
  --arg prompt "$PROMPT_TEXT" \
  --arg msg "🔄 Ralph iteration $NEXT_ITERATION" \
  '{
    "decision": "block",
    "reason": $prompt,
    "systemMessage": $msg
  }'

exit 0
```

**주요 패턴:**
1. **빠른 종료** (7-9행): 루프가 활성화되어 있지 않으면 건너뜀
2. **반복 추적** (11-20행): 횟수를 계산하고 최대 반복 횟수를 강제함
3. **완료 약속 감지** (25-33행): 출력에서 완료 신호가 감지되는지 검사
4. **프롬프트 추출** (38행): 마크다운 본문을 다음 프롬프트로 읽음
5. **상태 업데이트** (40-43행): 반복 횟수를 원자적으로 증가시킴
6. **루프 지속** (45-53행): 종료를 차단하고 프롬프트를 다시 제공

### 생성 (Creation)

**파일:** `scripts/setup-ralph-loop.sh`

```bash
#!/bin/bash
PROMPT="$1"
MAX_ITERATIONS="${2:-0}"
COMPLETION_PROMISE="${3:-}"

# Create state file
cat > ".claude/ralph-loop.local.md" <<EOF
---
iteration: 1
max_iterations: $MAX_ITERATIONS
completion_promise: "$COMPLETION_PROMISE"
started_at: "$(date -Iseconds)"
---

$PROMPT
EOF

echo "Ralph loop initialized: .claude/ralph-loop.local.md"
```

## 패턴 비교 (Pattern Comparison)

| 기능 (Feature) | multi-agent-swarm | ralph-loop |
|---------|-------------------|--------------|
| **파일** | `.claude/multi-agent-swarm.local.md` | `.claude/ralph-loop.local.md` |
| **목적** | 에이전트 조정 상태 | 루프 반복 상태 |
| **프론트매터** | 에이전트 메타데이터 | 루프 구성 |
| **본문** | 작업 할당 | 루프백할 프롬프트 |
| **업데이트** | PR 번호, 상태 | 반복 횟수 카운터 |
| **삭제** | 수동 또는 완료 시 | 루프 종료 시 |
| **훅** | Stop (알림 용도) | Stop (루프 제어 용도) |

## 실제 플러그인의 권장 사항 (Best Practices from Real Plugins)

### 1. 빠른 종료 패턴 (1. Quick Exit Pattern)

두 플러그인 모두 파일 존재 여부를 먼저 확인합니다:

```bash
if [[ ! -f "$STATE_FILE" ]]; then
  exit 0  # 활성화되지 않음
fi
```

**이유:** 플러그인이 구성되지 않았을 때 발생할 수 있는 오류를 방지하고 신속하게 처리됩니다.

### 2. Enabled 플래그 (2. Enabled Flag)

두 플러그인 모두 명시적 제어를 위해 `enabled` 필드를 사용합니다:

```yaml
enabled: true
```

**이유:** 파일을 삭제하지 않고도 일시적으로 비활성화할 수 있습니다.

### 3. 원자적 업데이트 (3. Atomic Updates)

두 플러그인 모두 임시 파일과 원자적 이동(move)을 사용합니다:

```bash
TEMP_FILE="${FILE}.tmp.$$"
sed "s/^field: .*/field: $NEW_VALUE/" "$FILE" > "$TEMP_FILE"
mv "$TEMP_FILE" "$FILE"
```

**이유:** 프로세스가 중간에 중단되더라도 파일이 손상되는 것을 방지합니다.

### 4. 따옴표 처리 (4. Quote Handling)

두 플러그인 모두 YAML 값에서 감싸고 있는 따옴표를 제거합니다:

```bash
sed 's/^"\(.*\)"$/\1/'
```

**이유:** YAML은 `field: value`와 `field: "value"` 형식을 모두 허용하기 때문입니다.

### 5. 에러 처리 (5. Error Handling)

두 플러그인 모두 누락되거나 손상된 파일을 정상적으로 처리합니다:

```bash
if [[ ! -f "$FILE" ]]; then
  exit 0  # 에러 없음, 단지 구성되지 않음
fi

if [[ -z "$CRITICAL_FIELD" ]]; then
  echo "Settings file corrupt" >&2
  rm "$FILE"  # 정리
  exit 0
fi
```

**이유:** 크래시를 내는 대신 오류 상황을 안전하게 복구합니다.

## 피해야 할 안티 패턴 (Anti-Patterns to Avoid)

### ❌ 하드코딩된 경로 (Hardcoded Paths)

```bash
# 나쁨 (BAD)
FILE="/Users/alice/.claude/my-plugin.local.md"

# 좋음 (GOOD)
FILE=".claude/my-plugin.local.md"
```

### ❌ 따옴표로 감싸지 않은 변수 (Unquoted Variables)

```bash
# 나쁨 (BAD)
echo $VALUE

# 좋음 (GOOD)
echo "$VALUE"
```

### ❌ 비원자적 업데이트 (Non-Atomic Updates)

```bash
# 나쁨 (BAD): 도중에 중단되면 파일이 손상될 수 있음
sed -i "s/field: .*/field: $VALUE/" "$FILE"

# 좋음 (GOOD): 원자적
TEMP_FILE="${FILE}.tmp.$$"
sed "s/field: .*/field: $VALUE/" "$FILE" > "$TEMP_FILE"
mv "$TEMP_FILE" "$FILE"
```

### ❌ 기본값 누락 (No Default Values)

```bash
# 나쁨 (BAD): 필드가 누락된 경우 실패함
if [[ $MAX -gt 100 ]]; then
  # MAX가 비어있을 수 있음!
fi

# 좋음 (GOOD): 기본값 제공
MAX=${MAX:-10}
```

### ❌ 예외 상황 무시 (Ignoring Edge Cases)

```bash
# 나쁨 (BAD): 정확히 2개의 --- 마커가 있다고 가정함
sed -n '/^---$/,/^---$/{ /^---$/d; p; }'

# 좋음 (GOOD): 본문 내에 ---가 있는 경우도 처리함
awk '/^---$/{i++; next} i>=2'  # 본문 추출
```

## 결론 (Conclusion)

`.claude/plugin-name.local.md` 패턴은 다음을 제공합니다:
- 간단하고 가독성 높은 구성
- 버전 관리 친화적 (gitignored)
- 프로젝트별 설정 지원
- 표준 bash 도구로 손쉬운 파싱 가능
- 구조화된 구성(YAML)과 자유 형식 콘텐츠(마크다운) 모두 지원

사용자가 구성해야 할 동작이나 지속성 있는 상태 관리가 필요한 모든 플러그인에 이 패턴을 사용하십시오.
