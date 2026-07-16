# iMessage

iMessage를 Claude Code 어시스턴트에 연결합니다. 기록 조회, 검색, 신규 메시지 감지를 위해 `~/Library/Messages/chat.db`를 직접 읽고, AppleScript를 통해 Messages.app으로 메시지를 보냅니다. 외부 서버도 없고 백그라운드 프로세스를 계속 켜둘 필요도 없습니다.

macOS 전용입니다.

## 빠른 설정
> 기본 설정: 본인에게 텍스트를 보냅니다. 허용 목록에 추가하기 전까지 다른 발신자의 메시지는 자동으로 무시됩니다(자동 회신 없음). 그룹 및 다중 사용자 설정에 대해서는 [ACCESS.md](./ACCESS.md)를 참조하십시오.

**1. 전체 디스크 접근 권한(Full Disk Access) 허용**

`chat.db`는 macOS TCC에 의해 보호됩니다. 서버가 이를 처음 읽을 때, macOS에서 터미널의 메시지 접근 허용 여부를 묻는 팝업 창이 뜹니다. **허용(Allow)**을 클릭하십시오. 프롬프트에는 bun을 실행한 앱(Terminal.app, iTerm, Ghostty, 또는 IDE)의 이름이 표시됩니다.

만약 허용하지 않음(Don't Allow)을 클릭했거나 프롬프트가 뜨지 않는 경우, 직접 권한을 허용하십시오: **시스템 설정 → 개인정보 보호 및 보안 → 전체 디스크 접근 권한** → 귀하의 터미널 앱을 추가합니다. 이 권한이 없으면 서버는 `authorization denied`와 함께 즉시 종료됩니다.

**2. 플러그인 설치**

이들은 Claude Code 명령어이므로, 먼저 `claude`를 실행하여 세션을 시작하십시오.

플러그인을 설치합니다. 별도의 환경 변수는 필요하지 않습니다.
```
/plugin install imessage@claude-plugins-official
```

**3. 채널 플래그와 함께 재실행**

이 설정 없이는 서버가 연결되지 않습니다. 세션을 종료하고 새 세션을 시작하십시오:

```sh
claude --channels plugin:imessage@claude-plugins-official
```

`/imessage:configure` 명령어가 탭 자동 완성으로 나타나는지 확인합니다.

**4. 자신에게 메시지 보내기**

아무 기기에서나 자신에게 iMessage를 보냅니다. 메시지는 즉시 어시스턴트에게 도달하며, 자신과의 대화(self-chat)는 접근 제어를 우회합니다.

> 첫 아웃바운드 답장 시 **자동화(Automation)** 권한 요청 프롬프트("Terminal이 Messages를 제어하려고 합니다")가 뜹니다. 확인(OK)을 클릭하십시오.

**5. Decide who else gets in.**

상대방의 핸들(주소)을 추가하기 전까지는 다른 사람의 문자가 어시스턴트에게 도달하지 않습니다:

```
/imessage:access allow +15551234567
```

핸들은 전화번호(`+15551234567`) 또는 Apple ID 이메일(`them@icloud.com`) 형식입니다. 어떻게 설정해야 할지 확실하지 않다면 Claude에게 설정을 검토해 달라고 요청하십시오.

## 작동 방식

| | |
| --- | --- |
| **인바운드 (Inbound)** | 초당 한 번씩 `chat.db`에서 `ROWID > watermark`를 확인하여 폴링합니다. 워터마크는 부팅 시 `MAX(ROWID)`로 초기화되므로, 재시작 시 이전 메시지들이 다시 실행되지 않습니다. |
| **아웃바운드 (Outbound)** | `tell application "Messages" to send …` 형태의 `osascript`를 사용합니다. 텍스트와 채팅 GUID가 argv를 통해 전달되므로 에스케이핑 문제가 발생하지 않습니다. |
| **히스토리 및 검색** | `chat.db`에 대해 직접 SQLite 쿼리를 수행합니다. 서버가 시작된 이후의 메시지뿐만 아니라 전체 히스토리를 제공합니다. |
| **첨부 파일** | `chat.db`에 절대 파일 시스템 경로가 저장됩니다. 메시지당 첫 번째 인바운드 이미지는 어시스턴트가 `Read`할 수 있도록 로컬 경로로 노출됩니다. 아웃바운드 첨부 파일은 텍스트 전송 후 별도의 메시지로 전송됩니다. |

## 환경 변수

| 변수 | 기본값 | 효과 |
| --- | --- | --- |
| `IMESSAGE_APPEND_SIGNATURE` | `true` | 아웃바운드 메시지 끝에 `\nSent by Claude` 문구를 추가합니다. 비활성화하려면 `false`로 설정하십시오. |
| `IMESSAGE_ALLOW_SMS` | `false` | iMessage 외에도 인바운드 SMS/RCS를 허용합니다. **SMS 발신자 ID는 스푸핑(spoofing)이 가능하기 때문에 기본적으로 꺼져 있습니다.** 꺼두지 않으면 본인 번호로 위장된 SMS가 접근 제어를 우회하게 될 수 있습니다. 위험 요소를 인지하고 있는 경우에만 활성화하십시오. |
| `IMESSAGE_ACCESS_MODE` | — | `static`으로 설정 시 런타임 페어링이 비활성화되며 `access.json` 파일만 읽습니다. |
| `IMESSAGE_STATE_DIR` | `~/.claude/channels/imessage` | `access.json` 및 페어링 상태 파일이 위치할 디렉터리를 변경합니다. |

## 액세스 제어

DM 정책, 그룹, 자신과의 대화(self-chat), 전달 설정, 스킬 명령어, 그리고 `access.json` 스키마에 대해서는 **[ACCESS.md](./ACCESS.md)**를 참조하십시오.

빠른 참조: ID는 **핸들 주소**(`+15551234567` 또는 `someone@icloud.com`)입니다. 기본 정책은 `allowlist`입니다. 이 스킬은 개인 `chat.db`를 읽습니다. 자신과의 대화는 항상 접근 제어 게이트를 통과합니다.

## 어시스턴트에게 노출되는 도구

| 도구 | 목적 |
| --- | --- |
| `reply` | 채팅으로 메시지를 보냅니다. `chat_id` + `text`, 선택적으로 `files`(절대 경로)를 지정할 수 있습니다. 긴 텍스트는 자동으로 분할되며, 파일은 개별 메시지로 전송됩니다. |
| `chat_messages` | 대화 스레드로 최근 히스토리를 가져옵니다. 각 스레드에는 참가자 목록과 함께 **DM** 또는 **Group** 라벨이 지정되며, 타임스탬프가 포함된 메시지(오래된 순서대로)가 반환됩니다. `chat_guid`를 생략하면 허용 목록에 있는 모든 대화를 한 번에 보거나, 특정 GUID를 전달하여 상세히 들여다볼 수 있습니다. 채팅당 기본 100개 메시지를 제공하며 `chat.db`에서 직접 네이티브 전체 히스토리를 읽습니다. |

## 지원되지 않는 기능

AppleScript는 메시지를 보낼 수는 있지만, 탭백(tapback), 편집, 스레드 답장 기능은 지원하지 않습니다. 이들 기능은 Apple의 비공개 API가 필요합니다. 이 기능들이 꼭 필요하다면 [BlueBubbles](https://bluebubbles.app)(SIP 비활성화 필요)를 알아보십시오.
