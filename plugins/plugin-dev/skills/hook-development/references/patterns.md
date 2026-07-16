# 공통 훅 패턴 (Common Hook Patterns)

이 참조 문서는 Claude Code 훅 구현을 위해 널리 검증된 공통 패턴들을 제공합니다. 일반적인 훅 사용 사례에 맞게 이 패턴들을 시작점으로 활용하십시오.

## 패턴 1: 보안 검증 (Pattern 1: Security Validation)

프롬프트 기반 훅을 사용하여 위험한 파일 쓰기를 차단합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "File path: $TOOL_INPUT.file_path. Verify: 1) Not in /etc or system directories 2) Not .env or credentials 3) Path doesn't contain '..' traversal. Return 'approve' or 'deny'."
        }
      ]
    }
  ]
}
```

**용도:** 민감한 파일이나 시스템 디렉토리에 대한 쓰기 작업을 방지합니다.

## 패턴 2: 테스트 강제 적용 (Pattern 2: Test Enforcement)

종료(Stop)하기 전에 테스트가 정상 실행되었는지 확인합니다:

```json
{
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Review transcript. If code was modified (Write/Edit tools used), verify tests were executed. If no tests were run, block with reason 'Tests must be run after code changes'."
        }
      ]
    }
  ]
}
```

**용도:** 코드 품질 기준을 강제하고 완료되지 않은 불완전한 작업 처리를 방지합니다.

## 패턴 3: 컨텍스트 로딩 (Pattern 3: Context Loading)

세션이 시작될 때 프로젝트 맞춤형 컨텍스트를 로드합니다:

```json
{
  "SessionStart": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/load-context.sh"
        }
      ]
    }
  ]
}
```

**예시 스크립트 (load-context.sh):**
```bash
#!/bin/bash
cd "$CLAUDE_PROJECT_DIR" || exit 1

# Detect project type
if [ -f "package.json" ]; then
  echo "📦 Node.js project detected"
  echo "export PROJECT_TYPE=nodejs" >> "$CLAUDE_ENV_FILE"
elif [ -f "Cargo.toml" ]; then
  echo "🦀 Rust project detected"
  echo "export PROJECT_TYPE=rust" >> "$CLAUDE_ENV_FILE"
fi
```

**용도:** 프로젝트 전용 설정을 자동으로 감지하고 구성합니다.

## 패턴 4: 알림 로깅 (Pattern 4: Notification Logging)

감사(audit)나 분석을 위해 모든 알림을 로깅합니다:

```json
{
  "Notification": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/log-notification.sh"
        }
      ]
    }
  ]
}
```

**용도:** 사용자 대상 알림을 추적하거나 외부 로깅 시스템과 통합하는 데 사용합니다.

## 패턴 5: MCP 도구 모니터링 (Pattern 5: MCP Tool Monitoring)

MCP 도구의 사용을 모니터링하고 검증합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "mcp__.*__delete.*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Deletion operation detected. Verify: Is this deletion intentional? Can it be undone? Are there backups? Return 'approve' only if safe."
        }
      ]
    }
  ]
}
```

**용도:** 파괴적으로 동작하는 MCP 작업의 무단 실행을 방지합니다.

## 패턴 6: 빌드 검증 (Pattern 6: Build Verification)

코드를 변경한 후 빌드가 정상 완료되는지 보장합니다:

```json
{
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Check if code was modified. If Write/Edit tools were used, verify the project was built (npm run build, cargo build, etc). If not built, block and request build."
        }
      ]
    }
  ]
}
```

**용도:** 작업을 커밋하거나 완료하기 전에 빌드 오류를 사전에 감지합니다.

## 패턴 7: 권한 승인 (Pattern 7: Permission Confirmation)

위험한 작업을 실행하기 전에 사용자에게 승인을 요청합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Command: $TOOL_INPUT.command. If command contains 'rm', 'delete', 'drop', or other destructive operations, return 'ask' to confirm with user. Otherwise 'approve'."
        }
      ]
    }
  ]
}
```

**용도:** 잠재적으로 파괴적인 명령어가 실행되기 전에 사용자의 명시적인 확인을 거치도록 합니다.

## 패턴 8: 코드 품질 검사 (Pattern 8: Code Quality Checks)

파일을 수정한 후 린터나 포맷터를 자동으로 실행합니다:

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/check-quality.sh"
        }
      ]
    }
  ]
}
```

**예시 스크립트 (check-quality.sh):**
```bash
#!/bin/bash
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Run linter if applicable
if [[ "$file_path" == *.js ]] || [[ "$file_path" == *.ts ]]; then
  npx eslint "$file_path" 2>&1 || true
fi
```

**용도:** 코드 품질 표준을 자동으로 준수하도록 강제합니다.

## 패턴 조합 (Pattern Combinations)

여러 패턴을 결합하여 다중 레이어 보안 및 자동화를 구축합니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate file write safety"
        }
      ]
    },
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate bash command safety"
        }
      ]
    }
  ],
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Verify tests run and build succeeded"
        }
      ]
    }
  ],
  "SessionStart": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/load-context.sh"
        }
      ]
    }
  ]
}
```

이를 통해 다중 보안 레이어 구축 및 프로세스 자동화를 수행할 수 있습니다.

## 패턴 9: 일시적으로 활성화되는 훅 (Pattern 9: Temporarily Active Hooks)

플래그 파일이 명시적으로 존재할 때만 작동하는 훅을 설계합니다:

```bash
#!/bin/bash
# Hook only active when flag file exists
FLAG_FILE="$CLAUDE_PROJECT_DIR/.enable-security-scan"

if [ ! -f "$FLAG_FILE" ]; then
  # Quick exit when disabled
  exit 0
fi

# Flag present, run validation
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Run security scan
security-scanner "$file_path"
```

**활성화 절차:**
```bash
# Enable the hook
touch .enable-security-scan

# Disable the hook
rm .enable-security-scan
```

**용도:**
- 일시적으로 훅의 동작을 디버깅할 때
- 개발용 기능 플래그(Feature Flags)로 활용
- 프로젝트별 검증 절차에 대해 옵트인(Opt-in) 방식으로 구성하고자 할 때
- 필요할 때만 성능 집약적인 무거운 검증 단계를 수행할 때

**주의:** 훅이 플래그 파일의 변경 사항을 인식하도록 하려면 플래그 생성/제거 후에 반드시 Claude Code를 재시작해야 합니다.

## 패턴 10: 구성 구동형 훅 (Pattern 10: Configuration-Driven Hooks)

JSON 구성 정보를 읽어와서 훅의 세부 동작을 제어합니다:

```bash
#!/bin/bash
CONFIG_FILE="$CLAUDE_PROJECT_DIR/.claude/my-plugin.local.json"

# Read configuration
if [ -f "$CONFIG_FILE" ]; then
  strict_mode=$(jq -r '.strictMode // false' "$CONFIG_FILE")
  max_file_size=$(jq -r '.maxFileSize // 1000000' "$CONFIG_FILE")
else
  # Defaults
  strict_mode=false
  max_file_size=1000000
fi

# Skip if not in strict mode
if [ "$strict_mode" != "true" ]; then
  exit 0
fi

# Apply configured limits
input=$(cat)
file_size=$(echo "$input" | jq -r '.tool_input.content | length')

if [ "$file_size" -gt "$max_file_size" ]; then
  echo '{"decision": "deny", "reason": "File exceeds configured size limit"}' >&2
  exit 2
fi
```

**구성 파일 (.claude/my-plugin.local.json):**
```json
{
  "strictMode": true,
  "maxFileSize": 500000,
  "allowedPaths": ["/tmp", "/home/user/projects"]
}
```

**용도:**
- 사용자 정의에 기반하여 훅의 세부 동작을 조율할 때
- 프로젝트별 개별 설정 지원
- 팀 또는 부서별 고유 정책 적용
- 동적인 검증 조건 기준 적용
