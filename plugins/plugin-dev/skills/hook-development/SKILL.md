---
name: hook-development
description: This skill should be used when the user asks to "create a hook", "add a PreToolUse/PostToolUse/Stop hook", "validate tool use", "implement prompt-based hooks", "use ${CLAUDE_PLUGIN_ROOT}", "set up event-driven automation", "block dangerous commands", or mentions hook events (PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, PreCompact, Notification). Provides comprehensive guidance for creating and implementing Claude Code plugin hooks with focus on advanced prompt-based hooks API.
version: 0.1.0
---

# Claude Code 플러그인을 위한 훅 개발 (Hook Development for Claude Code Plugins)

## 개요 (Overview)

훅(hooks)은 Claude Code 이벤트에 반응하여 실행되는 이벤트 기반 자동화 스크립트입니다. 훅을 사용하여 작업을 검증하고, 정책을 시행하고, 컨텍스트를 추가하고, 외부 도구를 워크플로우에 통합할 수 있습니다.

**핵심 기능:**
- 실행 전 도구 호출 검증 (PreToolUse)
- 도구 결과에 반응하여 처리 수행 (PostToolUse)
- 작업 완료 기준 검증 (Stop, SubagentStop)
- 프로젝트 컨텍스트 로드 (SessionStart)
- 개발 수명 주기 전반에 걸친 워크플로우 자동화

## 훅 유형 (Hook Types)

### 프롬프트 기반 훅 (권장) (Prompt-Based Hooks (Recommended))

컨텍스트 인식형 검증을 위해 LLM 구동 의사 결정을 활용합니다:

```json
{
  "type": "prompt",
  "prompt": "Evaluate if this tool use is appropriate: $TOOL_INPUT",
  "timeout": 30
}
```

**지원되는 이벤트:** Stop, SubagentStop, UserPromptSubmit, PreToolUse

**이점:**
- 자연어 추론에 기반한 컨텍스트 인식 의사 결정 가능
- bash 스크립팅이 필요 없는 유연한 검증 로직 구현
- 유연한 예외 상황 처리
- 유지보수 및 확장 용이

### 명령어 훅 (Command Hooks)

결정론적인 검사를 위해 bash 명령어를 실행합니다:

```json
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh",
  "timeout": 60
}
```

**용도:**
- 신속한 결정론적 유효성 검증
- 파일 시스템 연산 작업
- 외부 도구 연동
- 성능에 민감한 검증 단계

## 훅 구성 형식 (Hook Configuration Formats)

### 플러그인 hooks.json 형식 (Plugin hooks.json Format)

`hooks/hooks.json`에 정의하는 **플러그인용 훅**의 경우, 감싸는(wrapper) 형식을 사용합니다:

```json
{
  "description": "Brief explanation of hooks (optional)",
  "hooks": {
    "PreToolUse": [...],
    "Stop": [...],
    "SessionStart": [...]
  }
}
```

**핵심 사항:**
- `description` 필드는 선택 사항입니다.
- 실제 훅 이벤트를 포함하는 `hooks` 필드는 필수 감싸는(wrapper) 객체입니다.
- 이 방식이 **플러그인 전용 형식**입니다.

**예시:**
```json
{
  "description": "Validation hooks for code quality",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/hooks/validate.sh"
          }
        ]
      }
    ]
  }
}
```

### 설정 형식 (직접 지정) (Settings Format (Direct))

`.claude/settings.json`에 정의하는 **사용자 설정용 훅**의 경우, 직접 지정 형식을 사용합니다:

```json
{
  "PreToolUse": [...],
  "Stop": [...],
  "SessionStart": [...]
}
```

**핵심 사항:**
- 감싸는 wrapper가 없음 - 이벤트가 바로 최상위에 정의됨
- description 필드가 없음
- 이 방식이 **설정용 형식**입니다.

**중요:** 아래의 예시들은 두 형식 중 하나에 들어가는 훅 이벤트 구조를 보여줍니다. 플러그인 hooks.json의 경우, 이 예시들을 `{"hooks": {...}}` 내부에 넣어 사용하십시오.

## 훅 이벤트 (Hook Events)

### PreToolUse

도구가 실행되기 전에 동작합니다. 도구 호출을 승인(approve), 거부(deny), 혹은 수정(modify)할 때 사용합니다.

**예시 (프롬프트 기반):**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate file write safety. Check: system paths, credentials, path traversal, sensitive content. Return 'approve' or 'deny'."
        }
      ]
    }
  ]
}
```

**PreToolUse의 출력 구조:**
```json
{
  "hookSpecificOutput": {
    "permissionDecision": "allow|deny|ask",
    "updatedInput": {"field": "modified_value"}
  },
  "systemMessage": "Explanation for Claude"
}
```

### PostToolUse

도구 실행이 완료된 후 동작합니다. 결과에 반응하거나 피드백을 전달하고, 로그를 남길 때 사용합니다.

**예시:**
```json
{
  "PostToolUse": [
    {
      "matcher": "Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Analyze edit result for potential issues: syntax errors, security vulnerabilities, breaking changes. Provide feedback."
        }
      ]
    }
  ]
}
```

**출력 동작 방식:**
- 종료 코드 0: 표준 출력(stdout) 내용이 대화 기록에 표시됨
- 종료 코드 2: 표준 에러(stderr) 내용이 Claude에게 다시 피드백으로 전달됨
- systemMessage가 컨텍스트에 포함됨

### Stop

메인 에이전트가 동작을 멈추려고 할 때(Stop) 실행됩니다. 작업의 완성도와 종결 조건을 검증할 때 사용합니다.

**예시:**
```json
{
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Verify task completion: tests run, build succeeded, questions answered. Return 'approve' to stop or 'block' with reason to continue."
        }
      ]
    }
  ]
}
```

**결정 출력 구조:**
```json
{
  "decision": "approve|block",
  "reason": "Explanation",
  "systemMessage": "Additional context"
}
```

### SubagentStop

서브에이전트가 동작을 멈추려고 할 때 실행됩니다. 서브에이전트가 할당된 작업을 완료했는지 검증할 때 사용합니다.

Stop 훅과 동일하게 작동하나, 대상이 서브에이전트라는 점만 다릅니다.

### UserPromptSubmit

사용자가 프롬프트를 제출할 때 실행됩니다. 추가 컨텍스트를 주입하거나, 검증하고, 프롬프트를 차단할 때 사용합니다.

**예시:**
```json
{
  "UserPromptSubmit": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Check if prompt requires security guidance. If discussing auth, permissions, or API security, return relevant warnings."
        }
      ]
    }
  ]
}
```

### SessionStart

Claude Code 세션이 시작될 때 실행됩니다. 컨텍스트를 로드하고 환경을 설정할 때 사용합니다.

**예시:**
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

**특별 기능:** `$CLAUDE_ENV_FILE`을 사용해 환경 변수를 세션 전반에 유지할 수 있습니다:
```bash
echo "export PROJECT_TYPE=nodejs" >> "$CLAUDE_ENV_FILE"
```

동작하는 전체 예시는 `examples/load-context.sh`를 참조하십시오.

### SessionEnd

세션이 종료될 때 실행됩니다. 리소스 정리, 로그 기록, 상태 보존을 수행할 때 사용합니다.

### PreCompact

컨텍스트 압축(Compaction)이 발생하기 전에 동작합니다. 보존해야 하는 핵심 정보를 주입할 때 사용합니다.

### Notification

Claude가 알림을 보낼 때 실행됩니다. 사용자 알림에 반응하여 동작을 수행할 때 사용합니다.

## 훅 출력 형식 (Hook Output Format)

### 표준 출력 (모든 훅 공통) (Standard Output (All Hooks))

```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "Message for Claude"
}
```

- `continue`: false인 경우 처리를 중단합니다 (기본값: true)
- `suppressOutput`: 대화 기록에서 출력을 숨깁니다 (기본값: false)
- `systemMessage`: Claude에게 보여줄 메시지

### 종료 코드 (Exit Codes)

- `0` - 성공 (표준 출력 내용이 대화 기록에 표시됨)
- `2` - 차단 에러 (표준 에러 내용이 Claude에게 전달됨)
- 기타 - 비차단형 일반 에러

## 훅 입력 형식 (Hook Input Format)

모든 훅은 표준 입력(stdin)을 통해 다음 공통 필드를 가진 JSON 데이터를 전달받습니다:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.txt",
  "cwd": "/current/working/dir",
  "permission_mode": "ask|allow",
  "hook_event_name": "PreToolUse"
}
```

**이벤트별 추가 필드:**

- **PreToolUse/PostToolUse:** `tool_name`, `tool_input`, `tool_result`
- **UserPromptSubmit:** `user_prompt`
- **Stop/SubagentStop:** `reason`

프롬프트 내부에서 `$TOOL_INPUT`, `$TOOL_RESULT`, `$USER_PROMPT` 등을 사용해 해당 필드에 접근할 수 있습니다.

## 환경 변수 (Environment Variables)

모든 명령어 훅에서 사용 가능한 변수 목록:

- `$CLAUDE_PROJECT_DIR` - 프로젝트 루트 경로
- `$CLAUDE_PLUGIN_ROOT` - 플러그인 디렉토리 (이식 가능한 경로 설정을 위해 사용)
- `$CLAUDE_ENV_FILE` - SessionStart 전용: 여기에 작성하여 환경 변수를 유지함
- `$CLAUDE_CODE_REMOTE` - 원격 컨텍스트에서 실행 중일 때 설정됨

**이식성을 위해 훅 명령어 내부에서는 항상 ${CLAUDE_PLUGIN_ROOT}를 사용하십시오:**

```json
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## 플러그인 훅 구성 (Plugin Hook Configuration)

플러그인에서는 `hooks/hooks.json`에 훅을 정의합니다:

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
    }
  ],
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Verify task completion"
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
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/load-context.sh",
          "timeout": 10
        }
      ]
    }
  ]
}
```

플러그인 훅은 사용자의 훅과 병합되어 병렬로 실행됩니다.

## 매처 (Matchers)

### 도구 이름 매칭 (Tool Name Matching)

**정확한 일치:**
```json
"matcher": "Write"
```

**다중 도구 지정:**
```json
"matcher": "Read|Write|Edit"
```

**와일드카드 (모든 도구 대상):**
```json
"matcher": "*"
```

**정규식 패턴:**
```json
"matcher": "mcp__.*__delete.*"  // All MCP delete tools
```

**주의:** 매처는 대소문자를 구분합니다.

### 공통 패턴 (Common Patterns)

```json
// 모든 MCP 도구
"matcher": "mcp__.*"

// 특정 플러그인의 MCP 도구
"matcher": "mcp__plugin_asana_.*"

// 모든 파일 관련 연산
"matcher": "Read|Write|Edit"

// Bash 명령어 전용
"matcher": "Bash"
```

## 보안 권장 사항 (Security Best Practices)

### 입력 값 검증 (Input Validation)

명령어 훅 내에서는 항상 입력 값을 검증하십시오:

```bash
#!/bin/bash
set -euo pipefail

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')

# Validate tool name format
if [[ ! "$tool_name" =~ ^[a-zA-Z0-9_]+$ ]]; then
  echo '{"decision": "deny", "reason": "Invalid tool name"}' >&2
  exit 2
fi
```

### 경로 안전성 검증 (Path Safety)

디렉토리 탐색(path traversal) 시도 및 민감한 파일에 대한 쓰기 시도를 검증합니다:

```bash
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Deny path traversal
if [[ "$file_path" == *".."* ]]; then
  echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
  exit 2
fi

# Deny sensitive files
if [[ "$file_path" == *".env"* ]]; then
  echo '{"decision": "deny", "reason": "Sensitive file"}' >&2
  exit 2
fi
```

완전한 예시는 `examples/validate-write.sh` 및 `examples/validate-bash.sh`를 참조하십시오.

### 모든 변수 따옴표 감싸기 (Quote All Variables)

```bash
# 올바름 (GOOD)
echo "$file_path"
cd "$CLAUDE_PROJECT_DIR"

# 잘못됨 (BAD: 인젝션 위험이 있음)
echo $file_path
cd $CLAUDE_PROJECT_DIR
```

### 적절한 제한 시간 설정 (Set Appropriate Timeouts)

```json
{
  "type": "command",
  "command": "bash script.sh",
  "timeout": 10
}
```

**기본 제한 시간:** 명령어 훅 (60초), 프롬프트 훅 (30초)

## 성능 고려 사항 (Performance Considerations)

### 병렬 실행 (Parallel Execution)

조건과 매칭되는 모든 훅들은 **병렬로 동시 실행**됩니다:

```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {"type": "command", "command": "check1.sh"},  // Parallel
        {"type": "command", "command": "check2.sh"},  // Parallel
        {"type": "prompt", "prompt": "Validate..."}   // Parallel
      ]
    }
  ]
}
```

**설계 시 고려 사항:**
- 훅은 서로 다른 훅의 처리 결과를 볼 수 없습니다.
- 실행 순서가 결정론적이지 않습니다.
- 각 훅이 완전히 독립적으로 동작하도록 설계하십시오.

### 최적화 (Optimization)

1. 신속한 결정론적 확인 절차에는 명령어 훅을 사용하십시오.
2. 복잡한 추론과 분석이 필요한 영역에는 프롬프트 훅을 사용하십시오.
3. 임시 파일에 검증 결과를 캐싱해 두십시오.
4. 트래픽이 몰리는 핫 패스(hot paths) 구간에서 I/O 연산을 최소화하십시오.

## 일시적으로 활성화되는 훅 (Temporarily Active Hooks)

플래그 파일이나 구성을 체크하여 조건부로 활성화되는 훅을 설계합니다:

**패턴: 플래그 파일 활성화 방식**
```bash
#!/bin/bash
# Only active when flag file exists
FLAG_FILE="$CLAUDE_PROJECT_DIR/.enable-strict-validation"

if [ ! -f "$FLAG_FILE" ]; then
  # Flag not present, skip validation
  exit 0
fi

# Flag present, run validation
input=$(cat)
# ... validation logic ...
```

**패턴: 구성 정보 기반 활성화 방식**
```bash
#!/bin/bash
# Check configuration for activation
CONFIG_FILE="$CLAUDE_PROJECT_DIR/.claude/plugin-config.json"

if [ -f "$CONFIG_FILE" ]; then
  enabled=$(jq -r '.strictMode // false' "$CONFIG_FILE")
  if [ "$enabled" != "true" ]; then
    exit 0  # Not enabled, skip
  fi
fi

# Enabled, run hook logic
input=$(cat)
# ... hook logic ...
```

**사용 사례:**
- 필요할 때만 엄격한(strict) 검증 단계를 켬
- 임시 디버깅용 훅 가동
- 프로젝트별 개별적인 훅 행동 조율
- 훅에 대한 기능 플래그(Feature Flags) 적용

**권장 사항:** 사용자가 임시 훅을 켜고 끌 수 있는 방법을 알 수 있도록 플러그인 README 파일에 이 활성화 메커니즘을 함께 기술하십시오.

## 훅 수명 주기 및 제한 사항 (Hook Lifecycle and Limitations)

### 세션 시작 시 훅 로드 (Hooks Load at Session Start)

**중요:** 훅 정보는 Claude Code 세션이 시작될 때 메모리에 로드됩니다. 훅 구성을 수정한 경우 Claude Code를 다시 시작해야만 변경 사항이 반영됩니다.

**실시간 핫스왑 미지원:**
- `hooks/hooks.json`을 수정해도 현재 세션에 반영되지 않습니다.
- 새로 추가된 훅 스크립트 파일이 인식되지 않습니다.
- 변경된 훅 명령어/프롬프트가 적용되지 않습니다.
- 반드시 Claude Code를 종료하고 다시 `claude`를 실행하여 시작해야 합니다.

**훅 변경 사항 테스트 방법:**
1. 훅 구성 정보나 스크립트 코드를 수정합니다.
2. 실행 중인 Claude Code 세션을 종료합니다.
3. 재시작: `claude` 혹은 `cc` 명령어 실행
4. 새로운 훅 구성 정보가 로드됩니다.
5. `claude --debug` 명령어를 통해 훅 동작을 모니터링하며 테스트합니다.

### 시작 시 훅 검증 (Hook Validation at Startup)

Claude Code가 시작될 때 훅을 검증합니다:
- `hooks.json`에 유효하지 않은 JSON이 있으면 로드가 실패합니다.
- 참조된 스크립트가 존재하지 않으면 경고를 발생시킵니다.
- 구문 오류(syntax errors) 정보가 디버그 모드에서 표시됩니다.

현재 세션에 정상적으로 로드된 훅의 목록을 확인하려면 `/hooks` 명령어를 실행하십시오.

## 훅 디버깅 (Debugging Hooks)

### 디버그 모드 활성화 (Enable Debug Mode)

```bash
claude --debug
```

디버그 메시지 중에서 훅 등록, 실행 로그, 입력/출력 JSON 데이터 및 소요 시간 정보를 꼼꼼히 확인하십시오.

### 훅 스크립트 테스트 (Test Hook Scripts)

명령어 훅을 직접 격리 테스트합니다:

```bash
echo '{"tool_name": "Write", "tool_input": {"file_path": "/test"}}' | \
  bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh

echo "Exit code: $?"
```

### JSON 출력 검증 (Validate JSON Output)

훅 스크립트가 유효한 JSON 포맷을 반환하는지 검증합니다:

```bash
output=$(./your-hook.sh < test-input.json)
echo "$output" | jq .
```

## 빠른 참조 (Quick Reference)

### 훅 이벤트 요약 (Hook Events Summary)

| 이벤트 | 실행 시점 | 주요 용도 |
|-------|------|---------|
| PreToolUse | 도구 실행 직전 | 검증, 변경 |
| PostToolUse | 도구 완료 직후 | 피드백 전달, 로깅 |
| UserPromptSubmit | 사용자 입력 제출 시 | 컨텍스트 추가, 검증 |
| Stop | 에이전트 정지 시 | 완성도/조건 검사 |
| SubagentStop | 서브에이전트 종료 시 | 서브 작업 검증 |
| SessionStart | 세션 개시 시 | 컨텍스트 로딩 |
| SessionEnd | 세션 종료 시 | 리소스 정리, 로깅 |
| PreCompact | 압축 발생 직전 | 보존 컨텍스트 추가 |
| Notification | 알림 전달 시 | 알림 모니터링, 처리 |

### 권장 사항 (Best Practices)

**수행할 사항 (DO):**
- ✅ 복잡한 로직 검증을 위해 프롬프트 기반 훅 활용
- ✅ 이식 가능한 경로 조율을 위해 항상 `${CLAUDE_PLUGIN_ROOT}` 사용
- ✅ 명령어 훅 내부에서는 전달되는 모든 입력을 꼼꼼히 검증
- ✅ 모든 bash 변수를 큰따옴표로 감싸서 인젝션 방지
- ✅ 적절한 timeout 임계치 설정
- ✅ 잘 구조화된 JSON 데이터 반환
- ✅ 사전에 충분히 테스트 진행

**피해야 할 사항 (DON'T):**
- ❌ 절대 경로 하드코딩
- ❌ 검증하지 않은 사용자 입력을 그대로 신뢰
- ❌ 실행 시간이 오래 걸리는 무거운 훅 구성
- ❌ 다른 훅의 실행 순서에 기댄 로직 설계
- ❌ 전역 상태의 예측 불가능한 임의 수정
- ❌ 보안상 민감한 정보 로깅
- ❌ 거부 메시지를 누락

## 추가 리소스 (Additional Resources)

### 참조 파일

상세 패턴과 고급 기법은 아래 파일을 참조하십시오:

- **`references/patterns.md`** - 공통 훅 패턴 가이드 (8가지 이상의 모범 설계 패턴)
- **`references/migration.md`** - 기본 훅에서 고급 프롬프트 훅으로의 마이그레이션 가이드
- **`references/advanced.md`** - 고급 사용 사례 및 고난도 훅 설계 기법

### 예시 훅 스크립트

`examples/` 내 작동 가능한 예시:

- **`validate-write.sh`** - 파일 쓰기 검증 사례
- **`validate-bash.sh`** - Bash 명령어 실행 검증 사례
- **`load-context.sh`** - SessionStart 연동 컨텍스트 로딩 사례

### 개발 도구용 유틸리티 스크립트

`scripts/` 내 개발자 도구:

- **`validate-hook-schema.sh`** - hooks.json 구조 및 문법 유효성 검증 스크립트
- **`test-hook.sh`** - 배포 전 샘플 입력 기반의 격리 테스트 헬퍼
- **`hook-linter.sh`** - 훅 스크립트 코딩 규격 및 권장 설계 위반 여부 린터

### 외부 참조 링크

- **공식 문서**: https://docs.claude.com/en/docs/claude-code/hooks
- **마켓플레이스 사례**: 마켓플레이스에 등록된 security-guidance 플러그인 참조
- **로깅**: 상세 실행 흐름 확인을 위해 `claude --debug` 사용
- **검증**: 반환 JSON 검증을 위해 `jq` 활용

## 구현 워크플로우 (Implementation Workflow)

플러그인에 훅을 구현하는 절차:

1. 후킹하고자 하는 이벤트 유형을 식별합니다 (PreToolUse, Stop, SessionStart 등).
2. 프롬프트 기반 훅(유연한 비즈니스 로직)과 명령어 훅(결정론적 속도 검사) 중 적합한 유형을 결정합니다.
3. `hooks/hooks.json` 파일에 훅 구성을 작성합니다.
4. 명령어 훅을 선택한 경우 훅 스크립트를 구현합니다.
5. 모든 내부 파일 참조에는 반드시 `${CLAUDE_PLUGIN_ROOT}`를 사용합니다.
6. `scripts/validate-hook-schema.sh hooks/hooks.json` 명령어로 스키마 유효성을 검증합니다.
7. 배포 전에 `scripts/test-hook.sh`를 사용해 개별 스크립트 동작을 격리 테스트합니다.
8. `claude --debug` 명령어를 실행하여 실제 Claude Code 세션에서 동작을 모니터링하며 테스트합니다.
9. 플러그인 README 파일에 훅 구성 및 역할에 대해 친절하게 기술합니다.

대부분의 경우 프롬프트 기반의 유연한 훅 설정을 우선 고려하십시오. 연산 집약적이거나 단순 결정론적 확인에 대해서만 제한적으로 명령어 훅을 적용하십시오.
