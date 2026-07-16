---
name: build-mcpb
description: 이 스킬은 사용자가 "MCP 서버 패키징", "MCP 번들링", "MCPB 작성", "로컬 MCP 서버 배포", "로컬 MCP 배포"를 원할 때, ".mcpb 파일"에 대해 논의할 때, Node 또는 Python 런타임을 MCP 서버와 함께 번들링하는 것을 언급할 때, 또는 로컬 파일 시스템, 데스크톱 앱 또는 OS와 상호작용하고 사용자가 Node/Python을 설치하지 않고도 설치할 수 있어야 하는 MCP 서버가 필요할 때 사용해야 합니다.
version: 0.1.0
---

# MCPB 빌드하기 (번들형 로컬 MCP 서버)

MCPB는 **런타임과 함께 패키징된** 로컬 MCP 서버입니다. 사용자는 단 하나의 파일만 설치하면 되며, 해당 기기에 Node, Python 또는 별도의 툴체인이 없어도 실행됩니다. 이는 로컬 MCP 서버를 배포하는 공식적으로 권장되는 방법입니다.

> MCPB는 **보조** 배포 경로입니다. Anthropic은 디렉터리 조회를 위해 원격 MCP 서버를 사용하는 것을 권장합니다 — https://claude.com/docs/connectors/building/what-to-build 를 참고하세요.

**서버가 반드시 사용자의 기기에서 실행되어야 할 때 MCPB를 사용하세요** — 로컬 파일 읽기, 데스크톱 앱 제어, localhost 서비스 통신, OS 수준 API 호출 등이 이에 해당합니다. 서버가 클라우드 API만 호출한다면 원격 HTTP 서버를 사용하는 것이 훨씬 낫습니다 (`build-mcp-server` 참고). URL로 처리할 수 있는 작업에 굳이 MCPB 패키징 비용을 들이지 마세요.

---

## MCPB 번들 구성 요소

```
my-server.mcpb              (zip archive)
├── manifest.json           ← identity, entry point, config schema, compatibility
├── server/                 ← your MCP server code
│   ├── index.js
│   └── node_modules/       ← bundled dependencies (or vendored)
└── icon.png
```

호스트는 `manifest.json`을 읽고, `server.mcp_config.command`를 **stdio** MCP 서버로 실행한 뒤 메시지를 파이프합니다. 코드 관점에서는 로컬 stdio 서버와 완전히 동일하며, 유일한 차이점은 패키징 방식입니다.

---

## 매니페스트

```json
{
  "$schema": "https://raw.githubusercontent.com/anthropics/mcpb/main/schemas/mcpb-manifest-v0.4.schema.json",
  "manifest_version": "0.4",
  "name": "local-files",
  "version": "0.1.0",
  "description": "Read, search, and watch files on the local filesystem.",
  "author": { "name": "Your Name" },
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"],
      "env": {
        "ROOT_DIR": "${user_config.rootDir}"
      }
    }
  },
  "user_config": {
    "rootDir": {
      "type": "directory",
      "title": "Root directory",
      "description": "Directory to expose. Defaults to ~/Documents.",
      "default": "${HOME}/Documents",
      "required": true
    }
  },
  "compatibility": {
    "claude_desktop": ">=1.0.0",
    "platforms": ["darwin", "win32", "linux"]
  }
}
```

**`server.type`** — `node`, `python` 또는 `binary`. 정보 제공용이며, 실제 실행은 `mcp_config` 설정에 따라 수행됩니다.

**`server.mcp_config`** — 실행할 실제 명령어/인자/환경 변수. 번들 상대 경로에는 `${__dirname}`을 사용하고, 설치 시 설정을 대체하려면 `${user_config.<key>}`를 사용합니다. **자동 접두사는 없습니다** — 서버가 읽는 환경 변수 이름은 `env`에 입력한 것과 정확히 일치합니다.

**`user_config`** — 호스트 UI에 노출되는 설치 시 설정. `type: "directory"`는 네이티브 폴더 선택 창을 렌더링합니다. `sensitive: true`는 OS 키체인에 저장됩니다. 전체 필드는 `references/manifest-schema.md`를 참고하세요.

---

## 서버 코드: 로컬 stdio와 동일

서버 자체는 표준 stdio MCP 서버입니다. 도구 로직에는 MCPB에만 국한된 부분이 전혀 없습니다.

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { readFile, readdir } from "node:fs/promises";
import { join } from "node:path";
import { homedir } from "node:os";

// ROOT_DIR comes from what you put in manifest's server.mcp_config.env — no auto-prefix
const ROOT = (process.env.ROOT_DIR ?? join(homedir(), "Documents"));

const server = new McpServer({ name: "local-files", version: "0.1.0" });

server.registerTool(
  "list_files",
  {
    description: "List files in a directory under the configured root.",
    inputSchema: { path: z.string().default(".") },
    annotations: { readOnlyHint: true },
  },
  async ({ path }) => {
    const entries = await readdir(join(ROOT, path), { withFileTypes: true });
    const list = entries.map(e => ({ name: e.name, dir: e.isDirectory() }));
    return { content: [{ type: "text", text: JSON.stringify(list, null, 2) }] };
  },
);

server.registerTool(
  "read_file",
  {
    description: "Read a file's contents. Path is relative to the configured root.",
    inputSchema: { path: z.string() },
    annotations: { readOnlyHint: true },
  },
  async ({ path }) => {
    const text = await readFile(join(ROOT, path), "utf8");
    return { content: [{ type: "text", text }] };
  },
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

**샌드박싱은 전적으로 개발자의 몫입니다.** 매니페스트 수준의 샌드박스는 존재하지 않으며, 프로세스는 전체 사용자 권한으로 실행됩니다. 경로를 검증하고, `ROOT` 경로를 벗어나지 않도록 차단하고, 실행할 명령어를 허용 목록에 명시하세요. `references/local-security.md`를 참고하십시오.

환경 변수 설정에서 `ROOT`를 하드코딩하기 전에, 호스트가 `roots/list`(사용자가 승인한 디렉터리를 가져오는 사양 표준 방식)를 지원하는지 확인하세요. 이 패턴에 대해서는 `references/local-security.md`를 참고하십시오.

---

## 빌드 파이프라인

### Node

```bash
npm install
npx esbuild src/index.ts --bundle --platform=node --outfile=server/index.js
# or: copy node_modules wholesale if native deps resist bundling
npx @anthropic-ai/mcpb pack
```

`mcpb pack`은 디렉터리를 압축하고 `manifest.json`을 스키마에 맞게 검증합니다.

### Python

```bash
pip install -t server/vendor -r requirements.txt
npx @anthropic-ai/mcpb pack
```

종속성을 하위 디렉터리에 벤더링하고 진입 스크립트에서 `sys.path` 앞에 추가하세요. 네이티브 확장 모듈(numpy 등)은 각 대상 플랫폼에 맞게 빌드해야 하므로, 가능하면 네이티브 종속성은 피하는 것이 좋습니다.

---

## MCPB에는 샌드박스가 없습니다 — 보안은 여러분의 책임입니다

모바일 앱 스토어와 달리, MCPB는 권한을 강제하지 않습니다. 매니페스트에는 `permissions` 블록이 없으며, 서버는 완전한 사용자 권한으로 실행됩니다. `references/local-security.md`는 선택이 아닌 필수 권장 문서입니다. 플랫폼 수준에서 어떠한 제한도 가하지 않으므로, 모든 경로를 검증하고 모든 실행 명령을 허용 목록에 등록해야 합니다.

매니페스트를 통한 파일 시스템/네트워크 범위 설정을 기대했다면, 그런 기능은 존재하지 않습니다. 도구 핸들러에서 직접 구현해야 합니다.

서버가 클라우드 API만 호출한다면 작업을 멈추세요. 그것은 MCPB의 탈을 쓴 원격 서버일 뿐입니다. 사용자가 로컬에서 실행함으로써 얻는 이점이 없으며, 불필요하게 로컬 보안 책임을 짊어지게 됩니다.

---

## MCPB + UI 위젯

MCPB 서버는 원격 MCP 앱과 마찬가지로 UI 리소스를 제공할 수 있으며, 위젯 메커니즘은 전송 방식에 구애받지 않습니다. 실제 디스크를 탐색하는 로컬 파일 선택 창, 네이티브 앱을 제어하는 다이얼로그 등이 가능합니다.

위젯 작성법은 **`build-mcp-app`** 스킬에서 설명하고 있으며, 작동 방식은 동일합니다. 유일한 차이점은 서버가 실행되는 위치입니다.

---

## 테스트

```bash
# 대화형 매니페스트 생성 (최초 1회)
npx @anthropic-ai/mcpb init

# stdio를 통해 서버를 직접 실행하고 인스펙터로 조작하기
npx @modelcontextprotocol/inspector node server/index.js

# 매니페스트를 스키마에 맞게 검증한 후 압축
npx @anthropic-ai/mcpb validate
npx @anthropic-ai/mcpb pack

# 배포용 서명
npx @anthropic-ai/mcpb sign dist/local-files.mcpb

# 설치: .mcpb 파일을 Claude Desktop으로 드래그 앤 드롭
```

배포하기 전에 개발 툴체인이 설치되지 **않은** 머신에서 테스트해 보세요. MCPB에서 발생하는 "내 컴퓨터에서는 되는데" 식의 오류는 대부분 종속성이 실제로 번들링되지 않아 발생합니다.

---

## 참고 파일

- `references/manifest-schema.md` — 전체 `manifest.json` 필드 레퍼런스
- `references/local-security.md` — 경로 탐색, 샌드박싱, 최소 권한
