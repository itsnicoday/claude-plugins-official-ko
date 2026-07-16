---
description: 대화 분석 또는 명시적 지침을 통해 원치 않는 동작을 방지하는 훅(hook) 생성
argument-hint: 해결해야 할 선택적이고 구체적인 동작
allowed-tools: ["Read", "Write", "AskUserQuestion", "Task", "Grep", "TodoWrite", "Skill"]
---

# Hookify - 원치 않는 동작으로부터 훅(Hook) 생성

**가장 먼저: Skill 도구를 사용하여 hookify:writing-rules 스킬을 로드**하여 규칙 파일의 형식과 구문을 이해하세요.

대화를 분석하거나 사용자의 명시적 지침으로부터 문제가 되는 동작을 방지하는 훅 규칙을 생성합니다.

## 작업

원치 않는 동작을 방지하기 위한 hookify 규칙을 생성하도록 사용자를 돕습니다. 다음 단계를 따르세요:

### 1단계: 동작 정보 수집

**$ARGUMENTS가 제공되는 경우:**
- 사용자가 구체적인 지침을 제공했습니다: `$ARGUMENTS`
- 추가 컨텍스트를 위해 최근 대화(마지막 10~15개의 사용자 메시지)를 계속 분석합니다
- 동작이 발생하는 사례를 찾습니다

**$ARGUMENTS가 비어 있는 경우:**
- 문제가 되는 동작을 찾기 위해 conversation-analyzer 에이전트를 실행합니다
- 에이전트는 사용자 프롬프트에서 불만(frustration) 신호를 스캔합니다
- 에이전트는 구조화된 발견 사항을 반환합니다

**대화를 분석하려면:**
Task 도구를 사용하여 conversation-analyzer 에이전트를 실행합니다:
```
{
  "subagent_type": "general-purpose",
  "description": "Analyze conversation for unwanted behaviors",
  "prompt": "You are analyzing a Claude Code conversation to find behaviors the user wants to prevent.

Read user messages in the current conversation and identify:
1. Explicit requests to avoid something (\"don't do X\", \"stop doing Y\")
2. Corrections or reversions (user fixing Claude's actions)
3. Frustrated reactions (\"why did you do X?\", \"I didn't ask for that\")
4. Repeated issues (same problem multiple times)

For each issue found, extract:
- What tool was used (Bash, Edit, Write, etc.)
- Specific pattern or command
- Why it was problematic
- User's stated reason

Return findings as a structured list with:
- category: Type of issue
- tool: Which tool was involved
- pattern: Regex or literal pattern to match
- context: What happened
- severity: high/medium/low

Focus on the most recent issues (last 20-30 messages). Don't go back further unless explicitly asked."
}
```

### 2단계: 사용자에게 발견 사항 제시

(인수 또는 에이전트로부터) 동작을 수집한 후 AskUserQuestion을 사용하여 사용자에게 제시합니다:

**질문 1: 어떤 동작을 hookify 규칙으로 만들겠습니까?**
- Header: "규칙 생성"
- multiSelect: true
- Options: 감지된 각 동작을 나열합니다 (최대 4개)
  - Label: 간단한 설명 (예: "Block rm -rf")
  - Description: 문제가 되는 이유

**질문 2: 선택된 각 동작에 대해 취할 조치를 묻습니다:**
- "이 작업을 차단해야 합니까, 아니면 그냥 경고만 해야 합니까?"
- 옵션:
  - "Just warn" (action: warn - 메시지를 보여주지만 허용)
  - "Block operation" (action: block - 실행을 차단)

**질문 3: 예시 패턴을 묻습니다:**
- "어떤 패턴이 이 규칙을 트리거해야 합니까?"
- 감지된 패턴을 표시합니다
- 사용자가 정제하거나 추가할 수 있도록 허용합니다

### 3단계: 규칙 파일 생성

확인된 각 동작에 대해 `.claude/hookify.{rule-name}.local.md` 파일을 생성합니다:

**규칙 명명 규칙:**
- kebab-case를 사용합니다
- 구체적인 설명 사용: `block-dangerous-rm`, `warn-console-log`, `require-tests-before-stop`
- 동작 동사로 시작: block, warn, prevent, require

**파일 형식:**
```markdown
---
name: {rule-name}
enabled: true
event: {bash|file|stop|prompt|all}
pattern: {regex pattern}
action: {warn|block}
---

{규칙이 트리거될 때 Claude에게 보여줄 메시지}
```

**Action 값:**
- `warn`: 메시지를 표시하지만 작업을 허용 (기본값)
- `block`: 작업을 차단하거나 세션을 중단

**더 복잡한 규칙의 경우 (여러 조건):**
```markdown
---
name: {rule-name}
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.env$
  - field: new_text
    operator: contains
    pattern: API_KEY
---

{경고 메시지}
```

### 4단계: 파일 생성 및 확인

**중요**: 규칙 파일은 플러그인 디렉터리가 아닌 현재 작업 디렉터리의 `.claude/` 폴더에 생성되어야 합니다.

Claude Code가 시작된 현재 작업 디렉터리를 기준 경로로 사용합니다.

1. 현재 작업 디렉터리에 `.claude/` 디렉터리가 있는지 확인합니다.
   - 없는 경우, 먼저 `mkdir -p .claude` 명령어로 생성합니다.

2. Write 도구를 사용하여 각 `.claude/hookify.{name}.local.md` 파일을 만듭니다.
   - 현재 작업 디렉터리 기준의 상대 경로를 사용합니다: `.claude/hookify.{name}.local.md`
   - 이 경로는 플러그인이 아닌 프로젝트의 .claude 디렉터리로 결정되어야 합니다.

3. 생성된 항목을 사용자에게 보여줍니다:
   ```
   3개의 hookify 규칙이 생성되었습니다:
   - .claude/hookify.dangerous-rm.local.md
   - .claude/hookify.console-log.local.md
   - .claude/hookify.sensitive-files.local.md

   이 규칙들은 다음 상황에서 트리거됩니다:
   - dangerous-rm: "rm -rf"와 일치하는 Bash 명령어
   - console-log: console.log 구문을 추가하는 편집 작업
   - sensitive-files: .env 또는 자격 증명(credentials) 파일 편집
   ```

4. 목록을 조회하여 파일이 올바른 위치에 생성되었는지 확인합니다.

5. 사용자에게 알립니다: **"규칙은 즉시 활성화됩니다 - 재시작이 필요하지 않습니다!"**

   hookify 훅은 이미 로드되어 있으며 다음 도구 사용 시 새 규칙을 읽습니다.

## 이벤트 유형 참조

- **bash**: Bash 도구 명령어와 일치
- **file**: Edit, Write, MultiEdit 도구와 일치
- **stop**: 에이전트가 중단하고자 할 때 일치 (완료 검사에 사용)
- **prompt**: 사용자가 프롬프트를 제출할 때 일치
- **all**: 모든 이벤트와 일치

## 패턴 작성 팁

**Bash 패턴:**
- 위험한 명령어 매칭: `rm\s+-rf|chmod\s+777|dd\s+if=`
- 특정 도구 매칭: `npm\s+install\s+|pip\s+install`

**File 패턴:**
- 코드 패턴 매칭: `console\.log\(|eval\(|innerHTML\s*=`
- 파일 경로 매칭: `\.env$|\.git/|node_modules/`

**Stop 패턴:**
- 누락된 단계 검사: (대화 기록 또는 완료 기준 확인)

## 예시 워크플로우

**사용자 발화**: "/hookify Don't use rm -rf without asking me first"

**에이전트 응답**:
1. 분석: 사용자는 rm -rf 명령어를 방지하고자 합니다
2. 질문: "이 명령어를 차단할까요, 아니면 그냥 경고만 해드릴까요?"
3. 사용자 선택: "Just warn"
4. `.claude/hookify.dangerous-rm.local.md` 파일 생성:
   ```markdown
   ---
   name: warn-dangerous-rm
   enabled: true
   event: bash
   pattern: rm\s+-rf
   ---

   ⚠️ **위험한 rm 명령어가 감지되었습니다**

   rm -rf를 사용하기 전에 경고를 받도록 요청하셨습니다.
   경로가 올바른지 확인해 주세요.
   ```
5. 확인: "hookify 규칙이 생성되었습니다. 즉시 활성화되니 트리거해 보세요!"

## 중요 참고 사항

- **재시작 불필요**: 다음 도구 사용 시 규칙이 즉시 적용됩니다
- **파일 위치**: 플러그인의 .claude/가 아닌 프로젝트의 `.claude/` 디렉터리(현재 작업 디렉터리)에 파일을 생성합니다
- **정규식 구문**: Python 정규식 구문을 사용합니다 (원시 문자열, YAML에서 이스케이프할 필요 없음)
- **조치 유형**: 규칙은 작업을 `warn`(기본값) 또는 `block`할 수 있습니다
- **테스트**: 생성 후 즉시 규칙을 테스트하세요

## 문제 해결

**규칙 파일 생성이 실패하는 경우:**
1. pwd로 현재 작업 디렉터리를 확인합니다
2. `.claude/` 디렉터리가 존재하는지 확인합니다 (필요한 경우 mkdir로 생성)
3. 필요한 경우 절대 경로를 사용합니다: `{cwd}/.claude/hookify.{name}.local.md`
4. Glob 또는 ls로 파일이 생성되었는지 확인합니다

**생성 후 규칙이 트리거되지 않는 경우:**
1. 파일이 플러그인의 `.claude/`가 아닌 프로젝트의 `.claude/`에 있는지 확인합니다
2. Read 도구로 파일을 확인하여 패턴이 올바른지 확인합니다
3. 다음 명령어로 패턴을 테스트합니다: `python3 -c "import re; print(re.search(r'pattern', 'test text'))"`
4. 프론트매터에 `enabled: true`로 되어 있는지 확인합니다
5. 기억해 두세요: 규칙은 재시작 없이 즉시 적용됩니다

**차단이 너무 엄격해 보이는 경우:**
1. 규칙 파일에서 `action: block`을 `action: warn`으로 변경합니다
2. 또는 패턴을 더 구체적으로 조정합니다
3. 변경 사항은 다음 도구 사용 시 반영됩니다

TodoWrite를 사용하여 단계별 진행 상황을 추적하세요.
