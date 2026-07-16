# Ralph Loop 플러그인

Claude Code에서 반복적이고 자기참조적인 AI 개발 루프를 구현하는 Ralph Wiggum 기법을 적용한 플러그인입니다.

## Ralph Loop란 무엇인가요?

Ralph Loop는 연속적인 AI 에이전트 루프에 기반한 개발 방법론입니다. Geoffrey Huntley가 묘사했듯이, **"Ralph는 Bash 루프입니다"** - 간단한 `while true` 루프를 통해 AI 에이전트에게 프롬프트 파일을 반복적으로 제공하여 완료될 때까지 점진적으로 결과물을 개선하도록 만듭니다.

이 기법은 심슨 가족(The Simpsons)의 캐릭터 이름을 딴 Ralph Wiggum 코딩 기법에서 영감을 받았으며, 장애물에 개의치 않고 끈기 있게 반복 작업을 고수하는 철학을 담고 있습니다.

### 핵심 개념

이 플러그인은 Claude의 종료 시도를 가로채는 **Stop 훅(hook)**을 사용하여 Ralph를 구현합니다:

```bash
# 한 번만 실행하면 됩니다:
/ralph-loop "Your task description" --completion-promise "DONE"

# 이후 Claude Code가 자동으로 수행합니다:
# 1. 작업 수행
# 2. 종료 시도
# 3. Stop 훅이 종료를 차단
# 4. Stop 훅이 동일한 프롬프트를 다시 제공
# 5. 완료될 때까지 반복
```

루프는 외부 Bash 루프를 필요로 하지 않고 **현재 세션 내부**에서 동작합니다. `hooks/stop-hook.sh`의 Stop 훅이 정상적인 세션 종료를 차단하여 자기참조적 피드백 루프를 생성합니다.

이는 다음과 같은 **자기참조적 피드백 루프**를 구성합니다:
- 반복 주기(iteration) 사이에 프롬프트는 절대 변경되지 않음
- Claude의 이전 작업 결과가 파일에 유지됨
- 각 반복 주기마다 수정된 파일 및 Git 이력을 참조함
- Claude가 파일에 기록된 이전 작업을 분석하여 자율적으로 개선을 수행함

## 빠른 시작

```bash
/ralph-loop "Build a REST API for todos. Requirements: CRUD operations, input validation, tests. Output <promise>COMPLETE</promise> when done." --completion-promise "COMPLETE" --max-iterations 50
```

Claude가 다음을 수행합니다:
- API를 점진적으로 구현
- 테스트 실행 및 실패 확인
- 테스트 출력을 기반으로 버그 수정
- 모든 요구사항을 충족할 때까지 반복
- 완료 시 완료 약속(completion promise) 출력

## 명령어

### /ralph-loop

현재 세션에서 Ralph 루프를 시작합니다.

**사용법:**
```bash
/ralph-loop "<prompt>" --max-iterations <n> --completion-promise "<text>"
```

**옵션:**
- `--max-iterations <n>` - N번 반복 후 정지 (기본값: 무제한)
- `--completion-promise <text>` - 완료를 알리는 텍스트 문구

### /cancel-ralph

활성화된 Ralph 루프를 취소합니다.

**사용법:**
```bash
/cancel-ralph
```

## 프롬프트 작성 모범 사례

### 1. 명확한 완료 기준

❌ 나쁜 예: "할 일 API를 만들고 좋게 구현해줘."

✅ 좋은 예:
```markdown
Build a REST API for todos.

When complete:
- All CRUD endpoints working
- Input validation in place
- Tests passing (coverage > 80%)
- README with API docs
- Output: <promise>COMPLETE</promise>
```

### 2. 단계별 목표 설정

❌ 나쁜 예: "완전한 이커머스 플랫폼을 만들어줘."

✅ 좋은 예:
```markdown
Phase 1: User authentication (JWT, tests)
Phase 2: Product catalog (list/search, tests)
Phase 3: Shopping cart (add/remove, tests)

Output <promise>COMPLETE</promise> when all phases done.
```

### 3. 자가 수정 지침 포함

❌ 나쁜 예: "기능 X를 구현하는 코드를 작성해줘."

✅ 좋은 예:
```markdown
Implement feature X following TDD:
1. Write failing tests
2. Implement feature
3. Run tests
4. If any fail, debug and fix
5. Refactor if needed
6. Repeat until all green
7. Output: <promise>COMPLETE</promise>
```

### 4. 탈출구 마련

불가능한 작업에서 무한 루프에 빠지는 것을 방지하기 위해 항상 안전장치로 `--max-iterations`를 설정하십시오:

```bash
# 권장: 항상 합리적인 수준의 반복 제한을 설정하십시오
/ralph-loop "Try to implement feature X" --max-iterations 20

# 프롬프트에 진행이 막혔을 때의 지침을 포함하십시오:
# "After 15 iterations, if not complete:
#  - Document what's blocking progress
#  - List what was attempted
#  - Suggest alternative approaches"
```

**참고**: `--completion-promise`는 정확한 문자열 일치를 기반으로 작동하므로 여러 개의 완료 조건("SUCCESS" 대 "BLOCKED" 등)을 동시에 지정할 수 없습니다. 항상 `--max-iterations`를 주요 안전장치로 활용하십시오.

## 철학

Ralph는 몇 가지 핵심 원칙을 구현합니다:

### 1. 완벽함보다 반복
첫 시도에 완벽을 기하기보다 루프를 돌며 결과물을 정제해 나가십시오.

### 2. 실패는 데이터다
"결정론적 실패(deterministically bad)"는 실패가 예측 가능하며 유용한 정보를 제공함을 의미합니다. 실패 데이터를 활용해 프롬프트를 미세 조정하십시오.

### 3. 오퍼레이터의 역량이 중요함
성공 여부는 좋은 모델을 쓰는 것뿐 아니라 좋은 프롬프트를 작성하는 숙련도에 달려 있습니다.

### 4. 끈기가 승리한다
성공할 때까지 계속 시도하십시오. 루프가 재시도 로직을 자동으로 처리합니다.

## Ralph Loop 사용이 적합한 경우

**적합한 작업:**
- 성공 기준이 명확하게 정의된 작업
- 반복 작업 및 세부 조정이 필요한 작업 (예: 테스트 통과)
- 모니터링 없이 방치해두어도 괜찮은 신규(Greenfield) 프로젝트
- 자동 검증 수단(테스트, 린터 등)이 갖춰진 작업

**부적합한 작업:**
- 인간의 주관적 판단이나 디자인 결정이 필요한 작업
- 단발성 작업
- 성공 기준이 모호한 작업
- 프로덕션 버그 디버깅 (대신 타겟팅된 디버깅 기법을 사용하십시오)

## 실제 성과

- Y Combinator 해커톤 테스트에서 야간에 6개의 저장소를 성공적으로 생성
- 5만 달러 규모의 계약 건을 297달러의 API 비용으로 완료
- 이 접근 방식을 활용해 3개월 동안 프로그래밍 언어("cursed") 전체를 자체 제작

## Windows 호환성

stop 훅은 정상 작동을 위해 Git for Windows를 필요로 하는 Bash 스크립트를 사용합니다.

**문제**: Windows 환경에서는 `bash` 명령어가 Git Bash가 아닌 WSL bash(설정이 잘못된 경우가 많음)로 연결되어 다음과 같은 에러와 함께 훅이 실패할 수 있습니다:
- `wsl: Unknown key 'automount.crossDistro'`
- `execvpe(/bin/bash) failed: No such file or directory`

**해결책**: 캐시된 플러그인의 `hooks/hooks.json`을 편집하여 다음과 같이 Git Bash를 명시적으로 지정합니다:

```json
"command": "\"C:/Program Files/Git/bin/bash.exe\" ${CLAUDE_PLUGIN_ROOT}/hooks/stop-hook.sh"
```

**위치**: `~/.claude/plugins/cache/claude-plugins-official/ralph-wiggum/<hash>/hooks/hooks.json`

**참고**: PATH 설정이 제대로 되어 있는 래퍼 파일인 `Git/bin/bash.exe`를 사용해야 하며, PATH 내에 유틸리티가 누락된 raw MinGW bash인 `Git/usr/bin/bash.exe`는 피하십시오.

## 더 알아보기

- Original technique: https://ghuntley.com/ralph/
- Ralph Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator

## 도움말

상세한 명령어 참조 및 예시를 확인하려면 Claude Code에서 `/help`를 실행하십시오.
