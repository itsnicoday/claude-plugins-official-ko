---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
description: Git 커밋 생성
---

## 컨텍스트

- 현재 git 상태: !`git status`
- 현재 git diff (스테이징 및 언스테이징된 변경 사항): !`git diff HEAD`
- 현재 브랜치: !`git branch --show-current`
- 최근 커밋 내역: !`git log --oneline -10`

## 작업 내용

위 변경 사항을 바탕으로 단일 Git 커밋을 생성합니다.

귀하는 단일 응답에서 여러 도구를 호출할 수 있는 능력이 있습니다. 스테이징과 커밋 생성을 단일 메시지 내에서 동시에 처리하십시오. 다른 도구를 사용하거나 다른 작업을 수행하지 마십시오. 이 도구 호출 외에 다른 텍스트나 메시지를 전송하지 마십시오.
