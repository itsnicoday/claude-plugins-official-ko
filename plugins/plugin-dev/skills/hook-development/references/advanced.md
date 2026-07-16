# 고급 훅 사용 사례 (Advanced Hook Use Cases)

이 참조 문서는 고도화된 자동화 워크플로우를 위한 고급 훅 패턴 및 기법을 다룹니다.

## 다단계 검증 (Multi-Stage Validation)

명령어 훅과 프롬프트 훅을 결합하여 계층적 검증을 구현합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/quick-check.sh",
          "timeout": 5
        },
        {
          "type": "prompt",
          "prompt": "Deep analysis of bash command: $TOOL_INPUT",
          "timeout": 15
        }
      ]
    }
  ]
}
```

**사용 사례:** 신속한 결정론적 확인 후 지능적 분석 수행

**예시 quick-check.sh:**
```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

# Immediate approval for safe commands
if [[ "$command" =~ ^(ls|pwd|echo|date|whoami)$ ]]; then
  exit 0
fi

# Let prompt hook handle complex cases
exit 0
```

명령어 훅은 명백히 안전한 명령어를 빠르게 승인하고, 프롬프트 훅은 그 외의 모든 명령어를 상세히 분석합니다.

## 조건부 훅 실행 (Conditional Hook Execution)

환경 또는 컨텍스트에 따라 조건부로 훅을 실행합니다:

```bash
#!/bin/bash
# Only run in CI environment
if [ -z "$CI" ]; then
  echo '{"continue": true}' # Skip in non-CI
  exit 0
fi

# Run validation logic in CI
input=$(cat)
# ... validation code ...
```

**사용 사례:**
- CI 환경과 로컬 개발 환경에서의 서로 다른 동작 정의
- 프로젝트 전용 검증 규칙 설정
- 특정 사용자 전용 규칙 설정

**예시: 신뢰할 수 있는 사용자에 대한 특정 검증 건너뛰기:**
```bash
#!/bin/bash
# Skip detailed checks for admin users
if [ "$USER" = "admin" ]; then
  exit 0
fi

# Full validation for other users
input=$(cat)
# ... validation code ...
```

## 상태를 통한 훅 체이닝 (Hook Chaining via State)

임시 파일을 사용하여 훅 간에 상태를 공유합니다:

```bash
# Hook 1: Analyze and save state
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

# Analyze command
risk_level=$(calculate_risk "$command")
echo "$risk_level" > /tmp/hook-state-$$

exit 0
```

```bash
# Hook 2: Use saved state
#!/bin/bash
risk_level=$(cat /tmp/hook-state-$$ 2>/dev/null || echo "unknown")

if [ "$risk_level" = "high" ]; then
  echo "High risk operation detected" >&2
  exit 2
fi
```

**중요:** 이 방식은 순차적으로 발생히는 훅 이벤트(예: PreToolUse 실행 후 PostToolUse)에 대해서만 작동하며, 병렬로 실행되는 훅에서는 정상 동작하지 않습니다.

## 동적 훅 구성 (Dynamic Hook Configuration)

프로젝트별 구성 정보에 따라 훅의 동작을 동적으로 수정합니다:

```bash
#!/bin/bash
cd "$CLAUDE_PROJECT_DIR" || exit 1

# Read project-specific config
if [ -f ".claude-hooks-config.json" ]; then
  strict_mode=$(jq -r '.strict_mode' .claude-hooks-config.json)

  if [ "$strict_mode" = "true" ]; then
    # Apply strict validation
    # ...
  else
    # Apply lenient validation
    # ...
  fi
fi
```

**예시 .claude-hooks-config.json:**
```json
{
  "strict_mode": true,
  "allowed_commands": ["ls", "pwd", "grep"],
  "forbidden_paths": ["/etc", "/sys"]
}
```

## 컨텍스트 인식 프롬프트 훅 (Context-Aware Prompt Hooks)

대화 내용(transcript) 및 세션 컨텍스트를 활용하여 지능적인 결정을 내립니다:

```json
{
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Review the full transcript at $TRANSCRIPT_PATH. Check: 1) Were tests run after code changes? 2) Did the build succeed? 3) Were all user questions answered? 4) Is there any unfinished work? Return 'approve' only if everything is complete."
        }
      ]
    }
  ]
}
```

LLM은 대화 내용 파일을 읽어서 컨텍스트를 파악하고 지능적인 결정을 내릴 수 있습니다.

## 성능 최적화 (Performance Optimization)

### 검증 결과 캐싱 (Caching Validation Results)

```bash
#!/bin/bash
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')
cache_key=$(echo -n "$file_path" | md5sum | cut -d' ' -f1)
cache_file="/tmp/hook-cache-$cache_key"

# Check cache
if [ -f "$cache_file" ]; then
  cache_age=$(($(date +%s) - $(stat -f%m "$cache_file" 2>/dev/null || stat -c%Y "$cache_file")))
  if [ "$cache_age" -lt 300 ]; then  # 5 minute cache
    cat "$cache_file"
    exit 0
  fi
fi

# Perform validation
result='{"decision": "approve"}'

# Cache result
echo "$result" > "$cache_file"
echo "$result"
```

### 병렬 실행 최적화 (Parallel Execution Optimization)

훅은 병렬로 동시 실행되므로, 각 훅이 독립적으로 작동하도록 설계해야 합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {
          "type": "command",
          "command": "bash check-size.sh",      // Independent
          "timeout": 2
        },
        {
          "type": "command",
          "command": "bash check-path.sh",      // Independent
          "timeout": 2
        },
        {
          "type": "prompt",
          "prompt": "Check content safety",     // Independent
          "timeout": 10
        }
      ]
    }
  ]
}
```

세 개의 훅이 동시에 병렬로 동작하므로 전체 대기 시간이 크게 단축됩니다.

## 교차 이벤트 워크플로우 (Cross-Event Workflows)

서로 다른 훅 이벤트를 연계하여 작동하도록 구성합니다:

**SessionStart - 추적 환경 구성:**
```bash
#!/bin/bash
# Initialize session tracking
echo "0" > /tmp/test-count-$$
echo "0" > /tmp/build-count-$$
```

**PostToolUse - 이벤트 추적 및 로깅:**
```bash
#!/bin/bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')

if [ "$tool_name" = "Bash" ]; then
  command=$(echo "$input" | jq -r '.tool_result')
  if [[ "$command" == *"test"* ]]; then
    count=$(cat /tmp/test-count-$$ 2>/dev/null || echo "0")
    echo $((count + 1)) > /tmp/test-count-$$
  fi
fi
```

**Stop - 추적 상태에 따른 최종 검증:**
```bash
#!/bin/bash
test_count=$(cat /tmp/test-count-$$ 2>/dev/null || echo "0")

if [ "$test_count" -eq 0 ]; then
  echo '{"decision": "block", "reason": "No tests were run"}' >&2
  exit 2
fi
```

## 외부 시스템과의 통합 (Integration with External Systems)

### Slack 알림 (Slack Notifications)

```bash
#!/bin/bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
decision="blocked"

# Send notification to Slack
curl -X POST "$SLACK_WEBHOOK" \
  -H 'Content-Type: application/json' \
  -d "{\"text\": \"Hook ${decision} ${tool_name} operation\"}" \
  2>/dev/null

echo '{"decision": "deny"}' >&2
exit 2
```

### 데이터베이스 로깅 (Database Logging)

```bash
#!/bin/bash
input=$(cat)

# Log to database
psql "$DATABASE_URL" -c "INSERT INTO hook_logs (event, data) VALUES ('PreToolUse', '$input')" \
  2>/dev/null

exit 0
```

### 메트릭 수집 (Metrics Collection)

```bash
#!/bin/bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')

# Send metrics to monitoring system
echo "hook.pretooluse.${tool_name}:1|c" | nc -u -w1 statsd.local 8125

exit 0
```

## 보안 패턴 (Security Patterns)

### 속도 제한 (Rate Limiting)

```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

# Track command frequency
rate_file="/tmp/hook-rate-$$"
current_minute=$(date +%Y%m%d%H%M)

if [ -f "$rate_file" ]; then
  last_minute=$(head -1 "$rate_file")
  count=$(tail -1 "$rate_file")

  if [ "$current_minute" = "$last_minute" ]; then
    if [ "$count" -gt 10 ]; then
      echo '{"decision": "deny", "reason": "Rate limit exceeded"}' >&2
      exit 2
    fi
    count=$((count + 1))
  else
    count=1
  fi
else
  count=1
fi

echo "$current_minute" > "$rate_file"
echo "$count" >> "$rate_file"

exit 0
```

### 감사 로그 (Audit Logging)

```bash
#!/bin/bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
timestamp=$(date -Iseconds)

# Append to audit log
echo "$timestamp | $USER | $tool_name | $input" >> ~/.claude/audit.log

exit 0
```

### 비밀값 감지 (Secret Detection)

```bash
#!/bin/bash
input=$(cat)
content=$(echo "$input" | jq -r '.tool_input.content')

# Check for common secret patterns
if echo "$content" | grep -qE "(api[_-]?key|password|secret|token).{0,20}['\"]?[A-Za-z0-9]{20,}"; then
  echo '{"decision": "deny", "reason": "Potential secret detected in content"}' >&2
  exit 2
fi

exit 0
```

## 고급 훅 테스트 (Testing Advanced Hooks)

### 훅 스크립트 단위 테스트 (Unit Testing Hook Scripts)

```bash
# test-hook.sh
#!/bin/bash

# Test 1: Approve safe command
result=$(echo '{"tool_input": {"command": "ls"}}' | bash validate-bash.sh)
if [ $? -eq 0 ]; then
  echo "✓ Test 1 passed"
else
  echo "✗ Test 1 failed"
fi

# Test 2: Block dangerous command
result=$(echo '{"tool_input": {"command": "rm -rf /"}}' | bash validate-bash.sh)
if [ $? -eq 2 ]; then
  echo "✓ Test 2 passed"
else
  echo "✗ Test 2 failed"
fi
```

### 통합 테스트 (Integration Testing)

전체 훅 워크플로우를 동작시키는 통합 테스트 시나리오를 구성합니다:

```bash
# integration-test.sh
#!/bin/bash

# Set up test environment
export CLAUDE_PROJECT_DIR="/tmp/test-project"
export CLAUDE_PLUGIN_ROOT="$(pwd)"
mkdir -p "$CLAUDE_PROJECT_DIR"

# Test SessionStart hook
echo '{}' | bash hooks/session-start.sh
if [ -f "/tmp/session-initialized" ]; then
  echo "✓ SessionStart hook works"
else
  echo "✗ SessionStart hook failed"
fi

# Clean up
rm -rf "$CLAUDE_PROJECT_DIR"
```

## 고급 훅을 위한 권장 사항 (Best Practices for Advanced Hooks)

1. **훅의 독립성 유지**: 각 훅은 서로 실행 순서에 연동되어 설계되어서는 안 됩니다.
2. **제한 시간(Timeout) 적용**: 훅 유형별로 적절한 제한 임계치를 구성하십시오.
3. **정상적인 에러 처리**: 검증 실패 시 사용자에게 직관적이고 친절한 거절 사유를 보여주어야 합니다.
4. **복잡성 문서화**: 복잡한 작동 패턴은 README 등에 명확하게 기재합니다.
5. **철저한 테스트**: 예외 조건과 실패 모드에 대해 꼼꼼하게 검증합니다.
6. **성능 모니터링**: 훅의 런타임 지연 시간을 지속 추적하십시오.
7. **구성 정보 버전 관리**: 훅 설정 내역을 Git 등 버전 관리 솔루션으로 안전하게 관리합니다.
8. **비상 탈출구(Escape Hatches) 제공**: 필요한 경우 사용자가 훅을 우회(bypass)할 수 있는 수단을 마련합니다.

## 일반적인 실수 (Common Pitfalls)

### ❌ 훅 실행 순서 가정 (Assuming Hook Order)

```bash
# BAD: 훅이 순서대로 실행된다고 가정하는 경우
# Hook 1이 상태를 저장하고 Hook 2가 이를 읽음
# 훅은 병렬로 동시 실행되므로 이 방식은 실패할 수 있습니다!
```

### ❌ 실행 시간이 긴 훅 (Long-Running Hooks)

```bash
# BAD: 훅이 동작 완료되는 데 2분이 걸리는 경우
sleep 120
# 제한 시간이 초과되어 워크플로우 자체가 차단될 것입니다.
```

### ❌ 예외 처리 누락 (Uncaught Exceptions)

```bash
# BAD: 예기치 않은 입력이 들어왔을 때 스크립트 자체가 크래시됨
file_path=$(echo "$input" | jq -r '.tool_input.file_path')
cat "$file_path"  # 파일이 없는 경우 실패하여 에러 유발
```

### ✅ 적절한 에러 처리 (Proper Error Handling)

```bash
# GOOD: 에러가 발생해도 우아하게 복구하고 처리함
file_path=$(echo "$input" | jq -r '.tool_input.file_path')
if [ ! -f "$file_path" ]; then
  echo '{"continue": true, "systemMessage": "File not found, skipping check"}' >&2
  exit 0
fi
```

## 결론 (Conclusion)

고급 훅 패턴을 이용하면 시스템의 반응 속도와 신뢰성을 훼손하지 않으면서도 고도로 자동화된 세부 제어를 구현할 수 있습니다. 기본 훅으로 원하는 비즈니스 정합성을 만족시키기 어려울 때 이러한 기법을 사용하되, 언제나 단순성과 유지보수 편의성을 우선순위에 두십시오.
