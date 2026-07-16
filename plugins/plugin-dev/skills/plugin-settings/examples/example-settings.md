# 플러그인 설정 파일 예시 (Example Plugin Settings File)

## 템플릿: 기본 구성 (Template: Basic Configuration)

**.claude/my-plugin.local.md:**

```markdown
---
enabled: true
mode: standard
---

# My Plugin Configuration

Plugin is active in standard mode.
```

## 템플릿: 고급 구성 (Template: Advanced Configuration)

**.claude/my-plugin.local.md:**

```markdown
---
enabled: true
strict_mode: false
max_file_size: 1000000
allowed_extensions: [".js", ".ts", ".tsx"]
enable_logging: true
notification_level: info
retry_attempts: 3
timeout_seconds: 60
custom_path: "/path/to/data"
---

# My Plugin Advanced Configuration

This project uses custom plugin configuration with:
- Standard validation mode
- 1MB file size limit
- JavaScript/TypeScript files allowed
- Info-level logging
- 3 retry attempts

## Additional Notes

Contact @team-lead with questions about this configuration.
```

## 템플릿: 에이전트 상태 파일 (Template: Agent State File)

**.claude/multi-agent-swarm.local.md:**

```markdown
---
agent_name: database-implementation
task_number: 4.2
pr_number: 5678
coordinator_session: team-leader
enabled: true
dependencies: ["Task 3.5", "Task 4.1"]
additional_instructions: "Use PostgreSQL, not MySQL"
---

# Task Assignment: Database Schema Implementation

Implement the database schema for the new features module.

## Requirements

- Create migration files
- Add indexes for performance
- Write tests for constraints
- Document schema in README

## Success Criteria

- Migrations run successfully
- All tests pass
- PR created with CI green
- Schema documented

## Coordination

Depends on:
- Task 3.5: API endpoint definitions
- Task 4.1: Data model design

Report status to coordinator session 'team-leader'.
```

## 템플릿: 기능 플래그 패턴 (Template: Feature Flag Pattern)

**.claude/experimental-features.local.md:**

```markdown
---
enabled: true
features:
  - ai_suggestions
  - auto_formatting
  - advanced_refactoring
experimental_mode: false
---

# Experimental Features Configuration

Current enabled features:
- AI-powered code suggestions
- Automatic code formatting
- Advanced refactoring tools

Experimental mode is OFF (stable features only).
```

## 훅에서의 사용법 (Usage in Hooks)

이러한 템플릿은 훅에서 다음과 같이 읽을 수 있습니다:

```bash
# Check if plugin is configured
if [[ ! -f ".claude/my-plugin.local.md" ]]; then
  exit 0  # Not configured, skip hook
fi

# Read settings
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' ".claude/my-plugin.local.md")
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')

# Apply settings
if [[ "$ENABLED" == "true" ]]; then
  # Hook is active
  # ...
fi
```

## Gitignore 설정 (Gitignore)

항상 프로젝트 `.gitignore`에 추가하십시오:

```gitignore
# Plugin settings (user-local, not committed)
.claude/*.local.md
.claude/*.local.json
```

## 설정 편집 (Editing Settings)

사용자는 설정 파일을 수동으로 편집할 수 있습니다:

```bash
# Edit settings
vim .claude/my-plugin.local.md

# Changes take effect after restart
exit  # Exit Claude Code
claude  # Restart
```

변경 사항은 Claude Code 재시작이 필요합니다. 훅은 실행 중에 즉시 교체(hot-swap)할 수 없습니다.
