---
description: 대화형으로 hookify 규칙을 활성화하거나 비활성화합니다
allowed-tools: ["Glob", "Read", "Edit", "AskUserQuestion", "Skill"]
---

# Hookify 규칙 설정

규칙 형식을 이해하기 위해 **먼저 hookify:writing-rules 스킬을 로드**하세요.

대화형 인터페이스를 사용하여 기존 hookify 규칙을 활성화하거나 비활성화합니다.

## 단계

1. Glob 도구를 사용하여 모든 hookify 규칙 파일을 찾습니다:
```
pattern: ".claude/hookify.*.local.md"
```

규칙을 찾을 수 없는 경우 사용자에게 알립니다:
```
No hookify rules configured yet. Use `/hookify` to create your first rule.
```

### 2. 현재 상태 읽기

각 규칙 파일에 대해:
- 파일을 읽습니다
- 프론트매터에서 `name` 및 `enabled` 필드를 추출합니다
- 현재 상태를 포함한 규칙 목록을 빌드합니다

### 3. 전환할 규칙을 사용자에게 묻기

AskUserQuestion을 사용하여 사용자가 규칙을 선택할 수 있도록 합니다:

```json
{
  "questions": [
    {
      "question": "어떤 규칙을 활성화하거나 비활성화하시겠습니까?",
      "header": "설정",
      "multiSelect": true,
      "options": [
        {
          "label": "warn-dangerous-rm (현재 활성화됨)",
          "description": "rm -rf 명령어에 대해 경고합니다"
        },
        {
          "label": "warn-console-log (현재 비활성화됨)",
          "description": "코드 내 console.log에 대해 경고합니다"
        },
        {
          "label": "require-tests (현재 활성화됨)",
          "description": "중단하기 전에 테스트를 요구합니다"
        }
      ]
    }
  ]
}
```

**옵션 형식:**
- Label: `{rule-name} (현재 {enabled|disabled})`
- Description: 규칙의 메시지 또는 패턴의 간단한 설명

### 4. 사용자 선택 파싱

선택된 각 규칙에 대해:
- 라벨에서 현재 상태를 확인합니다 (enabled/disabled)
- 상태 전환: enabled → disabled, disabled → enabled

### 5. 규칙 파일 업데이트

전환할 각 규칙에 대해:
- Read 도구를 사용하여 현재 콘텐츠를 읽습니다
- Edit 도구를 사용하여 `enabled: true`를 `enabled: false`로 변경합니다 (또는 그 반대)
- 따옴표가 있는 경우와 없는 경우를 모두 처리합니다

**활성화를 위한 편집 패턴:**
```
old_string: "enabled: false"
new_string: "enabled: true"
```

**비활성화를 위한 편집 패턴:**
```
old_string: "enabled: true"
new_string: "enabled: false"
```

### 6. 변경 사항 확인

사용자에게 무엇이 변경되었는지 보여줍니다:

```
## Hookify 규칙 업데이트 완료

**활성화됨:**
- warn-console-log

**비활성화됨:**
- warn-dangerous-rm

**변경되지 않음:**
- require-tests

변경 사항은 즉시 적용되며, 재시작이 필요하지 않습니다
```

## 중요 참고 사항

- 변경 사항은 다음 도구 사용 시 즉시 적용됩니다
- .claude/hookify.*.local.md 파일을 수동으로 편집할 수도 있습니다
- 규칙을 영구적으로 제거하려면 .local.md 파일을 삭제하세요
- 설정된 모든 규칙을 보려면 `/hookify:list`를 사용하세요

## 예외 상황

**설정할 규칙이 없는 경우:**
- 먼저 규칙을 생성하기 위해 `/hookify`를 사용하라는 메시지를 표시합니다

**사용자가 규칙을 선택하지 않은 경우:**
- 변경 사항이 없음을 알립니다

**파일 읽기/쓰기 오류:**
- 사용자에게 구체적인 오류를 알립니다
- 대체 방법으로 수동 편집을 제안합니다
