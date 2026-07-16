---
name: example-command
description: 프론트매터 옵션과 skills/<name>/SKILL.md 레이아웃을 보여주는 사용자가 호출하는 예시 스킬
argument-hint: <required-arg> [optional-arg]
allowed-tools: [Read, Glob, Grep, Bash]
---

# Example Command (스킬 형식)

이 파일은 사용자가 직접 호출하는 슬래시 명령어를 위한 `skills/<name>/SKILL.md` 레이아웃을 보여줍니다. 기능적으로는 레거시인 `commands/example-command.md` 형식과 완전히 동일하며, 둘 다 같은 방식으로 로드됩니다. 단지 파일 레이아웃만 다릅니다.

## Arguments

사용자가 다음 인자로 호출했습니다: $ARGUMENTS

## Instructions

이 스킬이 호출되었을 때:

1. 사용자가 제공한 인자를 파싱합니다.
2. 허용된 도구(allowed tools)를 사용하여 요청된 작업을 수행합니다.
3. 결과를 사용자에게 보고합니다.

## Frontmatter Options Reference

이 레이아웃의 스킬은 다음 프론트매터 필드를 지원합니다:

- **name**: 스킬 식별자 (디렉터리 이름과 일치함)
- **description**: /help에 표시되는 짧은 설명
- **argument-hint**: 사용자에게 표시되는 명령어 인자 힌트
- **allowed-tools**: 이 스킬에 대해 사전 승인된 도구 (권한 요청 팝업을 줄여줌)
- **model**: 모델 오버라이드 (예: "haiku", "sonnet", "opus")

## Example Usage

```
/example-command my-argument
/example-command arg1 arg2
```
