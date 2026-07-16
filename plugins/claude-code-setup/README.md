# Claude Code Setup Plugin

코드베이스를 분석하여 hook, skill, MCP 서버 등 맞춤형 Claude Code 자동화를 추천합니다.

## 주요 기능

Claude는 이 skill을 사용하여 코드베이스를 스캔하고 각 카테고리별로 가장 적합한 1~2개의 자동화를 추천합니다:

- **MCP Servers** - 외부 통합 도구 (문서용 context7, 프론트엔드용 Playwright 등)
- **Skills** - 패키지화된 전문 지식 (Plan 에이전트, frontend-design)
- **Hooks** - 자동 실행 액션 (auto-format, auto-lint, 민감 파일 차단 등)
- **Subagents** - 전문 리뷰어 (보안, 성능, 웹 접근성 등)
- **Slash Commands** - 빠른 워크플로우 (/test, /pr-review, /explain)

이 skill은 **읽기 전용(read-only)**이며, 분석만 수행하고 파일을 수정하지는 않습니다.

## 사용법

```
"recommend automations for this project"
"help me set up Claude Code"
"what hooks should I use?"
```

<img src="automation-recommender-example.png" alt="Automation recommender analyzing a codebase and providing tailored recommendations" width="600">

## 작성자

Isabella He (isabella@anthropic.com)
