# Claude Code 플러그인 디렉토리

Claude Code를 위한 고품질 플러그인들이 엄선된 디렉토리입니다.

> **⚠️ 중요:** 플러그인을 설치, 업데이트 또는 사용하기 전에 해당 플러그인을 신뢰할 수 있는지 확인하십시오. Anthropic은 플러그인에 포함된 MCP 서버, 파일 또는 기타 소프트웨어를 제어하지 않으며, 의도한 대로 작동하거나 변경되지 않을 것임을 보장할 수 없습니다. 자세한 내용은 각 플러그인의 홈페이지를 참조하십시오.

## 구조

- **`/plugins`** - Anthropic에서 자체 개발하고 관리하는 내부 플러그인
- **`/external_plugins`** - 파트너 및 커뮤니티에서 제공하는 서드파티 플러그인

## 설치

플러그인은 Claude Code의 플러그인 시스템을 통해 이 마켓플레이스에서 직접 설치할 수 있습니다.

설치하려면 다음 명령어를 실행하십시오: `/plugin install {plugin-name}@claude-plugins-official`

또는 `/plugin > Discover` 메뉴에서 플러그인을 찾아볼 수 있습니다.

## 기여하기

### 내부 플러그인 (Internal Plugins)

내부 플러그인은 Anthropic 팀 멤버들에 의해 개발됩니다. 구현 시 참고하려면 `/plugins/example-plugin`을 참조하십시오.

### 외부 플러그인 (External Plugins)

서드파티 파트너는 마켓플레이스 등록을 위해 플러그인을 제출할 수 있습니다. 외부 플러그인은 승인을 받기 위해 품질 및 보안 표준을 충족해야 합니다. 새 플러그인을 제출하려면 [플러그인 디렉토리 제출 양식](https://clau.de/plugin-directory-submission)을 사용하십시오.

## 플러그인 구조

각 플러그인은 표준 구조를 따릅니다:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # 플러그인 메타데이터 (필수)
├── .mcp.json            # MCP 서버 설정 (선택)
├── commands/            # 슬래시 커맨드 (선택)
├── agents/              # 에이전트 정의 (선택)
├── skills/              # 스킬 정의 (선택)
└── README.md            # 문서
```

## 플러그인 이름은 변경 불가능합니다

마켓플레이스 항목의 `name` 필드는 **변경 불가능한 슬러그(immutable slug)**입니다. 플러그인이 한 번 게시되면 이름을 변경해서는 안 됩니다. 사용자들이 해당 슬러그로 플러그인을 설치했기 때문에, 이름을 바꾸면 `plugin-not-found` 오류와 함께 설치가 손상됩니다.

- UI에서 플러그인이 표시되는 방식을 변경하려면 대신 `displayName`을 설정하거나 업데이트하십시오.
- 이름 변경이 정말 불가피한 경우, 기존 설치가 자동으로 마이그레이션될 수 있도록 `.claude-plugin/marketplace.json`의 최상위 `renames` 맵에 항목을 추가하십시오:

```json
"renames": {
  "old-name": "new-name"
}
```

Claude Code 플러그인 로더는 이 맵을 읽고 사용자의 다음 동기화 시 기존 슬러그를 새 슬러그로 투명하게 재작성합니다.

## 스킬 번들 플러그인 (Skill-bundle plugins)

플러그인의 소스 저장소가 `.claude-plugin/plugin.json` 매니페스트 없이 스킬(`SKILL.md` 파일)을 제공하는 경우, 마켓플레이스 항목에서 `strict: false`와 명시적인 `skills` 배열을 사용하여 스킬을 직접 선언할 수 있습니다.

```json
{
  "name": "example-bundle",
  "description": "Brief description of the bundled skills.",
  "author": { "name": "Author Name" },
  "category": "development",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/example-org/sdk.git",
    "path": "packages/agent-skills",
    "ref": "main",
    "sha": "<commit sha>"
  },
  "strict": false,
  "skills": [
    "./skill-a",
    "./skill-b",
    "./skill-c"
  ],
  "homepage": "https://github.com/example-org/sdk"
}
```

`skills` 내의 각 경로는 `source.path`를 기준으로 하며 `SKILL.md`가 포함된 디렉토리를 가리킵니다. 경로는 단일 레벨보다 더 깊게 지정할 수 있습니다. 예를 들어, `["./libA/skill-1", "./libB/skill-2"]`는 여러 라이브러리 하위 디렉토리에 걸쳐 엄선된 하위 집합을 제공합니다. 각 스킬은 Claude Code에 `<plugin-name>:<skill-name>`으로 등록됩니다.

기본 스키마에 대해서는 마켓플레이스 문서의 [Strict mode](https://code.claude.com/docs/en/plugin-marketplaces)를 참조하십시오.

## 라이선스

링크된 각 플러그인의 관련 LICENSE 파일을 참조하십시오.

## 문서

Claude Code 플러그인 개발에 대한 자세한 내용은 [공식 문서](https://code.claude.com/docs/en/plugins)를 참조하십시오.
