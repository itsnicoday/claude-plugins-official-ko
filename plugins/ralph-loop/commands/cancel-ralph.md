---
description: "활성 Ralph Loop 취소"
allowed-tools: ["Bash(test -f .claude/ralph-loop.local.md:*)", "Bash(rm .claude/ralph-loop.local.md)", "Read(.claude/ralph-loop.local.md)"]
hide-from-slash-command-tool: "true"
---

# Ralph 취소

Ralph 루프를 취소하려면:

1. Bash를 사용하여 `.claude/ralph-loop.local.md` 파일이 존재하는지 확인합니다: `test -f .claude/ralph-loop.local.md && echo "EXISTS" || echo "NOT_FOUND"`

2. **만약 NOT_FOUND인 경우**: "활성화된 Ralph 루프를 찾을 수 없습니다."라고 안내합니다.

3. **만약 EXISTS인 경우**:
   - `.claude/ralph-loop.local.md` 파일을 읽어 `iteration:` 필드에서 현재 반복 횟수를 가져옵니다.
   - Bash를 사용하여 파일을 삭제합니다: `rm .claude/ralph-loop.local.md`
   - 다음과 같이 보고합니다: "Ralph 루프가 취소되었습니다 (진행된 반복 횟수: N)" (N은 실제 반복 횟수 값)
