# Example Plugin

Claude Code 확장 옵션을 보여주는 종합적인 예시 플러그인입니다.

## Structure

```
example-plugin/
├── .claude-plugin/
│   └── plugin.json            # 플러그인 메타데이터
├── .mcp.json                  # MCP 서버 설정
├── skills/
│   ├── example-skill/
│   │   └── SKILL.md           # 모델이 호출하는 스킬 (콘텍스트 기반 안내)
│   └── example-command/
│       └── SKILL.md           # 사용자가 호출하는 스킬 (슬래시 명령어)
└── commands/
    └── example-command.md     # 레거시 슬래시 명령어 형식 (아래 참고 사항 확인)
```

## Extension Options

### Skills (`skills/`)

Skills는 모델이 호출하는 기능과 사용자가 호출하는 슬래시 명령어 모두에 대해 권장되는 형식입니다. 하위 디렉터리에 `SKILL.md`를 생성하세요:

**모델 호출형 스킬 (Model-invoked skill)** (작업 콘텍스트에 의해 활성화됨):

```yaml
---
name: skill-name
description: Trigger conditions for this skill
version: 1.0.0
---
```

**사용자 호출형 스킬 (User-invoked skill)** (슬래시 명령어 — `/skill-name`):

```yaml
---
name: skill-name
description: Short description for /help
argument-hint: <arg1> [optional-arg]
allowed-tools: [Read, Glob, Grep]
---
```

### Commands (`commands/`) — 레거시

> **참고:** `commands/*.md` 레이아웃은 레거시 형식입니다. `skills/<name>/SKILL.md`와 동일하게 로드되며, 유일한 차이점은 파일 레이아웃입니다. 새로운 플러그인의 경우 `skills/` 디렉터리 형식을 권장합니다. 이 플러그인은 레거시 레이아웃에 대한 참조용으로 `commands/example-command.md`를 유지합니다.

### MCP Servers (`.mcp.json`)

Model Context Protocol을 통해 외부 도구 연동을 설정합니다:

```json
{
  "server-name": {
    "type": "http",
    "url": "https://mcp.example.com/api"
  }
}
```

## Usage

- `/example-command [args]` - 예시 슬래시 명령어 실행
- 예시 스킬은 작업 콘텍스트에 따라 활성화됩니다.
- 예시 MCP는 작업 콘텍스트에 따라 활성화됩니다.
