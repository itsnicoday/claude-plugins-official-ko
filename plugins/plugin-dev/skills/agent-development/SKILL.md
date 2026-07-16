---
name: agent-development
description: This skill should be used when the user asks to "create an agent", "add an agent", "write a subagent", "agent frontmatter", "when to use description", "agent examples", "agent tools", "agent colors", "autonomous agent", or needs guidance on agent structure, system prompts, triggering conditions, or agent development best practices for Claude Code plugins.
version: 0.1.0
---

# Claude Code 플러그인을 위한 에이전트 개발 (Agent Development for Claude Code Plugins)

## 개요 (Overview)

에이전트는 복잡하고 여러 단계로 이루어진 작업을 독립적으로 처리하는 자율적인 서브프로세스입니다. 에이전트 구조, 트리거 조건, 그리고 시스템 프롬프트 디자인을 이해하면 강력한 자율 기능을 구축할 수 있습니다.

**핵심 개념:**
- 에이전트는 자율적인 작업을 위한 것(FOR)이며, 명령(commands)은 사용자가 시작하는 액션을 위한 것(FOR)입니다.
- YAML frontmatter를 사용하는 마크다운 파일 형식
- 예시를 포함한 description 필드를 통한 트리거링
- 시스템 프롬프트가 에이전트 행동을 정의함
- 모델 및 색상 커스터마이징

## 에이전트 파일 구조 (Agent File Structure)

### 전체 포맷 (Complete Format)

```markdown
---
name: agent-identifier
description: Use this agent when [triggering conditions]. Typical triggers include [scenario 1 in prose], [scenario 2 in prose], and [scenario 3 in prose]. See "When to invoke" in the agent body for worked scenarios.
model: inherit
color: blue
tools: ["Read", "Write", "Grep"]
---

You are [agent role description]...

## When to invoke

[Two to four representative scenarios written as prose, e.g.:]
- **[Scenario name].** [What the situation looks like and what the agent should do.]
- **[Scenario name].** [Same.]

**Your Core Responsibilities:**
1. [Responsibility 1]
2. [Responsibility 2]

**Analysis Process:**
[Step-by-step workflow]

**Output Format:**
[What to return]
```

## Frontmatter 필드 (Frontmatter Fields)

### name (필수)

네임스페이스 지정 및 호출에 사용되는 에이전트 식별자입니다.

**형식:** 소문자, 숫자, 하이픈만 허용
**길이:** 3~50자
**패턴:** 반드시 알파벳이나 숫자로 시작하고 끝나야 함

**좋은 예:**
- `code-reviewer`
- `test-generator`
- `api-docs-writer`
- `security-analyzer`

**나쁜 예:**
- `helper` (너무 일반적임)
- `-agent-` (하이픈으로 시작/끝남)
- `my_agent` (언더스코어 허용되지 않음)
- `ag` (너무 짧음, 3자 미만)

### description (필수)

Claude가 언제 이 에이전트를 트리거해야 하는지 정의합니다. **이 필드는 가장 중요한 필드입니다.** 에이전트가 등록될 때마다 컨텍스트에 로드되므로, 하네스(harness)가 언제 디스패치할지 결정할 수 있습니다.

**필수 포함 사항:**
1. 트리거 조건 ("Use this agent when...")
2. 일반적인 트리거 시나리오의 짧은 줄글 요약
3. 구체적인 작동 시나리오가 들어있는 에이전트 본문의 "When to invoke" 섹션을 가리키는 포인터

**형식:**
```
Use this agent when [conditions]. Typical triggers include [scenario 1 in prose], [scenario 2 in prose], and [scenario 3 in prose]. See "When to invoke" in the agent body for worked scenarios.
```

**모범 사례:**
- 줄글 요약에 2~4개의 트리거 시나리오를 명시하십시오.
- 자발적 트리거링(어시스턴트 스스로 호출)과 반응적 트리거링(사용자 요청)을 모두 다루십시오.
- 동일한 의도의 다양한 표현 방식을 포괄하십시오.
- 에이전트를 사용하지 말아야 할 때를 구체적으로 명시하십시오.
- 본문의 "When to invoke" 아래에 구체적인 시나리오를 줄글 설명이 포함된 글머리 기호 목록으로 입력하십시오.

### model (필수)

에이전트가 사용할 모델입니다.

**옵션:**
- `inherit` - 부모 에이전트와 동일한 모델 사용 (권장)
- `sonnet` - Claude Sonnet (균형 잡힌 성능)
- `opus` - Claude Opus (가장 강력함, 고비용)
- `haiku` - Claude Haiku (빠름, 저비용)

**권장사항:** 에이전트에 특정 모델 기능이 특별히 필요한 경우가 아니라면 `inherit`을 사용하십시오.

### color (필수)

UI에서 에이전트를 시각적으로 구별하는 식별자입니다.

**옵션:** `blue`, `cyan`, `green`, `yellow`, `magenta`, `red`

**가이드라인:**
- 동일한 플러그인 내의 서로 다른 에이전트들에는 각기 다른 색상을 지정하십시오.
- 유사한 에이전트 유형에는 일관된 색상을 사용하십시오.
- Blue/cyan (파란색/청록색): 분석, 리뷰
- Green (초록색): 성공 지향적 작업
- Yellow (노란색): 주의, 검증
- Red (빨간색): 중요 작업, 보안
- Magenta (자홍색): 창의적 작업, 생성

### tools (선택)

에이전트가 사용할 수 있는 도구를 제한합니다.

**형식:** 도구 이름들의 배열

```yaml
tools: ["Read", "Write", "Grep", "Bash"]
```

**기본값:** 생략할 경우, 에이전트는 모든 도구에 액세스할 수 있습니다.

**모범 사례:** 필요한 최소한의 도구로 제한하십시오 (최소 권한 원칙).

**일반적인 도구 세트:**
- 읽기 전용 분석: `["Read", "Grep", "Glob"]`
- 코드 생성: `["Read", "Write", "Grep"]`
- 테스팅: `["Read", "Bash", "Grep"]`
- 전체 액세스: 필드를 생략하거나 `["*"]` 사용

## 시스템 프롬프트 디자인 (System Prompt Design)

마크다운 본문은 에이전트의 시스템 프롬프트가 됩니다. 에이전트를 직접 지칭하는 2인칭으로 작성하십시오.

### 구조 (Structure)

```markdown
You are [role] specializing in [domain].

**Your Core Responsibilities:**
1. [Primary responsibility]
2. [Secondary responsibility]
3. [Additional responsibilities...]

**Analysis Process:**
1. [Step one]
2. [Step two]
3. [Step three]
[...]

**Quality Standards:**
- [Standard 1]
- [Standard 2]

**Output Format:**
Provide results in this format:
- [What to include]
- [How to structure]

**Edge Cases:**
Handle these situations:
- [Edge case 1]: [How to handle]
- [Edge case 2]: [How to handle]
```

### 모범 사례 (Best Practices)

✅ **권장 사항 (DO):**
- 2인칭으로 작성하십시오 ("You are...", "You will...")
- 책임을 구체적으로 명시하십시오
- 단계별 프로세스를 제공하십시오
- 출력 형식을 정의하십시오
- 품질 표준을 포함하십시오
- 예외 사례를 처리하십시오
- 10,000자 미만으로 유지하십시오

❌ **금지 사항 (DON'T):**
- 1인칭으로 작성하지 마십시오 ("I am...", "I will...")
- 모호하거나 일반적인 표현을 피하십시오
- 프로세스 단계를 누락하지 마십시오
- 출력 형식을 정의하지 않은 채로 두지 마십시오
- 품질 안내를 생략하지 마십시오
- 오류 사례를 무시하지 마십시오

## 에이전트 생성하기 (Creating Agents)

### 방법 1: AI 지원 생성 (Method 1: AI-Assisted Generation)

다음 프롬프트 패턴을 사용하십시오 (Claude Code에서 추출):

```
Create an agent configuration based on this request: "[YOUR DESCRIPTION]"

Requirements:
1. Extract core intent and responsibilities
2. Design expert persona for the domain
3. Create comprehensive system prompt with:
   - Clear behavioral boundaries
   - Specific methodologies
   - Edge case handling
   - Output format
   - A "When to invoke" section listing 2-4 trigger scenarios as prose bullets
4. Create identifier (lowercase, hyphens, 3-50 chars)
5. Write description with triggering conditions and a short prose summary of trigger scenarios

Return JSON with:
{
  "identifier": "agent-name",
  "whenToUse": "Use this agent when... Typical triggers include [...]. See \"When to invoke\" in the agent body.",
  "systemPrompt": "You are..."
}
```

그런 다음 frontmatter를 포함한 에이전트 파일 형식으로 변환합니다.

전체 템플릿은 `examples/agent-creation-prompt.md`를 참고하십시오.

### 방법 2: 수동 생성 (Method 2: Manual Creation)

1. 에이전트 식별자를 선택합니다 (소문자, 하이픈 구성, 3~50자).
2. 예시를 포함하여 설명을 작성합니다.
3. 모델을 선택합니다 (일반적으로 `inherit`).
4. 시각적 구분을 위한 색상을 선택합니다.
5. 도구를 정의합니다 (액세스를 제한하려는 경우).
6. 위의 구조에 따라 시스템 프롬프트를 작성합니다.
7. `agents/agent-name.md`로 저장합니다.

## 검증 규칙 (Validation Rules)

### 식별자 검증 (Identifier Validation)

```
✅ Valid: code-reviewer, test-gen, api-analyzer-v2
❌ Invalid: ag (too short), -start (starts with hyphen), my_agent (underscore)
```

**규칙:**
- 3~50자
- 소문자, 숫자, 하이픈만 허용
- 반드시 알파벳이나 숫자로 시작하고 끝나야 함
- 언더스코어, 공백 또는 특수 문자 불가

### 설명(Description) 검증

**길이:** 10~5,000자
**필수 포함 사항:** 트리거 조건 및 예시
**가장 좋음:** 2~4개의 예시가 포함된 200~1,000자

### 시스템 프롬프트 검증

**길이:** 20~10,000자
**가장 좋음:** 500~3,000자
**구조:** 명확한 책임, 프로세스, 출력 형식

## 에이전트 조직화 (Agent Organization)

### 플러그인 에이전트 디렉토리 (Plugin Agents Directory)

```
plugin-name/
└── agents/
    ├── analyzer.md
    ├── reviewer.md
    └── generator.md
```

`agents/` 내의 모든 `.md` 파일은 자동으로 감지됩니다.

### 네임스페이스 지정 (Namespacing)

에이전트는 자동으로 네임스페이스가 지정됩니다:
- 단일 플러그인: `agent-name`
- 하위 디렉토리가 있는 경우: `plugin:subdir:agent-name`

## 에이전트 테스트하기 (Testing Agents)

### 트리거 테스트 (Test Triggering)

에이전트가 올바르게 트리거되는지 확인하기 위해 테스트 시나리오를 만듭니다:

1. 특정 트리거 예시를 명시하여 에이전트를 작성합니다.
2. 테스트에서 예시와 유사한 구문을 사용합니다.
3. Claude가 에이전트를 로드하는지 확인합니다.
4. 에이전트가 기대한 기능을 제공하는지 검증합니다.

### 시스템 프롬프트 테스트 (Test System Prompt)

시스템 프롬프트가 완전한지 확인하십시오:

1. 에이전트에게 일반적인 작업을 부여합니다.
2. 프로세스 단계를 따르는지 확인합니다.
3. 출력 형식이 올바른지 검증합니다.
4. 프롬프트에 언급된 예외 사례를 테스트합니다.
5. 품질 표준이 충족되는지 확인합니다.

## 빠른 참조 (Quick Reference)

### 최소 에이전트 구성 (Minimal Agent)

```markdown
---
name: simple-agent
description: Use this agent when [condition]. Typical triggers include [trigger 1] and [trigger 2]. See "When to invoke" in the agent body.
model: inherit
color: blue
---

You are an agent that [does X].

## When to invoke

- **[Scenario A].** [Description.]
- **[Scenario B].** [Description.]

Process:
1. [Step 1]
2. [Step 2]

Output: [What to provide]
```

### Frontmatter 필드 요약 (Frontmatter Fields Summary)

| 필드 | 필수 여부 | 형식 | 예시 |
|-------|----------|--------|---------|
| name | 예 | 소문자 및 하이픈 | code-reviewer |
| description | 예 | 줄글 형식의 트리거 | Use when... Typical triggers include... |
| model | 예 | inherit/sonnet/opus/haiku | inherit |
| color | 예 | 색상 이름 | blue |
| tools | 아니요 | 도구 이름들의 배열 | ["Read", "Grep"] |

### 모범 사례 (Best Practices)

**권장 사항 (DO):**
- ✅ description에 2~4개의 트리거 시나리오를 명명하십시오 (줄글 형태).
- ✅ 상세 작동 시나리오는 본문의 "When to invoke" 섹션에 줄글 글머리 기호 형태로 입력하십시오.
- ✅ 구체적인 트리거 조건을 작성하십시오.
- ✅ 특별한 요구사항이 없다면 모델에 `inherit`을 사용하십시오.
- ✅ 적절한 도구를 선택하십시오 (최소 권한).
- ✅ 명확하고 구조화된 시스템 프롬프트를 작성하십시오.
- ✅ 에이전트 트리거링을 철저하게 테스트하십시오.

**금지 사항 (DON'T):**
- ❌ 트리거 시나리오 없이 모호하고 일반적인 설명만 사용하지 마십시오.
- ❌ 트리거 조건을 생략하지 마십시오.
- ❌ 모든 에이전트에 동일한 색상을 지정하지 마십시오.
- ❌ 불필요한 도구 권한을 부여하지 마십시오.
- ❌ 모호한 시스템 프롬프트를 작성하지 마십시오.
- ❌ 테스트를 건너뛰지 마십시오.

## 추가 리소스 (Additional Resources)

### 참조 파일 (Reference Files)

상세한 가이드는 다음 파일을 참조하십시오:

- **`references/system-prompt-design.md`** - 전체 시스템 프롬프트 패턴
- **`references/triggering-examples.md`** - 예시 형식 및 모범 사례
- **`references/agent-creation-system-prompt.md`** - Claude Code의 실제 프롬프트

### 예시 파일 (Example Files)

`examples/` 내의 작동 가능한 예시들:

- **`agent-creation-prompt.md`** - AI 지원 에이전트 생성 템플릿
- **`complete-agent-examples.md`** - 다양한 유스케이스를 위한 전체 에이전트 예시

### 유틸리티 스크립트 (Utility Scripts)

`scripts/` 내의 개발 도구들:

- **`validate-agent.sh`** - 에이전트 파일 구조 검증
- **`test-agent-trigger.sh`** - 에이전트 트리거의 올바른 작동 여부 테스트

## 구현 워크플로우 (Implementation Workflow)

플러그인용 에이전트를 생성하는 절차:

1. 에이전트의 목적과 트리거 조건을 정의합니다.
2. 생성 방법(AI 지원 또는 수동)을 선택합니다.
3. `agents/agent-name.md` 파일을 생성합니다.
4. 필수 필드를 모두 갖춘 frontmatter를 작성합니다.
5. 모범 사례에 따라 시스템 프롬프트를 작성합니다.
6. description에 2~4개의 트리거 시나리오를 명시하고(줄글), 본문의 "When to invoke" 섹션에서 이를 상세히 설명합니다.
7. `scripts/validate-agent.sh`를 사용해 검증합니다.
8. 실제 시나리오로 트리거링을 테스트합니다.
9. 플러그인의 README에 에이전트를 문서화합니다.

자율적인 작동을 위해 명확한 트리거 조건과 포괄적인 시스템 프롬프트 작성에 초점을 맞추십시오.
