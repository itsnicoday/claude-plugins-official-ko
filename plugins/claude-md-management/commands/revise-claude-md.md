---
description: 이번 세션에서 학습한 내용으로 CLAUDE.md 업데이트
allowed-tools: Read, Edit, Glob
---

이 코드베이스에서 Claude Code와 함께 작업하며 얻은 학습 내용을 검토합니다. 향후 Claude 세션이 더 효율적으로 작동할 수 있도록 돕는 컨텍스트로 CLAUDE.md를 업데이트하십시오.

## 1단계: 반성 및 회고 (Reflect)

Claude가 더 효율적으로 작업하는 데 누락되었던 컨텍스트는 무엇인가요?
- 사용했거나 새로 발견한 Bash 명령어
- 준수한 코드 스타일 패턴
- 유효했던 테스트 접근 방식
- 환경/설정상의 특이 사항
- 마주친 경고나 주의 사항(gotchas)

## 2단계: CLAUDE.md 파일 찾기

```bash
find . -name "CLAUDE.md" -o -name ".claude.local.md" 2>/dev/null | head -20
```

추가할 내용을 어디에 배치할지 결정합니다:
- `CLAUDE.md` - 팀 공유용 (git에 커밋됨)
- `.claude.local.md` - 개인/로컬 전용 (gitignored)

## 3단계: 추가 항목 작성

**간결하게 유지하십시오** - 개념당 한 줄씩 작성합니다. CLAUDE.md는 프롬프트의 일부로 들어가므로 간결함이 중요합니다.

포맷: `<command or pattern>` - `<brief description>`

피해야 할 사항:
- 장황한 설명
- 너무나 당연한 정보
- 다시 발생할 가능성이 낮은 일회성 수정 사항

## 4단계: 제안된 변경 사항 보여주기

추가할 각 항목에 대해:

```
### Update: ./CLAUDE.md

**Why:** [한 줄로 적는 이유]

\`\`\`diff
+ [추가 내용 - 간결하게 작성]
\`\`\`
```

## 5단계: 승인 후 적용

사용자에게 변경 사항을 적용할지 묻습니다. 사용자가 승인한 파일만 수정하십시오.
