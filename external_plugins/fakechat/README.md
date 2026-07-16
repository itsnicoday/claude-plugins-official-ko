# fakechat

외부 서비스 없이 채널 규약(channel contract)을 테스트하기 위한 간단한 UI입니다. 브라우저를 열고 입력하면 메시지가 Claude Code 세션으로 이동하고 답장이 돌아옵니다.

## 설정

이들은 Claude Code 명령어이므로, 먼저 `claude`를 실행하여 세션을 시작하십시오.

플러그인을 설치합니다:
```
/plugin install fakechat@claude-plugins-official
```

**채널 플래그와 함께 재실행** — 이 설정 없이는 서버가 연결되지 않습니다. 세션을 종료하고 새 세션을 시작하십시오:

```sh
claude --channels plugin:fakechat@claude-plugins-official
```

서버는 시작 시 stderr에 URL을 출력합니다:

```
fakechat: http://localhost:8787
```

해당 URL을 브라우저로 엽니다. 메시지를 입력하면 어시스턴트가 스레드로 답장합니다.

포트를 변경하려면 `FAKECHAT_PORT`를 설정하십시오.

## 도구

| 도구 | 목적 |
| --- | --- |
| `reply` | UI로 메시지를 전송합니다. `text`를 인자로 받으며, 선택적으로 `reply_to`(메시지 ID)와 `files`(절대 경로, 50MB)를 지정할 수 있습니다. 첨부 파일은 텍스트 아래에 `[파일명]` 형식으로 표시됩니다. |
| `edit_message` | 이전에 보낸 메시지를 제자리에서 편집합니다. |

인바운드 이미지/파일은 `~/.claude/channels/fakechat/inbox/` 디렉터리에 저장되며 그 경로가 알림에 포함됩니다. 아웃바운드 파일은 `outbox/`에 복사되어 HTTP를 통해 제공됩니다.

## 실제 채널이 아님

히스토리, 검색, access.json, 스킬이 존재하지 않습니다. 새로 고침할 때마다 단일 브라우저 탭에서 초기화됩니다. 이 도구는 개발용 도구이며 실제 메시징 브릿지가 아닙니다.
