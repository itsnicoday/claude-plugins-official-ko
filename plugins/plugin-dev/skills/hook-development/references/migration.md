# 기본 훅에서 고급 훅으로의 마이그레이션 (Migrating from Basic to Advanced Hooks)

이 가이드는 더 나은 유지보수성과 유연성을 확보하기 위해 기본 명령어 훅에서 고급 프롬프트 기반 훅으로 마이그레이션하는 방법을 안내합니다.

## 마이그레이션이 필요한 이유 (Why Migrate?)

프롬프트 기반 훅은 다음과 같은 여러 이점을 제공합니다:

- **자연어 추론**: LLM이 컨텍스트와 의도를 이해합니다.
- **우수한 예외 상황 처리**: 예기치 않은 시나리오에 대처합니다.
- **Bash 스크립트 불필요**: 작성이 간편하며 유지보수가 쉽습니다.
- **유연한 검증**: 복잡한 코딩 없이도 복잡한 검증 로직을 구현할 수 있습니다.

## 마이그레이션 예시: Bash 명령어 검증 (Migration Example: Bash Command Validation)

### 이전 (기본 명령어 훅) (Before (Basic Command Hook))

**구성 정보:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "bash validate-bash.sh"
        }
      ]
    }
  ]
}
```

**스크립트 (validate-bash.sh):**
```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

# Hard-coded validation logic
if [[ "$command" == *"rm -rf"* ]]; then
  echo "Dangerous command detected" >&2
  exit 2
fi
```

**문제점:**
- 하드코딩된 "rm -rf" 문자열 패턴만 확인합니다.
- `rm -fr` 또는 `rm -r -f` 같은 변형 구문을 감지하지 못합니다.
- 다른 위험한 명령어(`dd`, `mkfs` 등)를 놓치게 됩니다.
- 컨텍스트를 인지하지 못합니다.
- bash 스크립팅에 대한 사전 지식이 필요합니다.

### 이후 (고급 프롬프트 훅) (After (Advanced Prompt Hook))

**구성 정보:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Command: $TOOL_INPUT.command. Analyze for: 1) Destructive operations (rm -rf, dd, mkfs, etc) 2) Privilege escalation (sudo) 3) Network operations without user consent. Return 'approve' or 'deny' with explanation.",
          "timeout": 15
        }
      ]
    }
  ]
}
```

**이점:**
- 모든 형태의 변형 구문 및 패턴을 감지합니다.
- 리터럴 문자열이 아닌 수행하려는 '의도'를 파악합니다.
- 별도의 스크립트 파일이 필요 없습니다.
- 새로운 검증 기준을 추가하기 쉽습니다.
- 컨텍스트 기반 의사 결정이 가능합니다.
- 거부 시 자연어 형식으로 거부 이유를 기술합니다.

## 마이그레이션 예시: 파일 쓰기 검증 (Migration Example: File Write Validation)

### 이전 (기본 명령어 훅) (Before (Basic Command Hook))

**구성 정보:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {
          "type": "command",
          "command": "bash validate-write.sh"
        }
      ]
    }
  ]
}
```

**스크립트 (validate-write.sh):**
```bash
#!/bin/bash
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Check for path traversal
if [[ "$file_path" == *".."* ]]; then
  echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
  exit 2
fi

# Check for system paths
if [[ "$file_path" == "/etc/"* ]] || [[ "$file_path" == "/sys/"* ]]; then
  echo '{"decision": "deny", "reason": "System file"}' >&2
  exit 2
fi
```

**문제점:**
- 경로 패턴이 하드코딩되어 있습니다.
- 심볼릭 링크(symlinks)를 인식하지 못합니다.
- 누락되는 경로 관련 엣지 케이스가 있습니다 (예: `/etc`와 `/etc/` 차이).
- 파일 내용을 고려하지 않습니다.

### 이후 (고급 프롬프트 훅) (After (Advanced Prompt Hook))

**구성 정보:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "File path: $TOOL_INPUT.file_path. Content preview: $TOOL_INPUT.content (first 200 chars). Verify: 1) Not system directories (/etc, /sys, /usr) 2) Not credentials (.env, tokens, secrets) 3) No path traversal 4) Content doesn't expose secrets. Return 'approve' or 'deny'."
        }
      ]
    }
  ]
}
```

**이점:**
- 컨텍스트 인식이 가능합니다 (내용도 함께 고려).
- 심볼릭 링크 및 다양한 예외 상황을 정상적으로 처리합니다.
- "시스템 디렉토리"의 의미를 자연어로 이해합니다.
- 콘텐츠에 포함된 보안 비밀값 노출을 감지할 수 있습니다.
- 조건 기준을 유연하게 확장할 수 있습니다.

## 명령어 훅을 계속 사용해야 하는 경우 (When to Keep Command Hooks)

명령어 훅도 여전히 다음과 같은 특정 영역에서 유용합니다:

### 1. 결정론적 성능 검증 (1. Deterministic Performance Checks)

```bash
#!/bin/bash
# Check file size quickly
file_path=$(echo "$input" | jq -r '.tool_input.file_path')
size=$(stat -f%z "$file_path" 2>/dev/null || stat -c%s "$file_path" 2>/dev/null)

if [ "$size" -gt 10000000 ]; then
  echo '{"decision": "deny", "reason": "File too large"}' >&2
  exit 2
fi
```

**명령어 훅을 사용할 때:** 검증 조건이 온전히 수학적이거나 결정론적(deterministic)일 때.

### 2. 외부 도구 통합 (2. External Tool Integration)

```bash
#!/bin/bash
# Run security scanner
file_path=$(echo "$input" | jq -r '.tool_input.file_path')
scan_result=$(security-scanner "$file_path")

if [ "$?" -ne 0 ]; then
  echo "Security scan failed: $scan_result" >&2
  exit 2
fi
```

**명령어 훅을 사용할 때:** 통과/실패 결과를 제공하는 외부 스캐너/도구와 통합할 때.

### 3. 매우 신속한 검증 (< 50ms) (3. Very Fast Checks (< 50ms))

```bash
#!/bin/bash
# Quick regex check
command=$(echo "$input" | jq -r '.tool_input.command')

if [[ "$command" =~ ^(ls|pwd|echo)$ ]]; then
  exit 0  # Safe commands
fi
```

**명령어 훅을 사용할 때:** 성능이 매우 중요하며 검증 로직이 간단할 때.

## 하이브리드 접근 방식 (Hybrid Approach)

다단계 검증을 위해 두 가지 방식을 결합할 수 있습니다:

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

명령어 훅이 신속하게 결정론적 검증을 처리하고, 프롬프트 훅이 복잡한 추론 영역을 전담합니다.

## 마이그레이션 체크리스트 (Migration Checklist)

훅 마이그레이션 절차:

- [ ] 명령어 훅 내부의 검증 로직 식별
- [ ] 하드코딩된 문자열 패턴을 자연어 검증 기준으로 반환
- [ ] 기존 훅이 차단하지 못했던 예외 상황을 테스트
- [ ] LLM이 검증 의도를 올바르게 이해하는지 확인
- [ ] 적절한 제한 시간 설정 (프롬프트 훅의 경우 보통 15-30초)
- [ ] README에 새로운 훅 정보 문서화
- [ ] 사용이 끝난 이전 스크립트 파일을 정리 및 삭제

## 마이그레이션 팁 (Migration Tips)

1. **점진적 시작**: 한 번에 모든 훅을 이전하지 말고 하나씩 처리하십시오.
2. **철저한 테스트**: 새 프롬프트 훅이 기존 명령어 훅의 검증 내역을 정상적으로 차단하는지 검증합니다.
3. **지속 개선**: 마이그레이션을 검증 규칙을 보완하고 강화하는 기회로 활용하십시오.
4. **이전 코드 보존**: 로직 참조를 위해 기존 스크립트를 아카이브 폴더에 저장해 둡니다.
5. **이유 기술**: README에 왜 프롬프트 기반 훅이 권장되는지 사유를 문서화합니다.

## 종합 마이그레이션 예시 (Complete Migration Example)

### 원래 플러그인 구조 (Original Plugin Structure)

```
my-plugin/
├── .claude-plugin/plugin.json
├── hooks/hooks.json
└── scripts/
    ├── validate-bash.sh
    ├── validate-write.sh
    └── check-tests.sh
```

### 마이그레이션 이후 (After Migration)

```
my-plugin/
├── .claude-plugin/plugin.json
├── hooks/hooks.json      # Now uses prompt hooks
└── scripts/              # Archive or delete
    └── archive/
        ├── validate-bash.sh
        ├── validate-write.sh
        └── check-tests.sh
```

### 업데이트된 hooks.json (Updated hooks.json)

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate bash command safety: destructive ops, privilege escalation, network access"
        }
      ]
    },
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate file write safety: system paths, credentials, path traversal, content secrets"
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
          "prompt": "Verify tests were run if code was modified"
        }
      ]
    }
  ]
}
```

**결과:** 더 단순하고, 유지보수가 쉬우며, 강력하게 동작합니다.

## 공통 마이그레이션 패턴 (Common Migration Patterns)

### 패턴: 문자열 포함 검사 → 자연어 검사 (Pattern: String Contains → Natural Language)

**이전:**
```bash
if [[ "$command" == *"sudo"* ]]; then
  echo "Privilege escalation" >&2
  exit 2
fi
```

**이후:**
```
"Check for privilege escalation (sudo, su, etc)"
```

### 패턴: 정규식 검사 → 의도 분석 (Pattern: Regex → Intent)

**이전:**
```bash
if [[ "$file" =~ \.(env|secret|key|token)$ ]]; then
  echo "Credential file" >&2
  exit 2
fi
```

**이후:**
```
"Verify not writing to credential files (.env, secrets, keys, tokens)"
```

### 패턴: 다중 조건 검사 → 기준 목록 검사 (Pattern: Multiple Conditions → Criteria List)

**이전:**
```bash
if [ condition1 ] || [ condition2 ] || [ condition3 ]; then
  echo "Invalid" >&2
  exit 2
fi
```

**이후:**
```
"Check: 1) condition1 2) condition2 3) condition3. Deny if any fail."
```

## 결론 (Conclusion)

프롬프트 기반 훅으로 마이그레이션하면 플러그인이 더 간단해지고 유지보수가 쉬워지며 강력하게 작동합니다. 명령어 훅은 결정론적 검증 및 외부 도구 통합 용도로만 최소한으로 활용하십시오.
