---
description: hookify 플러그인 도움말 확인
allowed-tools: ["Read"]
---

# Hookify 플러그인 도움말

hookify 플러그인의 작동 원리 및 사용법을 설명합니다.

## 개요

hookify 플러그인을 사용하면 원치 않는 동작을 차단하는 커스텀 훅을 쉽게 생성할 수 있습니다. `hooks.json` 파일을 편집하는 대신, 감시할 패턴을 정의하는 간단한 마크다운 설정 파일을 작성합니다.

## 작동 원리

### 1. 훅 시스템

Hookify는 다음 이벤트가 발생할 때 실행되는 공통 훅을 구성합니다:
- **PreToolUse**: 도구(Bash, Edit, Write 등)가 실행되기 직전
- **PostToolUse**: 도구가 실행된 직후
- **Stop**: Claude가 세션 종료를 시도할 때
- **UserPromptSubmit**: 사용자가 프롬프트를 전송할 때

이 훅들은 `.claude/hookify.*.local.md` 설정 파일을 읽어 현재 작업이 규칙에 매칭되는지 검사합니다.

### 2. 설정 파일

사용자는 `.claude/hookify.{rule-name}.local.md` 파일에 규칙을 생성합니다:

```markdown
---
name: warn-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
---

⚠️ **Dangerous rm command detected!**

This command could delete important files. Please verify the path.
```

**핵심 필드:**
- `name`: 규칙의 유일한 식별자
- `enabled`: 활성화/비활성화를 결정하는 true/false
- `event`: bash, file, stop, prompt 또는 all
- `pattern`: 매칭할 정규식 패턴

메시지 본문은 규칙이 트리거되었을 때 Claude에게 보여지는 내용입니다.

### 3. 규칙 생성 방법

**방법 A: /hookify 명령어 사용**
```
/hookify Don't use console.log in production files
```

이 요청을 분석하여 적절한 규칙 파일을 자동 생성합니다.

**방법 B: 수동 생성**
위 서식에 맞춰 `.claude/hookify.my-rule.local.md` 파일을 생성합니다.

**방법 C: 대화 분석**
```
/hookify
```

인자 없이 호출하는 경우, hookify는 최근 대화 내용을 분석하여 예방하고 싶은 동작을 탐색합니다.

## 사용 가능한 명령어

- **`/hookify`** - 대화 분석 또는 명시적 지시사항을 바탕으로 훅 생성
- **`/hookify:help`** - 본 도움말 보기
- **`/hookify:list`** - 설정된 모든 훅 목록 표시
- **`/hookify:configure`** - 대화형 인터페이스를 통해 기존 훅을 활성화/비활성화

## 예제 사용 사례

**위험한 명령어 방지:**
```markdown
---
name: block-chmod-777
enabled: true
event: bash
pattern: chmod\s+777
---

Don't use chmod 777 - it's a security risk. Use specific permissions instead.
```

**디버깅 코드에 대한 경고:**
```markdown
---
name: warn-console-log
enabled: true
event: file
pattern: console\.log\(
---

Console.log detected. Remember to remove debug logging before committing.
```

**종료 전 테스트 실행 요구:**
```markdown
---
name: require-tests
enabled: true
event: stop
pattern: .*
---

Did you run tests before finishing? Make sure `npm test` or equivalent was executed.
```

## 패턴 문법

Python 정규식 문법을 사용합니다:
- `\s` - 공백 문자
- `\.` - 점(.) 문자 자체
- `|` - OR 조건
- `+` - 1회 이상 반복
- `*` - 0회 이상 반복
- `\d` - 숫자
- `[abc]` - 문자 클래스

**예시:**
- `rm\s+-rf` - "rm -rf" 매칭
- `console\.log\(` - "console.log(" 매칭
- `(eval|exec)\(` - "eval(" 또는 "exec(" 매칭
- `\.env$` - .env로 끝나는 파일명 매칭

## 중요 주의사항

**재시작 불필요**: Hookify 규칙(`.local.md` 파일)은 다음 도구 사용 시점부터 즉시 적용됩니다. Hookify 훅은 이미 로드되어 있으며 규칙을 동적으로 읽어옵니다.

**Block 또는 Warn**: 규칙은 작업을 차단(`block`)하거나 경고(`warn` - 메시지를 표시하되 허용)할 수 있습니다. 규칙 프론트매터에 `action: block` 또는 `action: warn`을 설정하십시오.

**규칙 파일**: 규칙은 `.claude/hookify.*.local.md` 파일 형태로 보관하십시오. 이 파일들은 Git에서 제외되어야 합니다 (필요시 .gitignore에 추가).

**규칙 비활성화**: 프론트매터에 `enabled: false`로 설정하거나 해당 파일을 삭제하십시오.

## 트러블슈팅

**훅이 트리거되지 않는 경우:**
- 규칙 파일이 `.claude/` 디렉터리에 존재하는지 확인하십시오.
- 프론트매터에 `enabled: true`로 설정되어 있는지 확인하십시오.
- 패턴이 유효한 정규식인지 확인하십시오.
- 패턴 테스트 실행: `python3 -c "import re; print(re.search('your_pattern', 'test_text'))"`
- 규칙은 별도 재시작 없이 즉시 적용됩니다.

**임포트 오류:**
- Python 3가 사용 가능한지 확인: `python3 --version`
- hookify 플러그인이 올바르게 설치되었는지 확인

**패턴 매칭이 되지 않는 경우:**
- 정규식 테스트를 별도로 실행해 보십시오.
- 이스케이프 관련 문제를 확인하십시오 (YAML 내에서 따옴표 없는 패턴 사용 권장).
- 단순한 패턴부터 시작해서 고도화하십시오.

## 시작하기

1. 첫 번째 규칙 생성:
   ```
   /hookify Warn me when I try to use rm -rf
   ```

2. 트리거 테스트:
   - Claude에게 `rm -rf /tmp/test` 실행을 요청합니다.
   - 경고 메시지가 표시되는지 확인합니다.

4. `.claude/hookify.warn-rm.local.md` 파일을 편집하여 규칙을 정교화합니다.

5. 원치 않는 동작을 발견할 때마다 추가적인 규칙을 생성합니다.

더 많은 예제는 `${CLAUDE_PLUGIN_ROOT}/examples/` 디렉터리를 참고하십시오.
