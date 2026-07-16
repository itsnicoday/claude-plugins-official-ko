# 설정 파일 파싱 기법 (Settings File Parsing Techniques)

bash 스크립트에서 `.claude/plugin-name.local.md` 파일을 파싱하기 위한 전체 가이드.

## 파일 구조 (File Structure)

설정 파일은 YAML 프론트매터가 포함된 마크다운을 사용합니다:

```markdown
---
field1: value1
field2: "value with spaces"
numeric_field: 42
boolean_field: true
list_field: ["item1", "item2", "item3"]
---

# Markdown Content

This body content can be extracted separately.
It's useful for prompts, documentation, or additional context.
```

## 프론트매터 파싱 (Parsing Frontmatter)

### 프론트매터 블록 추출 (Extract Frontmatter Block)

```bash
#!/bin/bash
FILE=".claude/my-plugin.local.md"

# Extract everything between --- markers (excluding the markers themselves)
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")
```

**작동 방식:**
- `sed -n` - 자동 출력 생략
- `/^---$/,/^---$/` - 첫 번째 `---`에서 두 번째 `---`까지의 범위 지정
- `{ /^---$/d; p; }` - `---` 행은 삭제하고, 그 외의 다른 모든 행 출력

### 개별 필드 추출 (Extract Individual Fields)

**문자열 필드:**
```bash
# Simple value
VALUE=$(echo "$FRONTMATTER" | grep '^field_name:' | sed 's/field_name: *//')

# Quoted value (removes surrounding quotes)
VALUE=$(echo "$FRONTMATTER" | grep '^field_name:' | sed 's/field_name: *//' | sed 's/^"\(.*\)"$/\1/')
```

**불리언 필드:**
```bash
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')

# Use in condition
if [[ "$ENABLED" == "true" ]]; then
  # Enabled
fi
```

**숫자 필드:**
```bash
MAX=$(echo "$FRONTMATTER" | grep '^max_value:' | sed 's/max_value: *//')

# Validate it's a number
if [[ "$MAX" =~ ^[0-9]+$ ]]; then
  # Use in numeric comparison
  if [[ $MAX -gt 100 ]]; then
    # Too large
  fi
fi
```

**리스트 필드 (단순):**
```bash
# YAML: list: ["item1", "item2", "item3"]
LIST=$(echo "$FRONTMATTER" | grep '^list:' | sed 's/list: *//')
# Result: ["item1", "item2", "item3"]

# For simple checks:
if [[ "$LIST" == *"item1"* ]]; then
  # List contains item1
fi
```

**리스트 필드 (jq를 사용한 올바른 파싱):**
```bash
# For proper list handling, use yq or convert to JSON
# This requires yq to be installed (brew install yq)

# Extract list as JSON array
LIST=$(echo "$FRONTMATTER" | yq -o json '.list' 2>/dev/null)

# Iterate over items
echo "$LIST" | jq -r '.[]' | while read -r item; do
  echo "Processing: $item"
done
```

## 마크다운 본문 파싱 (Parsing Markdown Body)

### 본문 내용 추출 (Extract Body Content)

```bash
#!/bin/bash
FILE=".claude/my-plugin.local.md"

# Extract everything after the closing ---
# Counts --- markers: first is opening, second is closing, everything after is body
BODY=$(awk '/^---$/{i++; next} i>=2' "$FILE")
```

**작동 방식:**
- `/^---$/` - `---` 행 매칭
- `{i++; next}` - 카운터를 증가시키고 `---` 행 건너뜀
- `i>=2` - 두 번째 `---` 이후의 모든 행 출력

**예외 상황 처리:** 마크다운 본문에 `---`가 포함되어 있어도 여전히 올바르게 작동합니다. 파일의 시작 부분에서 처음 두 개의 `---`만 계산하기 때문입니다.

### 본문을 프롬프트로 사용 (Use Body as Prompt)

```bash
# Extract body
PROMPT=$(awk '/^---$/{i++; next} i>=2' "$RALPH_STATE_FILE")

# Feed back to Claude
echo '{"decision": "block", "reason": "'"$PROMPT"'"}' | jq .
```

**중요:** 사용자 콘텐츠가 포함된 보다 안전한 JSON 구성을 위해 `jq -n --arg`를 사용하십시오:

```bash
PROMPT=$(awk '/^---$/{i++; next} i>=2' "$FILE")

# Safe JSON construction
jq -n --arg prompt "$PROMPT" '{
  "decision": "block",
  "reason": $prompt
}'
```

## 공통 파싱 패턴 (Common Parsing Patterns)

### 패턴: 기본값이 있는 필드 (Pattern: Field with Default)

```bash
VALUE=$(echo "$FRONTMATTER" | grep '^field:' | sed 's/field: *//' | sed 's/^"\(.*\)"$/\1/')

# Use default if empty
if [[ -z "$VALUE" ]]; then
  VALUE="default_value"
fi
```

### 패턴: 선택적 필드 (Pattern: Optional Field)

```bash
OPTIONAL=$(echo "$FRONTMATTER" | grep '^optional_field:' | sed 's/optional_field: *//' | sed 's/^"\(.*\)"$/\1/')

# Only use if present
if [[ -n "$OPTIONAL" ]] && [[ "$OPTIONAL" != "null" ]]; then
  # Field is set, use it
  echo "Optional field: $OPTIONAL"
fi
```

### 패턴: 여러 필드를 한 번에 처리 (Pattern: Multiple Fields at Once)

```bash
# Parse all fields in one pass
while IFS=': ' read -r key value; do
  # Remove quotes if present
  value=$(echo "$value" | sed 's/^"\(.*\)"$/\1/')

  case "$key" in
    enabled)
      ENABLED="$value"
      ;;
    mode)
      MODE="$value"
      ;;
    max_size)
      MAX_SIZE="$value"
      ;;
  esac
done <<< "$FRONTMATTER"
```

## 설정 파일 업데이트 (Updating Settings Files)

### 원자적 업데이트 (Atomic Updates)

파일 손상을 방지하기 위해 항상 임시 파일 생성 및 원자적 이동을 사용하십시오:

```bash
#!/bin/bash
FILE=".claude/my-plugin.local.md"
NEW_VALUE="updated_value"

# Create temp file
TEMP_FILE="${FILE}.tmp.$$"

# Update field using sed
sed "s/^field_name: .*/field_name: $NEW_VALUE/" "$FILE" > "$TEMP_FILE"

# Atomic replace
mv "$TEMP_FILE" "$FILE"
```

### 단일 필드 업데이트 (Update Single Field)

```bash
# Increment iteration counter
CURRENT=$(echo "$FRONTMATTER" | grep '^iteration:' | sed 's/iteration: *//')
NEXT=$((CURRENT + 1))

# Update file
TEMP_FILE="${FILE}.tmp.$$"
sed "s/^iteration: .*/iteration: $NEXT/" "$FILE" > "$TEMP_FILE"
mv "$TEMP_FILE" "$FILE"
```

### 여러 필드 업데이트 (Update Multiple Fields)

```bash
# Update several fields at once
TEMP_FILE="${FILE}.tmp.$$"

sed -e "s/^iteration: .*/iteration: $NEXT_ITERATION/" \
    -e "s/^pr_number: .*/pr_number: $PR_NUMBER/" \
    -e "s/^status: .*/status: $NEW_STATUS/" \
    "$FILE" > "$TEMP_FILE"

mv "$TEMP_FILE" "$FILE"
```

## 검증 기법 (Validation Techniques)

### 파일 존재 여부 및 읽기 가능 여부 검증 (Validate File Exists and Is Readable)

```bash
FILE=".claude/my-plugin.local.md"

if [[ ! -f "$FILE" ]]; then
  echo "Settings file not found" >&2
  exit 1
fi

if [[ ! -r "$FILE" ]]; then
  echo "Settings file not readable" >&2
  exit 1
fi
```

### 프론트매터 구조 검증 (Validate Frontmatter Structure)

```bash
# Count --- markers (should be exactly 2 at start)
MARKER_COUNT=$(grep -c '^---$' "$FILE" 2>/dev/null || echo "0")

if [[ $MARKER_COUNT -lt 2 ]]; then
  echo "Invalid settings file: missing frontmatter markers" >&2
  exit 1
fi
```

### 필드 값 검증 (Validate Field Values)

```bash
MODE=$(echo "$FRONTMATTER" | grep '^mode:' | sed 's/mode: *//')

case "$MODE" in
  strict|standard|lenient)
    # Valid mode
    ;;
  *)
    echo "Invalid mode: $MODE (must be strict, standard, or lenient)" >&2
    exit 1
    ;;
esac
```

### 숫자 범위 검증 (Validate Numeric Ranges)

```bash
MAX_SIZE=$(echo "$FRONTMATTER" | grep '^max_size:' | sed 's/max_size: *//')

if ! [[ "$MAX_SIZE" =~ ^[0-9]+$ ]]; then
  echo "max_size must be a number" >&2
  exit 1
fi

if [[ $MAX_SIZE -lt 1 ]] || [[ $MAX_SIZE -gt 10000000 ]]; then
  echo "max_size out of range (1-10000000)" >&2
  exit 1
fi
```

## 예외 상황 및 주의 사항 (Edge Cases and Gotchas)

### 값 내부의 따옴표 (Quotes in Values)

YAML은 따옴표가 있는 문자열과 없는 문자열을 모두 허용합니다:

```yaml
# These are equivalent:
field1: value
field2: "value"
field3: 'value'
```

**둘 다 처리하는 방법:**
```bash
# Remove surrounding quotes if present
VALUE=$(echo "$FRONTMATTER" | grep '^field:' | sed 's/field: *//' | sed 's/^"\(.*\)"$/\1/' | sed "s/^'\\(.*\\)'$/\\1/")
```

### 마크다운 본문 내부의 --- (--- in Markdown Body)

마크다운 본문에 `---`가 포함되어 있어도 처음 두 개만 일치시키기 때문에 파싱이 정상적으로 이루어집니다:

```markdown
---
field: value
---

# Body

Here's a separator:
---

More content after the separator.
```

`awk '/^---$/{i++; next} i>=2'` 패턴은 이를 올바르게 처리합니다.

### 빈 값 (Empty Values)

누락되었거나 비어 있는 필드를 처리합니다:

```yaml
field1:
field2: ""
field3: null
```

**파싱 방식:**
```bash
VALUE=$(echo "$FRONTMATTER" | grep '^field1:' | sed 's/field1: *//')
# VALUE will be empty string

# Check for empty/null
if [[ -z "$VALUE" ]] || [[ "$VALUE" == "null" ]]; then
  VALUE="default"
fi
```

### 특수 문자 (Special Characters)

특수 문자가 있는 값은 신중하게 처리해야 합니다:

```yaml
message: "Error: Something went wrong!"
path: "/path/with spaces/file.txt"
regex: "^[a-zA-Z0-9_]+$"
```

**안전한 파싱 방식:**
```bash
# Always quote variables when using
MESSAGE=$(echo "$FRONTMATTER" | grep '^message:' | sed 's/message: *//' | sed 's/^"\(.*\)"$/\1/')

echo "Message: $MESSAGE"  # Quoted!
```

## 성능 최적화 (Performance Optimization)

### 파싱된 값 캐싱 (Cache Parsed Values)

설정을 여러 번 읽어야 하는 경우:

```bash
# Parse once
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")

# Extract multiple fields from cached frontmatter
FIELD1=$(echo "$FRONTMATTER" | grep '^field1:' | sed 's/field1: *//')
FIELD2=$(echo "$FRONTMATTER" | grep '^field2:' | sed 's/field2: *//')
FIELD3=$(echo "$FRONTMATTER" | grep '^field3:' | sed 's/field3: *//')
```

**권장하지 않음:** 각 필드를 읽을 때마다 파일을 다시 파싱하는 방식.

### 지연 로딩 (Lazy Loading)

필요할 때만 설정을 파싱합니다:

```bash
#!/bin/bash
input=$(cat)

# Quick checks first (no file I/O)
tool_name=$(echo "$input" | jq -r '.tool_name')
if [[ "$tool_name" != "Write" ]]; then
  exit 0  # Not a write operation, skip
fi

# Only now check settings file
if [[ -f ".claude/my-plugin.local.md" ]]; then
  # Parse settings
  # ...
fi
```

## 디버깅 (Debugging)

### 파싱된 값 출력 (Print Parsed Values)

```bash
#!/bin/bash
set -x  # Enable debug tracing

FILE=".claude/my-plugin.local.md"

if [[ -f "$FILE" ]]; then
  echo "Settings file found" >&2

  FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")
  echo "Frontmatter:" >&2
  echo "$FRONTMATTER" >&2

  ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
  echo "Enabled: $ENABLED" >&2
fi
```

### 파싱 결과 검증 (Validate Parsing)

```bash
# Show what was parsed
echo "Parsed values:" >&2
echo "  enabled: $ENABLED" >&2
echo "  mode: $MODE" >&2
echo "  max_size: $MAX_SIZE" >&2

# Verify expected values
if [[ "$ENABLED" != "true" ]] && [[ "$ENABLED" != "false" ]]; then
  echo "⚠️  Unexpected enabled value: $ENABLED" >&2
fi
```

## 대안: yq 사용 (Alternative: Using yq)

복잡한 YAML의 경우 `yq` 사용을 고려해 보십시오:

```bash
# Install: brew install yq

# Parse YAML properly
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$FILE")

# Extract fields with yq
ENABLED=$(echo "$FRONTMATTER" | yq '.enabled')
MODE=$(echo "$FRONTMATTER" | yq '.mode')
LIST=$(echo "$FRONTMATTER" | yq -o json '.list_field')

# Iterate list properly
echo "$LIST" | jq -r '.[]' | while read -r item; do
  echo "Item: $item"
done
```

**장점:**
- 올바른 YAML 파싱
- 복잡한 구조 처리
- 우수한 리스트/객체 지원

**단점:**
- yq 설치 필요
- 추가 종속성 발생
- 모든 시스템에서 사용하지 못할 수 있음

**권장 사항:** 단순한 필드는 sed/grep을 사용하고, 복잡한 구조에는 yq를 사용하십시오.

## 완전한 예시 (Complete Example)

```bash
#!/bin/bash
set -euo pipefail

# Configuration
SETTINGS_FILE=".claude/my-plugin.local.md"

# Quick exit if not configured
if [[ ! -f "$SETTINGS_FILE" ]]; then
  # Use defaults
  ENABLED=true
  MODE=standard
  MAX_SIZE=1000000
else
  # Parse frontmatter
  FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$SETTINGS_FILE")

  # Extract fields with defaults
  ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
  ENABLED=${ENABLED:-true}

  MODE=$(echo "$FRONTMATTER" | grep '^mode:' | sed 's/mode: *//' | sed 's/^"\(.*\)"$/\1/')
  MODE=${MODE:-standard}

  MAX_SIZE=$(echo "$FRONTMATTER" | grep '^max_size:' | sed 's/max_size: *//')
  MAX_SIZE=${MAX_SIZE:-1000000}

  # Validate values
  if [[ "$ENABLED" != "true" ]] && [[ "$ENABLED" != "false" ]]; then
    echo "⚠️  Invalid enabled value, using default" >&2
    ENABLED=true
  fi

  if ! [[ "$MAX_SIZE" =~ ^[0-9]+$ ]]; then
    echo "⚠️  Invalid max_size, using default" >&2
    MAX_SIZE=1000000
  fi
fi

# Quick exit if disabled
if [[ "$ENABLED" != "true" ]]; then
  exit 0
fi

# Use configuration
echo "Configuration loaded: mode=$MODE, max_size=$MAX_SIZE" >&2

# Apply logic based on settings
case "$MODE" in
  strict)
    # Strict validation
    ;;
  standard)
    # Standard validation
    ;;
  lenient)
    # Lenient validation
    ;;
esac
```

이 코드는 기본값 적용, 유효성 검사 및 에러 복구 기능을 포함하여 견고한 설정 처리를 지원합니다.
