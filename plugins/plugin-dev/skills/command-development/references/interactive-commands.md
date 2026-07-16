# 인터랙티브 명령어 패턴 (Interactive Command Patterns)

AskUserQuestion 도구를 사용하여 사용자 피드백을 수집하고 의사 결정을 진행하는 대화형 명령어를 제작하기 위한 종합 안내서입니다.

## 개요

일부 명령어는 단순한 인자(arguments)만으로는 수월하게 처리하기 힘든 사용자 입력이 필요합니다. 예를 들어:
- 장단점이 교차하는 복잡한 옵션들 사이에서 하나를 선택해야 할 때
- 목록에서 여러 개의 항목을 다중 선택해야 할 때
- 부가 설명이 수반되는 결정을 내려야 할 때
- 대화식으로 설정 파일 정보나 사용자 선호도를 수집할 때

이러한 사례들을 처리하려면 명령어 실행에 단순 인자를 사용하는 대신 명령어 내부에서 **AskUserQuestion 도구**를 사용하십시오.

## AskUserQuestion 사용 시점

### AskUserQuestion 사용이 권장되는 상황:

1. **객관식 결정**이 필요하며 세부 설명이 필요할 때
2. 선택을 위해 문맥(context) 파악이 필요한 **복잡한 옵션**들을 다룰 때
3. **다중 선택 시나리오** (여러 항목을 골라야 하는 경우)
4. 설정을 위해 **사용자 선호도 정보**를 수집할 때
5. 답변에 따라 다르게 조율되는 **대화식 워크플로우**를 구축할 때

### 명령어 인자(Arguments) 사용이 권장되는 상황:

1. **단순한 값** (파일 경로, 숫자, 이름 등)
2. 사용자가 이미 명확히 인지하고 준비해 둔 **기존 입력값**이 존재할 때
3. 자동화 처리가 가능해야 하는 **스크립트 방식의 워크플로우**일 때
4. 대화 과정이 불필요하여 **빠른 호출 및 실행**이 중요할 때

## AskUserQuestion 기본 사항

### 도구 매개변수 (Tool Parameters)

```typescript
{
  questions: [
    {
      question: "Which authentication method should we use?",
      header: "Auth method",  // Short label (max 12 chars)
      multiSelect: false,     // true for multiple selection
      options: [
        {
          label: "OAuth 2.0",
          description: "Industry standard, supports multiple providers"
        },
        {
          label: "JWT",
          description: "Stateless, good for APIs"
        },
        {
          label: "Session",
          description: "Traditional, server-side state"
        }
      ]
    }
  ]
}
```

**핵심 사항:**
- 사용자는 자동 생성된 "Other" 항목을 골라 직접 커스텀 입력값을 전달할 수도 있습니다.
- `multiSelect: true`로 설정하면 복잡한 옵션 중 여러 개를 동시에 선택할 수 있습니다.
- 옵션 개수는 2~4개 수준으로 유지하는 것이 바람직합니다.
- 도구 호출 1회당 1~4개의 질문을 한 번에 던질 수 있습니다.

## 사용자 상호작용을 위한 명령어 패턴

### 기본적인 대화형 명령어

```markdown
---
description: Interactive setup command
allowed-tools: AskUserQuestion, Write
---

# Interactive Plugin Setup

This command will guide you through configuring the plugin with a series of questions.

## Step 1: Gather Configuration

Use the AskUserQuestion tool to ask:

**Question 1 - Deployment target:**
- header: "Deploy to"
- question: "Which deployment platform will you use?"
- options:
  - AWS (Amazon Web Services with ECS/EKS)
  - GCP (Google Cloud with GKE)
  - Azure (Microsoft Azure with AKS)
  - Local (Docker on local machine)

**Question 2 - Environment strategy:**
- header: "Environments"
- question: "How many environments do you need?"
- options:
  - Single (Just production)
  - Standard (Dev, Staging, Production)
  - Complete (Dev, QA, Staging, Production)

**Question 3 - Features to enable:**
- header: "Features"
- question: "Which features do you want to enable?"
- multiSelect: true
- options:
  - Auto-scaling (Automatic resource scaling)
  - Monitoring (Health checks and metrics)
  - CI/CD (Automated deployment pipeline)
  - Backups (Automated database backups)

## Step 2: Process Answers

Based on the answers received from AskUserQuestion:

1. Parse the deployment target choice
2. Set up environment-specific configuration
3. Enable selected features
4. Generate configuration files

## Step 3: Generate Configuration

Create `.claude/plugin-name.local.md` with:

\`\`\`yaml
---
deployment_target: [answer from Q1]
environments: [answer from Q2]
features:
  auto_scaling: [true if selected in Q3]
  monitoring: [true if selected in Q3]
  ci_cd: [true if selected in Q3]
  backups: [true if selected in Q3]
---

# Plugin Configuration

Generated: [timestamp]
Target: [deployment_target]
Environments: [environments]
\`\`\`

## Step 4: Confirm and Next Steps

Confirm configuration created and guide user on next steps.
```

### 다단계 대화형 워크플로우

```markdown
---
description: Multi-stage interactive workflow
allowed-tools: AskUserQuestion, Read, Write, Bash
---

# Multi-Stage Deployment Setup

This command walks through deployment setup in stages, adapting based on your answers.

## Stage 1: Basic Configuration

Use AskUserQuestion to ask about deployment basics.

Based on answers, determine which additional questions to ask.

## Stage 2: Advanced Options (Conditional)

If user selected "Advanced" deployment in Stage 1:

Use AskUserQuestion to ask about:
- Load balancing strategy
- Caching configuration
- Security hardening options

If user selected "Simple" deployment:
- Skip advanced questions
- Use sensible defaults

## Stage 3: Confirmation

Show summary of all selections.

Use AskUserQuestion for final confirmation:
- header: "Confirm"
- question: "Does this configuration look correct?"
- options:
  - Yes (Proceed with setup)
  - No (Start over)
  - Modify (Let me adjust specific settings)

If "Modify", ask which specific setting to change.

## Stage 4: Execute Setup

Based on confirmed configuration, execute setup steps.
```

## 대화형 질문 설계

### 질문 구조

**올바른 질문 예시:**
```markdown
Question: "Which database should we use for this project?"
Header: "Database"
Options:
  - PostgreSQL (Relational, ACID compliant, best for complex queries)
  - MongoDB (Document store, flexible schema, best for rapid iteration)
  - Redis (In-memory, fast, best for caching and sessions)
```

**피해야 할 질문 예시:**
```markdown
Question: "Database?"  // 너무 모호함
Header: "DB"  // 불명확한 약어
Options:
  - Option 1  // 설명적이지 않음
  - Option 2
```

### 옵션 설계 권장 가이드

**명확한 라벨 지정:**
- 1~5단어 내외로 요약하세요.
- 구체적이고 설명적이어야 합니다.
- 설명 없이 전문 용어만 적지 마세요.

**도움이 되는 설명 기재:**
- 해당 옵션이 의미하는 바를 설명하세요.
- 주요 이점이나 장단점을 함께 서술해 주세요.
- 사용자가 충분히 숙지한 상태에서 결정하도록 도와주세요.
- 1~2문장 내로 짧게 작성하세요.

**적정 수의 옵션 구성:**
- 질문 하나당 2~4개의 옵션을 지정하세요.
- 선택지가 너무 많아 사용자가 지치지 않도록 조율하세요.
- 서로 관련된 선택지들을 하나로 묶으세요.
- "Other"는 자동으로 생성됩니다.

### 다중 선택형 질문 (Multi-Select Questions)

**multiSelect 사용을 권장하는 경우:**

```markdown
Use AskUserQuestion for enabling features:

Question: "Which features do you want to enable?"
Header: "Features"
multiSelect: true  // 여러 개를 동시 선택 가능하게 설정
Options:
  - Logging (Detailed operation logs)
  - Metrics (Performance monitoring)
  - Alerts (Error notifications)
  - Backups (Automatic backups)
```

사용자는 아무것도 고르지 않거나, 일부 또는 전체를 선택하는 임의의 선택 조합을 지정할 수 있습니다.

**multiSelect 사용을 피해야 하는 경우:**

```markdown
Question: "Which authentication method?"
multiSelect: false  // 오직 한 개의 인증 방법만 활성화되어야 함
```

동시에 성립될 수 없는 옵션들을 제공할 때는 multiSelect 기능을 끄고 단일 선택으로 설정해야 합니다.

## AskUserQuestion 활용 명령어 패턴

### 패턴 1: 단순 Yes/No 결정형 (Simple Yes/No Decision)

```markdown
---
description: Command with confirmation
allowed-tools: AskUserQuestion, Bash
---

# Destructive Operation

This operation will delete all cached data.

Use AskUserQuestion to confirm:

Question: "This will delete all cached data. Are you sure?"
Header: "Confirm"
Options:
  - Yes (Proceed with deletion)
  - No (Cancel operation)

If user selects "Yes":
  Execute deletion
  Report completion

If user selects "No":
  Cancel operation
  Exit without changes
```

### 패턴 2: 다중 설정 질문형 (Multiple Configuration Questions)

```markdown
---
description: Multi-question configuration
allowed-tools: AskUserQuestion, Write
---

# Project Configuration Setup

Gather configuration through multiple questions.

Use AskUserQuestion with multiple questions in one call:

**Question 1:**
- question: "Which programming language?"
- header: "Language"
- options: Python, TypeScript, Go, Rust

**Question 2:**
- question: "Which test framework?"
- header: "Testing"
- options: Jest, PyTest, Go Test, Cargo Test
  (Q1에서 대답한 프로그래밍 언어에 상응하도록 가변적으로 제안됨)

**Question 3:**
- question: "Which CI/CD platform?"
- header: "CI/CD"
- options: GitHub Actions, GitLab CI, CircleCI

**Question 4:**
- question: "Which features do you need?"
- header: "Features"
- multiSelect: true
- options: Linting, Type checking, Code coverage, Security scanning

Process all answers together to generate cohesive configuration.
```

### 패턴 3: 조건부 질문 흐름 제어형 (Conditional Question Flow)

```markdown
---
description: Conditional interactive workflow
allowed-tools: AskUserQuestion, Read, Write
---

# Adaptive Configuration

## Question 1: Deployment Complexity

Use AskUserQuestion:

Question: "How complex is your deployment?"
Header: "Complexity"
Options:
  - Simple (Single server, straightforward)
  - Standard (Multiple servers, load balancing)
  - Complex (Microservices, orchestration)

## Conditional Questions Based on Answer

If answer is "Simple":
  - 추가적인 질문 생략
  - 기본 제공되는 최소 설정을 활용

If answer is "Standard":
  - 로드 밸런싱(load balancing) 아키텍처에 대해 질문
  - 스케일링 정책에 대해 질문

If answer is "Complex":
  - 오케스트레이션 플랫폼 유형(Kubernetes, Docker Swarm) 선택 유도
  - 서비스 메시 아키텍처(Istio, Linkerd, None) 선택 유도
  - 모니터링 수단(Prometheus, Datadog, CloudWatch) 선택 유도
  - 로그 파일 수집 및 보관 수단에 대해 질문

## Process Conditional Answers

Generate configuration appropriate for selected complexity level.
```

### 패턴 4: 점진적 반복 수집형 (Iterative Collection)

```markdown
---
description: Collect multiple items iteratively
allowed-tools: AskUserQuestion, Write
---

# Collect Team Members

We'll collect team member information for the project.

## Question: How many team members?

Use AskUserQuestion:

Question: "How many team members should we set up?"
Header: "Team size"
Options:
  - 2 people
  - 3 people
  - 4 people
  - 6 people

## Iterate Through Team Members

For each team member (1 to N based on answer):

Use AskUserQuestion for member details:

Question: "What role for team member [number]?"
Header: "Role"
Options:
  - Frontend Developer
  - Backend Developer
  - DevOps Engineer
  - QA Engineer
  - Designer

Store each member's information.

## Generate Team Configuration

After collecting all N members, create team configuration file with all members and their roles.
```

### 패턴 5: 종속성 선택형 (Dependency Selection)

```markdown
---
description: Select dependencies with multi-select
allowed-tools: AskUserQuestion
---

# Configure Project Dependencies

## Question: Required Libraries

Use AskUserQuestion with multiSelect:

Question: "Which libraries does your project need?"
Header: "Dependencies"
multiSelect: true
Options:
  - React (UI framework)
  - Express (Web server)
  - TypeORM (Database ORM)
  - Jest (Testing framework)
  - Axios (HTTP client)

User can select any combination.

## Process Selections

For each selected library:
- Add to package.json dependencies
- Generate sample configuration
- Create usage examples
- Update documentation
```

## 대화형 명령어 권장 가이드

### 질문 설계:

1. **명확하고 구체적으로**: 질문이 모호하지 않아야 합니다.
2. **간결한 헤더**: 깔끔한 화면 표시를 위해 헤더는 최대 12자로 작성합니다.
3. **친절한 옵션 구성**: 라벨은 명확하게 작성하고, 장단점 분석을 추가해 줍니다.
4. **적절한 개수**: 질문당 2~4개 옵션, 1회 호출당 1~4개 질문이 적당합니다.
5. **논리적인 순서**: 질문이 자연스러운 인과 관계로 이어지게 합니다.

### 에러 처리:

```markdown
# Handle AskUserQuestion Responses

After calling AskUserQuestion, verify answers received:

If answers are empty or invalid:
  Something went wrong gathering responses.

  Please try again or provide configuration manually:
  [Show alternative approach]

  Exit.

If answers look correct:
  Process as expected
```

### 점진적 노출 (Progressive Disclosure):

```markdown
# Start Simple, Get Detailed as Needed

## Question 1: Setup Type

Use AskUserQuestion:

Question: "How would you like to set up?"
Header: "Setup type"
Options:
  - Quick (Use recommended defaults)
  - Custom (Configure all options)
  - Guided (Step-by-step with explanations)

If "Quick":
  Apply defaults, minimal questions

If "Custom":
  Ask all available configuration questions

If "Guided":
  Ask questions with extra explanation
  Provide recommendations along the way
```

### 다중 선택 기준 (Multi-Select Guidelines):

**올바른 다중 선택 사용 사례:**
```markdown
Question: "Which features do you want to enable?"
multiSelect: true
Options:
  - Logging
  - Metrics
  - Alerts
  - Backups

이유: 사용자가 원하는 기능 조합을 임의로 엮어 제공받을 수 있음
```

**잘못된 다중 선택 사용 사례:**
```markdown
Question: "Which database engine?"
multiSelect: true  // ❌ 단일 선택이어야 함

이유: 데이터베이스 엔진은 물리적으로 한 시점에 하나만 주 서버로 동작할 수 있음
```

## 고급 패턴 (Advanced Patterns)

### 검증 루프 패턴 (Validation Loop)

```markdown
---
description: Interactive with validation
allowed-tools: AskUserQuestion, Bash
---

# Setup with Validation

## Gather Configuration

Use AskUserQuestion to collect settings.

## Validate Configuration

Check if configuration is valid:
- Required dependencies available?
- Settings compatible with each other?
- No conflicts detected?

If validation fails:
  Show validation errors

  Use AskUserQuestion to ask:

  Question: "Configuration has issues. What would you like to do?"
  Header: "Next step"
  Options:
    - Fix (Adjust settings to resolve issues)
    - Override (Proceed despite warnings)
    - Cancel (Abort setup)

  Based on answer, retry or proceed or exit.
```

### 점진적 설정 빌더 패턴 (Build Configuration Incrementally)

```markdown
---
description: Incremental configuration builder
allowed-tools: AskUserQuestion, Write, Read
---

# Incremental Setup

## Phase 1: Core Settings

Use AskUserQuestion for core settings.

Save to `.claude/config-partial.yml`

## Phase 2: Review Core Settings

Show user the core settings:

Based on these core settings, you need to configure:
- [Setting A] (because you chose [X])
- [Setting B] (because you chose [Y])

Ready to continue?

## Phase 3: Detailed Settings

Use AskUserQuestion for settings based on Phase 1 answers.

Merge with core settings.

## Phase 4: Final Review

Present complete configuration.

Use AskUserQuestion for confirmation:

Question: "Is this configuration correct?"
Options:
  - Yes (Save and apply)
  - No (Start over)
  - Modify (Edit specific settings)
```

### 문맥 기반 동적 옵션 생성 패턴 (Dynamic Options Based on Context)

```markdown
---
description: Context-aware questions
allowed-tools: AskUserQuestion, Bash, Read
---

# Context-Aware Setup

## Detect Current State

Check existing configuration:
- Current language: !`detect-language.sh`
- Existing frameworks: !`detect-frameworks.sh`
- Available tools: !`check-tools.sh`

## Ask Context-Appropriate Questions

Based on detected language, ask relevant questions.

If language is TypeScript:

  Use AskUserQuestion:

  Question: "Which TypeScript features should we enable?"
  Options:
    - Strict Mode (Maximum type safety)
    - Decorators (Experimental decorator support)
    - Path Mapping (Module path aliases)

If language is Python:

  Use AskUserQuestion:

  Question: "Which Python tools should we configure?"
  Options:
    - Type Hints (mypy for type checking)
    - Black (Code formatting)
    - Pylint (Linting and style)

Questions adapt to project context.
```

## 실무 예시: 멀티 에이전트 스웜 구동 (Multi-Agent Swarm Launch)

**multi-agent-swarm 플러그인에서 빌췌:**

```markdown
---
description: Launch multi-agent swarm
allowed-tools: AskUserQuestion, Read, Write, Bash
---

# Launch Multi-Agent Swarm

## Interactive Mode (No Task List Provided)

If user didn't provide task list file, help create one interactively.

### Question 1: Agent Count

Use AskUserQuestion:

Question: "How many agents should we launch?"
Header: "Agent count"
Options:
  - 2 agents (Best for simple projects)
  - 3 agents (Good for medium projects)
  - 4 agents (Standard team size)
  - 6 agents (Large projects)
  - 8 agents (Complex multi-component projects)

### Question 2: Task Definition Approach

Use AskUserQuestion:

Question: "How would you like to define tasks?"
Header: "Task setup"
Options:
  - File (I have a task list file ready)
  - Guided (Help me create tasks interactively)
  - Custom (Other approach)

If "File":
  Ask for file path
  Validate file exists and has correct format

If "Guided":
  Enter iterative task creation mode (see below)

### Question 3: Coordination Mode

Use AskUserQuestion:

Question: "How should agents coordinate?"
Header: "Coordination"
Options:
  - Team Leader (One agent coordinates others)
  - Collaborative (Agents coordinate as peers)
  - Autonomous (Independent work, minimal coordination)

### Iterative Task Creation (If "Guided" Selected)

For each agent (1 to N from Question 1):

**Question A: Agent Name**
Question: "What should we call agent [number]?"
Header: "Agent name"
Options:
  - auth-agent
  - api-agent
  - ui-agent
  - db-agent
  (Provide relevant suggestions based on common patterns)

**Question B: Task Type**
Question: "What task for [agent-name]?"
Header: "Task type"
Options:
  - Authentication (User auth, JWT, OAuth)
  - API Endpoints (REST/GraphQL APIs)
  - UI Components (Frontend components)
  - Database (Schema, migrations, queries)
  - Testing (Test suites and coverage)
  - Documentation (Docs, README, guides)

**Question C: Dependencies**
Question: "What does [agent-name] depend on?"
Header: "Dependencies"
multiSelect: true
Options:
  - [List of previously defined agents]
  - No dependencies

**Question D: Base Branch**
Question: "Which base branch for PR?"
Header: "PR base"
Options:
  - main
  - staging
  - develop

Store all task information for each agent.

### Generate Task List File

After collecting all agent task details:

1. Ask for project name
2. Generate task list in proper format
3. Save to `.daisy/swarm/tasks.md`
4. Show user the file path
5. Proceed with launch using generated task list
```

## 베스트 프랙티스

### 질문 작성법:

1. **구체적으로**: "어떤 데이터베이스를 사용하시겠습니까?"와 같이 구체적으로 명시하세요.
2. **장단점 기술**: 각 선택 옵션별로 명확한 장점과 단점을 기재하세요.
3. **콘텍스트 제공**: 다른 부가 설명 없이도 질문 그 자체로 이해될 수 있어야 합니다.
4. **의사 결정 유도**: 사용자가 충분한 정보를 얻고 최종 판단을 내리도록 안내하세요.
5. **간결하게 유지**: 헤더는 최대 12자 이내로, 각 상세 설명은 1~2문장으로 작문하세요.

### 옵션 디자인:

1. **유의미한 라벨**: 모호하지 않고 명확한 단어로 표기하세요.
2. **정보를 주는 설명**: 각 옵션이 실행되었을 때 어떤 결과를 주는지 기술하세요.
3. **장단점 제시**: 해당 선택을 했을 때 뒤따르는 영향이나 비용 등을 언급하세요.
4. **일관된 상세 수준**: 모든 옵션들에 대해 비슷한 분량과 톤앤매너로 서술하세요.
5. **2~4개 옵션 구성**: 너무 선택지가 좁거나 과도하게 넓어지지 않게 만듭니다.

### 흐름 설계:

1. **논리적인 순서**: 질문들이 물 흐르듯 순차적으로 연동되게 디자인합니다.
2. **이전 단계 활용**: 후속 질문은 이전 답변에 기초하여 동적으로 변경되어야 합니다.
3. **질문 최소화**: 굳이 물어보지 않아도 되는 영역은 생략하고 필수 질문만 간추립니다.
4. **연관 질문 그룹화**: 성격이 서로 밀접한 질문들은 묶어서 함께 제시합니다.
5. **진행 상황 안내**: 전체 진행 과정 중 어느 단계에 와 있는지 보여줍니다.

### 사용자 경험:

1. **예측 가능성 부여**: 명령어를 통해 어떤 작업을 하게 되는지 사전에 예고합니다.
2. **이유 설명**: 사용자 질문 수집이 왜 필요한지 그 당위성을 명시합니다.
3. **기본값 제공**: 특별한 사유가 없을 때 선택할 수 있는 추천 설정을 제시합니다.
4. **중단 권한 보장**: 사용자가 중간에 작업을 취소하거나 처음부터 다시 시작할 수 있도록 돕습니다.
5. **최종 확인**: 작업을 실행에 옮기기 전, 선택한 정보를 한눈에 볼 수 있게 요약합니다.

## 일반적인 패턴

### 패턴: 기능 선택 (Feature Selection)

```markdown
Use AskUserQuestion:

Question: "Which features do you need?"
Header: "Features"
multiSelect: true
Options:
  - Authentication
  - Authorization
  - Rate Limiting
  - Caching
```

### 패턴: 환경 구성 (Environment Configuration)

```markdown
Use AskUserQuestion:

Question: "Which environment is this?"
Header: "Environment"
Options:
  - Development (Local development)
  - Staging (Pre-production testing)
  - Production (Live environment)
```

### 패턴: 우선순위 선택 (Priority Selection)

```markdown
Use AskUserQuestion:

Question: "What's the priority for this task?"
Header: "Priority"
Options:
  - Critical (Must be done immediately)
  - High (Important, do soon)
  - Medium (Standard priority)
  - Low (Nice to have)
```

### 패턴: 범위 지정 (Scope Selection)

```markdown
Use AskUserQuestion:

Question: "What scope should we analyze?"
Header: "Scope"
Options:
  - Current file (Just this file)
  - Current directory (All files in directory)
  - Entire project (Full codebase scan)
```

## 명령어 인자(Arguments)와 질문의 결합

### 적재적소에 올바른 도구 조합

**알고 있는 값에는 인자 사용:**
```markdown
---
argument-hint: [project-name]
allowed-tools: AskUserQuestion, Write
---

Setup for project: $1

Now gather additional configuration...

Use AskUserQuestion for options that require explanation.
```

**복잡한 결정 사항에는 질문 사용:**
```markdown
Project name from argument: $1

Now use AskUserQuestion to choose:
- Architecture pattern
- Technology stack
- Deployment strategy

이러한 항목들은 설명이 수반되어야 하므로 인자보다 질문 대화가 더 유용합니다.
```

## 문제 해결

**질문 목록이 화면에 나타나지 않는 현상:**
- allowed-tools 필드 리스트에 AskUserQuestion 도구가 누락되지 않았는지 점검합니다.
- 질문 구조 및 YAML 형식에 에러가 없는지 체크합니다.
- 옵션 배열 개수가 2~4개 수준으로 맞춰졌는지 점검합니다.

**사용자가 옵션을 올바르게 고르지 못하는 현상:**
- 옵션 라벨 텍스트가 직관적인지 점검합니다.
- 상세 설명 문구가 실제 동작을 자세하게 뒷받침하는지 확인합니다.
- 한 질문에 너무 많은 옵션들이 한꺼번에 기재되었는지 점검합니다.
- multiSelect 필드 설정이 의도와 일치하는지 재확인합니다.

**전체 질문 흐름이 번잡하게 느껴지는 현상:**
- 전체 질문의 총합 개수를 줄여나갑니다.
- 질문 유형 및 성격에 따라 질문들을 하나로 병합하거나 묶습니다.
- 중간 단계 단계 사이에 설명 문구나 안내 텍스트를 적재적소에 가미합니다.
- 사용자가 진행 흐름을 가늠할 수 있는 진행 상황 알림을 추가합니다.

AskUserQuestion 도구를 활용하면 명령어가 복잡한 의사 결정을 단계별로 정밀하게 안내해주는 지능형 템플릿 마법사로 작동하면서도, 기존의 인자 입력 방식이 지닌 단순하고 명쾌한 장점을 동시에 살릴 수 있게 됩니다.
