# 버전 고정 (Version pins)

이 skill에서 버전에 민감한 모든 정보를 한곳에 정리한 것입니다. skill을 업데이트할 때 이 부분을 먼저 확인하십시오.

| 대상 정보 | 명시된 위치 | 최종 확인일 |
|---|---|---|
| `@modelcontextprotocol/ext-apps@1.2.2` CDN 고정 | `build-mcp-app/SKILL.md`, `build-mcp-app/references/widget-templates.md` (4개 위치) | 2026-03 |
| Elicitation 지원을 위한 Claude Code ≥2.1.76 | `elicitation.md:15`, `build-mcp-server/SKILL.md:43,76` | 2026-03 |
| MCP 스펙 2025-11-25 CIMD/DCR 상태 | `auth.md:20,24,41` | 2026-03 |
| MCPB 매니페스트 스키마 v0.4 | `build-mcpb/references/manifest-schema.md` | 2026-03 |
| CF `agents` SDK / `McpAgent` API | `deploy-cloudflare-workers.md` | 2026-03 |
| CF 템플릿 경로 `cloudflare/ai/demos/remote-mcp-authless` | `deploy-cloudflare-workers.md` | 2026-03 |

## 검증 방법 (How to verify)

```bash
# ext-apps 최신 버전 확인
npm view @modelcontextprotocol/ext-apps version

# CF 템플릿 존재 여부 확인
gh api repos/cloudflare/ai/contents/demos/remote-mcp-authless/src/index.ts --jq '.sha'

# MCPB 스키마 확인
curl -sI https://raw.githubusercontent.com/anthropics/mcpb/main/schemas/mcpb-manifest-v0.4.schema.json | head -1
```
