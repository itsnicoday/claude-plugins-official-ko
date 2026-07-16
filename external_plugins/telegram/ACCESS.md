# Telegram — 접근 및 전송

Telegram 봇은 공개적으로 주소가 지정될 수 있습니다. 봇의 사용자 이름을 찾은 사람은 누구나 DM을 보낼 수 있으며, 제어 장치가 없다면 그 메시지들은 곧바로 귀하의 어시스턴트 세션으로 흘러들어가게 됩니다. 여기에 기술된 접근 모델을 통해 누구의 메시지를 통과시킬지 결정할 수 있습니다.

기본적으로 알 수 없는 발신자로부터 오는 DM은 **페어링(pairing)**을 트리거합니다. 봇은 6자리의 페어링 코드로 답장하고 메시지를 삭제합니다. 어시스턴트 세션에서 `/telegram:access pair <code>`를 실행하여 승인하면, 그 이후부터 그들의 메시지가 전달됩니다.

모든 상태는 `~/.claude/channels/telegram/access.json`에 저장됩니다. `/telegram:access` 스킬 명령어가 이 파일을 편집하며, 서버는 수신 메시지가 올 때마다 파일을 다시 읽으므로 재시작 없이 변경 사항이 적용됩니다. 부팅 시 디스크에 있던 설정으로 고정하려면 `TELEGRAM_ACCESS_MODE=static`으로 설정하십시오 (정적 모드에서는 런타임 쓰기가 불가능하므로 페어링을 사용할 수 없습니다).

## 요약

| | |
| --- | --- |
| 기본 정책 | `pairing` |
| 발신자 ID | 숫자 형식 사용자 ID (예: `412587349`) |
| 그룹 키 | 슈퍼그룹 ID (음수, `-100…` 접두사) |
| `ackReaction` 제한 | 고정된 허용 목록만 지원. 허용되지 않은 이모지는 자동으로 무시됨 |
| 설정 파일 | `~/.claude/channels/telegram/access.json` |

## DM 정책

`dmPolicy`는 허용 목록에 없는 발신자로부터 오는 DM을 처리하는 방식을 제어합니다.

| 정책 | 동작 |
| --- | --- |
| `pairing` (기본값) | 페어링 코드로 회신하고 메시지를 삭제합니다. `/telegram:access pair <code>`로 승인합니다. |
| `allowlist` | 자동으로 삭제합니다. 회신하지 않습니다. 봇 사용자 이름 유추가 가능하여 페어링 회신으로 인해 스팸이 발생할 우려가 있는 경우에 사용합니다. |
| `disabled` | 허용 목록에 있는 사용자와 그룹을 포함하여 모든 것을 차단(drop)합니다. |

```
/telegram:access policy allowlist
```

## 사용자 ID

Telegram은 사용자를 `412587349`와 같은 **숫자 ID**로 식별합니다. 사용자 이름은 선택사항이며 변경될 수 있지만, 숫자 ID는 영구적입니다. 허용 목록에는 숫자 ID가 저장됩니다.

페어링을 통해 ID는 자동으로 캡처됩니다. 수동으로 ID를 확인하려면, 해당 사용자에게 [@userinfobot](https://t.me/userinfobot)으로 메시지를 보내게 하십시오. 봇이 그들의 ID를 응답해 줍니다. 사용자가 보낸 메시지를 @userinfobot으로 전달하는 방법으로도 확인할 수 있습니다.

```
/telegram:access allow 412587349
/telegram:access remove 412587349
```

## 그룹

그룹은 기본적으로 비활성화되어 있습니다. 개별적으로 옵트인하십시오.

```
/telegram:access group add -1001654782309
```

슈퍼그룹 ID는 `-100` 접두사가 붙은 음수입니다 (예: `-1001654782309`). 이는 Telegram UI에는 표시되지 않습니다. ID를 찾으려면 [@RawDataBot](https://t.me/RawDataBot)을 그룹에 임시로 추가하여 채팅 ID가 포함된 JSON 정보를 받아보거나, 귀하의 봇을 그룹에 추가한 후 `/telegram:access`를 실행해 최근 메시지가 차단된 그룹 목록을 확인하십시오.

기본값인 `requireMention: true` 설정 시, 봇은 @멘션되거나 답장이 달렸을 때만 응답합니다. 모든 메시지를 처리하려면 `--no-mention`을 전달하고, 트리거할 수 있는 멤버를 제한하려면 `--allow id1,id2`를 전달하십시오.

```
/telegram:access group add -1001654782309 --no-mention
/telegram:access group add -1001654782309 --allow 412587349,628194073
/telegram:access group rm -1001654782309
```

**개인정보 보호 모드(Privacy mode).** Telegram 봇은 기본적으로 서버 측 개인정보 보호 모드가 활성화되어 있어 그룹 메시지가 코드에 도달하기 전에 필터링됩니다 (오직 @멘션과 답장만 전달됨). 이는 기본값인 `requireMention: true`와 일치하므로 보통은 드러나지 않습니다. `--no-mention`을 사용하려면 개인정보 보호 모드도 비활성화해야 합니다: [@BotFather](https://t.me/BotFather)에게 메시지를 보내고, `/setprivacy`를 전송한 뒤 봇을 선택하고 **Disable**을 선택하십시오. 이 단계를 수행하지 않으면 로컬 설정과 관계없이 Telegram이 메시지를 전달하지 않습니다.

## 멘션 감지

`requireMention: true` 상태인 그룹에서는 다음 중 하나라도 만족하면 봇이 트리거됩니다:

- 구조화된 `@botusername` 멘션
- 봇의 메시지에 대한 답장
- `mentionPatterns` 내의 정규식과 일치하는 항목

```
/telegram:access set mentionPatterns '["^hey claude\\b", "\\bassistant\\b"]'
```

## 전송

`/telegram:access set <key> <value>` 명령어로 아웃바운드 동작을 설정합니다.

**`ackReaction`**은 메시지 수신 시 반응을 보냅니다. Telegram은 오직 **고정된 허용 목록**에 있는 반응 이모지만 허용하며, 그 외의 것은 자동으로 무시됩니다. 전체 Bot API 목록은 다음과 같습니다:

> 👍 👎 ❤ 🔥 🥰 👏 😁 🤔 🤯 😱 🤬 😢 🎉 🤩 🤮 💩 🙏 👌 🕊 🤡 🥱 🥴 😍 🐳 ❤‍🔥 🌚 🌭 💯 🤣 ⚡ 🍌 🏆 💔 🤨 😐 🍓 🍾 💋 🖕 😈 😴 😭 🤓 👻 👨‍💻 👀 🎃 🙈 😇 😨 🤝 ✍ 🤗 🫡 🎅 🎄 ☃ 💅 🤪 🗿 🆒 💘 🙉 🦄 😘 💊 🙊 😎 👾 🤷‍♂ 🤷 🤷‍♀ 😡

```
/telegram:access set ackReaction 👀
/telegram:access set ackReaction ""
```

**`replyToMode`**는 청크(chunk) 단위로 나누어 응답할 때의 스레딩을 제어합니다. 긴 응답이 분할될 때, `first`(기본값)는 수신 메시지 아래에 첫 번째 청크만 스레드로 연결합니다. `all`은 모든 청크를 스레드로 연결하며, `off`는 모든 청크를 개별 메시지로 보냅니다.

**`textChunkLimit`**은 분할 기준을 설정합니다. Telegram은 4096자를 초과하는 메시지를 차단합니다.

**`chunkMode`**는 분할 전략을 선택합니다: `length`는 한계치에서 정확히 자르며, `newline`은 문단 경계를 우선적으로 고려하여 자릅니다.

## 스킬 참조

| 명령어 | 효과 |
| --- | --- |
| `/telegram:access` | 정책, 허용 목록, 대기 중인 페어링, 활성화된 그룹 등 현재 상태를 출력합니다. |
| `/telegram:access pair a4f91c` | 페어링 코드 `a4f91c`를 승인합니다. 발신자를 `allowFrom`에 추가하고 Telegram으로 확인 메시지를 보냅니다. |
| `/telegram:access deny a4f91c` | 대기 중인 코드를 폐기합니다. 발신자에게는 알림이 가지 않습니다. |
| `/telegram:access allow 412587349` | 사용자 ID를 직접 추가합니다. |
| `/telegram:access remove 412587349` | 허용 목록에서 제거합니다. |
| `/telegram:access policy allowlist` | `dmPolicy`를 설정합니다. 값: `pairing`, `allowlist`, `disabled`. |
| `/telegram:access group add -1001654782309` | 그룹을 활성화합니다. 플래그: `--no-mention` (개인정보 보호 모드 비활성화도 필요함), `--allow id1,id2`. |
| `/telegram:access group rm -1001654782309` | 그룹을 비활성화합니다. |
| `/telegram:access set ackReaction 👀` | 설정 키를 설정합니다: `ackReaction`, `replyToMode`, `textChunkLimit`, `chunkMode`, `mentionPatterns`. |

## 설정 파일

`~/.claude/channels/telegram/access.json`. 파일이 없으면 빈 목록이 적용된 `pairing` 정책과 동일하게 작동하므로, 첫 DM 시 페어링이 트리거됩니다.

```jsonc
{
  // allowFrom에 없는 발신자로부터 오는 DM 처리 방식.
  "dmPolicy": "pairing",

  // DM이 허용된 숫자 사용자 ID.
  "allowFrom": ["412587349"],

  // 봇이 활성화된 그룹. 빈 객체인 경우 DM 전용.
  "groups": {
    "-1001654782309": {
      // true: @멘션 및 답장에만 응답.
      // false 설정 시 BotFather를 통한 개인정보 보호 모드 비활성화도 필요함.
      "requireMention": true,
      // 이 발신자들로만 트리거를 제한합니다. 비어 있으면 모든 멤버 대상 (requireMention 적용).
      "allowFrom": []
    }
  },

  // 멘션으로 간주할 대소문자 구분 없는 정규식 목록.
  "mentionPatterns": ["^hey claude\\b"],

  // Telegram의 고정된 허용 목록에 속한 이모지. 빈 문자열로 설정 시 비활성화.
  "ackReaction": "👀",

  // 분할 응답 시 스레드 설정: first | all | off
  "replyToMode": "first",

  // 분할 기준 문자 수. Telegram은 4096 초과 시 전송을 거부함.
  "textChunkLimit": 4096,

  // length = 한계치에서 자름. newline = 문단 경계 선호.
  "chunkMode": "newline"
}
```
