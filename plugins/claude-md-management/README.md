# CLAUDE.md 관리 플러그인

CLAUDE.md 파일을 유지 보수하고 개선하는 도구로, 품질 감사, 세션 학습 내용 기록, 프로젝트 메모리 최신 상태 유지를 지원합니다.

## 기능 정보

서로 다른 목적을 가진 두 가지의 상호 보완적인 도구를 제공합니다:

| | claude-md-improver (skill) | /revise-claude-md (command) |
|---|---|---|
| **목적** | CLAUDE.md를 코드베이스에 맞춤 | 세션 학습 내용 기록 |
| **실행 트리거** | 코드베이스 변경 사항 | 세션 종료 시 |
| **사용 시점** | 정기적인 유지 보수 | 세션 진행 중 누락된 컨텍스트 발견 시 |

## 사용법

### 스킬: claude-md-improver

현재 코드베이스 상태와 비교하여 CLAUDE.md 파일을 감사(audit)합니다:

```
"내 CLAUDE.md 파일 검사해줘"
"내 CLAUDE.md가 최신 상태인지 확인해줘"
```

<img src="claude-md-improver-example.png" alt="품질 점수 및 권장 업데이트 사항을 보여주는 CLAUDE.md 개선기(improver)" width="600">

### 명령어: /revise-claude-md

현재 세션에서 학습한 내용을 기록합니다:

```
/revise-claude-md
```

<img src="revise-claude-md-example.png" alt="세션 학습 내용을 CLAUDE.md에 기록하는 수정(Revise) 명령어" width="600">

## 저자

Isabella He (isabella@anthropic.com)
