---
description: [gone]으로 표시된 모든 Git 브랜치(원격 저장소에서는 삭제되었으나 로컬에는 남아있는 브랜치)를 정리하며, 연관된 작업 트리(worktrees)도 제거합니다.
---

## 작업 내용

원격 저장소에서 이미 삭제된 로컬 브랜치들을 정리하기 위해 다음 Bash 명령어들을 실행해야 합니다.

## 실행할 명령어

1. **먼저, [gone] 상태인 브랜치를 식별하기 위해 브랜치 목록을 확인합니다.**
   다음 명령어를 실행합니다:
   ```bash
   git branch -v
   ```
   
   참고: 앞에 `+` 접두사가 붙은 브랜치는 연결된 작업 트리(worktrees)가 존재하므로, 삭제하기 전에 작업 트리를 먼저 제거해야 합니다.

2. **다음으로, [gone] 브랜치를 위해 제거해야 할 작업 트리를 식별합니다.**
   다음 명령어를 실행합니다:
   ```bash
   git worktree list
   ```

3. **마지막으로, 작업 트리를 제거하고 [gone] 브랜치를 삭제합니다 (일반 브랜치 및 작업 트리 브랜치 모두 처리).**
   다음 명령어를 실행합니다:
   ```bash
   # Process all [gone] branches, removing '+' prefix if present
   git branch -v | grep '\[gone\]' | sed 's/^[+* ]//' | awk '{print $1}' | while read branch; do
     echo "Processing branch: $branch"
     # Find and remove worktree if it exists
     worktree=$(git worktree list | grep "\\[$branch\\]" | awk '{print $1}')
     if [ ! -z "$worktree" ] && [ "$worktree" != "$(git rev-parse --show-toplevel)" ]; then
       echo "  Removing worktree: $worktree"
       git worktree remove --force "$worktree"
     fi
     # Delete the branch
     echo "  Deleting branch: $branch"
     git branch -D "$branch"
   done
   ```

## 기대 결과

이 명령어들을 실행한 후 다음과 같이 처리됩니다:

- 모든 로컬 브랜치 목록과 그 상태 확인
- [gone] 브랜치와 연관된 모든 작업 트리 식별 및 제거
- [gone]으로 표시된 모든 브랜치 삭제
- 제거된 작업 트리와 브랜치에 대한 피드백 제공

만약 [gone]으로 표시된 브랜치가 없는 경우, 정리가 필요하지 않음을 보고하십시오.
