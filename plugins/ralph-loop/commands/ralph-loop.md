---
description: "현재 세션에서 Ralph Loop 시작"
argument-hint: "PROMPT [--max-iterations N] [--completion-promise TEXT]"
allowed-tools: ["Bash(${CLAUDE_PLUGIN_ROOT}/scripts/setup-ralph-loop.sh:*)"]
hide-from-slash-command-tool: "true"
---

# Ralph Loop 명령어

Ralph 루프를 초기화하기 위해 설정 스크립트를 실행합니다:

```!
"${CLAUDE_PLUGIN_ROOT}/scripts/setup-ralph-loop.sh" $ARGUMENTS
```

작업을 진행해 주십시오. 종료하려고 할 때 Ralph 루프는 다음 반복 작업을 위해 동일한 프롬프트를 다시 입력합니다. 소스 파일 및 Git 이력에서 이전 작업 내용을 참조하여 개선을 반복할 수 있습니다.

치명적인 규칙: 완료 약속(completion promise)이 설정된 경우, 해당 상태가 완전히 그리고 명백히 참(TRUE)인 경우에만 완료 약속을 출력해야 합니다. 막혔다고 생각하거나 다른 이유로 탈출해야 한다고 해서 루프를 벗어나기 위해 허위 완료 약속을 출력하지 마십시오. 루프는 진정한 완료 상태에 도달할 때까지 지속되도록 설계되었습니다.
