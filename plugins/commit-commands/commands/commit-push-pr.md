---
allowed-tools: Bash(git checkout --branch:*), Bash(git add:*), Bash(git status:*), Bash(git push:*), Bash(git commit:*), Bash(gh pr create:*)
description: 커밋, 푸시 및 PR 오픈
---

## 컨텍스트

- 현재 git 상태: !`git status`
- 현재 git diff (스테이징 및 언스테이징된 변경 사항): !`git diff HEAD`
- 현재 브랜치: !`git branch --show-current`

## 작업 내용

위 변경 사항을 바탕으로 다음을 수행합니다:

1. main 브랜치인 경우 새로운 브랜치 생성
2. 적절한 메시지와 함께 단일 커밋 생성
3. 브랜치를 origin 원격 저장소로 푸시
4. `gh pr create`를 사용하여 풀 리퀘스트 생성
5. 귀하는 단일 응답에서 여러 도구를 호출할 수 있는 능력이 있습니다. 반드시 위의 모든 작업을 단일 메시지 내에서 수행해야 합니다. 다른 도구를 사용하거나 다른 작업을 수행하지 마십시오. 이 도구 호출 외에 다른 텍스트나 메시지를 전송하지 마십시오.
