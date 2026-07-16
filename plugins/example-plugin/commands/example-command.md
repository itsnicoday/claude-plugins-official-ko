---
description: 명령 프론트매터(frontmatter) 옵션을 보여주는 예시 슬래시 명령어 (레거시 형식)
argument-hint: <required-arg> [optional-arg]
allowed-tools: [Read, Glob, Grep, Bash]
---

# Example Command (레거시 `commands/` 형식)

> **참고:** 이 파일은 레거시 `commands/*.md` 레이아웃을 보여줍니다. 새로운 플러그인의 경우 `skills/<name>/SKILL.md` 디렉터리 형식을 권장합니다 (이 플러그인의 `skills/example-command/SKILL.md`를 참고하세요). 두 방식 모두 동일하게 로드되며, 유일한 차이점은 파일 레이아웃입니다.

이 명령어는 슬래시 명령어 구조와 프론트매터 옵션을 보여줍니다.

## Arguments

사용자가 다음 인자(argument)로 이 명령어를 호출했습니다: $ARGUMENTS

## Instructions

이 명령어가 호출되었을 때:

1. 사용자가 제공한 인자를 파싱합니다.
2. 허용된 도구(allowed tools)를 사용하여 요청된 작업을 수행합니다.
3. 결과를 사용자에게 보고합니다.

## Frontmatter Options Reference

명령어는 다음 프론트매터 필드를 지원합니다:

- **description**: /help에 표시되는 짧은 설명
- **argument-hint**: 사용자에게 표시되는 명령어 인자 힌트
- **allowed-tools**: 이 명령어에 대해 사전 승인된 도구 (권한 요청 팝업을 줄여줌)
- **model**: 모델 오버라이드 (예: "haiku", "sonnet", "opus")

## Example Usage

```
/example-command my-argument
/example-command arg1 arg2
```
