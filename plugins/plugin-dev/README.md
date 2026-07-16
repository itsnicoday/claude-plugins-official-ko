# Plugin Development Toolkit

훅, MCP 통합, 플러그인 구조 및 마켓플레이스 게시 등에 대한 전문적인 가이드를 제공하여 Claude Code 플러그인을 개발할 수 있도록 돕는 포괄적인 툴킷입니다.

## 개요

plugin-dev 툴킷은 고품질의 Claude Code 플러그인을 빌드할 수 있도록 7가지의 전문적인 스킬을 제공합니다:

1. **hook-development** - 고급 훅 API 및 이벤트 기반 자동화
2. **mcp-integration** - Model Context Protocol 서버 통합
3. **plugin-structure** - 플러그인 조직화 및 매니페스트 설정
4. **plugin-settings** - .claude/plugin-name.local.md 파일을 활용한 설정 패턴
5. **command-development** - 프론트매터 및 인자(argument)가 포함된 슬래시 명령어 생성
6. **agent-development** - AI 지원 생성을 활용한 자율 에이전트 생성
7. **skill-development** - 점진적 공개(progressive disclosure) 및 확실한 트리거가 포함된 스킬 생성

각 스킬은 점진적 공개(progressive disclosure)를 적용한 베스트 프랙티스를 따릅니다: 간결한 핵심 문서, 상세한 참조 자료, 작동하는 예시 코드 및 유틸리티 스크립트를 포함합니다.

## 가이드 제공 워크플로우 명령어

### /plugin-dev:create-plugin

feature-dev 워크플로우와 유사하게 처음부터 플러그인을 생성할 수 있는 종합적인 엔드투엔드 워크플로우 명령어입니다.

**8단계 프로세스:**
1. **발견 (Discovery)** - 플러그인 목적 및 요구 사항 파악
2. **구성 요소 계획 (Component Planning)** - 필요한 스킬, 명령어, 에이전트, 훅, MCP 결정
3. **상세 설계 (Detailed Design)** - 각 구성 요소 상세 정의 및 모호성 해결
4. **구조 생성 (Structure Creation)** - 디렉터리 및 매니페스트 설정
5. **구성 요소 구현 (Component Implementation)** - AI 지원 에이전트를 사용하여 각 구성 요소 생성
6. **검증 (Validation)** - plugin-validator 및 구성 요소별 검사 실행
7. **테스트 (Testing)** - Claude Code에서 플러그인이 정상 작동하는지 검증
8. **문서화 (Documentation)** - README 마무리 및 배포 준비

**주요 기능:**
- 각 단계마다 명확히 하기 위한 질문 제시
- 관련 스킬 자동 로드
- AI 지원 에이전트 생성을 위해 agent-creator 사용
- 검증 유틸리티(validate-agent.sh, validate-hook-schema.sh 등) 실행
- plugin-dev의 검증된 패턴 준수
- 테스트 및 검증 단계 안내

**사용법:**
```bash
/plugin-dev:create-plugin [optional description]

# 예시:
/plugin-dev:create-plugin
/plugin-dev:create-plugin A plugin for managing database migrations
```

개념 구상부터 완성까지 구조화된 고품질 플러그인 개발을 위해 이 워크플로우를 사용하세요.

## 스킬 목록

### 1. hook-development

**트리거 문구:** "create a hook", "add a PreToolUse hook", "validate tool use", "implement prompt-based hooks", "${CLAUDE_PLUGIN_ROOT}", "block dangerous commands"

**다루는 내용:**
- LLM 의사 결정을 활용하는 프롬프트 기반 훅 (권장)
- 결정론적(deterministic) 검증을 위한 명령어 훅
- 모든 훅 이벤트: PreToolUse, PostToolUse, Stop, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, PreCompact, Notification
- 훅 출력 형식 및 JSON 스키마
- 보안 베스트 프랙티스 및 입력 검증
- 이식성 있는 경로 지정을 위한 ${CLAUDE_PLUGIN_ROOT}

**제공 리소스:**
- 핵심 SKILL.md (1,619 단어)
- 3개의 예시 훅 스크립트 (validate-write, validate-bash, load-context)
- 3개의 참조 문서: patterns, migration, advanced techniques
- 3개의 유틸리티 스크립트: validate-hook-schema.sh, test-hook.sh, hook-linter.sh

**사용 시점:** 이벤트 기반 자동화 구축, 작업 검증 또는 플러그인의 정책 강제 적용 시

### 2. mcp-integration

**트리거 문구:** "add MCP server", "integrate MCP", "configure .mcp.json", "Model Context Protocol", "stdio/SSE/HTTP server", "connect external service"

**다루는 내용:**
- MCP 서버 설정 (.mcp.json vs plugin.json)
- 모든 서버 타입: stdio (로컬), SSE (호스팅/OAuth), HTTP (REST), WebSocket (실시간)
- 환경 변수 확장 (${CLAUDE_PLUGIN_ROOT}, 사용자 정의 변수)
- 명령어/에이전트 내에서의 MCP 도구 명명 및 사용
- 인증 패턴: OAuth, 토큰, 환경 변수
- 통합 패턴 및 성능 최적화

**제공 리소스:**
- 핵심 SKILL.md (1,666 단어)
- 3개의 예시 설정 (stdio, SSE, HTTP)
- 3개의 참조 문서: server-types (~3,200 단어), authentication (~2,800 단어), tool-usage (~2,600 단어)

**사용 시점:** 외부 서비스, API, 데이터베이스 또는 도구를 플러그인에 통합할 때

### 3. plugin-structure

**트리거 문구:** "plugin structure", "plugin.json manifest", "auto-discovery", "component organization", "plugin directory layout"

**다루는 내용:**
- 표준 플러그인 디렉터리 구조 및 자동 감지(auto-discovery)
- plugin.json 매니페스트 형식 및 모든 필드
- 구성 요소 조직화 (commands, agents, skills, hooks)
- 전반적인 ${CLAUDE_PLUGIN_ROOT} 활용
- 파일 명명 규칙 및 베스트 프랙티스
- 최소, 표준 및 고급 플러그인 패턴

**제공 리소스:**
- 핵심 SKILL.md (1,619 단어)
- 3개의 예시 구조 (minimal, standard, advanced)
- 2개의 참조 문서: component-patterns, manifest-reference

**사용 시점:** 새 플러그인을 시작하거나, 구성 요소를 정리하거나, 플러그인 매니페스트를 설정할 때

### 4. plugin-settings

**트리거 문구:** "plugin settings", "store plugin configuration", ".local.md files", "plugin state files", "read YAML frontmatter", "per-project plugin settings"

**다루는 내용:**
- 설정을 위한 .claude/plugin-name.local.md 패턴
- YAML 프론트매터 + 마크다운 본문 구조
- bash 스크립트용 파싱 기법 (sed, awk, grep 패턴)
- 일시적으로 활성화되는 훅 (플래그 파일 및 빠른 종료)
- multi-agent-swarm 및 ralph-loop 플러그인의 실제 적용 예시
- 원자적(atomic) 파일 업데이트 및 검증
- gitignore 및 라이프사이클 관리

**제공 리소스:**
- 핵심 SKILL.md (1,623 단어)
- 3개의 예시 (read-settings 훅, create-settings 명령어, 템플릿)
- 2개의 참조 문서: parsing-techniques, real-world-examples
- 2개의 유틸리티 스크립트: validate-settings.sh, parse-frontmatter.sh

**사용 시점:** 플러그인을 설정 가능하게 만들거나, 프로젝트별 상태를 저장하거나, 사용자 선호도를 구현할 때

### 5. command-development

**트리거 문구:** "create a slash command", "add a command", "command frontmatter", "define command arguments", "organize commands"

**다루는 내용:**
- 슬래시 명령어 구조 및 마크다운 형식
- YAML 프론트매터 필드 (description, argument-hint, allowed-tools)
- 동적 인자 및 파일 참조
- 콘텍스트 확보를 위한 Bash 실행
- 명령어 조직화 및 네임스페이스
- 명령어 개발을 위한 베스트 프랙티스

**제공 리소스:**
- 핵심 SKILL.md (1,535 단어)
- 예시 및 참조 문서
- 명령어 조직화 패턴

**사용 시점:** 슬래시 명령어 생성, 명령어 인자 정의 또는 플러그인 명령어 구성 시

### 6. agent-development

**트리거 문구:** "create an agent", "add an agent", "write a subagent", "agent frontmatter", "when to use description", "agent examples", "autonomous agent"

**다루는 내용:**
- 에이전트 파일 구조 (YAML 프론트매터 + 시스템 프롬프트)
- 모든 프론트매터 필드 (name, description, model, color, tools)
- 확실한 트리거 구동을 위해 <example> 블록이 포함된 설명 형식
- 시스템 프롬프트 디자인 패턴 (분석, 생성, 검증, 오케스트레이션)
- Claude Code의 검증된 프롬프트를 사용하는 AI 지원 에이전트 생성
- 검증 규칙 및 베스트 프랙티스
- 즉시 프로덕션 적용 가능한 완전한 에이전트 예시

**제공 리소스:**
- 핵심 SKILL.md (1,438 단어)
- 2개의 예시: agent-creation-prompt (AI 지원 워크플로우), complete-agent-examples (4개의 완전한 에이전트)
- 3개의 참조 문서: agent-creation-system-prompt (Claude Code 내부에서 추출), system-prompt-design (~4,000 단어), triggering-examples (~2,500 단어)
- 1개의 유틸리티 스크립트: validate-agent.sh

**사용 시점:** 자율 에이전트 생성, 에이전트 행동 정의 또는 AI 지원 에이전트 생성 구현 시

### 7. skill-development

**트리거 문구:** "create a skill", "add a skill to plugin", "write a new skill", "improve skill description", "organize skill content"

**다루는 내용:**
- 스킬 구조 (YAML 프론트매터가 포함된 SKILL.md)
- 점진적 공개 원칙 (metadata → SKILL.md → resources)
- 특정 문구를 사용한 확실한 트리거 설명
- 작성 스타일 (명령조/인칭 없는 형태, 3인칭 시점)
- 번들 리소스 구성 (references/, examples/, scripts/)
- 스킬 생성 워크플로우
- Claude Code 플러그인에 맞게 조정된 skill-creator 방법론 기반

**제공 리소스:**
- 핵심 SKILL.md (1,232 단어)
- 참조 자료: skill-creator 방법론, plugin-dev 패턴
- 예시: 템플릿으로 활용할 수 있는 plugin-dev 자체 스킬

**사용 시점:** 플러그인용 새 스킬 생성 또는 기존 스킬 품질 개선 시

## 설치

claude-code-marketplace에서 설치:

```bash
/plugin install plugin-dev@claude-code-marketplace
```

또는 개발 목적으로 직접 사용하는 경우:

```bash
cc --plugin-dir /path/to/plugin-dev
```

## 빠른 시작

### 첫 번째 플러그인 생성하기

1. **플러그인 구조 계획:**
   - 질문 예시: "명령어와 MCP 통합이 포함된 플러그인의 가장 적합한 디렉터리 구조는 무엇인가요?"
   - plugin-structure 스킬이 안내를 제공합니다.

2. **MCP 통합 추가 (필요시):**
   - 질문 예시: "데이터베이스 액세스를 위한 MCP 서버를 어떻게 추가하나요?"
   - mcp-integration 스킬이 예시와 패턴을 제공합니다.

3. **훅 구현 (필요시):**
   - 질문 예시: "파일 쓰기를 검증하는 PreToolUse 훅을 생성해줘"
   - hook-development 스킬이 작동하는 예시와 유틸리티를 제공합니다.

## 개발 워크플로우

plugin-dev 툴킷은 플러그인 개발 생명주기 전체를 지원합니다:

```
┌─────────────────────┐
│  Design Structure   │  → plugin-structure 스킬
│  (manifest, layout) │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  Add Components     │
│  (commands, agents, │  → 모든 스킬이 지침 제공
│   skills, hooks)    │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  Integrate Services │  → mcp-integration 스킬
│  (MCP servers)      │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  Add Automation     │  → hook-development 스킬
│  (hooks, validation)│     + 유틸리티 스크립트
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  Test & Validate    │  → hook-development 유틸리티
│                     │     validate-hook-schema.sh
└──────────┬──────────┘     test-hook.sh
           │                 hook-linter.sh
```

## 기능적 특징

### 점진적 공개 (Progressive Disclosure)

각 스킬은 3단계 공개 시스템을 사용합니다:
1. **메타데이터** (항상 로드됨): 확실한 트리거를 동반한 간결한 설명
2. **핵심 SKILL.md** (트리거 시 로드됨): 필수 API 참조 문서 (~1,500-2,000 단어)
3. **참조 자료/예시** (필요에 따라 로드됨): 상세 가이드, 패턴 및 실제 작동하는 코드

이를 통해 필요할 때 깊이 있는 지식을 제공하면서도 Claude Code의 콘텍스트 크기를 작고 효율적으로 유지합니다.

### 유틸리티 스크립트

hook-development 스킬에는 프로덕션 수준의 유틸리티가 포함되어 있습니다:

```bash
# hooks.json 구조 검증
./validate-hook-schema.sh hooks/hooks.json

# 배포 전에 훅 테스트
./test-hook.sh my-hook.sh test-input.json

# 베스트 프랙티스 준수 여부를 위해 훅 스크립트 린트
./hook-linter.sh my-hook.sh
```

### 작동하는 예시

모든 스킬은 작동하는 예시를 제공합니다:
- **hook-development**: 3개의 완전한 훅 스크립트 (bash, write 검증, context 로딩)
- **mcp-integration**: 3개의 서버 설정 (stdio, SSE, HTTP)
- **plugin-structure**: 3개의 플러그인 레이아웃 (minimal, standard, advanced)
- **plugin-settings**: 3개의 예시 (read-settings 훅, create-settings 명령어, 템플릿)
- **command-development**: 10개의 완전한 명령어 예시 (review, test, deploy, docs 등)

## 문서화 표준

모든 스킬은 일관된 표준을 따릅니다:
- 3인칭 시점 설명 ("This skill should be used when...")
- 안정적인 로드를 위한 확실한 트리거 문구
- 본문 전반에 걸친 명령조/인칭 없는 형태 사용
- 공식 Claude Code 문서를 기반으로 작성
- 베스트 프랙티스를 적용한 보안 우선 접근 방식

## 전체 규모

- **핵심 스킬**: 7개의 SKILL.md 파일에 걸쳐 약 11,065 단어
- **참조 문서**: 약 10,000단어 이상의 세부 가이드
- **예시**: 12개 이상의 작동 예시 (훅 스크립트, MCP 설정, 플러그인 레이아웃, 설정 파일)
- **유틸리티**: 6개의 즉시 사용 가능한 검증/테스트/파싱 스크립트

## 주요 사용 사례

### 데이터베이스 플러그인 구축

```
1. "MCP 통합이 포함된 플러그인의 구조는 어떻게 되나요?"
   → plugin-structure 스킬이 레이아웃 제공

2. "PostgreSQL용 stdio MCP 서버를 어떻게 설정하나요?"
   → mcp-integration 스킬이 설정 예시 제시

3. "연결이 올바르게 닫히도록 Stop 훅 추가"
   → hook-development 스킬이 패턴 제공
```

### 검증 플러그인 생성

```
1. "보안을 위해 모든 파일 쓰기를 검증하는 훅 생성"
   → hook-development 스킬과 예시 확인

2. "배포 전에 훅 테스트"
   → validate-hook-schema.sh 및 test-hook.sh 사용

3. "훅 및 설정 파일 구조화"
   → plugin-structure 스킬이 베스트 프랙티스 제시
```

### 외부 서비스 통합

```
1. "OAuth가 포함된 Asana MCP 서버 추가"
   → mcp-integration 스킬이 SSE 서버를 다룸

2. "내 명령어에서 Asana 도구 사용"
   → mcp-integration of tool-usage 참조 문서 확인

3. "명령어와 MCP를 포함하여 플러그인 구조화"
   → plugin-structure 스킬이 패턴 제공
```

## 베스트 프랙티스

모든 스킬이 다음 사항을 강조합니다:

✅ **보안 우선 (Security First)**
- 훅 내의 입력값 검증
- MCP 서버에 HTTPS/WSS 적용
- 자격 증명 관리에 환경 변수 사용
- 최소 권한 원칙 적용

✅ **이식성 (Portability)**
- 모든 위치에 ${CLAUDE_PLUGIN_ROOT} 사용
- 상대 경로만 사용
- 환경 변수 치환 사용

✅ **테스트 (Testing)**
- 배포 전 설정 검증
- 샘플 입력을 활용한 훅 테스트
- 디버그 모드 사용 (`claude --debug`)

✅ **문서화 (Documentation)**
- 명확한 README 파일
- 사용된 환경 변수의 문서화
- 사용 예시

## 기여 안내

이 플러그인은 claude-code-marketplace의 일부입니다. 개선에 기여하려면 다음 단계를 따르세요:

1. 마켓플레이스 리포지토리를 포크합니다.
2. `plugin-dev/` 아래에서 수정을 진행합니다.
3. `cc --plugin-dir` 명령어로 로컬에서 테스트합니다.
4. marketplace-publishing 가이드라인에 따라 PR을 생성합니다.

## 버전

0.1.0 - 7가지 종합 스킬과 3가지 검증 에이전트가 포함된 최초 릴리즈

## 작성자

Daisy Hollman (daisy@anthropic.com)

## 라이선스

MIT 라이선스 - 자세한 내용은 리포지토리를 참고하세요.

---

**참고:** 이 툴킷은 고품질의 플러그인을 구축할 수 있도록 설계되었습니다. 관련된 질문을 하면 스킬이 자동으로 로드되어, 필요한 시점에 정확히 전문적인 가이드를 제공합니다.
