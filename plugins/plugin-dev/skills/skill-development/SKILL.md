---
name: skill-development
description: This skill should be used when the user wants to "create a skill", "add a skill to plugin", "write a new skill", "improve skill description", "organize skill content", or needs guidance on skill structure, progressive disclosure, or skill development best practices for Claude Code plugins.
version: 0.1.0
---

# Claude Code 플러그인을 위한 스킬 개발 (Skill Development for Claude Code Plugins)

이 스킬은 Claude Code 플러그인을 위한 효과적인 스킬을 제작하는 데 필요한 지침을 제공합니다.

## 스킬 소개 (About Skills)

스킬은 전문 지식, 워크플로우 및 도구 통합을 제공하여 Claude의 기능을 확장하는 모듈식의 독립적인 패키지입니다. 특정 분야나 작업에 대한 "온보딩 가이드"라고 생각하면 됩니다. 모델이 완전히 보유할 수 없는 절차적 지식을 장착하여 Claude를 범용 에이전트에서 특화된 에이전트로 변환해 줍니다.

### 스킬이 제공하는 것 (What Skills Provide)

1. 전문 워크플로우 - 특정 분야를 위한 다단계 절차
2. 도구 통합 - 특정 파일 형식이나 API로 작업하기 위한 지침
3. 도메인 전문 지식 - 회사 고유의 지식, 스키마, 비즈니스 로직
4. 번들 리소스 - 복잡하고 반복적인 작업을 위한 스크립트, 참조 자료 및 자산(에셋)

### 스킬의 구성 (Anatomy of a Skill)

모든 스킬은 필수 파일인 `SKILL.md`와 선택 사항인 번들 리소스로 구성됩니다:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation intended to be loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

#### SKILL.md (필수)

**메타데이터 품질:** YAML 프론트매터의 `name`과 `description`은 Claude가 언제 이 스킬을 사용할지 결정합니다. 스킬이 수행하는 작업과 사용 시기를 구체적으로 명시하세요. 3인칭을 사용하세요(예: "이 스킬은 ~할 때 사용해야 합니다..." 사용. "이럴 때 이 스킬을 사용하세요..."는 피함).

#### 번들 리소스 (선택 사항) (Bundled Resources (optional))

##### 스크립트 (`scripts/`)

결정론적인 신뢰성이 필요하거나 반복적으로 재작성되는 작업을 위한 실행 가능한 코드(Python/Bash 등)입니다.

- **포함할 시기**: 동일한 코드가 반복해서 재작성되거나 결정론적인 신뢰성이 필요할 때
- **예시**: PDF 회전 작업을 위한 `scripts/rotate_pdf.py`
- **장점**: 토큰 효율성, 결정론적 동작, 컨텍스트에 로드하지 않고도 실행 가능할 수 있음
- **참고**: 스크립트는 여전히 패치 적용이나 환경별 조정을 위해 Claude가 읽어야 할 수 있습니다.

##### 참조 (`references/`)

Claude의 프로세스와 사고에 정보를 제공하기 위해 필요에 따라 컨텍스트에 로드되도록 고안된 문서 및 참조 자료입니다.

- **포함할 시기**: Claude가 작업 중에 참조해야 하는 문서의 경우
- **예시**: 금융 스키마를 위한 `references/finance.md`, 회사 NDA 템플릿을 위한 `references/mnda.md`, 회사 정책을 위한 `references/policies.md`, API 사양을 위한 `references/api_docs.md`
- **사용 사례**: 데이터베이스 스키마, API 문서, 도메인 지식, 회사 정책, 세부 워크플로우 가이드
- **장점**: `SKILL.md`를 가볍게 유지하며, Claude가 필요하다고 판단할 때만 로드됨
- **권장 사항**: 파일이 큰 경우(>10k 단어), `SKILL.md`에 grep 검색 패턴을 포함하세요.
- **중복 방지**: 정보는 `SKILL.md` 또는 참조 파일 중 하나에만 있어야 하며, 양쪽에 모두 있어서는 안 됩니다. 세부 정보는 컨텍스트 창을 독점하지 않으면서도 정보를 검색할 수 있도록 `SKILL.md`보다는 참조 파일에 두는 것을 권장합니다. `SKILL.md`에는 필수적인 절차 지침과 워크플로우 안내만 남겨두고, 세부 참조 자료, 스키마, 예시는 참조 파일로 이동하세요.

##### 에셋 (`assets/`)

컨텍스트에 로드하기 위한 것이 아니라, Claude가 생성하는 출력물 내에서 사용하기 위한 파일입니다.

- **포함할 시기**: 스킬이 최종 출력물에 사용될 파일을 필요로 할 때
- **예시**: 브랜드 자산을 위한 `assets/logo.png`, 파워포인트 템플릿을 위한 `assets/slides.pptx`, HTML/React 보일러플레이트를 위한 `assets/frontend-template/`, 타이포그래피를 위한 `assets/font.ttf`
- **사용 사례**: 템플릿, 이미지, 아이콘, 보일러플레이트 코드, 폰트, 복사되거나 수정되는 샘플 문서
- **장점**: 출력 리소스를 문서와 분리하여 Claude가 파일을 컨텍스트에 로드하지 않고도 사용할 수 있도록 함

### 점진적 공개 설계 원칙 (Progressive Disclosure Design Principle)

스킬은 컨텍스트를 효율적으로 관리하기 위해 3단계 로딩 시스템을 사용합니다:

1. **메타데이터 (name + description)** - 항상 컨텍스트에 있음 (~100 단어)
2. **SKILL.md 본문** - 스킬이 트리거될 때 (<5k 단어)
3. **번들 리소스** - Claude가 필요할 때 (제한 없음*)

*스크립트는 컨텍스트 창을 읽지 않고도 실행할 수 있으므로 제한이 없습니다.

## 스킬 제작 프로세스 (Skill Creation Process)

스킬을 제작하려면 "스킬 제작 프로세스"를 순서대로 따르며, 적용할 수 없는 명확한 이유가 있는 경우에만 단계를 건너뜁니다.

### 1단계: 구체적인 예시를 통한 스킬 이해 (Step 1: Understanding the Skill with Concrete Examples)

스킬의 사용 패턴이 이미 명확히 이해된 경우에만 이 단계를 건너뜁니다. 기존 스킬을 작업할 때도 이 단계는 여전히 유용합니다.

효과적인 스킬을 제작하려면 스킬이 어떻게 사용될지에 대한 구체적인 예시를 명확히 이해해야 합니다. 이러한 이해는 직접적인 사용자 예시나 사용자 피드백으로 검증된 생성된 예시를 통해 얻을 수 있습니다.

예를 들어, 이미지 편집기(image-editor) 스킬을 구축할 때 관련된 질문은 다음과 같습니다:

- "이미지 편집기 스킬은 어떤 기능을 지원해야 하나요? 편집, 회전, 그 외에 다른 기능이 있나요?"
- "이 스킬이 어떻게 사용될지에 대한 예시를 들어주실 수 있나요?"
- "사용자가 '이 이미지에서 적목 현상을 제거해줘' 또는 '이 이미지를 회전해줘'와 같은 요청을 할 것 같습니다. 그 외에 생각하시는 사용 방식이 있나요?"
- "사용자가 어떤 말을 했을 때 이 스킬이 트리거되어야 하나요?"

사용자에게 부담을 주지 않으려면 한 메시지에서 너무 많은 질문을 하지 마세요. 가장 중요한 질문부터 시작하고 더 나은 효과를 위해 필요에 따라 후속 질문을 하세요.

스킬이 지원해야 하는 기능에 대한 명확한 방향이 잡히면 이 단계를 완료합니다.

### 2단계: 재사용 가능한 스킬 콘텐츠 계획 (Step 2: Planning the Reusable Skill Contents)

구체적인 예시를 효과적인 스킬로 전환하기 위해 각 예시를 다음과 같이 분석합니다:

1. 처음부터 예시를 어떻게 실행할지 고려하기
2. 이러한 워크플로우를 반복해서 실행할 때 어떤 스크립트, 참조 자료, 에셋이 도움이 될지 식별하기

예시: "이 PDF 회전을 도와줘"와 같은 쿼리를 처리하기 위해 `pdf-editor` 스킬을 구축할 때 분석 결과는 다음과 같습니다:

1. PDF를 회전하려면 매번 동일한 코드를 재작성해야 합니다.
2. 스킬에 `scripts/rotate_pdf.py` 스크립트를 저장해 두면 유용할 것입니다.

예시: "할 일(todo) 앱을 만들어줘" 또는 "내 걸음 수를 추적할 대시보드를 만들어줘"와 같은 쿼리를 위해 `frontend-webapp-builder` 스킬을 설계할 때 분석 결과는 다음과 같습니다:

1. 프론트엔드 웹앱을 작성하려면 매번 동일한 보일러플레이트 HTML/React가 필요합니다.
2. 보일러플레이트 HTML/React 프로젝트 파일이 포함된 `assets/hello-world/` 템ำ릿을 스킬에 저장해 두면 유용할 것입니다.

예시: "오늘 로그인한 사용자는 몇 명인가요?"와 같은 쿼리를 처리하기 위해 `big-query` 스킬을 구축할 때 분석 결과는 다음과 같습니다:

1. BigQuery를 쿼리하려면 매번 테이블 스키마와 관계를 재검색해야 합니다.
2. 테이블 스키마를 문서화한 `references/schema.md` 파일을 스킬에 저장해 두면 유용할 것입니다.

**Claude Code 플러그인의 경우:** 훅(hooks) 스킬을 구축할 때 분석 결과는 다음과 같습니다:
1. 개발자는 반복적으로 `hooks.json`을 검증하고 훅 스크립트를 테스트해야 합니다.
2. `scripts/validate-hook-schema.sh` 및 `scripts/test-hook.sh` 유틸리티가 도움이 될 것입니다.
3. `SKILL.md`가 비대해지는 것을 방지하기 위해 세부 훅 패턴은 `references/patterns.md`에 배치합니다.

스킬 콘텐츠를 설정하려면 각 구체적인 예시를 분석하여 스크립트, 참조 자료 및 에셋을 포함할 재사용 가능한 리소스 목록을 만듭니다.

### 3단계: 스킬 구조 생성 (Step 3: Create Skill Structure)

Claude Code 플러그인의 경우 다음과 같이 스킬 디렉토리 구조를 생성합니다:

```bash
mkdir -p plugin-name/skills/skill-name/{references,examples,scripts}
touch plugin-name/skills/skill-name/SKILL.md
```

**참고:** `init_skill.py`를 사용하는 일반적인 스킬 제작기와 달리, 플러그인 스킬은 더 간단한 수동 구조를 통해 플러그인의 `skills/` 디렉토리에 직접 생성됩니다.

### 4단계: 스킬 편집 (Step 4: Edit the Skill)

(새로 생성되었거나 기존의) 스킬을 편집할 때, 이 스킬은 Claude의 다른 인스턴스가 사용하기 위해 만들어지는 것임을 기억하십시오. Claude에게 유익하고 분명하지 않은 정보를 포함하는 데 집중하십시오. 다른 Claude 인스턴스가 이러한 작업을 더 효과적으로 실행하는 데 도움이 될 절차적 지식, 도메인 특정 세부 정보 또는 재사용 가능한 에셋이 무엇인지 고려하십시오.

#### 재사용 가능한 스킬 콘텐츠부터 시작하기

구현을 시작하려면 위에서 식별한 재사용 가능한 리소스(`scripts/`, `references/`, `assets/` 파일)부터 시작하십시오. 이 단계는 사용자의 입력이 필요할 수 있습니다. 예를 들어, `brand-guidelines` 스킬을 구현할 때 사용자는 `assets/`에 저장할 브랜드 자산이나 템플릿, 또는 `references/`에 저장할 문서를 제공해야 할 수 있습니다.

또한 스킬에 필요하지 않은 예시 파일과 디렉토리는 삭제하십시오. 실제로 필요한 디렉토리(`references/`, `examples/`, `scripts/`)만 생성하십시오.

#### Update SKILL.md

**작성 스타일:** 2인칭이 아닌 **명령형/부정사형**(동사 먼저 시작하는 지침)을 사용하여 전체 스킬을 작성하십시오. 객관적이고 교육적인 언어를 사용하십시오(예: "X를 수행해야 합니다" 또는 "X를 수행해야 하는 경우" 대신 "X를 달성하려면 Y를 수행하십시오"). 이는 AI가 활용할 때 일관성과 명확성을 유지하게 해줍니다.

**설명 (프론트매터):** 특정 트리거 구문과 함께 3인칭 형식을 사용하십시오:

```yaml
---
name: Skill Name
description: This skill should be used when the user asks to "specific phrase 1", "specific phrase 2", "specific phrase 3". Include exact phrases users would say that should trigger this skill. Be concrete and specific.
version: 0.1.0
---
```

**좋은 설명 예시:**
```yaml
description: This skill should be used when the user asks to "create a hook", "add a PreToolUse hook", "validate tool use", "implement prompt-based hooks", or mentions hook events (PreToolUse, PostToolUse, Stop).
```

**나쁜 설명 예시:**
```yaml
description: Use this skill when working with hooks.  # 잘못된 인칭, 모호함
description: Load when user needs hook help.  # 3인칭이 아님
description: Provides hook guidance.  # 트리거 구문 없음
```

`SKILL.md` 본문을 완성하려면 다음 질문에 대답하십시오:

1. 몇 문장으로 요약한 스킬의 목적은 무엇인가요?
2. 스킬은 언제 사용해야 하나요? (프론트매터 설명에 구체적인 트리거와 함께 포함)
3. 실제로 Claude는 스킬을 어떻게 사용해야 하나요? 위에서 개발한 모든 재사용 가능한 스킬 콘텐츠는 Claude가 어떻게 사용하는지 알 수 있도록 참조되어야 합니다.

**SKILL.md를 가볍게 유지하기:** 본문은 1,500~2,000 단어를 목표로 하십시오. 상세 내용은 `references/` 디렉토리로 이동하세요:
- 상세 패턴 → `references/patterns.md`
- 고급 기법 → `references/advanced.md`
- 마이그레이션 가이드 → `references/migration.md`
- API 참조 → `references/api-reference.md`

**SKILL.md에서 리소스 참조하기:**
```markdown
## Additional Resources

### Reference Files

For detailed patterns and techniques, consult:
- **`references/patterns.md`** - Common patterns
- **`references/advanced.md`** - Advanced use cases

### Example Files

Working examples in `examples/`:
- **`example-script.sh`** - Working example
```

### 5단계: 검증 및 테스트 (Step 5: Validate and Test)

**플러그인 스킬의 경우, 검증은 일반 스킬과 다릅니다:**

1. **구조 확인**: `plugin-name/skills/skill-name/` 디렉토리에 스킬 폴더 확인
2. **SKILL.md 검증**: name과 description 필드가 포함된 프론트매터 존재 여부
3. **트리거 구문 확인**: 설명에 구체적인 사용자 쿼리 포함 여부
4. **작성 스타일 검증**: 본문이 2인칭이 아닌 명령형/부정사형으로 작성되었는지 확인
5. **점진적 공개 테스트**: SKILL.md가 슬림한지(~1,500-2,000 단어), 상세 내용은 references/에 있는지 확인
6. **참조 확인**: 참조된 모든 파일이 실제로 존재하는지 확인
7. **예시 검증**: 예시가 완전하고 올바른지 확인
8. **스크립트 테스트**: 스크립트가 실행 가능하고 올바르게 작동하는지 확인

**skill-reviewer 에이전트 사용:**
```
Ask: "Review my skill and check if it follows best practices"
```

skill-reviewer 에이전트는 설명의 품질, 콘텐츠 구성 및 점진적 공개 여부를 확인합니다.

### 6단계: 반복 개선 (Step 6: Iterate)

스킬을 테스트한 후 사용자가 개선 사항을 요청할 수 있습니다. 이는 종종 스킬을 사용한 직후, 스킬이 어떻게 작동했는지에 대한 생생한 컨텍스트가 있을 때 발생합니다.

**반복 워크플로우:**
1. 실제 작업에 스킬 사용
2. 문제점이나 비효율성 발견
3. `SKILL.md` 또는 번들 리소스를 어떻게 업데이트해야 하는지 파악
4. 변경 사항을 구현하고 다시 테스트

**공통 개선 사항:**
- 설명에 포함된 트리거 구문 강화
- `SKILL.md`에서 긴 섹션을 `references/`로 이동
- 누락된 예시 또는 스크립트 추가
- 모호한 지침 명확화
- 예외 케이스 처리 추가

## 플러그인 전용 고려 사항 (Plugin-Specific Considerations)

### 플러그인 내 스킬 위치 (Skill Location in Plugins)

플러그인 스킬은 플러그인의 `skills/` 디렉토리에 위치합니다:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
├── agents/
└── skills/
    └── my-skill/
        ├── SKILL.md
        ├── references/
        ├── examples/
        └── scripts/
```

### 자동 감지 (Auto-Discovery)

Claude Code는 스킬을 자동으로 감지합니다:
- `skills/` 디렉토리를 스캔합니다.
- `SKILL.md`를 포함하는 하위 디렉토리를 찾습니다.
- 스킬 메타데이터(name + description)를 항상 로드합니다.
- 스킬이 트리거될 때 `SKILL.md` 본문을 로드합니다.
- 필요할 때 references/examples를 로드합니다.

### 패키징 불필요 (No Packaging Needed)

플러그인 스킬은 별도의 ZIP 파일이 아니라 플러그인의 일부로 배포됩니다. 사용자는 플러그인을 설치할 때 스킬을 받게 됩니다.

### 플러그인 내 테스트 (Testing in Plugins)

플러그인을 로컬에 설치하여 스킬을 테스트합니다:

```bash
# --plugin-dir를 지정하여 테스트
cc --plugin-dir /path/to/plugin

# 스킬을 트리거할 만한 질문을 던져보십시오.
# 스킬이 올바르게 로드되는지 확인합니다.
```

## Plugin-Dev의 예시 (Examples from Plugin-Dev)

권장 사항 예시로 이 플러그인의 스킬들을 학습해 보십시오:

**hook-development 스킬:**
- 우수한 트리거 구문: "create a hook", "add a PreToolUse hook" 등
- 슬림한 SKILL.md (1,651 단어)
- 세부 내용을 담은 3개의 `references/` 파일
- 작동하는 훅에 대한 3개의 `examples/`
- 3개의 `scripts/` 유틸리티

**agent-development 스킬:**
- 강력한 트리거: "create an agent", "agent frontmatter" 등
- 집중도 높은 SKILL.md (1,438 단어)
- Claude Code의 AI 생성 프롬프트가 포함된 참조 자료
- 완전한 에이전트 예시

**plugin-settings 스킬:**
- 구체적인 트리거: "plugin settings", ".local.md files", "YAML frontmatter"
- 실제 구현 사례를 보여주는 참조 자료 (multi-agent-swarm, ralph-loop)
- 작동하는 파싱 스크립트

각 스킬은 점진적 공개와 강력한 트리거 설정을 잘 보여줍니다.

## 실무에서의 점진적 공개 (Progressive Disclosure in Practice)

### SKILL.md에 들어갈 내용 (What Goes in SKILL.md)

**포함할 내용 (스킬 트리거 시 항상 로드됨):**
- 핵심 개념 및 개요
- 필수 절차 및 워크플로우
- 빠른 참조 테이블
- references/examples/scripts에 대한 안내 포인터
- 가장 공통적인 사용 사례

**3,000 단어 미만으로 유지, 이상적으로는 1,500-2,000 단어**

### references/에 들어갈 내용 (What Goes in references/)

**references/로 이동할 내용 (필요할 때 로드됨):**
- 상세 패턴 및 고급 기법
- 종합적인 API 문서
- 마이그레이션 가이드
- 예외 상황 및 문제 해결
- 광범위한 예시 및 튜토리얼

**각 참조 파일의 크기는 크게 작성할 수 있습니다 (2,000-5,000+ 단어)**

### examples/에 들어갈 내용 (What Goes in examples/)

**작동하는 코드 예시:**
- 완전하고 실행 가능한 스크립트
- 구성 파일
- 템플릿 파일
- 실제 사용 예시

**사용자는 이를 직접 복사하고 조정할 수 있습니다.**

### scripts/에 들어갈 내용 (What Goes in scripts/)

**유틸리티 스크립트:**
- 검증 도구
- 테스트 헬퍼
- 파싱 유틸리티
- 자동화 스크립트

**실행이 가능하고 문서화되어 있어야 합니다.**

## 작성 스타일 요구 사항 (Writing Style Requirements)

### 명령형/부정사형 (Imperative/Infinitive Form)

2인칭이 아닌 동사 먼저 시작하는 지침을 사용하여 작성하십시오:

**올바름 (명령형):**
```
To create a hook, define the event type.
Configure the MCP server with authentication.
Validate settings before use.
```

**잘못됨 (2인칭):**
```
You should create a hook by defining the event type.
You need to configure the MCP server.
You must validate settings before use.
```

### 설명에서의 3인칭 사용 (Third-Person in Description)

프론트매터 설명(description)은 반드시 3인칭을 사용해야 합니다:

**올바름:**
```yaml
description: This skill should be used when the user asks to "create X", "configure Y"...
```

**잘못됨:**
```yaml
description: Use this skill when you want to create X...
description: Load this skill when user asks...
```

### 객관적이고 교육적인 언어 사용 (Objective, Instructional Language)

누가 해야 하는지가 아니라 무엇을 해야 하는지에 초점을 맞추십시오:

**올바름:**
```
Parse the frontmatter using sed.
Extract fields with grep.
Validate values before use.
```

**잘못됨:**
```
You can parse the frontmatter...
Claude should extract fields...
The user might validate values...
```

## 검증 체크리스트 (Validation Checklist)

스킬을 완성하기 전에 확인하십시오:

**구조:**
- [ ] 유효한 YAML 프론트매터가 포함된 `SKILL.md` 파일이 존재함
- [ ] 프론트매터에 `name`과 `description` 필드가 있음
- [ ] 마크다운 본문이 존재하고 내용이 충실함
- [ ] 참조된 파일이 실제로 존재함

**설명 품질:**
- [ ] 3인칭을 사용함 ("이 스킬은 ~할 때 사용해야 합니다...")
- [ ] 사용자가 말할 수 있는 구체적인 트리거 구문을 포함함
- [ ] 구체적인 시나리오를 나열함 ("create X", "configure Y")
- [ ] 모호하거나 일반적이지 않음

**콘텐츠 품질:**
- [ ] `SKILL.md` 본문이 명령형/부정사형을 사용함
- [ ] 본문이 요점에 집중되고 가벼움 (1,500-2,000 단어가 이상적, 최대 5,000 단어 미만)
- [ ] 상세 내용은 `references/`로 이동됨
- [ ] 예시가 완전하고 실제로 작동함
- [ ] 스크립트가 실행 가능하고 문서화됨

**점진적 공개:**
- [ ] 핵심 개념이 `SKILL.md`에 포함됨
- [ ] 세부 문서는 `references/`에 포함됨
- [ ] 작동하는 코드가 `examples/`에 포함됨
- [ ] 유틸리티가 `scripts/`에 포함됨
- [ ] `SKILL.md`가 이러한 리소스를 올바르게 참조함

**테스트:**
- [ ] 스킬이 예상되는 사용자 쿼리에 반응하여 트리거됨
- [ ] 콘텐츠가 의도한 작업에 유용함
- [ ] 파일 간에 중복된 정보가 없음
- [ ] 필요할 때 참조 자료가 올바르게 로드됨

## 피해야 할 일반적인 실수 (Common Mistakes to Avoid)

### 실수 1: 모호한 트리거 설명 (Weak Trigger Description)

❌ **나쁨:**
```yaml
description: Provides guidance for working with hooks.
```

**이유:** 모호하고, 특정 트리거 구문이 없으며, 3인칭이 아님

✅ **좋음:**
```yaml
description: This skill should be used when the user asks to "create a hook", "add a PreToolUse hook", "validate tool use", or mentions hook events. Provides comprehensive hooks API guidance.
```

**이유:** 3인칭 사용, 구체적인 구문 언급, 구체적인 시나리오 제시

### 실수 2: SKILL.md에 너무 많은 정보 포함 (Too Much in SKILL.md)

❌ **나쁨:**
```
skill-name/
└── SKILL.md  (8,000 단어 - 하나의 파일에 모든 것 포함)
```

**이유:** 스킬 로드 시 컨텍스트가 너무 무거워지고, 상세 콘텐츠가 항상 로드됨

✅ **좋음:**
```
skill-name/
├── SKILL.md  (1,800 단어 - 핵심 필수 사항)
└── references/
    ├── patterns.md (2,500 단어)
    └── advanced.md (3,700 단어)
```

**이유:** 점진적 공개 적용, 세부 콘텐츠는 필요할 때만 로드됨

### 실수 3: 2인칭 글쓰기 (Second Person Writing)

❌ **나쁨:**
```markdown
You should start by reading the configuration file.
You need to validate the input.
You can use the grep tool to search.
```

**이유:** 2인칭 사용, 명령형이 아님

✅ **좋음:**
```markdown
Start by reading the configuration file.
Validate the input before processing.
Use the grep tool to search for patterns.
```

**이유:** 명령형 사용, 직접적인 지침 제공

### 실수 4: 리소스 참조 누락 (Missing Resource References)

❌ **나쁨:**
```markdown
# SKILL.md

[핵심 콘텐츠]

[references/나 examples/에 대한 언급이 없음]
```

**이유:** Claude는 참조 자료가 존재하는지 알지 못함

✅ **좋음:**
```markdown
# SKILL.md

[핵심 콘텐츠]

## Additional Resources

### Reference Files
- **`references/patterns.md`** - Detailed patterns
- **`references/advanced.md`** - Advanced techniques

### Examples
- **`examples/script.sh`** - Working example
```

**이유:** Claude가 추가 정보를 찾을 수 있는 위치를 알게 됨

## 빠른 참조 (Quick Reference)

### 최소 구성 스킬 (Minimal Skill)

```
skill-name/
└── SKILL.md
```

적합한 용도: 간단한 지식 설명, 복잡한 리소스가 필요하지 않은 경우

### 표준 스킬 (권장) (Standard Skill (Recommended))

```
skill-name/
├── SKILL.md
├── references/
│   └── detailed-guide.md
└── examples/
    └── working-example.sh
```

적합한 용도: 상세 문서가 포함된 대부분의 플러그인 스킬

### 완전한 스킬 (Complete Skill)

```
skill-name/
├── SKILL.md
├── references/
│   ├── patterns.md
│   └── advanced.md
├── examples/
│   ├── example1.sh
│   └── example2.json
└── scripts/
    └── validate.sh
```

적합한 용도: 검증 유틸리티가 포함된 복잡한 분야

## 권장 사항 요약 (Best Practices Summary)

✅ **수행할 사항 (DO):**
- 설명에 3인칭 사용 ("이 스킬은 ~할 때 사용해야 합니다...")
- 구체적인 트리거 구문 포함 ("create X", "configure Y")
- `SKILL.md`를 슬림하게 유지 (1,500-2,000 단어)
- 점진적 공개 사용 (상세 내용은 references/로 이동)
- 명령형/부정사형으로 작성
- 관련 지원 파일을 명확하게 참조
- 실제로 작동하는 예시 제공
- 공통 작업을 위한 유틸리티 스크립트 작성
- plugin-dev의 스킬들을 템플릿으로 연구

❌ **피해야 할 사항 (DON'T):**
- 어디서나 2인칭 사용
- 모호한 트리거 조건 설정
- 모든 내용을 `SKILL.md`에 포함 (>3,000 단어, references/ 없이)
- 2인칭형 글쓰기 ("귀하는 ~해야 합니다...")
- 리소스를 참조하지 않은 채 방치
- 손상되었거나 불완전한 예시 포함
- 검증 단계를 건너뜀

## 추가 리소스 (Additional Resources)

### 스킬 학습 사례

이 플러그인의 스킬들은 모범 사례를 잘 보여줍니다:
- `../hook-development/` - 점진적 공개, 유틸리티
- `../agent-development/` - AI 지원 생성, 참조
- `../mcp-integration/` - 종합적인 참조 문서
- `../plugin-settings/` - 실무 예시
- `../command-development/` - 명확한 핵심 개념
- `../plugin-structure/` - 훌륭한 구성

### 참조 파일

스킬 제작기(skill-creator)의 전체 방법론:
- **`references/skill-creator-original.md`** - 오리지널 skill-creator 전체 콘텐츠

## 구현 워크플로우 (Implementation Workflow)

플러그인용 스킬을 제작하는 방법:

1. **사용 사례 이해**: 스킬 사용법의 구체적인 예시 식별
2. **리소스 계획**: 필요한 스크립트/참조/예시 결정
3. **구조 생성**: `mkdir -p skills/skill-name/{references,examples,scripts}`
4. **SKILL.md 작성**:
   - 3인칭 설명 및 트리거 구문이 포함된 프론트매터
   - 명령형 형식의 슬림한 본문 (1,500-2,000 단어)
   - 관련 지원 파일 참조
5. **리소스 추가**: 필요에 따라 references/, examples/, scripts/ 생성
6. **검증**: 설명, 글쓰기 스타일, 구성 상태 확인
7. **테스트**: 예상되는 트리거에 스킬이 로드되는지 검증
8. **반복 개선**: 사용 결과에 기초하여 지속 개선

필요할 때 로드되고 표적화된 지침을 제공하는 효과적인 스킬을 위해 강력한 트리거 설명, 점진적 공개 및 명령형 작성 스타일을 고수하십시오.
