---
description: "사용자 설정으로 플러그인 설정 파일 생성"
allowed-tools: ["Write", "AskUserQuestion"]
---

# 플러그인 설정 생성 (Create Plugin Settings)

이 명령어는 사용자가 `.claude/my-plugin.local.md` 설정 파일을 생성할 수 있도록 도와줍니다.

## 단계 (Steps)

### 1단계: 사용자에게 기본 설정 요청 (Step 1: Ask User for Preferences)

AskUserQuestion을 사용하여 구성을 수집합니다:

```json
{
  "questions": [
    {
      "question": "Enable plugin for this project?",
      "header": "Enable Plugin",
      "multiSelect": false,
      "options": [
        {
          "label": "Yes",
          "description": "Plugin will be active"
        },
        {
          "label": "No",
          "description": "Plugin will be disabled"
        }
      ]
    },
    {
      "question": "Validation mode?",
      "header": "Mode",
      "multiSelect": false,
      "options": [
        {
          "label": "Strict",
          "description": "Maximum validation and security checks"
        },
        {
          "label": "Standard",
          "description": "Balanced validation (recommended)"
        },
        {
          "label": "Lenient",
          "description": "Minimal validation only"
        }
      ]
    }
  ]
}
```

### 2단계: 답변 파싱 (Step 2: Parse Answers)

AskUserQuestion 결과에서 답변을 추출합니다:

- answers["0"]: enabled (Yes/No)
- answers["1"]: mode (Strict/Standard/Lenient)

### 3단계: 설정 파일 생성 (Step 3: Create Settings File)

Write 도구를 사용하여 `.claude/my-plugin.local.md`를 생성합니다:

```markdown
---
enabled: <true if Yes, false if No>
validation_mode: <strict, standard, or lenient>
max_file_size: 1000000
notify_on_errors: true
---

# Plugin Configuration

Your plugin is configured with <mode> validation mode.

To modify settings, edit this file and restart Claude Code.
```

### 4단계: 사용자에게 정보 제공 (Step 4: Inform User)

사용자에게 다음 사항을 안내합니다:
- `.claude/my-plugin.local.md`에 설정 파일이 생성됨
- 현재 구성 요약
- 필요한 경우 수동으로 편집하는 방법
- 주의: 변경 사항을 적용하려면 Claude Code를 재시작해야 함
- 설정 파일은 gitignore 대상임 (커밋되지 않음)

## 구현 참고 사항 (Implementation Notes)

작성하기 전에 항상 사용자 입력을 검증하십시오:
- 모드가 유효한지 확인
- 숫자 필드가 숫자인지 검증
- 경로에 디렉토리 탐색(traversal) 시도가 없는지 확인
- 자유 텍스트 필드 정제(sanitize)
