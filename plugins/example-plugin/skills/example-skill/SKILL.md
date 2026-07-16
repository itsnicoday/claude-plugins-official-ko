---
name: example-skill
description: 이 스킬은 사용자가 "스킬 시연(demonstrate skills)", "스킬 형식 보여주기(show skill format)", "스킬 템플릿 생성(create a skill template)"을 요청하거나 스킬 개발 패턴에 대해 논의할 때 사용해야 합니다. Claude Code 플러그인 스킬을 생성하기 위한 참조 템플릿을 제공합니다.
version: 1.0.0
---

# Example Skill

이 스킬은 Claude Code 플러그인 스킬의 구조와 형식을 보여줍니다.

## Overview

스킬은 작업 콘텍스트를 기반으로 Claude가 자체적으로 사용하는 모델 호출형(model-invoked) 기능입니다. 사용자가 직접 실행하는 명령어(command)나 Claude가 생성하는 에이전트(agent)와 달리, 스킬은 Claude가 자신의 답변에 반영할 수 있도록 맥락적 안내(contextual guidance)를 제공합니다.

## When This Skill Applies

이 스킬은 사용자의 요청이 다음과 같을 때 활성화됩니다:
- 플러그인 스킬 생성 또는 이해
- 스킬 템플릿 또는 참조 요구
- 스킬 개발 패턴

## Skill Structure

### Required Files

```
skills/
└── skill-name/
    └── SKILL.md          # Main skill definition (required)
```

### Optional Supporting Files

```
skills/
└── skill-name/
    ├── SKILL.md          # Main skill definition
    ├── README.md         # Additional documentation
    ├── references/       # Reference materials
    │   └── patterns.md
    ├── examples/         # Example files
    │   └── sample.md
    └── scripts/          # Helper scripts
        └── helper.sh
```

## Frontmatter Options

스킬은 다음 프론트매터 필드를 지원합니다:

- **name** (필수): 스킬 식별자
- **description** (필수): 트리거 조건 - Claude가 이 스킬을 언제 사용해야 하는지 설명
- **version** (선택): 시맨틱 버전 번호
- **license** (선택): 라이선스 정보 또는 참조

## Writing Effective Descriptions

description 필드는 매우 중요합니다. Claude가 스킬을 언제 호출할지 판단하는 기준이 되기 때문입니다.

**올바른 description 패턴:**
```yaml
description: This skill should be used when the user asks to "specific phrase", "another phrase", mentions "keyword", or discusses topic-area.
```

**포함할 내용:**
- 사용자가 말할 수 있는 특정 트리거 문구
- 관련성을 나타내는 키워드
- 스킬이 다루는 주제 영역

## Skill Content Guidelines

1. **명확한 목적**: 스킬이 무엇을 돕는지 서술
2. **사용 시점**: 활성화 조건 정의
3. **구조화된 안내**: 정보를 논리적으로 정리
4. **실행 가능한 지침**: 구체적인 단계 제공
5. **예시**: 유용한 경우 실제 예시 포함

## Best Practices

- 스킬이 단일 도메인에만 집중되도록 유지하세요.
- 활성화 시점을 명확히 나타내는 설명을 작성하세요.
- 복잡한 스킬의 경우 하위 디렉터리에 참조 자료를 포함하세요.
- 예상 질문에 대해 스킬이 제대로 활성화되는지 테스트하세요.
- 다른 스킬의 트리거 조건과 겹치지 않도록 하세요.
