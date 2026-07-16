# MCPB 매니페스트 스키마 (v0.4)

`github.com/anthropics/mcpb/schemas/mcpb-manifest-v0.4.schema.json`을 기준으로 검증되었습니다. 이 스키마는 `additionalProperties: false`를 사용하므로 알 수 없는 키는 거부됩니다. 에디터 검증을 위해 매니페스트에 `"$schema"`를 추가하세요.

---

## 최상위 필드

| 필드 | 필수 여부 | 설명 |
|---|---|---|
| `manifest_version` | ✅ | 스키마 버전. `"0.4"`를 사용합니다. |
| `name` | ✅ | 패키지 식별자 (소문자, 하이픈). 고유해야 합니다. |
| `version` | ✅ | 패키지의 Semver 버전. |
| `description` | ✅ | 한 줄 요약. 마켓플레이스에 표시됩니다. |
| `author` | ✅ | `{name, email?, url?}` |
| `server` | ✅ | 진입점 및 실행 설정. 아래를 참고하세요. |
| `display_name` | | 사용자 친화적인 이름. 기본값은 `name`입니다. |
| `long_description` | | 마크다운 형식의 설명. 상세 페이지에 표시됩니다. |
| `icon` / `icons` | | 번들 내 아이콘 파일 경로. |
| `homepage` / `repository` / `documentation` / `support` | | URL 목록. |
| `license` | | SPDX 식별자. |
| `keywords` | | 검색용 문자열 배열. |
| `user_config` | | 설치 시 설정 필드. 아래를 참고하세요. |
| `compatibility` | | 호스트/플랫폼/런타임 요구사항. 아래를 참고하세요. |
| `tools` / `prompts` | | 마켓플레이스 표시를 위한 선택적 선언형 리스트. 런타임에 강제되지 않습니다. |
| `tools_generated` / `prompts_generated` | | 도구/프롬프트가 동적인 경우 `true` (정적으로 나열할 수 없는 경우). |
| `screenshots` | | 이미지 경로 배열. |
| `localization` | | i18n 번들. |
| `privacy_policies` | | URL 목록. |

---

## `server` — 실행 설정

```json
"server": {
  "type": "node",
  "entry_point": "server/index.js",
  "mcp_config": {
    "command": "node",
    "args": ["${__dirname}/server/index.js"],
    "env": {
      "API_KEY": "${user_config.apiKey}",
      "ROOT_DIR": "${user_config.rootDir}"
    }
  }
}
```

| 필드 | 설명 |
|---|---|
| `type` | `"node"`, `"python"`, 또는 `"binary"` |
| `entry_point` | 메인 파일의 상대 경로. 정보 제공용. |
| `mcp_config.command` | 실행할 실행 파일. |
| `mcp_config.args` | Argv 배열. 번들 상대 경로에는 `${__dirname}`을 사용하세요. |
| `mcp_config.env` | 환경 변수. 사용자 설정을 대체하려면 `${user_config.KEY}`를 사용하세요. |

**대체 변수** (`args` 및 `env`에서만 사용 가능):
- `${__dirname}` — 압축이 풀린 번들 디렉터리의 절대 경로
- `${user_config.<key>}` — 사용자가 설치 시 입력한 값
- `${HOME}` — 사용자의 홈 디렉터리

**자동으로 접두사가 붙는 환경 변수는 없습니다.** 서버가 읽는 환경 변수 이름은 `mcp_config.env`에 선언한 것과 정확히 일치합니다. 예를 들어 `"ROOT_DIR": "${user_config.rootDir}"`로 작성하면, 서버는 `process.env.ROOT_DIR`을 읽습니다.

---

## `user_config` — 설치 시 설정

```json
"user_config": {
  "apiKey": {
    "type": "string",
    "title": "API Key",
    "description": "Your service API key. Stored encrypted.",
    "sensitive": true,
    "required": true
  },
  "rootDir": {
    "type": "directory",
    "title": "Root directory",
    "description": "Directory to expose to the server.",
    "default": "${HOME}/Documents"
  },
  "maxResults": {
    "type": "number",
    "title": "Max results",
    "description": "Maximum items returned per query.",
    "default": 50,
    "min": 1,
    "max": 500
  }
}
```

| 필드 | 필수 여부 | 설명 |
|---|---|---|
| `type` | ✅ | `"string"`, `"number"`, `"boolean"`, `"directory"`, `"file"` |
| `title` | ✅ | 폼 레이블. |
| `description` | ✅ | 입력란 아래의 도움말 텍스트. |
| `default` | | 미리 채워질 값. `${HOME}`을 지원합니다. |
| `required` | | `true`인 경우, 값을 입력할 때까지 설치가 중단됩니다. |
| `sensitive` | | `true`인 경우 OS 키체인에 저장되고 UI에서 마스킹됩니다. **`secret`이 아닙니다.** 해당 필드는 존재하지 않습니다. |
| `multiple` | | `true`인 경우 사용자가 여러 값(배열)을 입력할 수 있습니다. |
| `min` / `max` | | 숫자 범위 제한 (`type: "number"`인 경우). |

`directory` 및 `file` 타입은 OS 네이티브 선택 창을 제공합니다. UX 및 검증을 위해 텍스트 입력 경로보다 이 타입을 권장합니다.

---

## `compatibility` — 설치 제한

```json
"compatibility": {
  "claude_desktop": ">=1.0.0",
  "platforms": ["darwin", "win32", "linux"],
  "runtimes": { "node": ">=20" }
}
```

| 필드 | 설명 |
|---|---|
| `claude_desktop` | Semver 범위. 호스트 버전이 더 낮은 경우 설치가 차단됩니다. |
| `platforms` | OS 허용 목록. `["darwin", "win32", "linux"]` 중 일부. |
| `runtimes` | 필요한 런타임 버전. 예: `{"node": ">=20"}` 또는 `{"python": ">=3.11"}`. |

---

## 최소한의 유효한 매니페스트

```json
{
  "$schema": "https://raw.githubusercontent.com/anthropics/mcpb/main/schemas/mcpb-manifest-v0.4.schema.json",
  "manifest_version": "0.4",
  "name": "hello",
  "version": "0.1.0",
  "description": "Minimal MCPB server.",
  "author": { "name": "Your Name" },
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"]
    }
  }
}
```

---

## MCPB에 없는 것

- **`permissions` 블록 없음.** 매니페스트 수준의 파일 시스템/네트워크/프로세스 범위 제한은 제공하지 않습니다. 서버는 완전한 사용자 권한으로 실행됩니다. 도구 핸들러 내에서 경계를 강제 적용하세요. `local-security.md`를 참고하십시오.
- **자동 환경 변수 접두사 없음.** `MCPB_CONFIG_*` 규칙이 없습니다. `server.mcp_config.env`에서 직접 명시적으로 설정과 환경 변수를 연결해야 합니다.
- **`entry` 필드 없음.** `server` 내부에 `entry_point`가 위치합니다.
- **`minHostVersion` 없음.** `compatibility.claude_desktop`을 사용합니다.
