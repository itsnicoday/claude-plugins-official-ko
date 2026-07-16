---
name: m5-onboard
description: 새로 연결된 M5Stack ESP32 디바이스 (Cardputer, Cardputer-Adv, Core, CoreS3, Stick)의 엔드투엔드 온보딩 — USB 상에서 감지하고, UIFlow 2.0 펌웨어를 플래시하며, Claude Buddy MicroPython 앱 번들을 설치합니다. 사용자가 M5Stack 또는 ESP32 보드를 연결했거나, 플래시/프로비저닝/리셋하고 싶을 때, 또는 "m5-onboard go"라고 말할 때 사용합니다.
---

# M5Stack Onboarding

이 스킬은 M5Stack ESP32 디바이스의 콜드 스타트 워크플로우 전체를 자동화합니다. USB 상에서 디바이스를 감지하고, 모델을 식별하고, UIFlow 2.0을 플래시하고, MicroPython 앱 번들을 `/flash/`에 푸시하여 디바이스가 사용자 소프트웨어로 부팅되도록 합니다. 번들로 제공되는 앱(Claude Buddy, Snake, Hello)은 BLE 또는 USB를 통해 통신합니다. 이 워크플로우는 macOS, Linux, Windows에서 작동합니다. 이 스킬은 M5Stack Basic v2.6 (CH9102 브릿지, ESP32-D0WDQ6-V3, 16MB 플래시)을 기준으로 개발되었으며 Core 제품군 전체로 일반화되었으며, 현재는 Cardputer-Adv (ESP32-S3, 네이티브 USB)를 기본 대상으로 삼고 있습니다.

## Where the scripts live

이 스킬은 참고용으로 `cwc-makers` 플러그인의 일부로 포함되어 제공되지만, 실행 가능한 스크립트와 `buddy/` 앱 번들은 https://github.com/moremas/build-with-claude 의 로컬 클론에 위치합니다 (`/maker-setup` 명령어가 이 클론을 생성합니다). 아래의 모든 `scripts/*.py` 호출은 해당 클론의 `onboard/` 디렉터리 내부에서 실행해야 `--apps buddy`가 형제 `buddy/device/` 페이로드로 올바르게 확인됩니다.

## When to use

사용자가 M5Stack 디바이스를 연결하고 프로비저닝하기를 원할 때 이 스킬을 사용합니다. 의사 결정 트리:

- **완전히 새로운/알 수 없는 디바이스** → `onboard.py --apps buddy`를 엔드투엔드로 실행합니다 (감지 → 식별 → 플래시 → 앱 설치). 이것이 기본 경로입니다.
- **이미 플래시된 디바이스, 사용자가 앱만 설치/업데이트하려는 경우** → `install_apps.py --src buddy` (또는 `.py` 파일 디렉터리에 대한 임의의 `--src <path>`)를 실행합니다.
- **플래시된 디바이스, 뭔가 고장난 것 같은 경우** → `smoke_test.py`를 실행합니다 (I2C + LCD + 스피커 + 버튼 확인).
- **사용자가 버스에 무엇이 있는지 / 디바이스가 무엇을 할 수 있는지 알고 싶을 때** → `smoke_test.py`.

여러 디바이스가 연결되어 있는 경우, 어떤 포트를 대상으로 할지 사용자에게 물어보세요. 추측하지 마십시오. 사용자가 이전에 작업했던 디바이스(예: "지난번과 같은 것" 또는 "다른 Buddy")를 프로비저닝하는 경우, 사용자가 명시하지 않는 한 기본값으로 `--apps buddy`를 사용하십시오.

### Which variant to assume

이 스킬이 적용되는 장비는 압도적으로 **Cardputer-Adv** 보드를 프로비저닝하므로, 현재 `onboard.py`는 기본값으로 `--variant cardputer-adv`를 사용합니다. 실무적으로 이는 다음을 의미합니다:

- 사용자가 모델에 대해 아무 말도 하지 않으면 기본값으로 진행합니다. 사용자가 Cardputer-Adv를 가지고 있을 가능성이 거의 확실합니다.
- 사용자가 "Cardputer"라고만 말하면 ("Adv" 없이) 물어보세요. 두 모델은 외형은 공유하지만 서로 다른 펌웨어 이미지를 사용하며, 잘못 플래시하면 디바이스가 무한 부팅 루프(boot-loop)에 빠집니다.
- 사용자가 다른 보드("Core2", "CoreS3", "Basic", "Fire" 등)의 이름을 말하면 일치하는 `--variant`를 명시적으로 전달하세요. 기본값은 적용되지 않습니다.
- 칩은 둘 다 ESP32-S3이며, `detect.py`는 UIFlow가 플래시되기 전에는 Cardputer와 Cardputer-Adv를 구분할 수 없습니다 (동일한 네이티브 USB-JTAG VID를 사용하며 플래시 전 I2C 프로브가 없음). 따라서 이는 하드웨어 핑거프린트의 문제가 아니라 사용자의 의도를 묻는 질문입니다.

## The workflow

메인 오케스트레이터(orchestrator)는 `scripts/onboard.py`입니다. 이 스크립트는 하위 스크립트들을 순서대로 실행하고 이들 사이의 연계(재부팅 대기, MAC 주소 캡처, 진행 상황 보고)를 처리합니다. 사용자가 부분 실행을 요청하지 않는 한, 하위 스크립트들을 직접 엮어 호출하는 것보다 이 메인 스크립트를 직접 호출하는 것이 좋습니다.

기본 프로비저닝 명령어 (새 Cardputer-Adv에 buddy 번들 설치):

```
python3 scripts/onboard.py --apps buddy
```

**Claude Code의 Bash 도구에서 이 명령어를 호출하는 방법.** `onboard.py`를 포그라운드 Bash 명령어로 호출하지 마십시오. Bash 도구는 출력을 캡처하고 명령어가 종료될 때까지 어시스턴트에게 스트리밍하지 않습니다. 그런데 이 명령어는 실행하는 데 2~3분이 소요됩니다. 이러한 무응답 상태는 중단(hang)된 것처럼 보이며, 어시스턴트는 대개 다운로드 모드 버튼 동작 프롬프트가 사용자에게 도달하기도 전에 포기하게 됩니다. 대신 항상 `run_in_background: true` 옵션으로 실행하고 로그 파일로 `tee`를 한 다음, 모니터(Monitor) 도구(또는 Read를 통한 주기적인 `tail`)를 사용하여 단계별 배너, 하트비트, 프롬프트를 사용자에게 실시간으로 보여주세요. `2>&1`로는 해결되지 않습니다. 모든 진행 상황은 이미 stderr로 작성되고 있으며 터미널에는 정상적으로 표시됩니다. 해결책은 리디렉션이 아니라 스트리밍 의미론(streaming semantics)입니다. 권장되는 패턴:

```
# 실행 (백그라운드, 로그 tee):
python3 scripts/onboard.py --apps buddy 2>&1 | tee /tmp/m5-onboard.log

# 모니터링 (바이트 진행률 스팸에 묻히지 않고 핵심 이벤트 표시):
tail -f /tmp/m5-onboard.log | grep -E --line-buffered \
  "^====|heartbeat|Heads up|Enter download mode|download mode!|rebooted into UIFlow|Manual reset|DONE|ERROR|Error|Traceback|FAIL|failed|No USB|not detected|Attempt [0-9]|Device already in download|Download mode port|Post-flash port|Waiting for device"
```

### Relaying physical steps to the user (REQUIRED)

네이티브 USB 보드에서는 **수동 버튼 누르기 없이는 플래시 단계를 진행할 수 없습니다.** 소프트웨어적인 방법은 없습니다. 모니터링 중인 로그에 `Enter download mode`가 표시되거나(또는 플래시 단계에서 스크립트가 멈춘 것처럼 보일 때), 계속 진행하기 전에 반드시 사용자에게 **Cardputer 뒷면**에서 다음 단계를 수행하도록 사용자 자신의 언어로 전달해야 합니다:

1. **G0** 버튼을 누른 채로 **유지**합니다.
2. G0를 계속 누른 상태에서 **RST** 버튼을 짧게 눌렀다 뗍니다.
3. G0를 약 1초 동안 더 누르고 있다가 뗍니다.
4. 화면이 완전히 꺼져야 합니다 — 이는 다운로드 모드가 활성화되었음을 의미합니다.

디바이스 화면이 꺼지지 않고 UIFlow로 재부팅된다면, 사용자에게 G0를 너무 빨리 뗐으니 더 오래 누르고 있으라고 안내해 주세요. 사용자가 화면이 꺼졌음을 확인하기 전에는 스크립트를 재시도하거나 소프트웨어적 해결책을 시도하는 등 다음 단계로 넘어가지 마십시오. 그렇지 않으면 플래시가 시작되지 않습니다. 이후의 `Manual reset` 프롬프트에 대해서도 마찬가지입니다. 물리적 단계를 안내하고 사용자를 기다리십시오.

Claude Code를 거치지 않고 자체 터미널에서 직접 `onboard.py`를 실행하는 사용자는 모든 출력을 실시간으로 볼 수 있으므로 변경할 필요가 없습니다.

`--port`가 생략되면 `detect.py`는 세 가지 OS 전체에서 가장 가능성 높은 대상을 선택합니다: 네이티브 USB ESP32-S3 (macOS의 `/dev/cu.usbmodem*`, Linux의 `/dev/ttyACM*`, Windows의 `COMx`) 또는 오래된 보드의 CH9102/CP210x UART 브릿지. 블루투스 시리얼 포트는 제외됩니다. 후보가 여러 개 있으면 사용자에게 물어봅니다.

알려진 앱 이름인 `buddy`는 이 리포지토리 안의 `buddy/device/` 디렉터리로 확인됩니다 (커스텀 런처 + Hello + Claude Buddy BLE 클라이언트 + Snake). 다른 `--apps` 값은 파일 시스템 경로로 처리됩니다.

재플래시를 건너뛰고 이미 프로비저닝된 디바이스로 앱을 푸시(또는 업데이트)하려면:

```
python3 scripts/install_apps.py --port <PORT> --src buddy
```

여기서 `<PORT>`는 지난 전체 실행 시 `detect.py`가 출력한 포트 이름입니다 (예: `/dev/cu.usbmodem1101`, `/dev/ttyACM0`, `COM3`).

### Stages

1. **감지 (Detect)** (`detect.py`) — 시리얼 포트를 열거하고 USB-UART 브릿지 (CH9102 벤더 `0x1A86`, Silabs CP210x `0x10C4`, FTDI `0x0403`) 또는 ESP32-S3 네이티브 USB-JTAG 인터페이스 (`0x303A`)로 필터링합니다. esptool로 프로브하여 칩을 확인합니다. 포트 이름은 OS마다 다르지만 (macOS의 `/dev/cu.usbmodem*`, Linux의 `/dev/ttyACM*`/`ttyUSB*`, Windows의 `COMx`) pyserial이 이를 추상화합니다.
2. **식별 (Identify)** (`detect.py`) — 포트 발견과 동시에 `detect.py`는 공장 테스트 파티션 시그니처를 읽거나 UIFlow가 켜진 후 I2C를 스캔하고, `references/hardware_signatures.md`를 교차 참조하여 올바른 펌웨어 변형 (Basic-16MB, Core2, CoreS3, Cardputer-Adv 등)을 제안합니다. 사용자용 변형 선택은 `onboard.py --variant`를 통해 수행되며, 별도의 `detect.py --identify` 플래그는 없습니다.
3. **펌웨어 가져오기 (Fetch firmware)** (`fetch_firmware.py`) — M5Burner 매니페스트 API를 쿼리하고 적절한 UIFlow 2.0 바이너리를 시스템 임시 디렉터리에 다운로드합니다. 실행 간에 캐시되므로 언제든지 캐시를 삭제해도 무방하며, 단지 다시 다운로드할 뿐입니다.
4. **플래시 (Flash)** (`flash.py`) — UART 브릿지의 경우 **460800 baud**로 `esptool write_flash 0x0 <image>`를 수행하고, 네이티브 USB S3 디바이스의 경우 115200 baud로 `--no-stub`을 수행합니다. 921600 baud는 CH9102 브릿지에서 간헐적으로 실패하므로 속도를 올리지 마십시오. 네이티브 USB 플래시는 지우기(erase) 도중 간헐적으로 `Lost connection, retrying` 에러를 발생시킬 수 있으나 esptool이 복구합니다. 플래시 자체는 성공했더라도 포스트 플래시 `watchdog-reset` 정리 단계가 실패할 수 있습니다. `flash.py`는 esptool의 표준 출력을 파싱하여 `Hash of data verified`가 표시된 경우 이 특정 실패 패턴을 심각하지 않은 것으로 처리하고, `onboard.py`는 필요에 따라 `flash.native_reset()`과 수동 RESET 안내로 대체합니다.
5. **앱 설치 (Install apps)** (선택 사항, `install_apps.py`) — 소스 디렉터리의 모든 `.py` 파일을 붙여넣기 모드 REPL 업로드 방식을 통해 `/flash/`에 업로드한 다음, `repl_reset`을 통해 재부팅합니다 (네이티브 USB에서는 DTR/RTS가 아무런 동작도 하지 않으므로 건드리지 마세요). 소스 레이아웃: 루트 `*.py` → `/flash/`, `apps/*.py` → `/flash/apps/` (UIFlow의 기본 런처가 이를 스캔함). 번들에 루트 `main.py`가 포함된 경우, `install_apps.py`는 NVS `boot_option=2`를 설정하여 UIFlow의 자체 런처가 실행되지 않고 우리의 `main.py`가 부팅 흐름을 대신하도록 합니다. 이는 ESP32-S3에서 BLE를 사용하는 앱에 필수적입니다 (아래 주의 사항 참고).
6. **동작 테스트 (Smoke test)** (선택 사항, `smoke_test.py`) — I2C 스캔, LCD 테스트 패턴, 스피커 비프음, 버튼 입력을 테스트합니다.

## Critical gotchas (baked into the scripts — do not second-guess)

이들은 스크립트가 이미 올바르게 처리하고 있는 사항이지만, 사용자가 "그냥 esptool을 수동으로 실행해 줘"라고 요청하더라도 재정의하거나 우회해서는 안 되는 항목들입니다:

- **네이티브 USB ESP32-S3 보드 (Cardputer, Cardputer-Adv, CoreS3)는 다운로드 모드로 들어가기 위해 물리적인 BtnG0 + BtnRST 동작이 필요합니다.** 소프트웨어적인 방법은 없습니다. 칩에 DTR/RTS 브릿지가 없으므로 esptool이나 pyserial이 ROM 부트로더로 진입시킬 수 있는 방법은 없으며, 사용자가 하드웨어 버튼으로 리셋 펄스가 유지되는 동안 GPIO0을 로우(low) 상태로 유지해야 합니다. 특히 Cardputer-Adv의 경우 두 버튼(BtnG0와 BtnRST) 모두 **디바이스 뒷면**에 있으며, 작고 평평하여 손톱으로 누르는 것이 가장 쉽습니다. `onboard.py:_wait_for_download_port`는 플래시 단계 도중 런타임에 이를 안내합니다: *BtnG0를 누른 채 유지하고, BtnRST를 짧게 눌렀다 뗀 후, BtnRST를 먼저 떼고, BtnG0를 약 1초 더 누르고 있다가 뗍니다. 화면이 완전히 꺼져야 합니다.* 디바이스가 UIFlow로 다시 부팅되면 BtnG0를 너무 일찍 뗀 것이므로, 안내를 다시 보여주고 더 오래 누르고 있도록 유도합니다. 이를 `esptool --before default_reset` 또는 pyserial의 DTR/RTS로 자동화하려고 하지 마십시오. 네이티브 USB에서는 둘 다 아무런 동작도 하지 않으며 (핀이 EN에 연결되어 있지 않음), 이를 추가하면 진짜 프롬프트만 숨겨지게 됩니다.
- **플래시 도중에 디바이스를 분리하지 마십시오.** 특히 네이티브 USB에서는 더욱 그렇습니다. 플래시 도중에 연결이 끊어지면 내부 플래시가 일관되지 않은 상태가 됩니다. 마스크 ROM(Mask ROM)은 그 이후에도 접근이 가능하므로 (뒷면의 BtnG0를 단독으로 누르거나 BtnG0 + BtnRST 동작을 수행함) 복구 방법은 단지 `m5-onboard go`를 다시 실행하는 것입니다. 이 스킬은 멱등성(idempotent)이 있으므로 다시 다운로드 모드로 진입하고 플래시하고 앱을 푸시합니다. 당황하여 케이스를 열지 마십시오. USB 물리 계층(PHY)만 손상되지 않았다면 마스크 ROM은 실리콘 칩 레벨에서 제공되므로 손상된 플래시 상태에서도 살아남습니다.
- **Baud rate는 UART 브릿지에서는 460800, 네이티브 USB에서는 `--no-stub`과 함께 115200을 사용합니다.** 어느 쪽이든 921600을 사용하지 마십시오. CH9102 브릿지는 921600에서 `erase_flash` 시 동기화를 잃습니다 (이론적인 문제가 아니라 실제로 실패합니다). 네이티브 USB의 스터브(stub) 보오율 상승 경로는 플래시 중간에 "Lost connection"을 유발합니다. 115200 no-stub 방식은 실패하지 않기 때문에 결과적으로 엔드투엔드 관점에서 더 빠릅니다.
- **NVS 쓰기는 `set_blob`이 아닌 `set_str`을 사용해야 합니다** (`install_apps.py`의 `boot_option` 설정과 관련됨). UIFlow의 시작 코드는 `nvs.get_str()`을 호출하며 ESP-IDF는 blob과 string 항목을 개별적으로 태그합니다. blob 태그가 달린 키는 `get_str`에 `ESP_ERR_NVS_NOT_FOUND`를 반환하여 디바이스를 부팅 루프에 빠뜨립니다. 이전 시도에서 blob으로 썼다면 `set_str`을 하기 전에 `nvs.erase_key(name)`를 호출하십시오.
- **REPL 멀티라인 블록은 붙여넣기 모드(paste mode)가 필요합니다.** `try:`/`except:`를 한 줄씩 전송하면 REPL이 들여쓰기를 무한히 누적하게 됩니다. Ctrl-E를 눌러 붙여넣기 모드로 진입한 후 블록을 전송하고, Ctrl-D를 눌러 실행하십시오. `mpy_repl.py`가 이를 래핑하여 처리합니다.
- **하드 리셋은 DTR=False, RTS=True 상태로 100ms 유지 후 RTS=False를 취하는 것이지만, 이는 UART 브릿지 디바이스에서만 작동합니다.** 네이티브 USB ESP32-S3 보드에서는 DTR/RTS 핀이 EN/GPIO0에 연결되어 있지 않으므로 이 펄스는 조용히 아무 일도 하지 않고 지나갑니다. 이러한 디바이스의 설치 후 재부팅에는 `mpy_repl.repl_reset()` (REPL을 통해 `machine.reset()`을 보냄)을 사용하십시오. `install_apps.py`가 이미 이 방식으로 처리하고 있습니다. 만약 `install_apps.py`를 우회하여 직접 흐름을 만드는 경우, usbmodem 포트에서 DTR/RTS를 조작하여 재부팅을 기대하지 마십시오. 파일은 디스크에 저장되지만 이전 코드가 계속 실행 중일 것입니다. 이 회귀 버그로 고생한 적이 있습니다.
- **대기 상태(idle)의 heap-debug 루프는 정상적입니다.** UIFlow 2.0은 페이링 화면에서 대기하는 동안 asyncio 진단 정보를 출력합니다. 이를 멈춤(hang) 현상으로 오해하지 마십시오.
- **Cardputer-Adv (ESP32-S3) BLE 주변기기는 NVS `boot_option=2` + 커스텀 `main.py`가 필요합니다.** UIFlow의 기본값인 `boot_option=1`은 백그라운드에서 Flow 페어링 BLE 애드버타이즈(advertise)를 시작하여 NimBLE 컨트롤러를 굳게 만듭니다. 이로 인해 이후 사용자 코드에서 호출하는 `gap_advertise(adv_data=...)`가 페이로드 크기와 상관없이 OSError(-519) "Memory Capacity Exceeded" 에러를 발생시키고, 결국 디바이스는 iOS 및 데스크톱 Claude Buddy 앱이 필터링하는 빈 AD 필드로 애드버타이즈하게 됩니다. 번들의 `main.py`는 `/flash/`에 위치하며 부팅 흐름을 넘겨받아 `/flash/apps/` 목록을 보여주는 간단한 메뉴를 표시하므로, BLE 자체를 건드리지 않고 컨트롤러 상태를 깨끗하게 유지하여 사용자가 어떤 앱을 선택하든 정상 작동할 수 있도록 돕습니다. 이제 `install_apps.py`는 번들에 루트 `main.py`가 포함된 경우 자동으로 `boot_option=2`를 설정합니다. 이 동작이 퇴보하지 않도록 하십시오.

## After provisioning (what the user sees on the device)

`m5-onboard go`가 완료되어 `DONE` 배너가 표시되면 디바이스를 단독으로 사용할 준비가 된 것입니다:

- **전원.** Cardputer-Adv 우측 가장자리에 있는 스위치를 밀어 전원을 켭니다. 끌 때도 동일한 스위치를 사용합니다. 보드는 케이블이 분리되었을 때 내부 LiPo 배터리로 작동하며, USB-C로 충전됩니다.
- **부팅.** 짧은 부팅 로그가 스크롤된 후 런처 메뉴가 자동으로 나타납니다. 메뉴는 `/flash/apps/` 내의 모든 `.py` 파일과 최상위 `/flash/*.py` 항목들을 나열합니다.
- **탐색.** 화살표 키(또는 키보드의 빨간 트랙포인트 스타일 커서 키)를 사용하여 메뉴를 스크롤하고, Enter 키로 하이라이트된 앱을 실행하며, 앱 내에서 ESC 키를 누르면 런처로 돌아갑니다.
- **이벤트 WiFi 자동 연결.** 번들의 `main.py`는 부팅할 때마다 하드코딩된 이벤트 WiFi (SSID `cardputer`)에 연결하고 런처 메뉴가 나타나기 전에 LCD에 그 결과를 표시합니다. 자격 증명(credentials)은 `buddy/device/wifi_event.py`에 저장됩니다. 이 연결은 최선의 노력(best-effort)으로 진행되므로 연결이 실패하더라도 런처는 항상 계속 실행됩니다. 만약 이벤트 외부에서 이 번들을 사용하는 경우, `wifi_event.py`를 편집하거나 `main.py`에서 `_connect_wifi_with_splash()` 호출을 제거하십시오.
- **BLE를 통한 Claude Buddy.** 최초 1회만 수행: Claude Desktop에서 **Help → Troubleshooting → Enable Developer Tools**를 활성화합니다 (1회 설정으로 이후 실행 시에도 유지됨). 그 다음 **Developer menu → Hardware Buddy → Connect**를 선택합니다. BLE는 WiFi 상태와 무관하게 작동합니다. Claude.app으로의 링크는 로컬 링크입니다.
- **UIFlow로 돌아가기.** buddy 번들은 `/flash/`에 오직 `main.py`만 설치하므로 (대체용 `boot.py` 없음), 기본 UIFlow `boot.py`는 전혀 변경되지 않으며 복원해야 할 `boot_uiflow.py` 백업도 없습니다. 디바이스 REPL에서 `os.remove('/flash/main.py')`를 수행한 후 `machine.reset()`을 실행하여 우리 `main.py`를 디바이스에서 삭제하면 복원됩니다. 다음 부팅 시 UIFlow의 기본 런처가 제어권을 가져옵니다. 펌웨어를 포함하여 완전히 초기화하려면 `--apps` 없이 스킬을 다시 실행하십시오.

## Files

- `scripts/onboard.py` — 메인 오케스트레이터
- `scripts/detect.py` — 포트 발견 + 칩 ID 확인
- `scripts/fetch_firmware.py` — M5Burner API 조회 + 다운로드
- `scripts/flash.py` — esptool 래퍼 스크립트
- `scripts/install_apps.py` — 붙여넣기 모드 REPL을 사용하여 `.py` 파일 디렉터리를 `/flash/`로 푸시합니다. 덮어쓰기 전에 `boot.py`를 `boot_uiflow.py`로 백업하고, 번들에 루트 `main.py`가 포함된 경우 NVS `boot_option` 키도 작성합니다.
- `scripts/smoke_test.py` — I2C + LCD + 스피커 + 버튼 확인
- `scripts/mpy_repl.py` — 공통 시리얼/REPL 헬퍼 (붙여넣기 모드, 하드 리셋, 부팅 로그 캡처)
- `references/hardware_signatures.md` — 칩 + I2C 핑거프린트 → 모델 → 펌웨어 매핑
- `references/uiflow2_nvs.md` — 타입 및 실패 모드가 정리된 NVS 키 참조서

## Dependencies

- `pyserial` — `onboard/scripts/vendor/serial/`에 내장(vendored)되어 있습니다 (버전 3.5 고정, BSD-3-Clause 라이선스).
- `esptool` — pip 의존성으로, `requirements.txt`에 선언되어 있습니다. 가져오기 가능 여부는 `importlib.util.find_spec("esptool")`을 통해 확인하며, 바이너리 백스톱(backstop) 검색 경로는 macOS의 `~/Library/Python/*/bin/`, Linux의 `~/.local/bin/`, Windows의 `%APPDATA%\Python\Python3XX\Scripts\`를 커버합니다.

`onboard.py`는 시작 시 사전 확인(preflight check)을 실행합니다: `esptool` (또는 매우 드물지만 벤더가 정리된 경우 `pyserial`)이 누락된 경우 필요한 항목을 나열하고 사용자에게 지금 설치할지 묻습니다. `Y` (또는 Enter) 입력 시 현재 인터프리터에서 `python -m pip install --user <missing>`을 실행한 후 다시 확인합니다. 가상환경(venv) 내부에서는 `--user` 플래그가 제외되어 가상환경의 site-packages에 설치됩니다. 비대화형 호출자(파이프 입력 등)에게는 프롬프트 대신 수동 설치 힌트가 제공됩니다.

이 스킬이 작업을 시작하려면 Python 자체가 먼저 존재해야 합니다. 인터프리터 내부에서 인터프리터를 빌드(bootstrapping)할 수는 없기 때문입니다. `git`은 필수 사항이 **아닙니다** — `git --version`이 실패하면 `/maker-setup` 명령어는 `curl` + `tar`를 사용해 GitHub tarball을 다운로드하는 방식으로 대체합니다 (둘 다 macOS, Linux, Windows 10+에 기본 탑재되어 있음). Claude는 임의의 `scripts/*.py` 호출을 실행하기 *전에* Python을 감지하고 없는 경우 설치할 책임이 있습니다. 감지는 단순하게 `python3 --version` / `python --version`을 실행하는 방식이며, 실패 시 Claude는 다른 작업을 진행하기 전에 호스트의 네이티브 패키지 관리자를 통해 Python을 가져옵니다.

**OS별 Python 부트스트랩 (누락 시 Claude의 책임):**

- **Windows** — `winget install -e --id Python.Python.3.13 --silent --accept-source-agreements --accept-package-agreements`. 약 30초가 소요되며 UI가 표시되지 않고 PATH가 정상적으로 설정됩니다. 이후 현재 셸이 `python`을 인식하지 못하면, 사용자에게 터미널을 닫고 다시 열도록 안내하십시오 (Windows는 새로운 셸에서만 PATH를 갱신함).
- **macOS** — 최신 macOS에는 대개 Python 3가 `/usr/bin/python3`로 사전 설치되어 제공됩니다 (Apple 제공). 어떤 이유로 설치되어 있지 않다면 Homebrew를 통해 `brew install python@3.13`을 실행하는 것이 기본입니다. Homebrew 자체가 없는 경우 `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`를 통해 설치하도록 제안할 수 있습니다 (단, 사용자가 동의한 경우에만 — Homebrew는 winget보다 설치 용량이 크고 무겁습니다).
- **Linux** — 배포판 패키지 관리자를 사용합니다. Debian/Ubuntu: `sudo apt-get update && sudo apt-get install -y python3 python3-pip`. Fedora: `sudo dnf install -y python3 python3-pip`. Arch: `sudo pacman -S --noconfirm python python-pip`. sudo 권한이 필요할 수 있으며 필요시 사용자에게 비밀번호 입력 프롬프트를 표시해야 합니다.

**pyserial — 스킬과 함께 제공됨:**

고정된 `pyserial 3.5`가 `scripts/vendor/`에 포함되어 있습니다 (BSD-3-Clause 라이선스, Apache 호환). `serial`을 가져오는 모든 스크립트는 첫 번째 서드파티 가져오기(import) 전에 `vendor_path.ensure_on_syspath()`를 호출합니다. 이 함수는 `sys.path`의 맨 앞에 `scripts/vendor/`를 추가하므로, 사용자가 시스템 전체에 어떤 버전을 설치했든 상관없이 내장된 사본으로 확인됩니다. 결과적으로 포트 열거 및 REPL I/O는 별도의 pip 설치 단계 없이도 새로운 클론에서 정상적으로 동작합니다. 약 500KB 크기의 순수 Python(pure-Python) 파일이며 macOS, Linux, Windows 모두 동일한 트리 구조를 공유합니다.

**esptool — pip 의존성, 최초 실행 시 자동 설치:**

`esptool`은 GPLv2+ 라이선스이므로 의도적으로 리포지토리에 **포함시키지 않았습니다**. 리포지토리 라이선스를 순수한 Apache-2.0으로 유지하기 위해 GPL 구성 요소는 리포지토리 트리가 아닌 사용자의 pip 관리 환경에 위치합니다. 스킬의 사전 확인 단계에서 가져오기 가능한 `esptool`이 있는지 체크하고, 없다면 설치 프롬프트를 표시합니다 (`python -m pip install --user esptool` — 가상환경 내부에서는 `--user`가 생략되어 site-packages로 들어감). 서브프로세스 호출 시에는 `[sys.executable, "-m", "esptool", ...]`을 사용합니다. 서브프로세스는 user-site를 상속받으므로 pip로 설치된 모듈을 깨끗하게 가져옵니다. `requirements.txt`에 명시적 설정을 위해 이를 선언해 두었으며, 프롬프트 경로는 아직 pip를 실행하지 않은 최초 참여자를 위한 기본 동작 방식입니다.

비대화형 호출자(파이프 입력, CI)는 프롬프트를 건너보고 대신 `python -m pip install --user esptool` 힌트를 받게 됩니다.

**누군가 `scripts/vendor/`를 삭제한 경우의 대체 경로:**

동일한 사전 확인 경로를 거치며 벤더 사본이 사라진 경우 pip를 통해 pyserial을 재설치합니다. 이는 누군가 벤더 디렉터리가 제외된 소스 전용 zip 파일을 다운로드했거나, 공간 절약을 위해 리포지토리를 수동으로 정리한 경우를 처리하기 위함입니다.

**USB 드라이버 — Windows 전용, 오래된 보드 대상:**

CH9102 USB-UART 드라이버는 Windows에서 여전히 수동으로 설치해야 합니다 — WCH가 winget 매니페스트를 게시하지 않기 때문입니다. UART 브릿지 보드(Basic, Fire, Core2, StickC)에서만 필요합니다. 네이티브 USB ESP32-S3 보드(Cardputer, Cardputer-Adv, CoreS3)는 Windows 기본 제공 드라이버를 사용하여 복합 USB-CDC 디바이스로 나열되므로 추가 설치가 필요하지 않습니다.

## Platform notes

이 스킬은 macOS, Linux, Windows에서 실행됩니다. 명확하지 않을 수 있는 유용한 팁:

- **포트 이름 지정.** pyserial이 조회를 추상화하지만 사용자가 보는 형태는 OS마다 다릅니다. `detect.py`가 보고하는 형태를 그대로 전달하십시오:
  - macOS: `/dev/cu.usbmodem1101` (네이티브 USB) 또는 `/dev/cu.usbserial-XXXX` (CH9102)
  - Linux: `/dev/ttyACM0` (네이티브 USB) 또는 `/dev/ttyUSB0` (UART 브릿지)
  - Windows: `COM3`, `COM4` 등 (불확실한 경우 장치 관리자 → 포트 확인)
- **Linux 권한 — 하드웨어를 탓하기 전에 이 부분을 읽어보세요.** 대부분의 배포판에서 sudo 없이 `/dev/ttyUSB*` / `/dev/ttyACM*`에 접근하려면 그룹 멤버십이 필요합니다 (Debian/Ubuntu/Arch의 경우 `dialout`, Fedora의 경우 `uucp`). 증상: `detect.py`는 포트를 찾아내지만, 플래시 단계에서 `Permission denied` 또는 `Could not open port` 에러와 함께 실패합니다. 장기적인 단일 해결책:
  ```bash
  sudo usermod -aG dialout $USER
  # 로그아웃 후 다시 로그인 — 그룹 변경 사항은 새 세션에서만 적용됨
  ```
  `sudo python3 scripts/onboard.py ...`를 일회성으로 실행하는 것도 작동하지만, 그룹 멤버십을 추가하는 것이 이후 사용자 모드에서 pyserial의 포트 열기가 깨끗하게 성공하므로 훨씬 더 좋습니다.
- **Windows PATH 주의 사항.** Python의 `pip install --user esptool`은 실행 파일을 `%APPDATA%\Python\Python3XX\Scripts\`에 설치합니다. 이 디렉터리가 PATH에 설정되어 있지 않으면 `pip`는 경고만 출력하고 다른 도구들은 설치를 인식하지 못합니다. `detect.py`는 PATH가 해결되지 않은 상태에서도 작동할 수 있도록 이 위치를 백스톱으로 직접 조회합니다. 그러나 스킬 외부에서 esptool을 호출하거나 다른 도구에서 "esptool을 찾을 수 없음" 에러가 발생하는 경우 다음 중 하나를 수행하십시오:
  - Python 설치 프로그램을 다시 실행하고 "Add Python to PATH"에 체크합니다 (설치 기본값).
  - 시스템 속성 → 환경 변수를 통해 `%APPDATA%\Python\Python3XX\Scripts`를 PATH에 추가합니다.
  - PATH 설정에 상관없이 항상 작동하는 `python -m esptool ...` 형식을 사용합니다.
- **Windows Store Python.** 최신 Windows 11 머신에는 Microsoft Store를 통해 Python이 사전 설치되어 있을 수 있습니다. 잘 작동하지만 PATH 동작이 다소 기이합니다 (`%LOCALAPPDATA%\Packages\PythonSoftwareFoundation.Python.*\` 아래에 위치함). `detect.py`는 이 위치도 검사합니다. 선택이 가능하다면 `winget install Python.Python.3.13` 버전이 좀 더 예측 가능합니다.
- **번들 경로 확인.** `install_apps.py`의 `--src buddy` 단축키는 다음 순서로 경로를 확인합니다:
  1. `$M5_BUDDY_DIR` 환경 변수가 설정된 경우 — 명시적 오버라이드로 항상 우선 적용됩니다. 이 클론에 포함되지 않은 포크 버전이나 커스텀 번들을 지정하고 싶을 때 유용합니다.
  2. 이 리포지토리 내부의 `buddy/device/` 디렉터리로, `install_apps.py`에서 위로 거슬러 올라가며 `os.path.realpath(__file__)`을 통해 찾습니다. `~/.claude/skills/m5-onboard/`에 심볼릭 링크로 설치된 스킬을 포함하여 모든 클론 위치에서 작동합니다.
  3. `~/Downloads/m5stack/buddy/device`
  4. `~/Desktop/m5stack/buddy/device`

  대부분의 설치는 (2)번에 매칭됩니다. 이 클론 외부의 번들을 지정하려는 특별한 경우에만 `M5_BUDDY_DIR`을 설정하십시오: `export M5_BUDDY_DIR=/path/to/buddy/device` (Unix) 또는 `$env:M5_BUDDY_DIR="C:\path\to\buddy\device"` (PowerShell).
- **펌웨어 캐시.** 다운로드된 펌웨어는 `~/.cache/m5-onboard/` (또는 `$XDG_CACHE_HOME/m5-onboard/`)에 저장되며, 없으면 0700 권한으로 생성됩니다. 캐시 파일은 작성할 때 MD5 검증을 거치고 캐시 히트 시 다시 검증합니다. 캐시 삭제는 안전하며 다음 실행 시 다시 다운로드합니다.
