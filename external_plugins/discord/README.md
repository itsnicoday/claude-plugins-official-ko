# Discord

MCP 서버를 통해 Discord 봇을 Claude Code에 연결합니다.

봇이 메시지를 수신하면, MCP 서버는 이를 Claude로 전달하고 메시지 답장, 반응 추가, 메시지 편집을 위한 도구를 제공합니다.

## 사전 요구 사항

- [Bun](https://bun.sh) — MCP 서버는 Bun에서 실행됩니다. `curl -fsSL https://bun.sh/install | bash` 명령어로 설치하십시오.

## 빠른 설정
> 단일 사용자 DM 봇을 위한 기본 페어링 흐름입니다. 그룹 및 다중 사용자 설정에 대해서는 [ACCESS.md](./ACCESS.md)를 참조하십시오.

**1. Discord 애플리케이션 및 봇 생성**

[Discord Developer Portal](https://discord.com/developers/applications)로 이동하여 **New Application**을 클릭하고 이름을 지정합니다.

사이드바에서 **Bot**으로 이동하여 봇의 사용자 이름을 지정합니다.

**Privileged Gateway Intents**로 스크롤을 내려 **Message Content Intent**를 활성화합니다. 활성화하지 않으면 봇이 빈 내용의 메시지를 수신하게 됩니다.

**2. 봇 토큰 생성**

**Bot** 페이지에서 **Token** 섹션으로 스크롤을 올려 **Reset Token**을 누릅니다. 토큰을 복사합니다. 이 토큰은 한 번만 표시되므로 5단계를 위해 보관해 두십시오.

**3. 서버에 봇 초대**

봇과 같은 서버에 참여하고 있지 않다면 Discord에서 봇에게 DM을 보낼 수 없습니다.

**OAuth2** → **URL Generator**로 이동합니다. `bot` 범위를 선택합니다. **Bot Permissions** 아래에서 다음 권한을 활성화합니다:

- View Channels
- Send Messages
- Send Messages in Threads
- Read Message History
- Attach Files
- Add Reactions

통합 유형: **Guild Install**. **Generated URL**을 복사하고 이를 열어 귀하가 참여 중인 서버에 봇을 추가합니다.

> DM 전용으로 사용할 경우 기술적으로는 권한이 필요하지 않지만, 지금 권한을 활성화해 두면 나중에 길드 채널을 사용하고 싶을 때 다시 설정하러 오는 번거로움을 줄일 수 있습니다.

**4. 플러그인 설치**

이들은 Claude Code 명령어이므로, 먼저 `claude`를 실행하여 세션을 시작하십시오.

플러그인을 설치합니다:
```
/plugin install discord@claude-plugins-official
/reload-plugins
```

**5. 서버에 토큰 제공**

```
/discord:configure MTIz...
```

`~/.claude/channels/discord/.env` 파일에 `DISCORD_BOT_TOKEN=...`을 기록합니다. 이 파일을 수동으로 작성하거나 쉘 환경 변수로 설정할 수도 있으며, 쉘 환경 변수가 우선합니다.

> 단일 시스템에서 여러 봇을 실행하려면(서로 다른 토큰, 개별 허용 목록 사용), 인스턴스마다 `DISCORD_STATE_DIR`이 다른 디렉터리를 가리키도록 설정하십시오.

**6. 채널 플래그와 함께 재실행**

이 설정 없이는 서버가 연결되지 않습니다. 세션을 종료하고 새 세션을 시작하십시오:

```sh
claude --channels plugin:discord@claude-plugins-official
```

**7. 페어링**

이전 단계에서 Claude Code가 실행 중인 상태에서 Discord로 봇에게 DM을 보내면, 봇이 페어링 코드로 답장합니다. 봇이 응답하지 않는 경우 세션이 `--channels`와 함께 실행 중인지 확인하십시오. Claude Code 세션에서 다음을 실행합니다:

```
/discord:access pair <code>
```

이제 다음 DM이 어시스턴트에게 도달합니다.

**8. 잠금 설정**

페어링은 ID를 캡처하기 위한 것입니다. 연결이 완료되면 외부인이 페어링 코드 답장을 받지 못하도록 `allowlist`로 전환하십시오. Claude에게 요청하거나 `/discord:access policy allowlist`를 직접 실행하십시오.

## 액세스 제어

DM 정책, 길드 채널, 언급(mention) 감지, 전달 설정, 스킬 명령어, 그리고 `access.json` 스키마에 대해서는 **[ACCESS.md](./ACCESS.md)**를 참조하십시오.

빠른 참조: ID는 Discord **snowflakes**(숫자 — 개발자 모드를 활성화한 뒤 우클릭 → ID 복사)입니다. 기본 정책은 `pairing`입니다. 길드 채널은 채널 ID별로 옵트인(opt-in)해야 합니다.

## 어시스턴트에게 노출되는 도구

| 도구 | 목적 |
| --- | --- |
| `reply` | 채널로 메시지를 전송합니다. `chat_id` + `text`를 인자로 받으며, 선택적으로 네이티브 스레드 처리를 위한 `reply_to`(메시지 ID)와 첨부 파일을 위한 `files`(절대 경로)를 지정할 수 있습니다. 첨부 파일은 최대 10개, 개당 25MB로 제한됩니다. 긴 메시지는 자동으로 분할(chunk)되며, 첨부 파일은 첫 번째 분할 메시지에 포함됩니다. 전송된 메시지 ID(들)를 반환합니다. |
| `react` | ID를 지정하여 임의의 메시지에 이모지 반응을 추가합니다. 유니코드 이모지는 직접 작동하며, 커스텀 이모지는 `<:name:id>` 형식을 사용해야 합니다. |
| `edit_message` | 이전에 봇이 전송한 메시지를 편집합니다. "작업 중..." → 결과물 형태로 진행 상황을 업데이트할 때 유용합니다. 봇이 직접 보낸 메시지에만 작동합니다. |
| `fetch_messages` | 채널에서 최근 히스토리를 가져옵니다(오래된 순서대로). 호출당 최대 100개로 제한됩니다. 각 줄에는 모델이 `reply_to`로 참조할 수 있도록 메시지 ID가 포함되며, 첨부 파일이 있는 메시지는 `+Natt`로 표시됩니다. Discord의 검색 API는 봇에 노출되지 않으므로, 이것이 과거 메시지를 확인하는 유일한 방법입니다. |
| `download_attachment` | 특정 메시지 ID의 모든 첨부 파일을 `~/.claude/channels/discord/inbox/` 디렉터리로 다운로드합니다. 파일 경로 및 메타데이터를 반환합니다. `fetch_messages`에서 해당 메시지에 첨부 파일이 있는 것으로 나타날 때 사용하십시오. |

인바운드 메시지는 자동으로 입력 중 표시기(typing indicator)를 트리거합니다. 어시스턴트가 답변을 생성하는 동안 Discord에는 "봇이 입력 중..."으로 표시됩니다.

## 첨부 파일

첨부 파일은 자동으로 다운로드되지 **않습니다**. `<channel>` 알림은 각 첨부 파일의 이름, 유형, 크기를 나열하며, 어시스턴트가 실제로 파일이 필요할 때 `download_attachment(chat_id, message_id)`를 호출합니다. 다운로드 파일은 `~/.claude/channels/discord/inbox/`에 저장됩니다.

`fetch_messages`를 통해 찾은 이전 메시지의 첨부 파일도 동일한 경로에 저장됩니다 (첨부 파일이 있는 메시지는 `+Natt`로 표시됨).
