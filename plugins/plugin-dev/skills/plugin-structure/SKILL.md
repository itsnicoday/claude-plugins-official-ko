---
name: plugin-structure
description: This skill should be used when the user asks to "create a plugin", "scaffold a plugin", "understand plugin structure", "organize plugin components", "set up plugin.json", "use ${CLAUDE_PLUGIN_ROOT}", "add commands/agents/skills/hooks", "configure auto-discovery", or needs guidance on plugin directory layout, manifest configuration, component organization, file naming conventions, or Claude Code plugin architecture best practices.
version: 0.1.0
---

# Claude Code를 위한 플러그인 구조 (Plugin Structure for Claude Code)

## 개요 (Overview)

Claude Code 플러그인은 컴포넌트 자동 감지 기능을 제공하는 표준화된 디렉토리 구조를 따릅니다. 이 구조를 이해하면 잘 구성되어 유지보수가 쉽고, Claude Code와 원활하게 통합되는 플러그인을 개발할 수 있습니다.

**핵심 개념:**
- 자동 감지를 위한 관례적인 디렉토리 레이아웃
- `.claude-plugin/plugin.json`에서의 매니페스트 기반 구성
- 컴포넌트 기반 구성 (명령어, 에이전트, 스킬, 훅)
- `${CLAUDE_PLUGIN_ROOT}`를 사용한 이식 가능한 경로 참조
- 명시적 컴포넌트 로딩 vs. 자동 감지형 컴포넌트 로딩

## 디렉토리 구조 (Directory Structure)

모든 Claude Code 플러그인은 다음 구성 패턴을 따릅니다:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Required: Plugin manifest
├── commands/                 # Slash commands (.md files)
├── agents/                   # Subagent definitions (.md files)
├── skills/                   # Agent skills (subdirectories)
│   └── skill-name/
│       └── SKILL.md         # Required for each skill
├── hooks/
│   └── hooks.json           # Event handler configuration
├── .mcp.json                # MCP server definitions
└── scripts/                 # Helper scripts and utilities
```

**필수 규칙:**

1. **매니페스트 위치**: `plugin.json` 매니페스트 파일은 반드시 `.claude-plugin/` 디렉토리에 있어야 합니다.
2. **컴포넌트 위치**: 모든 컴포넌트 디렉토리(commands, agents, skills, hooks)는 플러그인 루트 수준에 있어야 하며, `.claude-plugin/` 내부에 중첩되어 있으면 안 됩니다.
3. **선택적 컴포넌트**: 플러그인이 실제로 사용하는 컴포넌트용 디렉토리만 생성하십시오.
4. **명명 규칙**: 모든 디렉토리 및 파일 이름은 kebab-case를 사용하십시오.

## 플러그인 매니페스트 (plugin.json) (Plugin Manifest (plugin.json))

매니페스트는 플러그인 메타데이터 및 구성을 정의합니다. 경로는 `.claude-plugin/plugin.json`입니다:

### 필수 필드 (Required Fields)

```json
{
  "name": "plugin-name"
}
```

**이름 요구 사항:**
- kebab-case 형식 사용 (소문자 및 하이픈)
- 설치된 플러그인 중에서 유일해야 함
- 공백이나 특수 문자 사용 불가
- 예시: `code-review-assistant`, `test-runner`, `api-docs`

### 권장 메타데이터 (Recommended Metadata)

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Brief explanation of plugin purpose",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://example.com"
  },
  "homepage": "https://docs.example.com",
  "repository": "https://github.com/user/plugin-name",
  "license": "MIT",
  "keywords": ["testing", "automation", "ci-cd"]
}
```

**버전 형식**: 시맨틱 버저닝 규격 준수 (MAJOR.MINOR.PATCH)
**키워드**: 플러그인 검색 및 카테고리 분류용 태그 목록

### 컴포넌트 경로 구성 (Component Path Configuration)

컴포넌트의 맞춤형 경로를 지정합니다 (기본 디렉토리 추가 정보 제공):

```json
{
  "name": "plugin-name",
  "commands": "./custom-commands",
  "agents": ["./agents", "./specialized-agents"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./.mcp.json"
}
```

**중요**: 맞춤형 경로는 기본 디렉토리를 대체하지 않고 보완합니다. 기본 디렉토리와 맞춤 지정된 경로의 컴포넌트가 모두 로드됩니다.

**경로 규칙:**
- 반드시 플러그인 루트에 대한 상대 경로여야 함
- 반드시 `./`로 시작해야 함
- 절대 경로 사용 불가
- 다중 위치 지정을 위해 배열 형식 지원

## 컴포넌트 구성 방식 (Component Organization)

### 명령어 (Commands)

**위치**: `commands/` 디렉토리
**형식**: YAML 프론트매터가 포함된 마크다운 파일
**자동 감지**: `commands/` 디렉토리 내의 모든 `.md` 파일이 자동으로 로드됨

**예시 구조**:
```
commands/
├── review.md        # /review command
├── test.md          # /test command
└── deploy.md        # /deploy command
```

**파일 형식**:
```markdown
---
name: command-name
description: Command description
---

Command implementation instructions...
```

**사용법**: 명령어는 Claude Code 내에서 기본 슬래시 명령어 형식으로 통합됩니다.

### 에이전트 (Agents)

**위치**: `agents/` 디렉토리
**형식**: YAML 프론트매터가 포함된 마크다운 파일
**자동 감지**: `agents/` 디렉토리 내의 모든 `.md` 파일이 자동으로 로드됨

**예시 구조**:
```
agents/
├── code-reviewer.md
├── test-generator.md
└── refactorer.md
```

**파일 형식**:
```markdown
---
description: Agent role and expertise
capabilities:
  - Specific task 1
  - Specific task 2
---

Detailed agent instructions and knowledge...
```

**사용법**: 사용자가 수동으로 에이전트를 호출할 수 있으며, 또는 Claude Code가 작업 컨텍스트에 따라 자동으로 에이전트를 선택합니다.

### 스킬 (Skills)

**위치**: `skills/` 디렉토리 및 각 스킬별 하위 디렉토리
**형식**: 각 스킬은 자체 디렉토리를 가지며 그 내부에 `SKILL.md` 파일이 포함됨
**자동 감지**: 스킬 하위 디렉토리 내의 모든 `SKILL.md` 파일이 자동으로 로드됨

**예시 구조**:
```
skills/
├── api-testing/
│   ├── SKILL.md
│   ├── scripts/
│   │   └── test-runner.py
│   └── references/
│       └── api-spec.md
└── database-migrations/
    ├── SKILL.md
    └── examples/
        └── migration-template.sql
```

**SKILL.md 형식**:
```markdown
---
name: Skill Name
description: When to use this skill
version: 1.0.0
---

Skill instructions and guidance...
```

**지원 파일**: 스킬은 하위 디렉토리에 스크립트, 참조 자료, 예시 또는 에셋을 포함할 수 있습니다.

**사용법**: Claude Code는 설명 정보와 일치하는 작업 컨텍스트가 주어지면 자율적으로 스킬을 활성화합니다.

### 훅 (Hooks)

**위치**: `hooks/hooks.json` 또는 `plugin.json` 내의 인라인 설정
**형식**: 이벤트 핸들러를 정의하는 JSON 구성
**등록**: 플러그인이 활성화될 때 자동으로 등록됨

**예시 구조**:
```
hooks/
├── hooks.json           # Hook configuration
└── scripts/
    ├── validate.sh      # Hook script
    └── check-style.sh   # Hook script
```

**구성 형식**:
```json
{
  "PreToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{
      "type": "command",
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/validate.sh",
      "timeout": 30
    }]
  }]
}
```

**사용 가능한 이벤트**: PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, PreCompact, Notification

**사용법**: 훅은 Claude Code 이벤트에 맞춰 자동으로 실행됩니다.

### MCP 서버 (MCP Servers)

**위치**: 플러그인 루트의 `.mcp.json` 또는 `plugin.json` 내의 인라인 설정
**형식**: MCP 서버 정의를 위한 JSON 구성
**자동 시작**: 플러그인이 활성화될 때 서버가 자동으로 실행됨

**예시 형식**:
```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**사용법**: MCP 서버는 Claude Code의 도구 시스템과 원활하게 통합됩니다.

## 이식 가능한 경로 참조 (Portable Path References)

### ${CLAUDE_PLUGIN_ROOT}

플러그인 내부의 모든 경로 참조에는 `${CLAUDE_PLUGIN_ROOT}` 환경 변수를 사용하십시오:

```json
{
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/run.sh"
}
```

**사용해야 하는 이유**: 플러그인은 다음에 따라 서로 다른 위치에 설치될 수 있습니다:
- 사용자 설치 방법 (마켓플레이스, 로컬 개발 폴더, npm)
- 운영체제별 관례
- 사용자 정의 환경

**사용 위치**:
- 훅 명령어 경로
- MCP 서버 명령어 인수
- 스크립트 실행 참조 경로
- 리소스 파일 경로

**피해야 할 사항**:
- 하드코딩된 절대 경로 (`/Users/name/plugins/...`)
- 작업 디렉토리 기준 상대 경로 (명령어 내 `./scripts/...`)
- 홈 디렉토리 단축 기호 (`~/plugins/...`)

### 경로 해석 규칙 (Path Resolution Rules)

**매니페스트 JSON 필드 내에서** (훅, MCP 서버):
```json
"command": "${CLAUDE_PLUGIN_ROOT}/scripts/tool.sh"
```

**컴포넌트 파일 내에서** (명령어, 에이전트, 스킬):
```markdown
Reference scripts at: ${CLAUDE_PLUGIN_ROOT}/scripts/helper.py
```

**실행되는 스크립트 내에서**:
```bash
#!/bin/bash
# ${CLAUDE_PLUGIN_ROOT} available as environment variable
source "${CLAUDE_PLUGIN_ROOT}/lib/common.sh"
```

## 파일 명명 규칙 (File Naming Conventions)

### 컴포넌트 파일 (Component Files)

**명령어**: kebab-case `.md` 파일 사용
- `code-review.md` → `/code-review`
- `run-tests.md` → `/run-tests`
- `api-docs.md` → `/api-docs`

**에이전트**: 역할을 묘사하는 kebab-case `.md` 파일 사용
- `test-generator.md`
- `code-reviewer.md`
- `performance-analyzer.md`

**스킬**: kebab-case 디렉토리 이름 사용
- `api-testing/`
- `database-migrations/`
- `error-handling/`

### 지원 파일 (Supporting Files)

**스크립트**: 적절한 확장자를 사용하고 목적을 나타내는 kebab-case 이름 사용
- `validate-input.sh`
- `generate-report.py`
- `process-data.js`

**문서**: kebab-case 마크다운 파일 사용
- `api-reference.md`
- `migration-guide.md`
- `best-practices.md`

**구성 정보**: 표준 이름 사용
- `hooks.json`
- `.mcp.json`
- `plugin.json`

## 자동 감지 메커니즘 (Auto-Discovery Mechanism)

Claude Code는 컴포넌트를 자동으로 발견하고 로드합니다:

1. **플러그인 매니페스트**: 플러그인이 활성화될 때 `.claude-plugin/plugin.json`을 읽습니다.
2. **명령어**: `commands/` 디렉토리에서 `.md` 파일을 스캔합니다.
3. **에이전트**: `agents/` 디렉토리에서 `.md` 파일을 스캔합니다.
4. **스킬**: `skills/` 디렉토리에서 `SKILL.md`를 포함하는 하위 디렉토리를 스캔합니다.
5. **훅**: `hooks/hooks.json` 또는 매니페스트에서 구성을 로드합니다.
6. **MCP 서버**: `.mcp.json` 또는 매니페스트에서 구성을 로드합니다.

**감지 타이밍**:
- 플러그인 설치 시: 컴포넌트가 Claude Code에 등록됩니다.
- 플러그인 활성화 시: 컴포넌트를 사용할 수 있게 됩니다.
- 재시작 불필요: 다음 Claude Code 세션에서 변경 사항이 즉시 적용됩니다.

**오버라이드 규칙**: `plugin.json` 내의 맞춤형 경로는 기본 디렉토리를 대체하지 않고 보완합니다.

## 권장 사항 (Best Practices)

### 구성 방식 (Organization)

1. **논리적 그룹화**: 관련된 컴포넌트들은 함께 배치합니다.
   - 테스트 관련 명령어, 에이전트, 스킬은 한 곳에 둡니다.
   - `scripts/` 내에 목적별로 하위 디렉토리를 만듭니다.

2. **최소한의 매니페스트**: `plugin.json`을 간단하게 유지합니다.
   - 필요한 경우에만 맞춤형 경로를 지정하십시오.
   - 표준적인 레이아웃 구조의 경우 자동 감지 메커니즘에 의존하십시오.
   - 아주 간단한 경우에만 인라인 설정을 사용하십시오.

3. **문서화**: README 파일을 포함합니다.
   - 플러그인 루트: 전체적인 목적 및 사용법 기술
   - 컴포넌트 디렉토리: 세부 사용 가이드
   - 스크립트 디렉토리: 사용법 및 선행 요구 사항

### 명명 규칙 (Naming)

1. **일관성**: 컴포넌트 전반에 걸쳐 일관성 있는 명명 규칙을 적용합니다.
   - 명령어가 `test-runner`이면, 관련 에이전트는 `test-runner-agent`로 명명합니다.
   - 스킬 디렉토리 이름을 목적에 맞춥니다.

2. **명확성**: 목적을 나타내는 직관적인 이름을 사용합니다.
   - 올바름: `api-integration-testing/`, `code-quality-checker.md`
   - 피할 사항: `utils/`, `misc.md`, `temp.sh`

3. **길이**: 간결함과 명확성의 균형을 맞춥니다.
   - 명령어: 2-3 단어 (`review-pr`, `run-ci`)
   - 에이전트: 역할을 명확히 묘사 (`code-reviewer`, `test-generator`)
   - 스킬: 주제에 포커스 (`error-handling`, `api-design`)

### 이식성 (Portability)

1. **항상 ${CLAUDE_PLUGIN_ROOT} 사용**: 경로를 하드코딩하지 마십시오.
2. **다중 시스템 교차 검증**: macOS, Linux, Windows에서 모두 테스트하십시오.
3. **종속성 문서화**: 필요한 외부 도구와 버전을 기재하십시오.
4. **시스템 특정 기능 자제**: 이식 가능한 bash/Python 구조를 사용하십시오.

### 유지보수 (Maintenance)

1. **버전 관리의 일관성**: 릴리스 시 `plugin.json` 내의 버전을 함께 업데이트합니다.
2. **우아한 사용 중단(Deprecation)**: 구성 요소를 삭제하기 전에 구버전 대상임을 명확히 표시합니다.
3. **변경 내역(Breaking Changes) 문서화**: 기존 사용자에게 영향을 미칠 수 있는 변경 사항을 기술합니다.
4. **철저한 테스트**: 변경 사항을 배포하기 전에 모든 컴포넌트가 작동하는지 확인합니다.

## 공통 패턴 (Common Patterns)

### 최소 구성 플러그인 (Minimal Plugin)

의존성이 없는 단일 명령어 구성:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json    # Just name field
└── commands/
    └── hello.md       # Single command
```

### 전체 기능 플러그인 (Full-Featured Plugin)

모든 컴포넌트 유형을 갖춘 완전한 플러그인:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/          # User-facing commands
├── agents/            # Specialized subagents
├── skills/            # Auto-activating skills
├── hooks/             # Event handlers
│   ├── hooks.json
│   └── scripts/
├── .mcp.json          # External integrations
└── scripts/           # Shared utilities
```

### 스킬 중심 플러그인 (Skill-Focused Plugin)

스킬만 제공하는 플러그인:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    ├── skill-one/
    │   └── SKILL.md
    └── skill-two/
        └── SKILL.md
```

## 문제 해결 (Troubleshooting)

**컴포넌트가 로드되지 않음**:
- 파일이 올바른 디렉토리에 있고 올바른 확장자를 가졌는지 확인하십시오.
- YAML 프론트매터 구문을 검증하십시오 (명령어, 에이전트, 스킬).
- 스킬 디렉토리에 `SKILL.md`가 포함되어 있는지 확인하십시오 (`README.md` 등 다른 이름은 불가).
- Claude Code 설정에서 플러그인이 활성화되어 있는지 확인하십시오.

**경로 해석 오류**:
- 하드코딩된 모든 경로를 `${CLAUDE_PLUGIN_ROOT}`로 변경하십시오.
- 매니페스트 내의 경로가 상대 경로이며 `./`로 시작하는지 검증하십시오.
- 참조된 파일이 해당 경로에 실제로 존재하는지 확인하십시오.
- 훅 스크립트 내에서 `echo $CLAUDE_PLUGIN_ROOT`를 출력하여 테스트하십시오.

**자동 감지가 작동하지 않음**:
- 디렉토리가 플러그인 루트에 위치해 있는지 확인하십시오 (`.claude-plugin/` 내부가 아님).
- 파일 이름이 규칙(kebab-case, 올바른 확장자)을 따르는지 검증하십시오.
- 매니페스트에 구성된 맞춤형 경로가 올바른지 확인하십시오.
- 플러그인 구성을 다시 로드하기 위해 Claude Code를 재시작하십시오.

**플러그인 간 충돌**:
- 고유하고 설명적인 컴포넌트 이름을 사용하십시오.
- 필요한 경우 명령어 이름 앞에 플러그인 이름을 접두사로 붙이십시오.
- 플러그인 README 파일에 잠재적인 충돌 문제를 기술하십시오.
- 관련된 일련의 기능에 대해 공통의 명령어 접두사를 고려하십시오.

---

세부 예시 및 고급 패턴에 대해서는 `references/` 및 `examples/` 디렉토리 내의 파일들을 참조하십시오.
