# 플러그인 명령어 예시 (Plugin Command Examples)

Claude Code 플러그인용으로 설계된 명령어의 실용적인 예시로, 플러그인 전용 패턴과 기능들을 보여줍니다.

## 목차 (Table of Contents)

1. [단순 플러그인 명령어](#1-simple-plugin-command)
2. [스크립트 기반 분석](#2-script-based-analysis)
3. [템플릿 기반 생성](#3-template-based-generation)
4. [다중 스크립트 워크플로우](#4-multi-script-workflow)
5. [설정 기반 배포](#5-configuration-driven-deployment)
6. [에이전트 통합](#6-agent-integration)
7. [스킬 통합](#7-skill-integration)
8. [다중 컴포넌트 워크플로우](#8-multi-component-workflow)
9. [입력 검증 명령어](#9-validated-input-command)
10. [환경 감지 명령어](#10-environment-aware-command)

---

## 1. 단순 플러그인 명령어 (Simple Plugin Command)

**사용 사례:** 플러그인 스크립트를 사용하는 기본적인 명령어

**파일:** `commands/analyze.md`

```markdown
---
description: Analyze code quality using plugin tools
argument-hint: [file-path]
allowed-tools: Bash(node:*), Read
---

Analyze @$1 using plugin's quality checker:

!`node ${CLAUDE_PLUGIN_ROOT}/scripts/quality-check.js $1`

Review the analysis output and provide:
1. Summary of findings
2. Priority issues to address
3. Suggested improvements
4. Code quality score interpretation
```

**주요 특징:**
- 이식 가능한 경로를 위해 `${CLAUDE_PLUGIN_ROOT}` 사용
- 파일 참조와 스크립트 실행을 결합
- 단순한 단일 목적 명령어

---

## 2. 스크립트 기반 분석 (Script-Based Analysis)

**사용 사례:** 여러 플러그인 스크립트를 사용하여 종합적인 분석 실행

**파일:** `commands/full-audit.md`

```markdown
---
description: Complete code audit using plugin suite
argument-hint: [directory]
allowed-tools: Bash(*)
model: sonnet
---

Running complete audit on $1:

**Security scan:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/security-scan.sh $1`

**Performance analysis:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/perf-analyze.sh $1`

**Best practices check:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/best-practices.sh $1`

Analyze all results and create comprehensive report including:
- Critical issues requiring immediate attention
- Performance optimization opportunities
- Security vulnerabilities and fixes
- Overall health score and recommendations
```

**주요 특징:**
- 여러 스크립트 실행
- 정리된 출력 섹션
- 종합적인 워크플로우
- 명확한 보고 구조

---

## 3. 템플릿 기반 생성 (Template-Based Generation)

**사용 사례:** 플러그인 템플릿을 따른 문서 생성

**파일:** `commands/gen-api-docs.md`

```markdown
---
description: Generate API documentation from template
argument-hint: [api-file]
---

Template structure: @${CLAUDE_PLUGIN_ROOT}/templates/api-documentation.md

API implementation: @$1

Generate complete API documentation following the template format above.

Ensure documentation includes:
- Endpoint descriptions with HTTP methods
- Request/response schemas
- Authentication requirements
- Error codes and handling
- Usage examples with curl commands
- Rate limiting information

Format output as markdown suitable for README or docs site.
```

**주요 특징:**
- 플러그인 템플릿 사용
- 템플릿과 소스 파일 결합
- 표준화된 출력 형식
- 명확한 문서 구조

---

## 4. 다중 스크립트 워크플로우 (Multi-Script Workflow)

**사용 사례:** 빌드, 테스트 및 배포 워크플로우 조율

**파일:** `commands/release.md`

```markdown
---
description: Execute complete release workflow
argument-hint: [version]
allowed-tools: Bash(*), Read
---

Executing release workflow for version $1:

**Step 1 - Pre-release validation:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/pre-release-check.sh $1`

**Step 2 - Build artifacts:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build-release.sh $1`

**Step 3 - Run test suite:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/run-tests.sh`

**Step 4 - Package release:**
!`bash ${CLAUDE_PLUGIN_ROOT}/scripts/package.sh $1`

Review all step outputs and report:
1. Any failures or warnings
2. Build artifacts location
3. Test results summary
4. Next steps for deployment
5. Rollback plan if needed
```

**주요 특징:**
- 다단계 워크플로우
- 순차적인 스크립트 실행
- 명확한 단계 번호 지정
- 종합적인 보고

---

## 5. 설정 기반 배포 (Configuration-Driven Deployment)

**사용 사례:** 환경별 플러그인 설정을 사용하여 배포

**파일:** `commands/deploy.md`

```markdown
---
description: Deploy application to environment
argument-hint: [environment]
allowed-tools: Read, Bash(*)
---

Deployment configuration for $1: @${CLAUDE_PLUGIN_ROOT}/config/$1-deploy.json

Current git state: !`git rev-parse --short HEAD`

Build info: !`cat package.json | grep -E '(name|version)'`

Execute deployment to $1 environment using configuration above.

Deployment checklist:
1. Validate configuration settings
2. Build application for $1
3. Run pre-deployment tests
4. Deploy to target environment
5. Run smoke tests
6. Verify deployment success
7. Update deployment log

Report deployment status and any issues encountered.
```

**주요 특징:**
- 환경별 설정
- 동적 설정 파일 로딩
- 배포 전 검증
- 구조화된 체크리스트

---

## 6. 에이전트 통합 (Agent Integration)

**사용 사례:** 복잡한 작업을 위해 플러그인 에이전트를 실행하는 명령어

**파일:** `commands/deep-review.md`

```markdown
---
description: Deep code review using plugin agent
argument-hint: [file-or-directory]
---

Initiate comprehensive code review of @$1 using the code-reviewer agent.

The agent will perform:
1. **Static analysis** - Check for code smells and anti-patterns
2. **Security audit** - Identify potential vulnerabilities
3. **Performance review** - Find optimization opportunities
4. **Best practices** - Ensure code follows standards
5. **Documentation check** - Verify adequate documentation

The agent has access to:
- Plugin's linting rules: ${CLAUDE_PLUGIN_ROOT}/config/lint-rules.json
- Security checklist: ${CLAUDE_PLUGIN_ROOT}/checklists/security.md
- Performance guidelines: ${CLAUDE_PLUGIN_ROOT}/docs/performance.md

Note: This uses the Task tool to launch the plugin's code-reviewer agent for thorough analysis.
```

**주요 특징:**
- 플러그인 에이전트에 위임
- 에이전트 기능 문서화
- 플러그인 자원 참조
- 명확한 범위 정의

---

## 7. 스킬 통합 (Skill Integration)

**사용 사례:** 전문 지식을 위해 플러그인 스킬을 활용하는 명령어

**파일:** `commands/document-api.md`

```markdown
---
description: Document API following plugin standards
argument-hint: [api-file]
---

API source code: @$1

Generate API documentation following the plugin's API documentation standards.

Use the api-documentation-standards skill to ensure:
- **OpenAPI compliance** - Follow OpenAPI 3.0 specification
- **Consistent formatting** - Use plugin's documentation style
- **Complete coverage** - Document all endpoints and schemas
- **Example quality** - Provide realistic usage examples
- **Error documentation** - Cover all error scenarios

The skill provides:
- Standard documentation templates
- API documentation best practices
- Common patterns for this codebase
- Quality validation criteria

Generate production-ready API documentation.
```

**주요 특징:**
- 이름으로 플러그인 스킬 호출
- 스킬 목적 문서화
- 명확한 기대치
- 스킬 지식 활용

---

## 8. 다중 컴포넌트 워크플로우 (Multi-Component Workflow)

**사용 사례:** 에이전트, 스킬, 스크립트를 사용하는 복잡한 워크플로우

**파일:** `commands/complete-review.md`

```markdown
---
description: Comprehensive review using all plugin components
argument-hint: [file-path]
allowed-tools: Bash(node:*), Read
---

Target file: @$1

Execute comprehensive review workflow:

**Phase 1: Automated Analysis**
Run plugin analyzer: !`node ${CLAUDE_PLUGIN_ROOT}/scripts/analyze.js $1`

**Phase 2: Deep Review (Agent)**
Launch the code-quality-reviewer agent for detailed analysis.
Agent will examine:
- Code structure and organization
- Error handling patterns
- Testing coverage
- Documentation quality

**Phase 3: Standards Check (Skill)**
Use the coding-standards skill to validate:
- Naming conventions
- Code formatting
- Best practices adherence
- Framework-specific patterns

**Phase 4: Report Generation**
Template: @${CLAUDE_PLUGIN_ROOT}/templates/review-report.md

Compile all findings into comprehensive report following template.

**Phase 5: Recommendations**
Generate prioritized action items:
1. Critical issues (must fix)
2. Important improvements (should fix)
3. Nice-to-have enhancements (could fix)

Include specific file locations and suggested changes for each item.
```

**주요 특징:**
- 다단계 워크플로우
- 스크립트, 에이전트, 스킬 결합
- 템플릿 기반 보고
- 우선순위가 지정된 출력

---

## 9. 입력 검증 명령어 (Validated Input Command)

**사용 사례:** 입력 검증 및 에러 처리가 포함된 명령어

**파일:** `commands/build-env.md`

```markdown
---
description: Build for specific environment with validation
argument-hint: [environment]
allowed-tools: Bash(*)
---

Validate environment argument: !`echo "$1" | grep -E "^(dev|staging|prod)$" && echo "VALID" || echo "INVALID"`

Check build script exists: !`test -x ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh && echo "EXISTS" || echo "MISSING"`

Verify configuration available: !`test -f ${CLAUDE_PLUGIN_ROOT}/config/$1.json && echo "FOUND" || echo "NOT_FOUND"`

If all validations pass:

**Configuration:** @${CLAUDE_PLUGIN_ROOT}/config/$1.json

**Execute build:** !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/build.sh $1 2>&1`

**Validation results:** !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate-build.sh $1 2>&1`

Report build status and any issues.

If validations fail:
- Explain which validation failed
- Provide expected values/locations
- Suggest corrective actions
- Document troubleshooting steps
```

**주요 특징:**
- 입력 검증
- 자원 존재 여부 확인
- 에러 처리
- 유용한 에러 메시지
- 부드러운 실패 처리

---

## 10. 환경 감지 명령어 (Environment-Aware Command)

**사용 사례:** 환경에 따라 동작이 달라지는 명령어

**파일:** `commands/run-checks.md`

```markdown
---
description: Run environment-appropriate checks
argument-hint: [environment]
allowed-tools: Bash(*), Read
---

Environment: $1

Load environment configuration: @${CLAUDE_PLUGIN_ROOT}/config/$1-checks.json

Determine check level: !`echo "$1" | grep -E "^prod$" && echo "FULL" || echo "BASIC"`

**For production environment:**
- Full test suite: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/test-full.sh`
- Security scan: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/security-scan.sh`
- Performance audit: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/perf-check.sh`
- Compliance check: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/compliance.sh`

**For non-production environments:**
- Basic tests: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/test-basic.sh`
- Quick lint: !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh`

Analyze results based on environment requirements:

**Production:** All checks must pass with zero critical issues
**Staging:** No critical issues, warnings acceptable
**Development:** Focus on blocking issues only

Report status and recommend proceed/block decision.
```

**주요 특징:**
- 환경 감지 로직
- 조건부 실행
- 다양한 검증 수준
- 환경별 적절한 보고

---

## 공통 패턴 요약 (Common Patterns Summary)

### 패턴: 플러그인 스크립트 실행 (Plugin Script Execution)
```markdown
!`node ${CLAUDE_PLUGIN_ROOT}/scripts/script-name.js $1`
```
용도: 플러그인에서 제공하는 Node.js 스크립트 실행

### 패턴: 플러그인 설정 로드 (Plugin Configuration Loading)
```markdown
@${CLAUDE_PLUGIN_ROOT}/config/config-name.json
```
용도: 플러그인 설정 파일 로드

### 패턴: 플러그인 템플릿 사용 (Plugin Template Usage)
```markdown
@${CLAUDE_PLUGIN_ROOT}/templates/template-name.md
```
용도: 생성을 위해 플러그인 템플릿 사용

### 패턴: 에이전트 호출 (Agent Invocation)
```markdown
Launch the [agent-name] agent for [task description].
```
용도: 복잡한 작업을 플러그인 에이전트에 위임

### 패턴: 스킬 참조 (Skill Reference)
```markdown
Use the [skill-name] skill to ensure [requirements].
```
용도: 전문 지식을 위해 플러그인 스킬 활용

### 패턴: 입력 검증 (Input Validation)
```markdown
Validate input: !`echo "$1" | grep -E "^pattern$" && echo "OK" || echo "ERROR"`
```
용도: 명령어 인수 검증

### 패턴: 자원 검증 (Resource Validation)
```markdown
Check exists: !`test -f ${CLAUDE_PLUGIN_ROOT}/path/file && echo "YES" || echo "NO"`
```
용도: 필수 플러그인 파일이 존재하는지 확인

---

## 개발 팁 (Development Tips)

### 플러그인 명령어 테스트 (Testing Plugin Commands)

1. **플러그인이 설치된 상태로 테스트:**
   ```bash
   cd /path/to/plugin
   claude /command-name args
   ```

2. **${CLAUDE_PLUGIN_ROOT} 확장 확인:**
   ```bash
   # 명령어에 디버그 출력 추가
   !`echo "Plugin root: ${CLAUDE_PLUGIN_ROOT}"`
   ```

3. **다양한 작업 디렉토리에서 테스트:**
   ```bash
   cd /tmp && claude /command-name
   cd /other/project && claude /command-name
   ```

4. **자원 가용성 검증:**
   ```bash
   # 모든 플러그인 자원이 존재하는지 확인
   !`ls -la ${CLAUDE_PLUGIN_ROOT}/scripts/`
   !`ls -la ${CLAUDE_PLUGIN_ROOT}/config/`
   ```

### 피해야 할 일반적인 실수 (Common Mistakes to Avoid)

1. **${CLAUDE_PLUGIN_ROOT} 대신 상대 경로 사용:**
   ```markdown
   # 잘못된 예시
   !`node ./scripts/analyze.js`

   # 올바른 예시
   !`node ${CLAUDE_PLUGIN_ROOT}/scripts/analyze.js`
   ```

2. **필수 도구 허용 누락:**
   ```markdown
   # allowed-tools 누락
   !`bash script.sh`  # Bash 권한 없이 실패함

   # 올바른 예시
   ---
   allowed-tools: Bash(*)
   ---
   !`bash ${CLAUDE_PLUGIN_ROOT}/scripts/script.sh`
   ```

3. **입력 검증 누락:**
   ```markdown
   # 위험함 - 검증 없음
   Deploy to $1 environment

   # 권장 - 검증 포함
   Validate: !`echo "$1" | grep -E "^(dev|staging|prod)$" || echo "INVALID"`
   Deploy to $1 environment (if valid)
   ```

4. **플러그인 경로 하드코딩:**
   ```markdown
   # 잘못된 예시 - 다른 설치 환경에서 깨짐
   @/home/user/.claude/plugins/my-plugin/config.json

   # 올바른 예시 - 어디서나 작동함
   @${CLAUDE_PLUGIN_ROOT}/config.json
   ```

---

자세한 플러그인 전용 기능은 `references/plugin-features-reference.md`를 참고하세요.
일반적인 명령어 개발은 메인 `SKILL.md`를 참고하세요.
