# 스킬(Skills) 추천

스킬은 워크플로우, 참조 자료, 모범 사례 등이 패키지화된 전문 지식입니다. `.claude/skills/<name>/SKILL.md` 경로에 작성하여 생성합니다. 스킬은 관련 상황이 생겼을 때 Claude가 자동으로 호출하거나, 사용자가 직접 `/skill-name` 명령어를 통해 명시적으로 호출할 수 있습니다.

일부 사전 정의된 스킬은 공식 플러그인(설치 명령어: `/plugin install`)을 통해 제공됩니다.

**참고**: 아래는 일반적인 패턴들입니다. 코드베이스의 도구와 프레임워크에 특화된 스킬 아이디어를 얻으려면 웹 검색을 활용하십시오.

---

## 공식 플러그인을 통해 사용 가능한 스킬 (Available from Official Plugins)

### 플러그인 개발 (plugin-dev)

| 스킬 | 적합한 용도 |
|-------|----------|
| **skill-development** | 올바른 구조를 갖춘 새로운 스킬 생성 |
| **hook-development** | 자동화를 위한 훅 작성 |
| **command-development** | 슬래시(/) 명령어 생성 |
| **agent-development** | 특화된 서브에이전트 구축 |
| **mcp-integration** | 플러그인에 MCP 서버 연동 |
| **plugin-structure** | 플러그인 아키텍처 이해 |

### Git 워크플로우 (commit-commands)

| 스킬 | 적합한 용도 |
|-------|----------|
| **commit** | 올바른 메시지 형식의 git 커밋 작성 |
| **commit-push-pr** | 커밋, 푸시 및 PR 발행까지의 전체 워크플로우 |

### 프론트엔드 (frontend-design)

| 스킬 | 적합한 용도 |
|-------|----------|
| **frontend-design** | 완성도 높은 UI 컴포넌트 생성 |

**효과**: 일반적인 AI 특유의 디자인에서 벗어나 개성 있고 고품질의 UI를 제작합니다.

### 자동화 규칙 (hookify)

| 스킬 | 적합한 용도 |
|-------|----------|
| **writing-rules** | 자동화를 위한 hookify 규칙 작성 |

### 기능 개발 (feature-dev)

| 스킬 | 적합한 용도 |
|-------|----------|
| **feature-dev** | 엔드투엔드 기능 개발 워크플로우 진행 |

---

## 빠른 참조: 공식 플러그인 스킬 (Quick Reference: Official Plugin Skills)

| 코드베이스 신호 | 추천 스킬 | 플러그인 |
|-----------------|-------|--------|
| 플러그인 제작 중 | skill-development | plugin-dev |
| Git 커밋 작업 | commit | commit-commands |
| React/Vue/Angular 사용 | frontend-design | frontend-design |
| 자동화 규칙 필요 | writing-rules | hookify |
| 기능 개발 계획 수립 | feature-dev | feature-dev |

---

## 커스텀 프로젝트 스킬 (Custom Project Skills)

프로젝트별 특화 스킬은 `.claude/skills/<name>/SKILL.md` 파일에 생성합니다.

### 스킬 디렉토리 구조 (Skill Structure)

```
.claude/skills/
└── my-skill/
    ├── SKILL.md           # 메인 지침 파일 (필수)
    ├── template.yaml      # 적용할 템플릿
    ├── scripts/
    │   └── validate.sh    # 실행할 스크립트
    └── examples/          # 참고용 예제 코드
```

### 프론트매터(Frontmatter) 참조 가이드

```yaml
---
name: skill-name
description: What this skill does and when to use it
disable-model-invocation: true  # Only user can invoke (for side effects)
user-invocable: false           # Only Claude can invoke (for background knowledge)
allowed-tools: Read, Grep, Glob # Restrict tool access
context: fork                   # Run in isolated subagent
agent: Explore                  # Which agent type when forked
---
```

### 호출 제어 설정 (Invocation Control)

| 설정 값 | 사용자 호출 | Claude 호출 | 주요 사용 목적 |
|---------|------|--------|---------|
| (기본값) | ✓ | ✓ | 일반 목적의 스킬 |
| `disable-model-invocation: true` | ✓ | ✗ | 사이드 이펙트 발생 작업 (배포, 전송 등) |
| `user-invocable: false` | ✗ | ✓ | 백그라운드 지식 제공 |

---

## 커스텀 스킬 작성 예시 (Custom Skill Examples)

### OpenAPI 템플릿을 활용한 API 문서화

YAML 템플릿을 적용하여 일관된 형태의 API 문서를 생성합니다:

```
.claude/skills/api-doc/
├── SKILL.md
└── openapi-template.yaml
```

**SKILL.md:**
```yaml
---
name: api-doc
description: Generate OpenAPI documentation for an endpoint. Use when documenting API routes.
---

Generate OpenAPI documentation for the endpoint at $ARGUMENTS.

Use the template in [openapi-template.yaml](openapi-template.yaml) as the structure.

1. Read the endpoint code
2. Extract path, method, parameters, request/response schemas
3. Fill in the template with actual values
4. Output the completed YAML
```

**openapi-template.yaml:**
```yaml
paths:
  /{path}:
    {method}:
      summary: ""
      description: ""
      parameters: []
      requestBody:
        content:
          application/json:
            schema: {}
      responses:
        "200":
          description: ""
          content:
            application/json:
              schema: {}
```

---

### 스크립트를 동반한 데이터베이스 마이그레이션 생성기

함께 번들링된 스크립트를 사용하여 마이그레이션 코드를 생성하고 검증합니다:

```
.claude/skills/create-migration/
├── SKILL.md
└── scripts/
    └── validate-migration.sh
```

**SKILL.md:**
```yaml
---
name: create-migration
description: Create a database migration file
disable-model-invocation: true
allowed-tools: Read, Write, Bash
---

Create a migration for: $ARGUMENTS

1. Generate migration file in `migrations/` with timestamp prefix
2. Include up and down functions
3. Run validation: `bash ~/.claude/skills/create-migration/scripts/validate-migration.sh`
4. Report any issues found
```

**scripts/validate-migration.sh:**
```bash
#!/bin/bash
# Validate migration syntax
npx prisma validate 2>&1 || echo "Validation failed"
```

---

### 예제 기반 테스트 생성기

프로젝트의 패턴을 준수하는 테스트 코드를 생성합니다:

```
.claude/skills/gen-test/
├── SKILL.md
└── examples/
    ├── unit-test.ts
    └── integration-test.ts
```

**SKILL.md:**
```yaml
---
name: gen-test
description: Generate tests for a file following project conventions
disable-model-invocation: true
---

Generate tests for: $ARGUMENTS

Reference these examples for the expected patterns:
- Unit tests: [examples/unit-test.ts](examples/unit-test.ts)
- Integration tests: [examples/integration-test.ts](examples/integration-test.ts)

1. Analyze the source file
2. Identify functions/methods to test
3. Generate tests matching project conventions
4. Place in appropriate test directory
```

---

### 템플릿 기반 컴포넌트 생성기

템플릿으로부터 새로운 컴포넌트의 기본 구조를 생성합니다:

```
.claude/skills/new-component/
├── SKILL.md
└── templates/
    ├── component.tsx.template
    ├── component.test.tsx.template
    └── component.stories.tsx.template
```

**SKILL.md:**
```yaml
---
name: new-component
description: Scaffold a new React component with tests and stories
disable-model-invocation: true
---

Create component: $ARGUMENTS

Use templates in [templates/](templates/) directory:
1. Generate component from component.tsx.template
2. Generate tests from component.test.tsx.template
3. Generate Storybook story from component.stories.tsx.template

Replace {{ComponentName}} with the PascalCase name.
Replace {{component-name}} with the kebab-case name.
```

---

### 체크리스트 기반 PR 리뷰

프로젝트 전용 체크리스트를 기준으로 PR 내용을 검토합니다:

```
.claude/skills/pr-check/
├── SKILL.md
└── checklist.md
```

**SKILL.md:**
```yaml
---
name: pr-check
description: Review PR against project checklist
disable-model-invocation: true
context: fork
---

## PR Context
- Diff: !`gh pr diff`
- Description: !`gh pr view`

Review against [checklist.md](checklist.md).

For each item, mark ✅ or ❌ with explanation.
```

**checklist.md:**
```markdown
## PR Checklist

- [ ] Tests added for new functionality
- [ ] No console.log statements
- [ ] Error handling includes user-facing messages
- [ ] API changes are backwards compatible
- [ ] Database migrations are reversible
```

---

### 릴리스 노트 생성기

Git 이력을 파악하여 릴리스 노트를 생성합니다:

**SKILL.md:**
```yaml
---
name: release-notes
description: Generate release notes from commits since last tag
disable-model-invocation: true
---

## Recent Changes
- Commits since last tag: !`git log $(git describe --tags --abbrev=0)..HEAD --oneline`
- Last tag: !`git describe --tags --abbrev=0`

Generate release notes:
1. Group commits by type (feat, fix, docs, etc.)
2. Write user-friendly descriptions
3. Highlight breaking changes
4. Format as markdown
```

---

### 프로젝트 컨벤션 정의 스킬 (Claude 전용)

Claude가 자동으로 적용하는 백그라운드 지식 설정 스킬입니다:

**SKILL.md:**
```yaml
---
name: project-conventions
description: Code style and patterns for this project. Apply when writing or reviewing code.
user-invocable: false
---

## Naming Conventions
- React components: PascalCase
- Utilities: camelCase
- Constants: UPPER_SNAKE_CASE
- Files: kebab-case

## Patterns
- Use `Result<T, E>` for fallible operations, not exceptions
- Prefer composition over inheritance
- All API responses use `{ data, error, meta }` shape

## Forbidden
- No `any` types
- No `console.log` in production code
- No synchronous file I/O
```

---

### 환경 설정 스킬

설정 스크립트를 사용하여 신규 개발자의 환경 구축을 돕습니다:

```
.claude/skills/setup-dev/
├── SKILL.md
└── scripts/
    └── check-prerequisites.sh
```

**SKILL.md:**
```yaml
---
name: setup-dev
description: Set up development environment for new contributors
disable-model-invocation: true
---

Set up development environment:

1. Check prerequisites: `bash scripts/check-prerequisites.sh`
2. Install dependencies: `npm install`
3. Copy environment template: `cp .env.example .env`
4. Set up database: `npm run db:setup`
5. Verify setup: `npm test`

Report any issues encountered.
```

---

## 인자 전달 패턴 (Argument Patterns)

| 패턴 | 의미 | 예시 |
|---------|---------|---------|
| `$ARGUMENTS` | 모든 인자를 문자열로 전달 | `/deploy staging` 실행 시 → "staging" |

스킬 설명 파일 내에 `$ARGUMENTS`가 지정되지 않은 경우, 인자 값은 `ARGUMENTS: <값>` 형태로 덧붙여집니다.

## 동적 컨텍스트 주입 (Dynamic Context Injection)

스킬이 실행되기 전에 `!`명령어`` 구문을 활용해 실시간 데이터를 주입할 수 있습니다:

```yaml
## Current State
- Branch: !`git branch --show-current`
- Status: !`git status --short`
```

명령어의 실행 결과는 Claude가 스킬 내용을 확인하기 전 플레이스홀더 영역에 치환되어 반영됩니다.
