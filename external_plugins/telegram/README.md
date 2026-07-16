# Telegram

MCP 서버를 통해 Telegram 봇을 Claude Code에 연결합니다.

MCP 서버는 Telegram에 봇으로 로그인하고 메시지 답장, 반응 추가, 메시지 편집을 위한 도구를 Claude에 제공합니다. 봇에게 메시지를 보내면, 서버는 그 메시지를 귀하의 Claude Code 세션으로 전달합니다.

## 사전 요구 사항

- [Bun](https://bun.sh) — MCP 서버는 Bun에서 실행됩니다. `curl -fsSL https://bun.sh/install | bash` 명령어로 설치하십시오.

## 빠른 설정
> 단일 사용자 DM 봇을 위한 기본 페어링 흐름입니다. 그룹 및 다중 사용자 설정에 대해서는 [ACCESS.md](./ACCESS.md)를 참조하십시오.

**1. Create a bot with BotFather.**

Telegram에서 [@BotFather](https://t.me/BotFather)와 대화를 시작하고 `/newbot`을 전송합니다. BotFather는 두 가지 정보를 요청합니다:

- **Name** — 채팅 헤더에 표시될 이름 (공백 포함 가능)
- **Username** — `bot`으로 끝나는 고유한 핸들 이름 (예: `my_assistant_bot`). 이것이 봇의 링크가 됩니다: `t.me/my_assistant_bot`.

BotFather는 `123456789:AAHfiqksKZ8...` 형태의 토큰으로 응답합니다. 이 토큰 전체를 복사하십시오. 앞에 있는 숫자와 콜론을 모두 포함해야 합니다.

**2. 플러그인 설치**

이들은 Claude Code 명령어이므로, 먼저 `claude`를 실행하여 세션을 시작하십시오.

플러그인을 설치합니다:
```
/plugin install telegram@claude-plugins-official
/reload-plugins
```

**3. 서버에 토큰 제공**

```
/telegram:configure 123456789:AAHfiqksKZ8...
```

`~/.claude/channels/telegram/.env` 파일에 `TELEGRAM_BOT_TOKEN=...`을 기록합니다. 이 파일을 수동으로 작성하거나 쉘 환경 변수로 설정할 수도 있으며, 쉘 환경 변수가 우선합니다.

> 단일 시스템에서 여러 봇을 실행하려면(서로 다른 토큰, 개별 허용 목록 사용), 인스턴스마다 `TELEGRAM_STATE_DIR`이 다른 디렉터리를 가리키도록 설정하십시오.

**4. 채널 플래그와 함께 재실행**

이 설정 없이는 서버가 연결되지 않습니다. 세션을 종료하고 새 세션을 시작하십시오:

```sh
claude --channels plugin:telegram@claude-plugins-official
```

**5. 페어링**

이전 단계에서 Claude Code가 실행 중인 상태에서 Telegram으로 봇에게 DM을 보내면, 봇이 6자리의 페어링 코드로 답장합니다. 봇이 응답하지 않는 경우 세션이 `--channels`와 함께 실행 중인지 확인하십시오. Claude Code 세션에서 다음을 실행합니다:

```
/telegram:access pair <code>
```

이제 다음 DM이 어시스턴트에게 도달합니다.

> Discord와 달리 서버 초대 단계가 없습니다. Telegram 봇은 즉시 DM을 수신할 수 있습니다. 페어링을 통해 사용자 ID 조회가 자동으로 수행되므로 숫자 ID를 직접 다룰 필요가 없습니다.

**6. 잠금 설정**

페어링은 ID를 캡처하기 위한 것입니다. 연결이 완료되면 외부인이 페어링 코드 답장을 받지 못하도록 `allowlist`로 전환하십시오. Claude에게 요청하거나 `/telegram:access policy allowlist`를 직접 실행하십시오.

## 액세스 제어

DM 정책, 그룹, 언급(mention) 감지, 전달 설정, 스킬 명령어, 그리고 `access.json` 스키마에 대해서는 **[ACCESS.md](./ACCESS.md)**를 참조하십시오.

빠른 참조: ID는 **숫자 형식의 사용자 ID**입니다 (본인 ID는 [@userinfobot](https://t.me/userinfobot)을 통해 확인할 수 있습니다). 기본 정책은 `pairing`입니다. `ackReaction`은 Telegram의 고정된 이모지 허용 목록만 지원합니다.

## 어시스턴트에게 노출되는 도구

| 도구 | 목적 |
| --- | --- |
| `reply` | 채팅으로 메시지를 전송합니다. `chat_id` + `text`를 인자로 받으며, 선택적으로 네이티브 스레드 처리를 위한 `reply_to`(메시지 ID)와 첨부 파일을 위한 `files`(절대 경로)를 지정할 수 있습니다. 이미지(`.jpg`/`.png`/`.gif`/`.webp`)는 인라인 미리보기가 포함된 사진으로 전송되고, 그 외의 파일 형식은 문서로 전송됩니다. 첨부 파일은 개당 최대 50MB로 제한됩니다. 긴 텍스트는 자동으로 분할되며, 첨부 파일은 텍스트가 전송된 후 별도의 메시지로 전송됩니다. 전송된 메시지 ID(들)를 반환합니다. |
| `react` | ID를 지정하여 메시지에 이모지 반응을 추가합니다. **Telegram의 고정된 허용 목록**에 속한 이모지(👍 👎 ❤ 🔥 👀 등)만 사용할 수 있습니다. |
| `edit_message` | 이전에 봇이 전송한 메시지를 편집합니다. "작업 중..." → 결과물 형태로 진행 상황을 업데이트할 때 유용합니다. 봇이 직접 보낸 메시지에만 작동합니다. |

인바운드 메시지는 자동으로 입력 중 표시기(typing indicator)를 트리거합니다. 어시스턴트가 답변을 생성하는 동안 Telegram에는 "봇이 입력 중..."으로 표시됩니다.

## 사진

인바운드 사진은 `~/.claude/channels/telegram/inbox/` 디렉터리로 다운로드되며, 어시스턴트가 이를 `Read` 할 수 있도록 로컬 경로가 `<channel>` 알림에 포함됩니다. Telegram은 사진을 압축합니다. 만약 원본 파일이 필요하다면 문서 형식으로 전송하십시오 (길게 누르기 → 파일로 보내기).

## 히스토리 및 검색 없음

Telegram의 봇 API는 메시지 히스토리와 검색 기능을 **제공하지 않습니다**. 봇은 메시지가 도달하는 시점에만 이를 볼 수 있으며, `fetch_messages` 도구는 존재하지 않습니다. 어시스턴트가 이전 컨텍스트를 필요로 하는 경우, 사용자에게 붙여넣기나 요약을 요청할 것입니다.

이것은 또한 과거 메시지에 대한 `download_attachment` 도구가 존재하지 않음을 의미합니다. 나중에 사진을 가져올 수 있는 방법이 없기 때문에, 사진은 도착 즉시 다운로드됩니다.
