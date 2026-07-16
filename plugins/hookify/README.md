# Hookify 플러그인

대화 패턴 분석 또는 명시적인 지시사항을 통해 원치 않는 동작을 차단하는 커스텀 훅(hooks)을 쉽게 생성하십시오.

## 개요

hookify 플러그인을 사용하면 복잡한 `hooks.json` 파일을 직접 편집하지 않고도 훅을 간단히 만들 수 있습니다. 대신 감시할 패턴과 해당 패턴이 매칭되었을 때 표시할 메시지를 정의하는 경량 마크다운 설정 파일을 작성합니다.

**주요 특징:**
- 🎯 대화 내용을 분석하여 원치 않는 동작을 자동으로 탐색
- 📝 YAML 프론트매터(frontmatter)를 사용한 단순한 마크다운 설정 파일 구조
- 🔍 강력한 규칙 설정을 위한 정규식(regex) 패턴 매칭
- 🚀 코딩이 불필요하며 단지 동작을 설명하면 됨
- 🔄 재시작 없이 간편한 활성화/비활성화 처리

## 빠른 시작

### 1. 첫 번째 규칙 생성

```bash
/hookify Warn me when I use rm -rf commands
```

이는 귀하의 요청을 분석하여 `.claude/hookify.warn-rm.local.md` 파일을 생성합니다.

### 2. 즉각적인 테스트

**재시작이 필요하지 않습니다!** 설정된 규칙은 다음 도구 사용 시점부터 바로 적용됩니다.

규칙이 트리거되도록 Claude에게 특정 명령 실행을 요청해 보세요:
```
Run rm -rf /tmp/test
```

경고 메시지가 즉시 나타나는 것을 확인할 수 있습니다!

## 사용법

### 메인 명령어: /hookify

**인자를 포함하여 호출하는 경우:**
```
/hookify Don't use console.log in TypeScript files
```
사용자의 명시적인 지시사항을 바탕으로 규칙을 생성합니다.

**인자 없이 호출하는 경우:**
```
/hookify
```
최근 대화 기록을 분석하여 사용자가 수정했거나 시행착오를 겪었던 동작들을 탐색합니다.

### 헬퍼 명령어

**모든 규칙 목록 표시:**
```
/hookify:list
```

**대화형으로 규칙 구성:**
```
/hookify:configure
```
대화형 인터페이스를 통해 기존 규칙을 활성화/비활성화합니다.

**도움말 보기:**
```
/hookify:help
```

## 규칙 설정 서식

### 단순 규칙 (단일 패턴)

`.claude/hookify.dangerous-rm.local.md` 파일 예시:
```markdown
---
name: block-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

⚠️ **Dangerous rm command detected!**

This command could delete important files. Please:
- Verify the path is correct
- Consider using a safer approach
- Make sure you have backups
```

**Action 필드:**
- `warn`: 경고를 표시하되 작업을 허용함 (기본값)
- `block`: 작업 실행을 차단(PreToolUse)하거나 세션을 중단함 (Stop 이벤트)

### 고급 규칙 (다중 조건)

`.claude/hookify.sensitive-files.local.md` 파일 예시:
```markdown
---
name: warn-sensitive-files
enabled: true
event: file
action: warn
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.env$|credentials|secrets
  - field: new_text
    operator: contains
    pattern: KEY
---

🔐 **Sensitive file edit detected!**

Ensure credentials are not hardcoded and file is in .gitignore.
```

**규칙이 트리거되려면 모든 조건이 일치해야 합니다.**

## 이벤트 유형

- **`bash`**: Bash 도구 명령 실행 시 트리거
- **`file`**: Edit, Write, MultiEdit 도구 실행 시 트리거
- **`stop`**: Claude가 종료를 시도할 때 트리거 (완료 여부 검사 등)
- **`prompt`**: 사용자가 프롬프트를 전송할 때 트리거
- **`all`**: 모든 이벤트 발생 시 트리거

## 패턴 문법

Python 정규식 문법을 사용합니다:

| 패턴 | 매칭 대상 | 예시 |
|---------|---------|---------|
| `rm\s+-rf` | rm -rf | rm -rf /tmp |
| `console\.log\(` | console.log( | console.log("test") |
| `(eval\|exec)\(` | eval( 또는 exec( | eval("code") |
| `\.env$` | .env로 끝나는 파일 | .env, .env.local |
| `chmod\s+777` | chmod 777 | chmod 777 file.txt |

**팁:**
- 공백 문자 매칭을 위해 `\s`를 사용하십시오.
- 특수 문자는 이스케이프 처리하십시오: 점(.) 문자 자체를 의미하려면 `\.`
- OR 조건에는 `|`를 사용하십시오: `(foo|bar)`
- 임의의 문자 매칭에는 `.*`를 사용하십시오.
- 위험한 작업에는 `action: block`을 지정하십시오.
- 안내성 경고에는 `action: warn`을 지정하십시오. (또는 생략 가능)

## 예제

### 예제 1: 위험한 명령어 차단

```markdown
---
name: block-destructive-ops
enabled: true
event: bash
pattern: rm\s+-rf|dd\s+if=|mkfs|format
action: block
---

🛑 **Destructive operation detected!**

This command can cause data loss. Operation blocked for safety.
Please verify the exact path and use a safer approach.
```

**이 규칙은 작업을 차단합니다** - Claude가 해당 명령을 실행하는 것을 금지합니다.

### 예제 2: 디버그 코드에 대한 경고

```markdown
---
name: warn-debug-code
enabled: true
event: file
pattern: console\.log\(|debugger;|print\(
action: warn
---

🐛 **Debug code detected**

Remember to remove debugging statements before committing.
```

**이 규칙은 경고하되 작업을 허용합니다** - Claude가 메시지를 확인한 후 계속 작업을 진행할 수 있습니다.

### 예제 3: 종료 전 테스트 실행 요구

```markdown
---
name: require-tests-run
enabled: false
event: stop
action: block
conditions:
  - field: transcript
    operator: not_contains
    pattern: npm test|pytest|cargo test
---

**Tests not detected in transcript!**

Before stopping, please run tests to verify your changes work correctly.
```

세션 기록에 테스트 명령이 실행되지 않은 경우 **Claude가 종료되는 것을 차단합니다.** 엄격한 규칙 준수가 필요할 때만 활성화하십시오.

## 고급 사용법

### 다중 조건 설정

```markdown
---
name: api-key-in-typescript
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.tsx?$
  - field: new_text
    operator: regex_match
    pattern: (API_KEY|SECRET|TOKEN)\s*=\s*["']
---

🔐 **Hardcoded credential in TypeScript!**

Use environment variables instead of hardcoded values.
```

### 연산자 레퍼런스

- `regex_match`: 패턴이 매칭되어야 함 (가장 흔함)
- `contains`: 문자열이 패턴을 포함해야 함
- `equals`: 문자열이 정확히 일치해야 함
- `not_contains`: 문자열이 패턴을 포함하지 않아야 함
- `starts_with`: 문자열이 패턴으로 시작해야 함
- `ends_with`: 문자열이 패턴으로 끝나야 함

### 필드 레퍼런스

**bash 이벤트의 경우:**
- `command`: Bash 명령어 문자열

**file 이벤트의 경우:**
- `file_path`: 수정되는 파일 경로
- `new_text`: 새로 추가되는 내용 (Edit, Write)
- `old_text`: 대체되어 삭제되는 기존 내용 (Edit 전용)
- `content`: 파일 내용 (Write 전용)

**prompt 이벤트의 경우:**
- `user_prompt`: 사용자가 전송한 프롬프트 텍스트

**stop 이벤트의 경우:**
- 세션 상태에 대한 일반적인 매칭을 사용합니다.

## 관리 방법

### 규칙 활성화/비활성화

**일시적 비활성화:**
`.local.md` 파일을 편집하여 `enabled: false`로 설정합니다.

**재활성화:**
`enabled: true`로 설정합니다.

**또는 대화형 도구 사용:**
```
/hookify:configure
```

### 규칙 삭제

간단히 `.local.md` 파일을 삭제하십시오:
```bash
rm .claude/hookify.my-rule.local.md
```

### 모든 규칙 확인

```
/hookify:list
```

## 설치

이 플러그인은 Claude Code 마켓플레이스의 일부입니다. 마켓플레이스가 설치되면 자동으로 탐색됩니다.

**수동 테스트:**
```bash
cc --plugin-dir /path/to/hookify
```

## 요구사항

- Python 3.7+
- 외부 의존성 없음 (표준 라이브러리만 사용)

## 트러블슈팅

**규칙이 트리거되지 않는 경우:**
1. 규칙 파일이 `.claude/` 디렉터리에 존재하는지 확인하십시오. (플러그인 디렉터리가 아닌 프로젝트 루트 기준)
2. 프론트매터에 `enabled: true`로 설정되어 있는지 확인하십시오.
3. 정규식 패턴을 별도로 테스트하십시오.
4. 규칙은 별도 재시작 없이 즉시 적용됩니다.
5. `/hookify:list`를 실행하여 규칙이 로드되었는지 확인하십시오.

**임포트 오류:**
- Python 3가 사용 가능한지 확인: `python3 --version`
- hookify 플러그인이 설치되었는지 확인

**패턴 매칭이 되지 않는 경우:**
- 정규식 테스트 실행: `python3 -c "import re; print(re.search(r'pattern', 'text'))"`
- 이스케이프 문제를 피하기 위해 YAML 내에서 따옴표 없는 패턴을 사용하십시오.
- 단순하게 시작한 뒤 복잡도를 높이십시오.

**훅 동작이 느린 경우:**
- 패턴을 간단히 유지하십시오. (복잡한 정규식 회피)
- "all" 대신 구체적인 이벤트 유형(bash, file)을 지정하십시오.
- 활성화되는 규칙 개수를 제한하십시오.

## 기여하기

유용한 규칙 패턴을 발견하셨나요? 풀 리퀘스트(PR)를 통해 예제 파일을 공유해 주세요!

## 향후 개선 사항

- 심각도 수준 세분화 (error/warning/info 구분)
- 규칙 템플릿 라이브러리 제공
- 대화형 패턴 빌더 구성
- 훅 테스트 유틸리티 지원
- JSON 서식 지원 (마크다운에 추가 지원)

## 라이선스

MIT License
