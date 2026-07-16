---
description: Code-with-Claude Makers Cardputer 온보딩 — build-with-claude 리포지토리를 가져오고, 펌웨어를 플래시하며, Claude Buddy 앱을 설치합니다.
disable-model-invocation: true
---

사용자가 USB-C로 claude.com/cwc-makers의 Cardputer-Adv를 연결했습니다.

1. 현재 작업 디렉터리(cwd) 아래 `build-with-claude/` 디렉터리에 https://github.com/moremas/build-with-claude 리포지토리를 가져옵니다:
   - git을 사용할 수 있는 경우: `git clone` (이미 존재하는 경우 `git pull`)
   - git을 사용할 수 **없는** 경우: 설치하지 마세요. 대신 GitHub tarball을 다운로드합니다. curl과 tar는 macOS, Linux, Windows 10+에 기본 탑재되어 제공됩니다:
     - macOS / Linux: `curl -L https://github.com/moremas/build-with-claude/archive/refs/heads/main.tar.gz | tar xz && mv build-with-claude-main build-with-claude`
     - Windows (PowerShell): `curl.exe -L -o bwc.zip https://github.com/moremas/build-with-claude/archive/refs/heads/main.zip; tar -xf bwc.zip; Rename-Item build-with-claude-main build-with-claude`
   - 나중에 `/maker-setup`을 다시 실행하면 단순히 다시 다운로드합니다 (~500KB). 별도의 업데이트 메커니즘은 필요하지 않습니다.
2. `m5-onboard` 스킬을 호출하고 이를 따라 `build-with-claude/` 내부에서 `onboard/scripts/onboard.py --apps buddy`를 실행하며, 사용자에게 다운로드 모드 버튼 프롬프트를 표시합니다.
3. 완료되면 사용자에게 Claude Buddy를 실행하는 방법을 안내하고 다음으로 무엇을 빌드하고 싶은지 물어봅니다 (반복 작업에 대해서는 `cardputer-buddy` 스킬 참고).
