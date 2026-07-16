---
name: writing-hookify-rules
description: 사용자가 "hookify 규칙 생성", "훅 규칙 작성", "hookify 설정", "hookify 규칙 추가"를 요청하거나 hookify 규칙 구문 및 패턴에 대한 안내가 필요할 때 이 스킬을 사용해야 합니다.
version: 0.1.0
---

# Hookify 규칙 작성

## 개요

Hookify 규칙은 감시할 패턴과 해당 패턴이 일치할 때 보여줄 메시지를 정의하는 YAML 프론트매터가 포함된 마크다운 파일입니다. 규칙은 `.claude/hookify.{rule-name}.local.md` 파일에 저장됩니다.

## 규칙 파일 형식

### 기본 구조

```markdown
---
name: rule-identifier
enabled: true
event: bash|file|stop|prompt|all
pattern: regex-pattern-here
---

이 규칙이 트리거될 때 Claude에게 보여줄 메시지.
마크다운 서식, 경고, 제안 등을 포함할 수 있습니다.
```

### 프론트매터 필드

**name** (필수): 규칙의 고유 식별자
- kebab-case 사용: `warn-dangerous-rm`, `block-console-log`
- 설명적이고 작업 지향적으로 작성
- 동사로 시작: warn, prevent, block, require, check

**enabled** (필수): 활성화/비활성화를 위한 불리언(Boolean) 값
- `true`: 규칙 활성화 상태
- `false`: 규칙 비활성화 상태 (트리거되지 않음)
- 규칙을 삭제하지 않고도 전환할 수 있음

**event** (필수): 트리거할 훅 이벤트
- `bash`: Bash 도구 명령어
- `file`: Edit, Write, MultiEdit 도구
- `stop`: 에이전트가 중단하고자 할 때
- `prompt`: 사용자가 프롬프트를 제출할 때
- `all`: 모든 이벤트

**action** (선택): 규칙이 일치할 때 취할 조치
- `warn`: 메시지를 표시하지만 작업을 허용 (기본값)
- `block`: 작업 차단(PreToolUse) 또는 세션 중단(Stop 이벤트)
- 생략할 경우 기본값은 `warn`

**pattern** (단순 형식): 일치시킬 정규식 패턴
- 단순한 단일 조건 규칙에 사용됨
- 명령어(bash) 또는 new_text(file)와 매칭
- Python 정규식 구문

**예시:**
```yaml
event: bash
pattern: rm\s+-rf
```

### 고급 형식 (다중 조건)

여러 조건이 있는 복잡한 규칙의 경우:

```markdown
---
name: warn-env-file-edits
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

.env 파일에 API 키를 추가하고 있습니다. 이 파일이 .gitignore에 포함되어 있는지 확인하세요!
```

**조건 필드:**
- `field`: 확인할 필드
  - bash의 경우: `command`
  - file의 경우: `file_path`, `new_text`, `old_text`, `content`
- `operator`: 매칭 방법
  - `regex_match`: 정규식 패턴 매칭
  - `contains`: 부분 문자열 확인
  - `equals`: 정확히 일치
  - `not_contains`: 부분 문자열이 포함되어 있지 않아야 함
  - `starts_with`: 접두사 확인
  - `ends_with`: 접미사 확인
- `pattern`: 일치시킬 패턴 또는 문자열

**규칙이 트리거되려면 모든 조건이 일치해야 합니다.**

## 메시지 본문

프론트매터 이후의 마크다운 콘텐츠는 규칙이 트리거될 때 Claude에게 보여집니다.

**좋은 메시지 예시:**
- 감지된 내용을 설명합니다
- 그것이 왜 문제가 되는지 설명합니다
- 대안 또는 모범 사례를 제안합니다
- 명확성을 위해 서식(굵은 글씨, 목록 등)을 사용합니다

**예시:**
```markdown
⚠️ **Console.log 감지됨!**

프로덕션 코드에 console.log를 추가하고 있습니다.

**이것이 중요한 이유:**
- 디버그 로그는 프로덕션에 배포되어서는 안 됩니다
- Console.log는 민감한 데이터를 노출할 수 있습니다
- 브라우저 성능에 영향을 줍니다

**대안:**
- 적절한 로깅 라이브러리를 사용하세요
- 커밋하기 전에 제거하세요
- 조건부 디버그 빌드를 사용하세요
```

## 이벤트 유형 가이드

### bash 이벤트

Bash 명령어 패턴과 일치:

```markdown
---
event: bash
pattern: sudo\s+|rm\s+-rf|chmod\s+777
---

위험한 명령어가 감지되었습니다!
```

**공통 패턴:**
- 위험한 명령어: `rm\s+-rf`, `dd\s+if=`, `mkfs`
- 권한 상승: `sudo\s+`, `su\s+`
- 권한 설정 문제: `chmod\s+777`, `chown\s+root`

### file 이벤트

Edit/Write/MultiEdit 작업과 일치:

```markdown
---
event: file
pattern: console\.log\(|eval\(|innerHTML\s*=
---

잠재적으로 문제가 될 수 있는 코드 패턴이 감지되었습니다!
```

**다른 필드 매칭:**
```markdown
---
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.tsx?$
  - field: new_text
    operator: regex_match
    pattern: console\.log\(
---

TypeScript 파일에 console.log 존재!
```

**공통 패턴:**
- 디버그 코드: `console\.log\(`, `debugger`, `print\(`
- 보안 위험: `eval\(`, `innerHTML\s*=`, `dangerouslySetInnerHTML`
- 민감한 파일: `\.env$`, `credentials`, `\.pem$`
- 생성된 파일: `node_modules/`, `dist/`, `build/`

### stop 이벤트

에이전트가 중단하고자 할 때 일치 (완료 검사):

```markdown
---
event: stop
pattern: .*
---

중단하기 전에 다음을 확인하세요:
- [ ] 테스트 실행 여부
- [ ] 빌드 성공 여부
- [ ] 문서 업데이트 여부
```

**용도:**
- 필요한 단계에 대한 리마인더
- 완료 체크리스트
- 프로세스 강제 적용

### prompt 이벤트

사용자 프롬프트 콘텐츠 일치 (고급):

```markdown
---
event: prompt
conditions:
  - field: user_prompt
    operator: contains
    pattern: deploy to production
---

프로덕션 배포 체크리스트:
- [ ] 테스트 통과 여부?
- [ ] 팀의 리뷰를 거쳤는지?
- [ ] 모니터링이 준비되었는지?
```

## 패턴 작성 팁

### 정규식 기초

**리터럴 문자**: 대부분의 문자는 자기 자신과 일치합니다
- `rm`은 "rm"과 일치합니다
- `console.log`는 "console.log"와 일치합니다

**이스케이프가 필요한 특수 문자:**
- `.` (모든 문자) → `\.` (리터럴 점)
- `(` `)` → `\(` `\)` (리터럴 괄호)
- `[` `]` → `\[` `\]` (리터럴 대괄호)

**공통 메타문자:**
- `\s` - 공백 (띄어쓰기, 탭, 줄바꿈)
- `\d` - 숫자 (0-9)
- `\w` - 단어 문자 (a-z, A-Z, 0-9, _)
- `.` - 모든 문자
- `+` - 하나 이상
- `*` - 0개 이상
- `?` - 0개 또는 1개
- `|` - 또는(OR)

**예시:**
```
rm\s+-rf         매칭 대상: rm -rf, rm  -rf
console\.log\(   매칭 대상: console.log(
(eval|exec)\(    매칭 대상: eval( 또는 exec(
chmod\s+777      매칭 대상: chmod 777, chmod  777
API_KEY\s*=      매칭 대상: API_KEY=, API_KEY =
```

### 패턴 테스트

사용하기 전에 정규식 패턴을 테스트하세요:

```bash
python3 -c "import re; print(re.search(r'your_pattern', 'test text'))"
```

혹은 온라인 정규식 테스터(Python 버전의 regex101.com)를 사용하세요.

### 자주 발생하는 실수

**너무 광범위함:**
```yaml
pattern: log    # "log", "login", "dialog", "catalog"와 일치
```
더 나은 방법: `console\.log\(|logger\.`

**너무 구체적임:**
```yaml
pattern: rm -rf /tmp  # 정확히 이 경로에만 일치
```
더 나은 방법: `rm\s+-rf`

**이스케이프 문제:**
- 따옴표가 있는 YAML 문자열: `"pattern"`은 이중 백슬래시 `\\s`가 필요합니다
- 따옴표가 없는 YAML: `pattern: \s`는 그대로 작동합니다
- **권장 사항**: YAML에서는 따옴표를 사용하지 않는 패턴을 권장합니다

## 파일 구성

**위치**: 모든 규칙은 `.claude/` 디렉터리에 위치합니다
**이름**: `.claude/hookify.{설명적인-이름}.local.md`
**Gitignore**: `.gitignore`에 `.claude/*.local.md`를 추가합니다

**올바른 이름 예시:**
- `hookify.dangerous-rm.local.md`
- `hookify.console-log.local.md`
- `hookify.require-tests.local.md`
- `hookify.sensitive-files.local.md`

**잘못된 이름 예시:**
- `hookify.rule1.local.md` (설명적이지 않음)
- `hookify.md` (.local 누락)
- `danger.local.md` (hookify 접두사 누락)

## 워크플로우

### 규칙 생성하기

1. 원치 않는 동작을 식별합니다
2. 어떤 도구가 관련되어 있는지 확인합니다 (Bash, Edit 등)
3. 이벤트 유형을 선택합니다 (bash, file, stop 등)
4. 정규식 패턴을 작성합니다
5. 프로젝트 루트에 `.claude/hookify.{name}.local.md` 파일을 생성합니다
6. 즉시 테스트합니다 - 규칙은 다음 도구 사용 시 동적으로 읽힙니다

### 규칙 개선하기

1. `.local.md` 파일을 편집합니다
2. 패턴 또는 메시지를 조정합니다
3. 즉시 테스트합니다 - 변경 사항은 다음 도구 사용 시 반영됩니다

### 규칙 비활성화하기

**임시**: 프론트매터에서 `enabled: false`로 설정합니다
**영구**: `.local.md` 파일을 삭제합니다

## 예시

전체 예시는 `${CLAUDE_PLUGIN_ROOT}/examples/`를 확인하세요:
- `dangerous-rm.local.md` - 위험한 rm 명령어 차단
- `console-log-warning.local.md` - console.log에 대한 경고
- `sensitive-files-warning.local.md` - .env 파일 편집에 대한 경고

## 빠른 참조

**최소 작동 가능 규칙:**
```markdown
---
name: my-rule
enabled: true
event: bash
pattern: dangerous_command
---

여기에 경고 메시지 작성
```

**조건이 있는 규칙:**
```markdown
---
name: my-rule
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.ts$
  - field: new_text
    operator: contains
    pattern: any
---

경고 메시지
```

**이벤트 유형:**
- `bash` - Bash 명령어
- `file` - 파일 편집
- `stop` - 완료 검사
- `prompt` - 사용자 입력
- `all` - 모든 이벤트

**필드 옵션:**
- Bash: `command`
- File: `file_path`, `new_text`, `old_text`, `content`
- Prompt: `user_prompt`

**연산자:**
- `regex_match`, `contains`, `equals`, `not_contains`, `starts_with`, `ends_with`
