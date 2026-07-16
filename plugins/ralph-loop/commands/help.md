---
description: "Ralph Loop 플러그인 및 사용 가능한 명령어 설명"
---

# Ralph Loop 플러그인 도움말

사용자에게 다음 내용을 설명하십시오:

## Ralph Loop란 무엇인가요?

Ralph Loop는 Geoffrey Huntley가 개척한 연속적인 AI 루프 기반의 반복적 개발 방법론인 Ralph Wiggum 기법을 구현한 것입니다.

**핵심 개념:**
```bash
while :; do
  cat PROMPT.md | claude-code --continue
done
```

동일한 프롬프트가 Claude에게 반복적으로 제공됩니다. "자기참조적(self-referential)"이라는 성격은 출력을 입력으로 다시 되돌리는 것이 아니라, Claude가 소스 파일 및 Git 이력에 남아 있는 본인의 이전 작업을 참조하는 데서 비롯됩니다.

**각 반복 주기(iteration):**
1. Claude가 동일한 프롬프트를 받음
2. 작업을 수행하여 파일을 수정함
3. 종료를 시도함
4. Stop 훅이 이를 가로채 동일한 프롬프트를 다시 입력함
5. Claude가 파일에서 본인의 이전 작업 내용을 확인함
6. 완료될 때까지 점진적으로 개선해 나감

이 기법은 "비결정론적 세상에서의 결정론적 실패(deterministically bad)"로 묘사됩니다. 즉, 실패가 예측 가능하므로 프롬프트 튜닝을 통해 체계적인 개선을 도모할 수 있습니다.

## 사용 가능한 명령어

### /ralph-loop <PROMPT> [OPTIONS]

현재 세션에서 Ralph 루프를 시작합니다.

**사용법:**
```
/ralph-loop "Refactor the cache layer" --max-iterations 20
/ralph-loop "Add tests" --completion-promise "TESTS COMPLETE"
```

**옵션:**
- `--max-iterations <n>` - 자동 정지 전 최대 반복 횟수
- `--completion-promise <text>` - 완료를 감지하기 위한 약속 텍스트 문구

**작동 방식:**
1. `.claude/.ralph-loop.local.md` 상태 파일 생성
2. 작업 수행
3. 종료 시도 시 Stop 훅이 감지
4. 동일한 프롬프트가 다시 입력됨
5. 이전 작업 내역을 참조함
6. 완료 약속이 감지되거나 최대 반복 횟수에 도달할 때까지 지속

---

### /cancel-ralph

활성화된 Ralph 루프를 취소합니다 (루프 상태 파일을 삭제함).

**사용법:**
```
/cancel-ralph
```

**작동 방식:**
- 활성 루프 상태 파일 확인
- `.claude/.ralph-loop.local.md` 파일 삭제
- 반복 횟수와 함께 취소 결과 보고

---

## 핵심 개념

### 완료 약속 (Completion Promises)

완료를 알리기 위해 Claude는 반드시 `<promise>` 태그를 출력해야 합니다:

```
<promise>TASK COMPLETE</promise>
```

Stop 훅은 이 특정 태그를 감지합니다. 이 태그가 없거나 `--max-iterations`가 설정되어 있지 않으면 Ralph 루프가 무한히 실행됩니다.

### 자기참조 메커니즘 (Self-Reference Mechanism)

"루프"는 Claude가 자기 자신과 대화한다는 의미가 아닙니다. 다음을 의미합니다:
- 동일한 프롬프트의 반복
- Claude의 작업물이 파일에 유지됨
- 각 반복 주기마다 이전의 시도 내용을 참조함
- 목표를 향해 점진적으로 단계를 밟아 나감

## 예시

### 대화형 버그 수정

```
/ralph-loop "Fix the token refresh logic in auth.ts. Output <promise>FIXED</promise> when all tests pass." --completion-promise "FIXED" --max-iterations 10
```

Ralph가 다음을 수행하는 것을 보게 될 것입니다:
- 수정 시도
- 테스트 실행
- 실패 감증
- 해결책 구체화 반복
- 현재 세션 내에서 진행

## Ralph Loop 사용이 적합한 경우

**적합한 작업:**
- 성공 기준이 명확하게 정의된 작업
- 반복 작업 및 개선 세부 조정이 필요한 작업
- 자가 수정을 동반한 점진적 개발 작업
- 신규(Greenfield) 프로젝트

**부적합한 작업:**
- 인간의 주관적 판단이나 디자인 결정이 필요한 작업
- 단발성 작업
- 성공 기준이 모호한 작업
- 프로덕션 버그 디버깅 (대신 타겟팅된 디버깅 기법을 사용하십시오)

## 더 알아보기

- Original technique: https://ghuntley.com/ralph/
- Ralph Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator
