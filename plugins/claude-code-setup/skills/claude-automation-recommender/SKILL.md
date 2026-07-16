---
name: claude-automation-recommender
description: Analyze a codebase and recommend Claude Code automations (hooks, subagents, skills, plugins, MCP servers). Use when user asks for automation recommendations, wants to optimize their Claude Code setup, mentions improving Claude Code workflows, asks how to first set up Claude Code for a project, or wants to know what Claude Code features they should use.
tools: Read, Glob, Grep, Bash
---

# Claude 자동화 추천 도구 (Claude Automation Recommender)

코드베이스의 패턴을 분석하여 다양한 확장 옵션(extensibility options) 중에서 최적의 맞춤형 Claude Code 자동화 설정을 추천합니다.

**이 스킬은 읽기 전용(read-only)입니다.** 코드베이스를 분석하고 추천 보고서를 출력할 뿐, 어떠한 파일도 직접 생성하거나 수정하지 않습니다. 사용자는 제공된 추천 사항을 직접 적용하거나, Claude에게 개별적으로 적용을 도와달라고 요청해야 합니다.

## 출력 지침 (Output Guidelines)

- **각 카테고리당 1~2개 추천**: 사용자가 복잡하게 느끼지 않도록 카테고리별로 가장 유용한 핵심 자동화 도구를 1~2개만 골라 추천하십시오.
- **특정 종류를 명시해 요청한 경우**: 사용자가 특정 유형의 자동화만 요구한 경우, 해당 유형에 집중하여 더 많은 대안(3~5개)을 추천하십시오.
- **참조 목록 외 정보 탐색**: 참조용 파일에 일반적인 패턴들이 기재되어 있지만, 웹 검색을 활용하여 해당 코드베이스의 도구, 프레임워크, 라이브러리에 특화된 새로운 추천 사항을 찾아 제안하십시오.
- **추가 요청 가능 여부 명시**: 보고서 마지막에 특정 카테고리에 대해 더 많은 추천을 요청할 수 있다는 안내 문구를 포함시키십시오.

## 자동화 유형 개요 (Automation Types Overview)

| 유형 | 적합한 용도 |
|------|----------|
| **Hooks** | 도구 이벤트(저장 시 포맷팅, lint, 수정 차단 등) 발생 시 자동으로 실행되는 액션 |
| **Subagents** | 병렬로 실행되며 특정 도구 권한을 갖는 전문적인 리뷰어 및 분석 에이전트 |
| **Skills** | 워크플로우나 반복 작업이 패키지화된 전문 지식 (Claude가 자동으로 감지해 실행하거나 사용자가 `/skill-name`으로 직접 실행) |
| **Plugins** | 설치하여 바로 사용할 수 있는 스킬들의 모음 |
| **MCP Servers** | 외부 도구 및 서비스 연동 (데이터베이스, API, 브라우저, 외부 문서 등) |

## 워크플로우 (Workflow)

### 1단계: 코드베이스 분석 (Phase 1: Codebase Analysis)

프로젝트 컨텍스트를 파악합니다:

```bash
# 프로젝트 유형 및 사용 도구 감지
ls -la package.json pyproject.toml Cargo.toml go.mod pom.xml 2>/dev/null
cat package.json 2>/dev/null | head -50

# MCP 서버 추천을 위한 종속성 확인
cat package.json 2>/dev/null | grep -E '"(react|vue|angular|next|express|fastapi|django|prisma|supabase|convex|stripe)"'

# 기존 Claude Code 설정 존재 여부 확인
ls -la .claude/ CLAUDE.md 2>/dev/null

# 프로젝트 디렉토리 구조 분석
ls -la src/ app/ lib/ tests/ components/ pages/ api/ 2>/dev/null
```

**수집해야 할 주요 지표 (Key Indicators to Capture):**

| 카테고리 | 확인 사항 | 추천 판단에 반영되는 요소 |
|----------|------------------|----------------------------|
| 언어/프레임워크 | package.json, pyproject.toml, import 패턴 | Hooks, MCP servers |
| 프론트엔드 스택 | React, Vue, Angular, Next.js | Playwright MCP, frontend skills |
| 백엔드 스택 | Express, FastAPI, Django | API 문서화 도구 |
| 데이터베이스 | Prisma, Supabase, Convex, raw SQL | 데이터베이스 / 백엔드 MCP servers |
| 외부 API | Stripe, OpenAI, AWS SDK | 문서 조회용 context7 MCP |
| 테스트 도구 | Jest, pytest, Playwright 설정 파일 | 테스팅 관련 hooks, subagents |
| CI/CD | GitHub Actions, CircleCI | GitHub MCP server |
| 이슈 트래커 | Linear, Jira 참조 흔적 | 이슈 트래커 MCP |
| 문서 패턴 | OpenAPI, JSDoc, docstrings | 문서화 관련 skills |

### 2단계: 추천 사항 생성 (Phase 2: Generate Recommendations)

분석 결과를 바탕으로 각 카테고리별 추천 사항을 도출합니다:

#### A. MCP 서버 추천 (MCP Server Recommendations)

자세한 패턴은 [references/mcp-servers.md](references/mcp-servers.md)를 참고하십시오.

| 코드베이스 신호 | 추천 MCP 서버 |
|-----------------|------------------------|
| 대중적인 라이브러리 사용 (React, Express 등) | **context7** - 실시간 라이브러리 문서 조회 |
| UI 테스트가 필요한 프론트엔드 프로젝트 | **Playwright** - 브라우저 자동화 및 테스트 실행 |
| Supabase 사용 | **Supabase MCP** - 데이터베이스 직접 조작 |
| Convex 사용 | **Convex MCP** - 실시간 배포 환경 분석, 쿼리/뮤테이션 실행, 환경 변수 및 로그 관리 |
| PostgreSQL/MySQL 데이터베이스 사용 | **Database MCP** - 쿼리 실행 및 스케마 조회 도구 |
| GitHub 저장소 | **GitHub MCP** - 이슈, PR, Actions 연동 |
| 이슈 관리에 Linear 사용 | **Linear MCP** - 이슈 관리 |
| AWS 인프라 구축 환경 | **AWS MCP** - 클라우드 리소스 관리 |
| Slack 사용 워크스페이스 | **Slack MCP** - 팀 알림 전송 |
| 세션 간의 메모리/컨텍스트 유지 | **Memory MCP** - 대화 세션 간 기억 공유 |
| Sentry 에러 추적 도구 사용 | **Sentry MCP** - 에러 상세 원인 분석 |
| Docker 컨테이너 사용 | **Docker MCP** - 컨테이너 관리 |

#### B. 스킬 추천 (Skills Recommendations)

자세한 설정 방법은 [references/skills-reference.md](references/skills-reference.md)를 참고하십시오.

스킬은 `.claude/skills/<name>/SKILL.md` 경로에 작성하며, 플러그인을 통해서도 설치할 수 있습니다:

| 코드베이스 신호 | 추천 스킬 | 플러그인 |
|-----------------|-------|--------|
| 플러그인 제작 중 | skill-development | plugin-dev |
| Git 커밋 작업 | commit | commit-commands |
| React/Vue/Angular 사용 | frontend-design | frontend-design |
| 자동화 규칙 필요 | writing-rules | hookify |
| 기능 개발 계획 수립 | feature-dev | feature-dev |

**제작을 추천할 만한 커스텀 스킬 (Custom skills to create):**

| 코드베이스 신호 | 제안할 커스텀 스킬 | 호출 권한 |
|-----------------|-----------------|------------|
| API 라우트 존재 | **api-doc** (OpenAPI 템플릿 포함) | 사용자 & Claude 모두 |
| 데이터베이스 프로젝트 | **create-migration** (검증 스크립트 포함) | 사용자만 실행 가능 |
| 테스트 코드 작성 시 | **gen-test** (테스트 예제 포함) | 사용자만 실행 가능 |
| 컴포넌트 라이브러리 개발 | **new-component** (기본 템플릿 포함) | 사용자만 실행 가능 |
| PR 검토 워크플로우 | **pr-check** (체크리스트 파일 포함) | 사용자만 실행 가능 |
| 릴리스 작업 | **release-notes** (git 컨텍스트 연계) | 사용자만 실행 가능 |
| 코드 스타일 강제 | **project-conventions** | Claude 전용 (백그라운드 지식) |
| 개발 환경 구축 | **setup-dev** (사전 요구사항 스크립트 포함) | 사용자만 실행 가능 |

#### C. 훅 추천 (Hooks Recommendations)

상세 설정법은 [references/hooks-patterns.md](references/hooks-patterns.md)를 참고하십시오.

| 코드베이스 신호 | 추천 훅 설정 |
|-----------------|------------------|
| Prettier 설정 감지 | PostToolUse: 파일 수정 시 자동 포맷팅 |
| ESLint/Ruff 설정 감지 | PostToolUse: 파일 수정 시 자동 Lint 검사 및 수정 |
| TypeScript 프로젝트 | PostToolUse: 파일 수정 시 자동 타입 검사 |
| 테스트 디렉토리 존재 | PostToolUse: 관련 테스트 자동 실행 |
| `.env` 파일 존재 | PreToolUse: `.env` 파일 직접 수정 차단 |
| 락(Lock) 파일 존재 | PreToolUse: 패키지 락 파일 직접 수정 차단 |
| 보안에 민감한 코드 영역 | PreToolUse: require confirmation |

#### D. 서브에이전트 추천 (Subagent Recommendations)

상세 템플릿 정보는 [references/subagent-templates.md](references/subagent-templates.md)를 참고하십시오.

| 코드베이스 신호 | 추천 서브에이전트 |
|-----------------|---------------------|
| 대규모 코드베이스 (>500개 파일) | **code-reviewer** - 백그라운드 코드 리뷰 수행 |
| 인증 및 결제 관련 코드 존재 | **security-reviewer** - 보안 취약점 감사 |
| API 기반 프로젝트 | **api-documenter** - OpenAPI 명세 자동화 |
| 성능이 매우 중요함 | **performance-analyzer** - 병목 구간 분석 |
| 프론트엔드 비중 높음 | **ui-reviewer** - 웹 접근성 및 UX 검토 |
| 테스트 보완 필요 | **test-writer** - 테스트 코드 일괄 생성 |

#### E. 플러그인 추천 (Plugin Recommendations)

제공되는 전체 플러그인 목록은 [references/plugins-reference.md](references/plugins-reference.md)를 참고하십시오.

| 코드베이스 신호 | 추천 플러그인 |
|-----------------|-------------------|
| 생산성 향상 전반 | **anthropic-agent-skills** - 핵심 스킬 묶음 |
| 문서 변환 워크플로우 | docx, xlsx, pdf 등 문서 처리 스킬 설치 |
| 프론트엔드 개발 | **frontend-design** 플러그인 |
| AI 도구 제작 프로젝트 | MCP 서버 개발을 위한 **mcp-builder** |

### 3단계: 추천 보고서 출력 (Phase 3: Output Recommendations Report)

추천 사항을 깔끔하게 정리하여 출력합니다. **카테고리별로 가장 유용한 1~2개의 핵심 사항만 포함**하고, 해당하지 않는 카테고리는 생략하십시오.

```markdown
## Claude Code 자동화 추천 보고서

코드베이스를 분석하여 가장 적합한 자동화 방안을 도출했습니다. 카테고리별로 우선순위가 높은 1~2개의 추천 사항을 제안합니다:

### 코드베이스 개요
- **언어/런타임**: [detected language/runtime]
- **프레임워크**: [detected framework]
- **핵심 라이브러리**: [relevant libraries detected]

---

### 🔌 MCP 서버

#### context7
**추천 이유**: [specific reason based on detected libraries]
**설치 방법**: `claude mcp add context7`

---

### 🎯 스킬 (Skills)

#### [스킬 이름]
**추천 이유**: [specific reason]
**생성 경로**: `.claude/skills/[name]/SKILL.md`
**호출 형태**: 사용자 전용(User-only) / 모두 가능(Both) / Claude 전용(Claude-only)
**기타 제공**: [plugin-name] 플러그인을 통해서도 설치 가능 (해당하는 경우)
```yaml
---
name: [skill-name]
description: [what it does]
disable-model-invocation: true  # 사용자 전용 스킬인 경우 설정
---
```

---

### ⚡ 훅 (Hooks)

#### [훅 이름]
**추천 이유**: [specific reason based on detected config]
**설정 경로**: `.claude/settings.json`

---

### 🤖 서브에이전트 (Subagents)

#### [에이전트 이름]
**추천 이유**: [specific reason based on codebase patterns]
**생성 경로**: `.claude/agents/[name].md`

---

**더 많은 자동화 설정을 원하시나요?** 특정 카테고리에 대해 더 많은 정보를 요구할 수 있습니다. (예: "MCP 서버 추천을 더 보여줘" 또는 "적용하면 좋은 다른 훅은 없을까?")

**설정을 직접 적용하는 데 도움이 필요하신가요?** 요청하시면 위의 추천 설정 파일 작성을 직접 도와드리겠습니다.
```

## 의사결정 프레임워크 (Decision Framework)

### MCP 서버 추천 기준
- 외부 서비스 연동이 필요한 경우 (데이터베이스, 제3자 API)
- 사용 중인 라이브러리/SDK의 공식 문서를 실시간으로 참조해야 할 때
- 브라우저 자동화나 테스트 실행이 필요할 때
- 협업 도구 연동이 필요할 때 (GitHub, Linear, Slack)
- 클라우드 인프라 자원을 직접 제어해야 할 때

### 스킬 추천 기준
- 문서 생성 작업이 잦을 때 (docx, xlsx, pptx, pdf — 플러그인으로도 제공됨)
- 반복적으로 사용되는 프롬프트나 정형화된 작업 흐름이 있을 때
- 인자(arguments)를 전달해야 하는 프로젝트 내 고유 작업이 있을 때
- 파일 템플릿을 생성하거나 검증 스크립트를 동반해야 할 때 (스킬 폴더 내에 부속 파일 번들링 가능)
- `/skill-name`과 같이 명령어로 빠르게 실행하고자 할 때
- 외부 실행 환경과 격리하여 병렬로 안전하게 실행하고 싶을 때 (`context: fork`)

**호출 권한 설정 제어:**
- `disable-model-invocation: true` — 사용자 전용 (배포, 커밋, 메시지 전송 등 외부 영향이 있는 작업용)
- `user-invocable: false` — Claude 전용 (백그라운드 코딩 지침 및 규칙 제공용)
- 기본값 (둘 다 생략) — 사용자 및 Claude 모두 호출 가능

### 훅 추천 기준
- 파일 수정 후 매번 반복적으로 수행하는 작업 (코드 포맷팅, lint 수정)
- 코드 안전 가이드 (민감 파일의 수정 차단)
- 유효성 사전 검사 (테스트 자동 실행, 타입 검사)

### 서브에이전트 추천 기준
- 보안, 성능 분석 등 특정 영역의 고도화된 전문 지식이 요구될 때
- 여러 파일을 병렬로 리뷰하는 워크플로우를 태우고 싶을 때
- 백그라운드에서 코드 품질 검사를 비동기적으로 수행할 때

### 플러그인 추천 기준
- 서로 연관된 여러 개의 스킬이 묶음으로 필요할 때
- 이미 잘 패키징된 자동화 도구 세트를 즉시 적용하고 싶을 때
- 팀 전체에 공통 워크플로우를 표준화하여 보급하고자 할 때

---

## 설정 팁 (Configuration Tips)

### MCP 서버 설정

**팀 공유**: `.mcp.json` 파일을 저장소에 커밋하여 모든 팀원이 동일한 MCP 서버 환경을 갖추도록 제안하십시오.

**디버깅**: 설정 상의 문제를 확인할 수 있도록 `--mcp-debug` 플래그 사용을 안내하십시오.

**필수 요구사항 확인:**
- GitHub CLI (`gh`) - 로컬 환경에 설치되어 있어야 원활한 GitHub 도구 실행 가능
- Puppeteer/Playwright CLI - 브라우저 제어형 MCP 서버 실행 시 사전 설치 필요

### 헤드리스 모드 (Headless Mode - CI 및 자동화 파이프라인용)

CI 파이프라인이나 자동화 스크립트 실행을 위해 헤드리스 Claude 구동법을 제안하십시오:

```bash
# Pre-commit 훅 구성 예시
claude -p "fix lint errors in src/" --allowedTools Edit,Write

# 구조화된 출력(JSON)을 받는 CI 파이프라인 구성 예시
claude -p "<prompt>" --output-format stream-json | your_command
```

### 훅 실행 권한 설정

`.claude/settings.json` 내에 허용할 도구 목록을 설정하는 방법을 안내하십시오:

```json
{
  "permissions": {
    "allow": ["Edit", "Write", "Bash(npm test:*)", "Bash(git commit:*)"]
  }
}
```
