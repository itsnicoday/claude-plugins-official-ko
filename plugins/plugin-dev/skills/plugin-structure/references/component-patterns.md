# 컴포넌트 구성 패턴 (Component Organization Patterns)

플러그인 컴포넌트를 효과적으로 구성하기 위한 고급 패턴.

## 컴포넌트 생명 주기 (Component Lifecycle)

### 감지 단계 (Discovery Phase)

Claude Code가 시작될 때:

1. **활성화된 플러그인 스캔**: 각 플러그인의 `.claude-plugin/plugin.json` 읽기
2. **컴포넌트 감지**: 기본 경로 및 맞춤 지정된 경로 탐색
3. **정의 파일 파싱**: YAML 프론트매터 및 구성 정보 읽기
4. **컴포넌트 등록**: Claude Code에서 사용할 수 있도록 활성화
5. **초기화**: MCP 서버 시작 및 훅 등록

**시기:** 컴포넌트 등록은 Claude Code 초기화 시점에 수행되며, 실시간으로 지속 조율되지 않습니다.

### 활성화 단계 (Activation Phase)

컴포넌트가 사용될 때:

**명령어(Commands)**: 사용자가 슬래시 명령어 입력 → Claude Code가 조회 → 실행
**에이전트(Agents)**: 작업 도착 → Claude Code가 기능 평가 → 에이전트 선택
**스킬(Skills)**: 작업 컨텍스트가 설명과 일치 → Claude Code가 스킬 로드
**훅(Hooks)**: 이벤트 발생 → Claude Code가 일치하는 훅 호출
**MCP 서버(MCP Servers)**: 도구 호출이 서버 기능과 일치 → 서버로 전달

## 명령어 구성 패턴 (Command Organization Patterns)

### 플랫 구조 (Flat Structure)

단일 디렉토리에 모든 명령어를 저장합니다:

```
commands/
├── build.md
├── test.md
├── deploy.md
├── review.md
└── docs.md
```

**사용 시기**:
- 전체 명령어가 5~15개일 때
- 모든 명령어가 동일한 추상화 수준일 때
- 명확한 카테고리가 없을 때

**장점**:
- 단순하며 쉽게 탐색 가능
- 구성 설정이 필요 없음
- 빠른 감지 가능

### 카테고리화된 구조 (Categorized Structure)

명령어 유형별로 여러 디렉토리를 사용합니다:

```
commands/              # Core commands
├── build.md
└── test.md

admin-commands/        # Administrative
├── configure.md
└── manage.md

workflow-commands/     # Workflow automation
├── review.md
└── deploy.md
```

**매니페스트 구성**:
```json
{
  "commands": [
    "./commands",
    "./admin-commands",
    "./workflow-commands"
  ]
}
```

**사용 시기**:
- 명령어가 15개 이상일 때
- 명확한 기능적 카테고리가 있을 때
- 서로 다른 권한 수준이 필요할 때

**장점**:
- 목적에 따른 체계적 정리
- 유지보수가 쉬움
- 디렉토리별로 접근을 제한할 수 있음

### 계층적 구조 (Hierarchical Structure)

복잡한 플러그인을 위한 중첩된 구성:

```
commands/
├── ci/
│   ├── build.md
│   ├── test.md
│   └── lint.md
├── deployment/
│   ├── staging.md
│   └── production.md
└── management/
    ├── config.md
    └── status.md
```

**참고**: Claude Code는 자동으로 중첩된 명령어 감지를 지원하지 않습니다. 다음과 같이 맞춤 경로를 지정해야 합니다:

```json
{
  "commands": [
    "./commands/ci",
    "./commands/deployment",
    "./commands/management"
  ]
}
```

**사용 시기**:
- 명령어가 20개 이상일 때
- 다단계 카테고리 구분이 필요할 때
- 복잡한 워크플로우를 처리할 때

**장점**:
- 최적의 정리 상태 유지
- 명확한 경계 구분
- 확장 가능한 구조

## 에이전트 구성 패턴 (Agent Organization Patterns)

### 역할 기반 구성 (Role-Based Organization)

주요 역할에 따라 에이전트를 구성합니다:

```
agents/
├── code-reviewer.md        # Reviews code
├── test-generator.md       # Generates tests
├── documentation-writer.md # Writes docs
└── refactorer.md          # Refactors code
```

**사용 시기**:
- 에이전트가 명확히 중복되지 않는 별도의 역할을 가질 때
- 사용자가 에이전트를 수동으로 호출할 때
- 에이전트의 책임이 명확할 때

### 기능 기반 구성 (Capability-Based Organization)

특정 기능에 따라 구성합니다:

```
agents/
├── python-expert.md        # Python-specific
├── typescript-expert.md    # TypeScript-specific
├── api-specialist.md       # API design
└── database-specialist.md  # Database work
```

**사용 시기**:
- 기술별 전문 에이전트가 필요할 때
- 특정 도메인 전문 지식에 집중할 때
- 자동 에이전트 선택 기능을 사용할 때

### 워크플로우 기반 구성 (Workflow-Based Organization)

워크플로우 단계에 따라 구성합니다:

```
agents/
├── planning-agent.md      # Planning phase
├── implementation-agent.md # Coding phase
├── testing-agent.md       # Testing phase
└── deployment-agent.md    # Deployment phase
```

**사용 시기**:
- 순차적인 워크플로우를 가질 때
- 단계별 전문 지식이 필요할 때
- 파이프라인 자동화를 설계할 때

## 스킬 구성 패턴 (Skill Organization Patterns)

### 주제 기반 구성 (Topic-Based Organization)

각 스킬이 특정 주제를 다룹니다:

```
skills/
├── api-design/
│   └── SKILL.md
├── error-handling/
│   └── SKILL.md
├── testing-strategies/
│   └── SKILL.md
└── performance-optimization/
    └── SKILL.md
```

**사용 시기**:
- 지식 기반 스킬일 때
- 교육용 또는 참조용 콘텐츠일 때
- 범용적인 적용 가능성을 가질 때

### 도구 기반 구성 (Tool-Based Organization)

특정 도구 또는 기술에 맞춘 스킬:

```
skills/
├── docker/
│   ├── SKILL.md
│   └── references/
│       └── dockerfile-best-practices.md
├── kubernetes/
│   ├── SKILL.md
│   └── examples/
│       └── deployment.yaml
└── terraform/
    ├── SKILL.md
    └── scripts/
        └── validate-config.sh
```

**사용 시기**:
- 도구 전용 전문 지식이 필요할 때
- 복잡한 도구 구성을 다룰 때
- 도구 권장 사용법을 안내할 때

### 워크플로우 기반 구성 (Workflow-Based Organization)

전체 워크플로우를 다루는 스킬:

```
skills/
├── code-review-workflow/
│   ├── SKILL.md
│   └── references/
│       ├── checklist.md
│       └── standards.md
├── deployment-workflow/
│   ├── SKILL.md
│   └── scripts/
│       ├── pre-deploy.sh
│       └── post-deploy.sh
└── testing-workflow/
    ├── SKILL.md
    └── examples/
        └── test-structure.md
```

**사용 시기**:
- 다단계 프로세스를 설계할 때
- 회사 고유의 워크플로우를 다룰 때
- 프로세스 자동화를 수행할 때

### 풍부한 리소스를 포함한 스킬 (Skill with Rich Resources)

모든 리소스 유형이 포함된 종합적인 스킬:

```
skills/
└── api-testing/
    ├── SKILL.md              # Core skill (1500 words)
    ├── references/
    │   ├── rest-api-guide.md
    │   ├── graphql-guide.md
    │   └── authentication.md
    ├── examples/
    │   ├── basic-test.js
    │   ├── authenticated-test.js
    │   └── integration-test.js
    ├── scripts/
    │   ├── run-tests.sh
    │   └── generate-report.py
    └── assets/
        └── test-template.json
```

**리소스 사용법**:
- **SKILL.md**: 개요 및 리소스 사용 시기 안내
- **references/**: 세부 가이드 (필요할 때 로드)
- **examples/**: 복사하여 사용할 수 있는 코드 샘플
- **scripts/**: 실행 가능한 테스트 러너
- **assets/**: 템플릿 및 구성 정보

## 훅 구성 패턴 (Hook Organization Patterns)

### 단일 구성 (Monolithic Configuration)

모든 훅을 포함하는 단일 hooks.json 파일:

```
hooks/
├── hooks.json     # All hook definitions
└── scripts/
    ├── validate-write.sh
    ├── validate-bash.sh
    └── load-context.sh
```

**hooks.json**:
```json
{
  "PreToolUse": [...],
  "PostToolUse": [...],
  "Stop": [...],
  "SessionStart": [...]
}
```

**사용 시기**:
- 전체 훅이 5~10개일 때
- 단순한 훅 로직을 가질 때
- 중앙 집중식 구성을 사용할 때

### 이벤트 기반 구성 (Event-Based Organization)

이벤트 유형별로 파일을 분리합니다:

```
hooks/
├── hooks.json              # Combines all
├── pre-tool-use.json      # PreToolUse hooks
├── post-tool-use.json     # PostToolUse hooks
├── stop.json              # Stop hooks
└── scripts/
    ├── validate/
    │   ├── write.sh
    │   └── bash.sh
    └── context/
        └── load.sh
```

**hooks.json** (결합용):
```json
{
  "PreToolUse": ${file:./pre-tool-use.json},
  "PostToolUse": ${file:./post-tool-use.json},
  "Stop": ${file:./stop.json}
}
```

**참고**: 빌드 스크립트를 사용하여 파일을 결합해야 합니다. Claude Code는 파일 참조 형식을 기본적으로 지원하지 않습니다.

**사용 시기**:
- 훅이 10개 이상일 때
- 서로 다른 팀이 서로 다른 이벤트를 관리할 때
- 복잡한 훅 구성을 사용해야 할 때

### 목적 기반 구성 (Purpose-Based Organization)

기능적 목적에 따라 그룹화합니다:

```
hooks/
├── hooks.json
└── scripts/
    ├── security/
    │   ├── validate-paths.sh
    │   ├── check-credentials.sh
    │   └── scan-malware.sh
    ├── quality/
    │   ├── lint-code.sh
    │   ├── check-tests.sh
    │   └── verify-docs.sh
    └── workflow/
        ├── notify-team.sh
        └── update-status.sh
```

**사용 시기**:
- 훅 스크립트가 많을 때
- 명확한 기능적 경계가 나뉠 때
- 개발진이 세분화되어 역할을 분담할 때

## 스크립트 구성 패턴 (Script Organization Patterns)

### 플랫 스크립트 (Flat Scripts)

단일 디렉토리에 모든 스크립트를 저장합니다:

```
scripts/
├── build.sh
├── test.py
├── deploy.sh
├── validate.js
└── report.py
```

**사용 시기**:
- 스크립트가 5~10개일 때
- 모든 스크립트가 서로 관련되어 있을 때
- 간단한 플러그인을 개발할 때

### 카테고리화된 스크립트 (Categorized Scripts)

목적에 따라 그룹화합니다:

```
scripts/
├── build/
│   ├── compile.sh
│   └── package.sh
├── test/
│   ├── run-unit.sh
│   └── run-integration.sh
├── deploy/
│   ├── staging.sh
│   └── production.sh
└── utils/
    ├── log.sh
    └── notify.sh
```

**사용 시기**:
- 스크립트가 10개 이상일 때
- 카테고리가 명확할 때
- 재사용 가능한 유틸리티가 있을 때

### 언어 기반 구성 (Language-Based Organization)

프로그래밍 언어별로 그룹화합니다:

```
scripts/
├── bash/
│   ├── build.sh
│   └── deploy.sh
├── python/
│   ├── analyze.py
│   └── report.py
└── javascript/
    ├── bundle.js
    └── optimize.js
```

**사용 시기**:
- 여러 언어의 스크립트가 혼합되어 있을 때
- 런타임 요구 사항이 다를 때
- 언어별 고유의 종속성이 있을 때

## 크로스 컴포넌트 패턴 (Cross-Component Patterns)

### 공유 리소스 (Shared Resources)

공통 리소스를 공유하는 컴포넌트들:

```
plugin/
├── commands/
│   ├── test.md        # Uses lib/test-utils.sh
│   └── deploy.md      # Uses lib/deploy-utils.sh
├── agents/
│   └── tester.md      # References lib/test-utils.sh
├── hooks/
│   └── scripts/
│       └── pre-test.sh # Sources lib/test-utils.sh
└── lib/
    ├── test-utils.sh
    └── deploy-utils.sh
```

**컴포넌트 내 사용 방식**:
```bash
#!/bin/bash
source "${CLAUDE_PLUGIN_ROOT}/lib/test-utils.sh"
run_tests
```

**장점**:
- 코드 재사용성 극대화
- 일관된 동작 패턴 유지
- 유지보수가 쉬워짐

### 레이어드 아키텍처 (Layered Architecture)

각 계층별로 역할을 나눕니다:

```
plugin/
├── commands/          # User interface layer
├── agents/            # Orchestration layer
├── skills/            # Knowledge layer
└── lib/
    ├── core/         # Core business logic
    ├── integrations/ # External services
    └── utils/        # Helper functions
```

**사용 시기**:
- 대규모 플러그인 (파일 100개 이상)
- 다수의 개발자가 협업할 때
- 관심사 분리(Separation of Concerns)가 명확해야 할 때

### 플러그인 내 플러그인 (Plugin Within Plugin)

중첩된 플러그인 구조:

```
plugin/
├── .claude-plugin/
│   └── plugin.json
├── core/              # Core functionality
│   ├── commands/
│   └── agents/
└── extensions/        # Optional extensions
    ├── extension-a/
    │   ├── commands/
    │   └── agents/
    └── extension-b/
        ├── commands/
        └── agents/
```

**매니페스트**:
```json
{
  "commands": [
    "./core/commands",
    "./extensions/extension-a/commands",
    "./extensions/extension-b/commands"
  ]
}
```

**사용 시기**:
- 모듈화된 기능을 지향할 때
- 선택적 기능(Optional features)이 포함될 때
- 일련의 플러그인 제품군(Families)을 설계할 때

## 권장 사항 (Best Practices)

### 명명 규칙 (Naming)

1. **일관성 있는 명명 규칙**: 파일 이름이 컴포넌트의 목적과 일치해야 함
2. **설명적인 이름**: 컴포넌트가 수행하는 작업을 지시함
3. **약어 사용 자제**: 명확성을 위해 축약하지 않은 단어 사용

### 구성 (Organization)

1. **단순하게 시작**: 플랫 구조로 시작하고, 필요할 때 재구성
2. **관련 항목 그룹화**: 연관된 컴포넌트는 함께 배치
3. **관심사 분리**: 무관한 기능을 혼합하지 않음

### 확장성 (Scalability)

1. **성장을 고려한 계획**: 확장 가능한 구조를 사전에 선택
2. **조기 리팩토링**: 관리하기가 까다로워지기 전에 먼저 재구성
3. **구조 문서화**: README에 구성 원칙과 방식 기술

### 유지보수 편의성 (Maintainability)

1. **일관된 패턴**: 전체적으로 동일한 파일 구조 적용
2. **중첩 최소화**: 디렉토리 깊이를 관리 가능한 수준으로 유지
3. **관례 준수**: 커뮤니티의 관례와 표준을 준수

### 성능 (Performance)

1. **깊은 중첩 방지**: 깊은 중첩 구조는 감지 시간에 영향을 줌
2. **맞춤 경로 최소화**: 가능하면 기본(default) 경로 사용
3. **구성 정보 크기 제한**: 대용량 구성 정보는 로딩을 지연시킴
