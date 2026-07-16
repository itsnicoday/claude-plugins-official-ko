---
description: 설정된 모든 hookify 규칙 목록 표시
allowed-tools: ["Glob", "Read", "Skill"]
---

# Hookify 규칙 목록

규칙 형식을 이해하기 위해 **먼저 hookify:writing-rules 스킬을 로드**하세요.

프로젝트에 설정된 모든 hookify 규칙을 표시합니다.

## 단계

1. Glob 도구를 사용하여 모든 hookify 규칙 파일을 찾습니다:
   ```
   pattern: ".claude/hookify.*.local.md"
   ```

2. 발견된 각 파일에 대해:
   - Read 도구를 사용하여 파일을 읽습니다
   - 프론트매터(frontmatter) 필드 추출: name, enabled, event, pattern
   - 메시지 미리보기 추출 (처음 100자)

3. 결과를 표로 제시합니다:

```
## 설정된 Hookify 규칙

| 이름 | 활성화됨 | 이벤트 | 패턴 | 파일 |
|------|---------|-------|---------|------|
| warn-dangerous-rm | ✅ 예 | bash | rm\s+-rf | hookify.dangerous-rm.local.md |
| warn-console-log | ✅ 예 | file | console\.log\( | hookify.console-log.local.md |
| check-tests | ❌ 아니오 | stop | .* | hookify.require-tests.local.md |

**합계**: 규칙 3개 (활성화됨 2개, 비활성화됨 1개)
```

4. 각 규칙에 대해 간단한 미리보기를 보여줍니다:
```
### warn-dangerous-rm
**이벤트**: bash
**패턴**: `rm\s+-rf`
**메시지**: "⚠️ **위험한 rm 명령어가 감지되었습니다!** 이 명령어는 삭제할 수 있습니다..."

**상태**: ✅ 활성
**파일**: .claude/hookify.dangerous-rm.local.md
```

5. 유용한 바닥글을 추가합니다:
```
---

규칙을 수정하려면: .local.md 파일을 직접 편집하세요
규칙을 비활성화하려면: 프론트매터에서 `enabled: false`로 설정하세요
규칙을 활성화하려면: 프론트매터에서 `enabled: true`로 설정하세요
규칙을 삭제하려면: .local.md 파일을 제거하세요
규칙을 생성하려면: `/hookify` 명령어를 사용하세요

**기억해 두세요**: 변경 사항은 즉시 반영되며, 재시작이 필요하지 않습니다
```

## 규칙을 찾을 수 없는 경우

hookify 규칙이 존재하지 않는 경우:

```
## 설정된 Hookify 규칙 없음

아직 생성된 hookify 규칙이 없습니다.

시작하려면 다음을 수행하세요:
1. `/hookify`를 사용하여 대화를 분석하고 규칙을 만듭니다
2. 또는 `.claude/hookify.my-rule.local.md` 파일을 수동으로 생성합니다
3. 문서에 대해서는 `/hookify:help`를 참조하세요

예시:
```
/hookify Warn me when I use console.log
```

예시 규칙 파일은 `${CLAUDE_PLUGIN_ROOT}/examples/`를 확인하세요.
```
