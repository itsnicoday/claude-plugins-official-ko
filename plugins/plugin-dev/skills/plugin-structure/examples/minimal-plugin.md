# 최소 구성 플러그인 예시 (Minimal Plugin Example)

단일 명령어만 포함하는 최소 구조의 플러그인 예시.

## 디렉토리 구조 (Directory Structure)

```
hello-world/
├── .claude-plugin/
│   └── plugin.json
└── commands/
    └── hello.md
```

## 파일 내용 (File Contents)

### .claude-plugin/plugin.json

```json
{
  "name": "hello-world"
}
```

### commands/hello.md

```markdown
---
name: hello
description: Prints a friendly greeting message
---

# Hello Command

Print a friendly greeting to the user.

## Implementation

Output the following message to the user:

> Hello! This is a simple command from the hello-world plugin.
>
> Use this as a starting point for building more complex plugins.

Include the current timestamp in the greeting to show the command executed successfully.
```

## 사용법 (Usage)

플러그인을 설치한 후:

```
$ claude
> /hello
Hello! This is a simple command from the hello-world plugin.

Use this as a starting point for building more complex plugins.

Executed at: 2025-01-15 14:30:22 UTC
```

## 주요 특징 (Key Points)

1. **최소 구성 매니페스트**: 필수 필드인 `name`만 포함
2. **단일 명령어**: `commands/` 디렉토리에 하나의 마크다운 파일만 포함
3. **자동 감지**: Claude Code가 명령어를 자동으로 검색
4. **종속성 없음**: 스크립트, 훅 또는 외부 리소스 미사용

## 이 패턴의 사용 시기 (When to Use This Pattern)

- 빠른 프로토타이핑
- 단일 목적 유틸리티
- 플러그인 개발 학습
- 하나의 특정 기능만 갖춘 내부 팀용 도구

## 이 플러그인의 확장 방법 (Extending This Plugin)

기능을 추가하려면:

1. **명령어 추가**: `commands/` 디렉토리에 추가적인 `.md` 파일 생성
2. **메타데이터 추가**: `plugin.json`에 `version`, `description`, `author` 정보 업데이트
3. **에이전트 추가**: 에이전트 정의가 포함된 `agents/` 디렉토리 생성
4. **훅 추가**: 이벤트 처리를 위한 `hooks/hooks.json` 파일 생성
