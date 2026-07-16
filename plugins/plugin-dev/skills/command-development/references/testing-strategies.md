# 명령어 테스트 전략 (Command Testing Strategies)

배포 및 배포 전 슬래시 명령어를 테스트하기 위한 종합 전략입니다.

## 개요

명령어를 테스트함으로써 제대로 작동하는지, 예외 상황(edge cases)을 잘 처리하는지, 우수한 사용자 경험(UX)을 주는지 확인할 수 있습니다. 체계적인 테스트 접근 방식은 이슈를 일찍 포착하고 명령어의 신뢰도를 높여줍니다.

## 테스트 단계 (Testing Levels)

### 1단계: 구문 및 구조 검증

**테스트 항목:**
- YAML 프론트매터 구문
- 마크다운 형식
- 파일 위치 및 명명 규칙

**테스트 방법:**

```bash
# Validate YAML frontmatter
head -n 20 .claude/commands/my-command.md | grep -A 10 "^---"

# Check for closing frontmatter marker
head -n 20 .claude/commands/my-command.md | grep -c "^---" # 2가 나와야 함

# Verify file has .md extension
ls .claude/commands/*.md

# Check file is in correct location
test -f .claude/commands/my-command.md && echo "Found" || echo "Missing"
```

**자동 검증 스크립트:**

```bash
#!/bin/bash
# validate-command.sh

COMMAND_FILE="$1"

if [ ! -f "$COMMAND_FILE" ]; then
  echo "ERROR: File not found: $COMMAND_FILE"
  exit 1
fi

# Check .md extension
if [[ ! "$COMMAND_FILE" =~ \.md$ ]]; then
  echo "ERROR: File must have .md extension"
  exit 1
fi

# Validate YAML frontmatter if present
if head -n 1 "$COMMAND_FILE" | grep -q "^---"; then
  # Count frontmatter markers
  MARKERS=$(head -n 50 "$COMMAND_FILE" | grep -c "^---")
  if [ "$MARKERS" -ne 2 ]; then
    echo "ERROR: Invalid YAML frontmatter (need exactly 2 '---' markers)"
    exit 1
  fi
  echo "✓ YAML frontmatter syntax valid"
fi

# Check for empty file
if [ ! -s "$COMMAND_FILE" ]; then
  echo "ERROR: File is empty"
  exit 1
fi

echo "✓ Command file structure valid"
```

### 2단계: 프론트매터 필드 검증

**테스트 항목:**
- 올바른 필드 타입
- 유효 범위 내의 값
- 필수 필드 존재 여부 (있는 경우)

**검증 스크립트:**

```bash
#!/bin/bash
# validate-frontmatter.sh

COMMAND_FILE="$1"

# Extract YAML frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/p' "$COMMAND_FILE" | sed '1d;$d')

if [ -z "$FRONTMATTER" ]; then
  echo "No frontmatter to validate"
  exit 0
fi

# Check 'model' field if present
if echo "$FRONTMATTER" | grep -q "^model:"; then
  MODEL=$(echo "$FRONTMATTER" | grep "^model:" | cut -d: -f2 | tr -d ' ')
  if ! echo "sonnet opus haiku" | grep -qw "$MODEL"; then
    echo "ERROR: Invalid model '$MODEL' (must be sonnet, opus, or haiku)"
    exit 1
  fi
  echo "✓ Model field valid: $MODEL"
fi

# Check 'allowed-tools' field format
if echo "$FRONTMATTER" | grep -q "^allowed-tools:"; then
  echo "✓ allowed-tools field present"
  # 필요에 따라 정교한 검증 로직 추가 가능
fi

# Check 'description' length
if echo "$FRONTMATTER" | grep -q "^description:"; then
  DESC=$(echo "$FRONTMATTER" | grep "^description:" | cut -d: -f2-)
  LENGTH=${#DESC}
  if [ "$LENGTH" -gt 80 ]; then
    echo "WARNING: Description length $LENGTH (recommend < 60 chars)"
  else
    echo "✓ Description length acceptable: $LENGTH chars"
  fi
fi

echo "✓ Frontmatter fields valid"
```

### 3단계: 수동 명령어 호출 테스트

**테스트 항목:**
- 명령어가 `/help` 도움말 목록에 표시되는지
- 명령어 실행 시 에러가 나지 않는지
- 출력이 예상대로 나오는지

**테스트 절차:**

```bash
# 1. Start Claude Code
claude --debug

# 2. Check command appears in help
> /help
# 도움말 목록에서 개발한 명령어 이름을 찾아봅니다.

# 3. Invoke command without arguments
> /my-command
# 수용 가능한 에러나 기본 반응이 나오는지 확인합니다.

# 4. Invoke with valid arguments
> /my-command arg1 arg2
# 정상적으로 동작하는지 검사합니다.

# 5. Check debug logs
tail -f ~/.claude/debug-logs/latest
# 에러나 경고가 발생하는지 모니터링합니다.
```

### 4단계: 인자(Argument) 테스트

**테스트 항목:**
- 위치 인자가 잘 동작하는지 ($1, $2 등)
- $ARGUMENTS가 모든 인자를 잘 캡처하는지
- 누락된 인자를 우아하게 처리하는지
- 잘못된 인자를 잘 감지해내는지

**테스트 매트릭스:**

| 테스트 케이스 | 명령어 | 예상 결과 |
|---------------|--------|-----------|
| 인자 없음 | `/cmd` | 예외 상황의 부드러운 처리 및 사용법 안내 출력 |
| 단일 인자 | `/cmd arg1` | $1 위치에 정상 치환됨 |
| 두 개의 인자 | `/cmd arg1 arg2` | $1 및 $2 위치에 각각 정상 치환됨 |
| 초과 인자 | `/cmd a b c d` | 전체가 정상 캡처되거나, 초과분이 적절히 무시됨 |
| 특수 문자 | `/cmd "arg with spaces"` | 따옴표가 정상적으로 처리되어 전달됨 |
| 빈 인자 | `/cmd ""` | 빈 문자열에 맞게 정상 처리됨 |

**테스트 스크립트:**

```bash
#!/bin/bash
# test-command-arguments.sh

COMMAND="$1"

echo "Testing argument handling for /$COMMAND"
echo

echo "Test 1: No arguments"
echo "  Command: /$COMMAND"
echo "  Expected: [예상 동작 기술]"
echo "  Manual test required"
echo

echo "Test 2: Single argument"
echo "  Command: /$COMMAND test-value"
echo "  Expected: 'test-value'가 출력에 표시됨"
echo "  Manual test required"
echo

echo "Test 3: Multiple arguments"
echo "  Command: /$COMMAND arg1 arg2 arg3"
echo "  Expected: 모든 인자가 쓰임새에 맞게 적절히 처리됨"
echo "  Manual test required"
echo

echo "Test 4: Special characters"
echo "  Command: /$COMMAND \"value with spaces\""
echo "  Expected: 공백을 포함한 문구 전체가 하나의 인자로 캡처됨"
echo "  Manual test required"
```

### 5단계: 파일 참조 테스트

**테스트 항목:**
- `@` 구문이 파일 내용을 올바르게 로드하는지
- 존재하지 않는 파일이 주어졌을 때 잘 처리하는지
- 대용량 파일을 적절히 제어하는지
- 여러 파일 참조가 정상 작동하는지

**테스트 절차:**

```bash
# 임시 테스트 파일 생성
echo "Test content" > /tmp/test-file.txt
echo "Second file" > /tmp/test-file-2.txt

# 단일 파일 참조 테스트
> /my-command /tmp/test-file.txt
# 파일 내용을 올바르게 읽는지 검증합니다.

# 존재하지 않는 파일 테스트
> /my-command /tmp/nonexistent.txt
# 에러를 부드럽게 핸들링하는지 확인합니다.

# 다중 파일 참조 테스트
> /my-command /tmp/test-file.txt /tmp/test-file-2.txt
# 두 파일 모두 누락 없이 처리되는지 검사합니다.

# 대용량 파일 테스트
dd if=/dev/zero of=/tmp/large-file.bin bs=1M count=100
> /my-command /tmp/large-file.bin
# 대용량 파일에 대해 경고를 띄우거나 적절히 자르는 등 안정성을 유지하는지 확인합니다.

# 정리
rm /tmp/test-file*.txt /tmp/large-file.bin
```

### 6단계: Bash 실행 테스트

**테스트 항목:**
- `!` 명령어가 올바르게 실행되는지
- 명령어 출력이 프롬프트에 포함되는지
- 명령어 실행 실패를 감지하는지
- 보안: 허용된 명령어만 실행되는지

**테스트 절차:**

```bash
# Bash 실행 구문이 포함된 테스트 명령어 생성
cat > .claude/commands/test-bash.md << 'EOF'
---
description: Test bash execution
allowed-tools: Bash(echo:*), Bash(date:*)
---

Current date: !`date`
Test output: !`echo "Hello from bash"`

Analysis of output above...
EOF

# Claude Code에서 테스트 실행
> /test-bash
# 다음 사항을 확인합니다:
# 1. 날짜 정보가 정상적으로 나타남
# 2. echo 문구 내용이 출력됨
# 3. 디버그 로그에 에러 흔적이 없음

# 허용되지 않은 금지 명령어 테스트 (차단되거나 실패해야 함)
cat > .claude/commands/test-forbidden.md << 'EOF'
---
description: Test forbidden command
allowed-tools: Bash(echo:*)
---

Trying forbidden: !`ls -la /`
EOF

> /test-forbidden
# 확인: 권한 에러(Permission denied) 등 적절한 오류 처리가 수행됨
```

### 7단계: 통합 테스트

**테스트 항목:**
- 명령어가 다른 플러그인 구성 요소와 유기적으로 잘 작동하는지
- 명령어끼리 상호작용이 올바르게 이루어지는지
- 여러 실행 간에 상태 관리가 작동하는지
- 워크플로우 명령어가 순차적으로 실행되는지

**테스트 시나리오:**

**시나리오 1: 명령어 + 훅 통합**

```bash
# 설정: 훅을 트리거하는 명령어 준비
# 테스트: 명령어를 실행하여 훅이 정상 트리거되는지 검증

# 명령어 파일: .claude/commands/risky-operation.md
# 훅 파일: 해당 작업을 사전에 감사 및 제어하는 PreToolUse 훅

> /risky-operation
# 검증: 명령어가 실행을 끝내기 전, 훅이 정상 가동되어 검증을 마침
```

**시나리오 2: 명령어 시퀀스**

```bash
# 설정: 다단계 워크플로우 시퀀스 구성
> /workflow-init
# 검증: 상태 기록 파일이 정상 생성됨

> /workflow-step2
# 검증: 기존 상태 파일을 성공적으로 읽어와 다음 단계로 연동됨

> /workflow-complete
# 검증: 최종 처리를 마치고 상태 파일이 깔끔하게 지워짐
```

**시나리오 3: 명령어 + MCP 통합**

```bash
# 설정: MCP 도구를 사용하는 명령어 구성
# 테스트: MCP 서버에 접근 가능한지 검증

> /mcp-command
# 검증:
# 1. MCP 서버가 정상 시작됨 (stdio 연결 형태 등)
# 2. 도구 호출(Tool calls)이 성공적으로 수행됨
# 3. 획득한 결과가 출력물에 반영됨
```

## 자동화 테스트 접근 방식 (Automated Testing Approaches)

### 명령어 테스트 스위트 (Command Test Suite)

테스트 스위트 스크립트 작성:

```bash
#!/bin/bash
# test-commands.sh - Command test suite

TEST_DIR=".claude/commands"
FAILED_TESTS=0

echo "Command Test Suite"
echo "=================="
echo

for cmd_file in "$TEST_DIR"/*.md; do
  cmd_name=$(basename "$cmd_file" .md)
  echo "Testing: $cmd_name"

  # 구조 검사
  if ./validate-command.sh "$cmd_file"; then
    echo "  ✓ Structure valid"
  else
    echo "  ✗ Structure invalid"
    ((FAILED_TESTS++))
  fi

  # 프론트매터 검사
  if ./validate-frontmatter.sh "$cmd_file"; then
    echo "  ✓ Frontmatter valid"
  else
    echo "  ✗ Frontmatter invalid"
    ((FAILED_TESTS++))
  fi

  echo
done

echo "=================="
echo "Tests complete"
echo "Failed: $FAILED_TESTS"

exit $FAILED_TESTS
```

### Pre-Commit 훅

커밋하기 전에 명령어를 검증합니다:

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Validating commands..."

COMMANDS_CHANGED=$(git diff --cached --name-only | grep "\.claude/commands/.*\.md")

if [ -z "$COMMANDS_CHANGED" ]; then
  echo "No commands changed"
  exit 0
fi

for cmd in $COMMANDS_CHANGED; do
  echo "Checking: $cmd"

  if ! ./scripts/validate-command.sh "$cmd"; then
    echo "ERROR: Command validation failed: $cmd"
    exit 1
  fi
done

echo "✓ All commands valid"
```

### 지속적 테스트 (Continuous Testing)

CI/CD 환경에서 명령어를 테스트합니다:

```yaml
# .github/workflows/test-commands.yml
name: Test Commands

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Validate command structure
        run: |
          for cmd in .claude/commands/*.md; do
            echo "Testing: $cmd"
            ./scripts/validate-command.sh "$cmd"
          done

      - name: Validate frontmatter
        run: |
          for cmd in .claude/commands/*.md; do
            ./scripts/validate-frontmatter.sh "$cmd"
          done

      - name: Check for TODOs
        run: |
          if grep -r "TODO" .claude/commands/; then
            echo "ERROR: TODOs found in commands"
            exit 1
          fi
```

## 예외 상황(Edge Case) 테스트

### 예외 상황 테스트 항목

**빈 인자:**
```bash
> /cmd ""
> /cmd '' ''
```

**특수 문자:**
```bash
> /cmd "arg with spaces"
> /cmd arg-with-dashes
> /cmd arg_with_underscores
> /cmd arg/with/slashes
> /cmd 'arg with "quotes"'
```

**너무 긴 인자:**
```bash
> /cmd $(python -c "print('a' * 10000)")
```

**특이한 파일 경로:**
```bash
> /cmd ./file
> /cmd ../file
> /cmd ~/file
> /cmd "/path with spaces/file"
```

**Bash 명령어 예외 상황:**
```markdown
# 실패 가능성이 높은 명령어들
!`exit 1`
!`false`
!`command-that-does-not-exist`

# 내용이 특이한 출력 결과들
!`echo ""`
!`cat /dev/null`
!`yes | head -n 1000000`
```

## 성능 테스트 (Performance Testing)

### 응답 시간 테스트

```bash
#!/bin/bash
# test-command-performance.sh

COMMAND="$1"

echo "Testing performance of /$COMMAND"
echo

for i in {1..5}; do
  echo "Run $i:"
  START=$(date +%s%N)

  # 명령어 호출 안내 (종료 시간은 수동 체크가 필요함)
  echo "  Invoke: /$COMMAND"
  echo "  Start time: $START"
  echo "  (Record end time manually)"
  echo
done

echo "Analyze results:"
echo "  - Average response time"
echo "  - Variance"
echo "  - Acceptable threshold: 빠른 명령어의 경우 3초 미만"
```

### 리소스 사용량 테스트

```bash
# 명령어 실행 중 Claude Code 프로세스 추적
# 터미널 1:
claude --debug

# 터미널 2:
watch -n 1 'ps aux | grep claude'

# 명령어를 실행하며 다음 지표를 모니터링:
# - 메모리 사용량
# - CPU 사용량
# - 프로세스 개수
```

## 사용자 경험(UX) 테스트

### 사용성 체크리스트

- [ ] 명령어 이름이 직관적이고 기억하기 쉬움
- [ ] 설명이 `/help` 목록에 올바르게 잘 표시됨
- [ ] 예상되는 인자(arguments)들이 친절히 설명됨
- [ ] 에러 메시지가 구체적이고 대안을 포함하여 유용함
- [ ] 가독성을 높일 수 있는 텍스트 포맷팅이 출력에 적용됨
- [ ] 실행 시간이 긴 명령어의 경우 진행률(progress)을 보여줌
- [ ] 분석 또는 체크 결과가 명확하고 실행 가능한 가이드를 제공함
- [ ] 오류 상황이나 예외 처리 시 좋은 UX가 가미되어 있음

### 사용자 인수 테스트

베타 테스터 구성 및 테스트 가이드 배포:

```markdown
# Testing Guide for Beta Testers

## Command: /my-new-command

### Test Scenarios

1. **Basic usage:**
   - Run: `/my-new-command`
   - Expected: [상세 동작 기술]
   - Rate clarity: 1-5

2. **With arguments:**
   - Run: `/my-new-command arg1 arg2`
   - Expected: [상세 동작 기술]
   - Rate usefulness: 1-5

3. **Error case:**
   - Run: `/my-new-command invalid-input`
   - Expected: 유용한 오류 메시지 출력
   - Rate error message: 1-5

### Feedback Questions

1. 명령어가 직관적이고 쉽게 이해되셨나요?
2. 출력 결과가 본래 의도와 잘 들어맞았나요?
3. 개선하거나 수정하고 싶으신 부분이 있으신가요?
4. 앞으로 이 명령어를 일상에서 자주 사용하실 생각이 있으신가요?
```

## 테스트 체크리스트

명령어 릴리즈 전 체크리스트:

### 구조 검증
- [ ] 파일이 올바른 위치에 저장됨
- [ ] 파일 확장자가 `.md`인지 확인
- [ ] 프론트매터의 YAML 형식에 에러가 없음
- [ ] 마크다운 구문이 제대로 렌더링됨

### 기능 검증
- [ ] 명령어가 `/help` 도움말 목록에 정상 등록됨
- [ ] 설명이 목적에 맞게 작성됨
- [ ] 실행 시 예외 에러 없이 정상적으로 마침
- [ ] 각 매개변수 인자들이 적절하게 해석됨
- [ ] 파일 참조가 성공적으로 파일 내용을 로드함
- [ ] Bash 실행 구문(`!`)이 의도대로 동작함

### 예외 상황 처리
- [ ] 누락된 인자(arguments)가 부드럽게 무시되거나 에러 안내됨
- [ ] 비정상적인 인자를 감지하여 차단함
- [ ] 존재하지 않는 파일 참조 시 에러가 명시됨
- [ ] 입력값에 포함된 특수 문자가 정상적으로 탈출(escape) 처리됨
- [ ] 너무 긴 입력값에도 무너지지 않고 안정적으로 버팀

### 통합 연동
- [ ] 연계된 다른 명령어들과 함께 잘 맞물림
- [ ] 훅(hooks) 작동과 충돌 없이 잘 연계됨
- [ ] MCP 서버를 올바르게 호출함
- [ ] 런타임 간 상태 보존 파일이 정상 갱신 및 유지됨

### 품질 지표
- [ ] 실행 속도가 적정 수준을 만족함
- [ ] 명령어 사용으로 인한 보안 취약점이 감지되지 않음
- [ ] 에러 메시지가 실제 복구에 도움이 되도록 구체적임
- [ ] 결과 텍스트가 가독성 높게 표시됨
- [ ] 동반 매뉴얼 및 문서 작성이 성실하게 완료됨

### 배포 준비
- [ ] 제3자를 통해 로컬 동작 검증을 거침
- [ ] 테스트 피드백 사항이 코드에 성실히 반영됨
- [ ] 동반 README 파일 갱신 완료
- [ ] 예시 코드 검증 완료

## 실패한 테스트 디버깅 (Debugging Failed Tests)

### 일반적인 문제 및 해결책

**문제: 명령어가 /help에 나타나지 않음**

```bash
# 파일 위치 확인
ls -la .claude/commands/my-command.md

# 권한 권장 설정 적용
chmod 644 .claude/commands/my-command.md

# 파일 내 구문 에러 검사
head -n 20 .claude/commands/my-command.md

# Claude Code 재시작 후 확인
claude --debug
```

**문제: 인자가 대체(substitution)되지 않음**

```bash
# 파일 내 인자 치환 기호 검증
grep '\$1' .claude/commands/my-command.md
grep '\$ARGUMENTS' .claude/commands/my-command.md

# 아주 간단한 예시로 치환 로직 단독 테스트
echo "Test: \$1 and \$2" > .claude/commands/test-args.md
```

**문제: Bash 명령어가 실행되지 않음**

```bash
# allowed-tools 선언 내역 검증
grep "allowed-tools" .claude/commands/my-command.md

# 명령어 실행 기호 검사
grep '!\`' .claude/commands/my-command.md

# 해당 셸 명령어가 터미널 내에서 작동 가능한지 단독 실행
date
echo "test"
```

**문제: 파일 참조가 작동하지 않음**

```bash
# @ 기호 사용법 검증
grep '@' .claude/commands/my-command.md

# 참조하려는 파일이 실제로 해당 경로에 존재하는지 단독 확인
ls -la /path/to/referenced/file

# 파일 권한 확인
chmod 644 /path/to/referenced/file
```

## 베스트 프랙티스

1. **일찍 테스트하고, 자주 테스트하세요**: 개발하는 과정에서 꾸준히 검증을 진행하세요.
2. **검증 자동화**: 반복적인 검사를 위해 검증 스크립트를 사용하세요.
3. **예외 상황 테스트**: 정상 작동 경로(happy path)만 테스트하지 마세요.
4. **피드백 받기**: 널리 배포하기 전에 다른 사람들이 테스트해 보게 하세요.
5. **테스트 문서화**: 회귀 테스트(regression testing)를 위해 테스트 시나리오를 기록해 둡니다.
6. **운영 환경 모니터링**: 릴리즈 후 발생할 수 있는 문제를 모니터링합니다.
7. **지속적인 보완**: 실제 사용 데이터를 바탕으로 개선해 나갑니다.
