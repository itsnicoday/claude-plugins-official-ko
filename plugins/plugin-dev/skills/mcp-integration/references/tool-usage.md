# 명령어 및 에이전트에서 MCP 도구 활용하기 (Using MCP Tools in Commands and Agents)

Claude Code 플러그인의 명령어 및 에이전트에서 MCP 도구를 효과적으로 활용하는 방법 전체 가이드.

## 개요 (Overview)

MCP 서버가 올바르게 구성되면 서버가 제공하는 도구들은 `mcp__plugin_<plugin-name>_<server-name>__<tool-name>` 접두사가 붙은 형태로 시스템에 노출됩니다. 이 도구들을 명령어 및 에이전트 구성 내에서 기본 기본 도구(built-in tools)들과 마찬가지로 동일하게 호출하여 사용할 수 있습니다.

## 도구 명명 규칙 (Tool Naming Convention)

### 형식 (Format)

```
mcp__plugin_<plugin-name>_<server-name>__<tool-name>
```

### 예시 (Examples)

**asana 서버를 지닌 Asana 플러그인 사례:**
- `mcp__plugin_asana_asana__asana_create_task`
- `mcp__plugin_asana_asana__asana_search_tasks`
- `mcp__plugin_asana_asana__asana_get_project`

**database 서버를 지닌 custom 플러그인 사례:**
- `mcp__plugin_myplug_database__query`
- `mcp__plugin_myplug_database__execute`
- `mcp__plugin_myplug_database__list_tables`

### 가용한 도구 이름 탐색하기 (Discovering Tool Names)

**/mcp 명령어 실행:**
```bash
/mcp
```

이 명령어는 다음 정보를 표시합니다:
- 현재 가동 중인 모든 MCP 서버 목록
- 각 서버가 노출하여 공급하는 세부 도구 목록
- 도구의 매개변수 정의 스키마 및 목적 설명
- 구성 설정에 직접 기입하여 사용할 전체 도구 이름

## 명령어 내부에서 도구 사용하기 (Using Tools in Commands)

### 도구 사전 허용 구성 (Pre-Allowing Tools)

명령어 마크다운의 YAML 프론트매터 영역에 활용할 MCP 도구를 명시적으로 지정하십시오:

```markdown
---
description: Create a new Asana task
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task"
]
---

# Create Task Command

To create a task:
1. Gather task details from user
2. Use mcp__plugin_asana_asana__asana_create_task with the details
3. Confirm creation to user
```

### 다중 도구 사전 허용 (Multiple Tools)

```markdown
---
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task",
  "mcp__plugin_asana_asana__asana_search_tasks",
  "mcp__plugin_asana_asana__asana_get_project"
]
---
```

### 와일드카드 사용 (신중히 사용) (Wildcard (Use Sparingly))

```markdown
---
allowed-tools: ["mcp__plugin_asana_asana__*"]
---
```

**주의:** 해당 명령어에서 특정 서버의 모든 도구를 빠짐없이 활용해야 하는 불가피한 상황에서만 와일드카드 매칭을 활용하십시오.

### 명령어 지침 내부의 도구 사용법 (Tool Usage in Command Instructions)

**예시 마크다운 명령어 지침:**
```markdown
---
description: Search and create Asana tasks
allowed-tools: [
  "mcp__plugin_asana_asana__asana_search_tasks",
  "mcp__plugin_asana_asana__asana_create_task"
]
---

# Asana Task Management

## Searching Tasks

To search for tasks:
1. Use mcp__plugin_asana_asana__asana_search_tasks
2. Provide search filters (assignee, project, etc.)
3. Display results to user

## Creating Tasks

To create a task:
1. Gather task details:
   - Title (required)
   - Description
   - Project
   - Assignee
   - Due date
2. Use mcp__plugin_asana_asana__asana_create_task
3. Show confirmation with task link
```

## 에이전트 내부에서 도구 사용하기 (Using Tools in Agents)

### 에이전트 구성 (Agent Configuration)

에이전트는 사전 허용(pre-allowing) 설정 없이도 가용한 MCP 도구들을 컨텍스트에 맞춰 자율적으로 호출해 활용합니다:

```markdown
---
name: asana-status-updater
description: This agent should be used when the user asks to "update Asana status", "generate project report", or "sync Asana tasks"
model: inherit
color: blue
---

## Role

Autonomous agent for generating Asana project status reports.

## Process

1. **Query tasks**: Use mcp__plugin_asana_asana__asana_search_tasks to get all tasks
2. **Analyze progress**: Calculate completion rates and identify blockers
3. **Generate report**: Create formatted status update
4. **Update Asana**: Use mcp__plugin_asana_asana__asana_create_comment to post report

## Available Tools

The agent has access to all Asana MCP tools without pre-approval.
```

### 에이전트의 도구 접근 권한 (Agent Tool Access)

에이전트는 명령어와 대비하여 유연한 도구 호출 범위를 보장받습니다:
- Claude가 판단하기에 작업 완수에 필수적인 임의의 가용한 도구를 호출합니다.
- 사전 정의 허용 목록(`allowed-tools`)을 필수로 기입할 의무가 없습니다.
- 원활한 실행을 위해 일반적으로 자주 쓰이는 핵심 도구 목록은 지침에 기재해 두는 것이 좋습니다.

## 도구 호출 패턴 (Tool Call Patterns)

### 패턴 1: 단순 도구 호출 (Pattern 1: Simple Tool Call)

단일 도구에 대해 파라미터 사전 유효성 확인을 진행하는 기본 흐름:

```markdown
Steps:
1. Validate user provided required fields
2. Call mcp__plugin_api_server__create_item with validated data
3. Check for errors
4. Display confirmation
```

### 패턴 2: 순차적인 도구 체이닝 호출 (Pattern 2: Sequential Tools)

복수의 도구를 선후 관계를 맺어 체이닝 형태로 순차 호출하는 흐름:

```markdown
Steps:
1. Search for existing items: mcp__plugin_api_server__search
2. If not found, create new: mcp__plugin_api_server__create
3. Add metadata: mcp__plugin_api_server__update_metadata
4. Return final item ID
```

### 패턴 3: 일괄 배치 연산 (Pattern 3: Batch Operations)

동일한 도구를 이용하여 반복 처리를 수행하는 일괄 배치 흐름:

```markdown
Steps:
1. Get list of items to process
2. For each item:
   - Call mcp__plugin_api_server__update_item
   - Track success/failure
3. Report results summary
```

### 패턴 4: 에러 처리 및 복구 (Pattern 4: Error Handling)

지연이나 네트워크 일시 오류 상황에 견고하게 대처하는 우아한 예외 복구 흐름:

```markdown
Steps:
1. Try to call mcp__plugin_api_server__get_data
2. If error (rate limit, network, etc.):
   - Wait and retry (max 3 attempts)
   - If still failing, inform user
   - Suggest checking configuration
3. On success, process data
```

## 도구 매개변수 (Tool Parameters)

### 도구 스키마 이해하기 (Understanding Tool Schemas)

각 MCP 도구는 전달 인수 매개변수를 선언한 스키마 정의 규격을 갖습니다. `/mcp`로 이를 미리 살펴보십시오.

**예시 JSON 스키마:**
```json
{
  "name": "asana_create_task",
  "description": "Create a new Asana task",
  "inputSchema": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "description": "Task title"
      },
      "notes": {
        "type": "string",
        "description": "Task description"
      },
      "workspace": {
        "type": "string",
        "description": "Workspace GID"
      }
    },
    "required": ["name", "workspace"]
  }
}
```

### 매개변수를 지정하여 도구 호출하기 (Calling Tools with Parameters)

Claude는 선언된 스키마에 기초해 인수 형식을 규격화하여 전송합니다:

```typescript
// Claude generates this internally
{
  toolName: "mcp__plugin_asana_asana__asana_create_task",
  input: {
    name: "Review PR #123",
    notes: "Code review for new feature",
    workspace: "12345",
    assignee: "67890",
    due_on: "2025-01-15"
  }
}
```

### 매개변수 값 유효성 검증 (Parameter Validation)

**명령어 내부 가이드라인 단계에서 매개변수 사전 검증 단계를 명시하십시오:**

```markdown
Steps:
1. Check required parameters:
   - Title is not empty
   - Workspace ID is provided
   - Due date is valid format (YYYY-MM-DD)
2. If validation fails, ask user to provide missing data
3. If validation passes, call MCP tool
4. Handle tool errors gracefully
```

## 응답 처리 (Response Handling)

### 성공 응답 처리 (Success Responses)

```markdown
Steps:
1. Call MCP tool
2. On success:
   - Extract relevant data from response
   - Format for user display
   - Provide confirmation message
   - Include relevant links or IDs
```

### 에러 응답 처리 (Error Responses)

```markdown
Steps:
1. Call MCP tool
2. On error:
   - Check error type (auth, rate limit, validation, etc.)
   - Provide helpful error message
   - Suggest remediation steps
   - Don't expose internal error details to user
```

### 부분 성공 처리 (Partial Success)

```markdown
Steps:
1. Batch operation with multiple MCP calls
2. Track successes and failures separately
3. Report summary:
   - "Successfully processed 8 of 10 items"
   - "Failed items: [item1, item2] due to [reason]"
   - Suggest retry or manual intervention
```

## 성능 최적화 (Performance Optimization)

### 요청 일괄 처리 (Batching Requests)

**권장 형태: 필터 지정을 통한 단일 조회 처리**
```markdown
Steps:
1. Call mcp__plugin_api_server__search with filters:
   - project_id: "123"
   - status: "active"
   - limit: 100
2. Process all results
```

**피해야 할 형태: 개별 ID 루프 순회 조회 반복**
```markdown
Steps:
1. For each item ID:
   - Call mcp__plugin_api_server__get_item
   - Process item
```

### 결과 캐싱 (Caching Results)

```markdown
Steps:
1. Call expensive MCP operation: mcp__plugin_api_server__analyze
2. Store results in variable for reuse
3. Use cached results for subsequent operations
4. Only re-fetch if data changes
```

### 병렬 도구 호출 (Parallel Tool Calls)

호출 결과 간 종속성이 없는 비연관 명령들은 동시 병렬 발송 형태로 조율하십시오:

```markdown
Steps:
1. Make parallel calls (Claude handles this automatically):
   - mcp__plugin_api_server__get_project
   - mcp__plugin_api_server__get_users
   - mcp__plugin_api_server__get_tags
2. Wait for all to complete
3. Combine results
```

## 연동 권장 사항 (Integration Best Practices)

### 사용자 경험 향상 (User Experience)

**상태 알림 및 피드백 제공:**
```markdown
Steps:
1. Inform user: "Searching Asana tasks..."
2. Call mcp__plugin_asana_asana__asana_search_tasks
3. Show progress: "Found 15 tasks, analyzing..."
4. Present results
```

**오래 걸리는 장시간 연산 제어:**
```markdown
Steps:
1. Warn user: "This may take a minute..."
2. Break into smaller steps with updates
3. Show incremental progress
4. Final summary when complete
```

### 직관적인 에러 메시지 작성 (Error Messages)

**사용자 친화적 에러 가이드라인:**
```
❌ "Could not create task. Please check:
   1. You're logged into Asana
   2. You have access to workspace 'Engineering'
   3. The project 'Q1 Goals' exists"
```

**불친절하고 불명확한 시스템 에러 노출:**
```
❌ "Error: MCP tool returned 403"
```

### 문서화 (Documentation)

**명령어 내부 가이드 파일에 연동하는 외부 MCP 명세 기재:**
```markdown
## MCP Tools Used

This command uses the following Asana MCP tools:
- **asana_search_tasks**: Search for tasks matching criteria
- **asana_create_task**: Create new task with details
- **asana_update_task**: Update existing task properties

Ensure you're authenticated to Asana before running this command.
```

## 도구 사용 모니터링 및 테스트 (Testing Tool Usage)

### 로컬 테스트 수행 (Local Testing)

1. 로컬 개발 경로의 `.mcp.json` 파일에 **MCP 서버 설정을 등록**합니다.
2. 플러그인을 로컬 개발 대상 디렉토리인 `.claude-plugin/`에 **심볼릭 링크 혹은 직접 설치**합니다.
3. `/mcp` 명령어를 수행하여 **해당 도구들이 시스템에 정상 감지되는지 확인**합니다.
4. 해당 도구를 내부에서 유기적으로 호출하도록 코딩한 **마크다운 명령을 가동해 테스트**합니다.
5. 상세 트레이스 및 동작 오류 분석을 위해 `claude --debug` **디버그 출력을 지속 추적**합니다.

### 테스트 시나리오 구성 (Test Scenarios)

**정상 성공 시나리오:**
```markdown
Steps:
1. Create test data in external service
2. Run command that queries this data
3. Verify correct results returned
```

**예외 발생 에러 대응 시나리오:**
```markdown
Steps:
1. Test with missing authentication
2. Test with invalid parameters
3. Test with non-existent resources
4. Verify graceful error handling
```

**엣지 케이스 및 극한 상황 시나리오:**
```markdown
Steps:
1. Test with empty results
2. Test with maximum results
3. Test with special characters
4. Test with concurrent access
```

## 공통 패턴 (Common Patterns)

### 패턴: CRUD 작업 (Pattern: CRUD Operations)

```markdown
---
allowed-tools: [
  "mcp__plugin_api_server__create_item",
  "mcp__plugin_api_server__read_item",
  "mcp__plugin_api_server__update_item",
  "mcp__plugin_api_server__delete_item"
]
---

# Item Management

## Create
Use create_item with required fields...

## Read
Use read_item with item ID...

## Update
Use update_item with item ID and changes...

## Delete
Use delete_item with item ID (ask for confirmation first)...
```

### 패턴: 검색 및 후처리 (Pattern: Search and Process)

```markdown
Steps:
1. **Search**: mcp__plugin_api_server__search with filters
2. **Filter**: Apply additional local filtering if needed
3. **Transform**: Process each result
4. **Present**: Format and display to user
```

### 패턴: 다단계 워크플로우 (Pattern: Multi-Step Workflow)

```markdown
Steps:
1. **Setup**: Gather all required information
2. **Validate**: Check data completeness
3. **Execute**: Chain of MCP tool calls:
   - Create parent resource
   - Create child resources
   - Link resources together
   - Add metadata
4. **Verify**: Confirm all steps succeeded
5. **Report**: Provide summary to user
```

## 문제 해결 (Troubleshooting)

### 도구를 사용할 수 없는 상태일 때 (Tools Not Available)

**확인할 사항:**
- `.mcp.json`이나 `plugin.json`에 서버 정보가 정확한 구조로 등록되어 있는지 확인합니다.
- 서버 프로세스가 현재 무리 없이 실행 수립되어 가용한 상태인지 확인합니다 (`/mcp` 명령어 체크).
- 스키마에 정의된 이름과 마크다운 본문에 기재된 도구 식별자의 대소문자 매칭이 완전 일치하는지 확인합니다.
- 설정을 수정한 후에는 Claude Code 터미널 세션을 완전히 껐다가 다시 기동시켰는지 확인합니다.

### 도구 호출 자체가 실패할 때 (Tool Calls Failing)

**확인할 사항:**
- 원격 서비스 연동 토큰이 만료되지 않고 유효한지 인증 상태를 점검합니다.
- 송신 매개변수의 구조나 자료형 타입이 JSON inputSchema 명세 구조와 부합하는지 점검합니다.
- 필수 요구 매개변수 항목 누락 여부를 확인합니다.
- 상세 정보 확인을 위해 `claude --debug` 시스템 트레이스 로그를 확인합니다.

### 성능 병목 및 지연 문제 (Performance Issues)

**확인할 사항:**
- 루프 순회 순차 개별 호출 방식을 단일 요청 기반의 일괄 처리 쿼리 모델로 개선 가능한지 점검합니다.
- 자주 호출되며 변화가 적은 조회 목적 데이터에 캐시 데이터 사용 패턴을 적용했는지 점검합니다.
- 중복되거나 불필요한 도구 조회를 지속 발송하고 있지 않은지 확인합니다.
- 선후 의존 관계가 없는 조회의 경우 병렬 동시 발송 설정을 적극 활용했는지 점검합니다.

## 결론 (Conclusion)

안정적인 MCP 연동 구현을 위한 핵심 요건:
1. `/mcp`로 스키마 명세를 **완벽히 이해하고 설계**할 것
2. 필요한 도구 정보만 지명하여 **안전하게 사전 허용(`allowed-tools`)**할 것
3. 어떠한 시스템 예외나 인증 만료 장애가 오더라도 **직관적인 에러 복구**를 구성할 것
4. 무분별한 조회를 방지하고 **캐싱 및 일괄 조회를 지향**할 것
5. 진행 단계 알림 및 우아한 예외 처리를 통해 **UX 완성도를 제고**할 것
6. 릴리스 배포 전에 로컬 샌드박스 영역에서 **충분한 예외 시나리오 교차 테스트를 수행**할 것

위 모범 패턴들을 숙지하고 준수하여 명령어 및 에이전트가 완벽히 동작하는 탄탄한 MCP 플러그인 생태계를 설계하십시오.
