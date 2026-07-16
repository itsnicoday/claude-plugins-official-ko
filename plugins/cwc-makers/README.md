# cwc-makers

[Code-with-Claude Makers](https://claude.com/cwc-makers) Cardputer 키트를 위한 매끄러운 온보딩을 제공합니다.

## 주요 기능

USB-C로 M5Stack Cardputer-Adv를 연결하고 `/maker-setup`을 입력하면 Claude가 다음을 수행합니다:

1. [`moremas/build-with-claude`](https://github.com/moremas/build-with-claude) 리포지토리를 클론합니다.
2. 디바이스를 감지하고, UIFlow 2.0 펌웨어를 플래시하며, Claude Buddy + Hello + Snake 앱 번들을 설치합니다.
3. 한 가지 물리적 단계(디바이스 뒷면의 다운로드 모드 버튼 누르기)를 안내합니다.
4. BLE를 통해 Claude Desktop과 페어링되는 작동 가능한 포켓 컴퓨터를 준비해 드립니다.

그 다음 Claude에게 매직 8볼, 픽셀 펫, 날씨 전광판 등 원하는 것을 빌드해 달라고 요청하면, 재플래시 없이 MicroPython 코드를 작성하여 디바이스로 푸시해 줍니다.

## 설치

```
/plugin install cwc-makers@claude-plugins-official
```

## 구성 요소

| 경로 | 타입 | 사용자 호출 가능 | 목적 |
|------|------|----------------|------|
| `commands/maker-setup.md` | 슬래시 명령어 | ✅ `/maker-setup` | 진입점 — 리포지토리 클론 + 전체 온보딩 실행 |
| `skills/m5-onboard/` | 스킬 | ✅ `/m5-onboard` | 전체 프로비저닝 플레이북 (감지, 플래시, 설치, 모든 주의 사항) |
| `skills/cardputer-buddy/` | 스킬 | ✅ `/cardputer-buddy` | 온보딩 후 앱 개발 반복 작업 (푸시, tail, REPL) |

`/maker-setup`이 설계된 진입점이며, 관련이 있을 때 Claude가 스킬들을 자동으로 트리거하기도 합니다. 스킬 콘텐츠는 업스트림 리포지토리에서 벤더링되어 있으므로, `~/.claude/skills/`에 심볼릭 링크를 걸지 않고도 Claude가 해당 분야 지식을 콘텍스트 안에서 활용할 수 있습니다.

## 요구 사양

호스트 머신에 Python 3.10+ 필요 (git은 선택 사항 — 없을 경우 `/maker-setup`이 curl+tar 다운로드로 대체 실행함). 온보딩 스크립트는 최초 실행 시 `esptool`을 자동 설치하며, `pyserial`은 업스트림 리포지토리에 포함(vendored)되어 있습니다.

## 라이선스

Apache-2.0. 스킬 콘텐츠는 [`moremas/build-with-claude`](https://github.com/moremas/build-with-claude) (Apache-2.0)에서 가져왔습니다.
