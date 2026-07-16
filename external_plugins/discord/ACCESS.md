# Discord — 접근 및 전송

Discord는 공통 서버에 속한 계정 간에만 DM을 허용합니다. 누가 귀하의 봇에게 DM을 보낼 수 있는지는 봇이 설치된 위치에 따라 다릅니다: 하나의 비공개 서버인 경우 해당 서버의 멤버만 봇에게 도달할 수 있으며, 공개 커뮤니티인 경우 거기 속한 모든 멤버가 DM을 보낼 수 있습니다.

개발자 포털(Bot 탭, 기본적으로 켜져 있음)의 **Public Bot** 토글은 누가 봇을 새로운 서버에 추가할 수 있는지 제어합니다. 이 설정을 끄면 본인 계정으로만 설치할 수 있습니다. 이는 첫 번째 진입 장벽이며, 본 프로세스가 아닌 Discord 자체에서 강제합니다.

전송된 DM의 경우 기본 정책은 **페어링(pairing)**입니다. 알 수 없는 발신자는 답장으로 6자리 코드를 받으며 해당 메시지는 삭제됩니다. 이들을 승인하려면 어시스턴트 세션에서 `/discord:access pair <code>`를 실행하십시오. 승인되면 그들의 메시지가 전달됩니다.

모든 상태는 `~/.claude/channels/discord/access.json`에 저장됩니다. `/discord:access` 스킬 명령어가 이 파일을 편집하며, 서버는 수신 메시지가 올 때마다 파일을 다시 읽으므로 재시작 없이 변경 사항이 적용됩니다. 부팅 시 디스크에 있던 설정으로 고정하려면 `DISCORD_ACCESS_MODE=static`으로 설정하십시오 (정적 모드에서는 런타임 쓰기가 불가능하므로 페어링을 사용할 수 없습니다).

## 요약

| | |
| --- | --- |
| 기본 정책 | `pairing` |
| 발신자 ID | 사용자 스노우플레이크 (숫자 형식, 예: `184695080709324800`) |
| 그룹 키 | 채널 스노우플레이크 — 길드 ID가 아님 |
| 설정 파일 | `~/.claude/channels/discord/access.json` |

## DM 정책

`dmPolicy`는 허용 목록에 없는 발신자로부터 오는 DM을 처리하는 방식을 제어합니다.

| 정책 | 동작 |
| --- | --- |
| `pairing` (기본값) | 페어링 코드로 회신하고 메시지를 삭제합니다. `/discord:access pair <code>`로 승인합니다. |
| `allowlist` | 자동으로 삭제합니다. 회신하지 않습니다. 접근 권한이 필요한 모든 사용자가 이미 목록에 등록되었거나, 페어링 회신으로 인해 스팸이 발생할 우려가 있는 경우에 사용합니다. |
| `disabled` | 허용 목록에 있는 사용자와 길드 채널을 포함하여 모든 것을 차단(drop)합니다. |

```
/discord:access policy allowlist
```

## 사용자 ID

Discord는 사용자를 **스노우플레이크(snowflakes)**(예: `184695080709324800`와 같은 고유한 숫자 ID)로 식별합니다. 사용자 이름은 변경될 수 있지만 스노우플레이크는 불변입니다. 허용 목록에는 스노우플레이크가 저장됩니다.

페어링을 통해 ID는 자동으로 캡처됩니다. 수동으로 사용자를 추가하려면 Discord에서 **사용자 설정 → 고급 → 개발자 모드**를 활성화한 뒤, 임의의 사용자를 우클릭하고 **사용자 ID 복사**를 선택하십시오. 자신의 ID는 왼쪽 아래의 프로필 아바타를 우클릭하여 가져올 수 있습니다.

```
/discord:access allow 184695080709324800
/discord:access remove 184695080709324800
```

## 길드 채널

길드 채널은 기본적으로 비활성화되어 있습니다. 길드가 아닌 **채널** 스노우플레이크를 키로 하여 개별적으로 옵트인하십시오. 스레드는 상위 채널의 옵트인 설정을 그대로 상속받으므로 별도의 등록이 필요하지 않습니다. 채널 ID는 사용자 ID와 동일한 방법으로 찾을 수 있습니다: 개발자 모드를 켜고 채널을 우클릭하여 채널 ID 복사를 클릭합니다.

```
/discord:access group add 846209781206941736
```

기본값인 `requireMention: true` 설정 시, 봇은 @멘션되거나 답장이 달렸을 때만 응답합니다. 채널 내의 모든 메시지를 처리하려면 `--no-mention`을 전달하고, 트리거할 수 있는 멤버를 제한하려면 `--allow id1,id2`를 전달하십시오.

```
/discord:access group add 846209781206941736 --no-mention
/discord:access group add 846209781206941736 --allow 184695080709324800,221773638772129792
/discord:access group rm 846209781206941736
```

## 멘션 감지

`requireMention: true` 상태인 채널에서는 다음 중 하나라도 만족하면 봇이 트리거됩니다:

- 구조화된 `@botname` 멘션 (Discord의 자동 완성을 통해 입력)
- 봇의 최근 메시지에 대한 답장
- `mentionPatterns` 내의 정규식과 일치하는 항목

닉네임 트리거를 위한 정규식 설정 예시:

```
/discord:access set mentionPatterns '["^hey claude\\b", "\\bassistant\\b"]'
```

## 전송

`/discord:access set <key> <value>` 명령어로 아웃바운드 동작을 설정합니다.

**`ackReaction`**은 "확인함" 표시의 일환으로 수신된 메시지에 이모지 반응을 남깁니다. 유니코드 이모지는 바로 사용 가능하며, 커스텀 서버 이모지는 `<:name:id>` 전체 형식이 필요합니다. 이모지 ID는 이모지를 우클릭하고 링크를 복사했을 때 URL 끝부분에 나오는 숫자입니다. 빈 문자열로 설정 시 비활성화됩니다.

```
/discord:access set ackReaction 🔨
/discord:access set ackReaction ""
```

**`replyToMode`**는 청크(chunk) 단위로 나누어 응답할 때의 스레딩을 제어합니다. 긴 응답이 분할될 때, `first`(기본값)는 수신 메시지 아래에 첫 번째 청크만 스레드로 연결합니다. `all`은 모든 청크를 스레드로 연결하며, `off`는 모든 청크를 개별 메시지로 보냅니다.

**`textChunkLimit`**은 분할 기준을 설정합니다. Discord는 2000자를 초과하는 메시지를 차단하므로, 이것이 고정된 최대 한계(hard ceiling)입니다.

**`chunkMode`**는 분할 전략을 선택합니다: `length`는 한계치에서 정확히 자르며, `newline`은 문단 경계를 우선적으로 고려하여 자릅니다.

## 스킬 참조

| 명령어 | 효과 |
| --- | --- |
| `/discord:access` | 정책, 허용 목록, 대기 중인 페어링, 활성화된 채널 등 현재 상태를 출력합니다. |
| `/discord:access pair a4f91c` | 페어링 코드 `a4f91c`를 승인합니다. 발신자를 `allowFrom`에 추가하고 Discord로 확인 메시지를 보냅니다. |
| `/discord:access deny a4f91c` | 대기 중인 코드를 폐기합니다. 발신자에게는 알림이 가지 않습니다. |
| `/discord:access allow 184695080709324800` | 사용자 스노우플레이크를 직접 추가합니다. |
| `/discord:access remove 184695080709324800` | 허용 목록에서 제거합니다. |
| `/discord:access policy allowlist` | `dmPolicy`를 설정합니다. 값: `pairing`, `allowlist`, `disabled`. |
| `/discord:access group add 846209781206941736` | 길드 채널을 활성화합니다. 플래그: `--no-mention`, `--allow id1,id2`. |
| `/discord:access group rm 846209781206941736` | 길드 채널을 비활성화합니다. |
| `/discord:access set ackReaction 🔨` | 설정 키를 설정합니다: `ackReaction`, `replyToMode`, `textChunkLimit`, `chunkMode`, `mentionPatterns`. |

## 설정 파일

`~/.claude/channels/discord/access.json`. 파일이 없으면 빈 목록이 적용된 `pairing` 정책과 동일하게 작동하므로, 첫 DM 시 페어링이 트리거됩니다.

```jsonc
{
  // allowFrom에 없는 발신자로부터 오는 DM 처리 방식.
  "dmPolicy": "pairing",

  // DM이 허용된 사용자 스노우플레이크.
  "allowFrom": ["184695080709324800"],

  // 봇이 활성화된 길드 채널. 빈 객체인 경우 DM 전용.
  "groups": {
    "846209781206941736": {
      // true: @멘션 및 답장에만 응답.
      "requireMention": true,
      // 이 발신자들로만 트리거를 제한합니다. 비어 있으면 모든 멤버 대상 (requireMention 적용).
      "allowFrom": []
    }
  },

  // 멘션으로 간주할 대소문자 구분 없는 정규식 목록.
  "mentionPatterns": ["^hey claude\\b"],

  // 수신 시 반응할 이모지. 빈 문자열로 설정 시 비활성화.
  "ackReaction": "👀",

  // 분할 응답 시 스레드 설정: first | all | off
  "replyToMode": "first",

  // 분할 기준 문자 수. Discord는 2000자 초과 시 전송을 거부함.
  "textChunkLimit": 2000,

  // length = 한계치에서 자름. newline = 문단 경계 선호.
  "chunkMode": "newline"
}
```
