# 고급 플러그인 예시 (Advanced Plugin Example)

MCP 통합 및 고급 구성을 특징으로 하는 엔터프라이즈급의 복잡한 플러그인 사례.

## 디렉토리 구조 (Directory Structure)

```
enterprise-devops/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── ci/
│   │   ├── build.md
│   │   ├── test.md
│   │   └── deploy.md
│   ├── monitoring/
│   │   ├── status.md
│   │   └── logs.md
│   └── admin/
│       ├── configure.md
│       └── manage.md
├── agents/
│   ├── orchestration/
│   │   ├── deployment-orchestrator.md
│   │   └── rollback-manager.md
│   └── specialized/
│       ├── kubernetes-expert.md
│       ├── terraform-expert.md
│       └── security-auditor.md
├── skills/
│   ├── kubernetes-ops/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   ├── deployment-patterns.md
│   │   │   ├── troubleshooting.md
│   │   │   └── security.md
│   │   ├── examples/
│   │   │   ├── basic-deployment.yaml
│   │   │   ├── stateful-set.yaml
│   │   │   └── ingress-config.yaml
│   │   └── scripts/
│   │       ├── validate-manifest.sh
│   │       └── health-check.sh
│   ├── terraform-iac/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   └── best-practices.md
│   │   └── examples/
│   │       └── module-template/
│   └── ci-cd-pipelines/
│       ├── SKILL.md
│       └── references/
│           └── pipeline-patterns.md
├── hooks/
│   ├── hooks.json
│   └── scripts/
│       ├── security/
│       │   ├── scan-secrets.sh
│       │   ├── validate-permissions.sh
│       │   └── audit-changes.sh
│       ├── quality/
│       │   ├── check-config.sh
│       │   └── verify-tests.sh
│       └── workflow/
│           ├── notify-team.sh
│           └── update-status.sh
├── .mcp.json
├── servers/
│   ├── kubernetes-mcp/
│   │   ├── index.js
│   │   ├── package.json
│   │   └── lib/
│   ├── terraform-mcp/
│   │   ├── main.py
│   │   └── requirements.txt
│   └── github-actions-mcp/
│       ├── server.js
│       └── package.json
├── lib/
│   ├── core/
│   │   ├── logger.js
│   │   ├── config.js
│   │   └── auth.js
│   ├── integrations/
│   │   ├── slack.js
│   │   ├── pagerduty.js
│   │   └── datadog.js
│   └── utils/
│       ├── retry.js
│       └── validation.js
└── config/
    ├── environments/
    │   ├── production.json
    │   ├── staging.json
    │   └── development.json
    └── templates/
        ├── deployment.yaml
        └── service.yaml
```

## 파일 내용 (File Contents)

### .claude-plugin/plugin.json

```json
{
  "name": "enterprise-devops",
  "version": "2.3.1",
  "description": "Comprehensive DevOps automation for enterprise CI/CD pipelines, infrastructure management, and monitoring",
  "author": {
    "name": "DevOps Platform Team",
    "email": "devops-platform@company.com",
    "url": "https://company.com/teams/devops"
  },
  "homepage": "https://docs.company.com/plugins/devops",
  "repository": {
    "type": "git",
    "url": "https://github.com/company/devops-plugin.git"
  },
  "license": "Apache-2.0",
  "keywords": [
    "devops",
    "ci-cd",
    "kubernetes",
    "terraform",
    "automation",
    "infrastructure",
    "deployment",
    "monitoring"
  ],
  "commands": [
    "./commands/ci",
    "./commands/monitoring",
    "./commands/admin"
  ],
  "agents": [
    "./agents/orchestration",
    "./agents/specialized"
  ],
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./.mcp.json"
}
```

### .mcp.json

```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/kubernetes-mcp/index.js"],
      "env": {
        "KUBECONFIG": "${KUBECONFIG}",
        "K8S_NAMESPACE": "${K8S_NAMESPACE:-default}"
      }
    },
    "terraform": {
      "command": "python",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/terraform-mcp/main.py"],
      "env": {
        "TF_STATE_BUCKET": "${TF_STATE_BUCKET}",
        "AWS_REGION": "${AWS_REGION}"
      }
    },
    "github-actions": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/github-actions-mcp/server.js"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_ORG": "${GITHUB_ORG}"
      }
    }
  }
}
```

### commands/ci/build.md

```markdown
---
name: build
description: Trigger and monitor CI build pipeline
---

# Build Command

Trigger CI/CD build pipeline and monitor progress in real-time.

## Process

1. **Validation**: Check prerequisites
   - Verify branch status
   - Check for uncommitted changes
   - Validate configuration files

2. **Trigger**: Start build via MCP server
   ```javascript
   // Uses github-actions MCP server
   const build = await tools.github_actions_trigger_workflow({
     workflow: 'build.yml',
     ref: currentBranch
   })
   ```

3. **Monitor**: Track build progress
   - Display real-time logs
   - Show test results as they complete
   - Alert on failures

4. **Report**: Summarize results
   - Build status
   - Test coverage
   - Performance metrics
   - Deploy readiness

## Integration

After successful build:
- Offer to deploy to staging
- Suggest performance optimizations
- Generate deployment checklist
```

### agents/orchestration/deployment-orchestrator.md

```markdown
---
description: Orchestrates complex multi-environment deployments with rollback capabilities and health monitoring
capabilities:
  - Plan and execute multi-stage deployments
  - Coordinate service dependencies
  - Monitor deployment health
  - Execute automated rollbacks
  - Manage deployment approvals
---

# Deployment Orchestrator Agent

Specialized agent for orchestrating complex deployments across multiple environments.

## Expertise

- **Deployment strategies**: Blue-green, canary, rolling updates
- **Dependency management**: Service startup ordering, dependency injection
- **Health monitoring**: Service health checks, metric validation
- **Rollback automation**: Automatic rollback on failure detection
- **Approval workflows**: Multi-stage approval processes

## Orchestration Process

1. **Planning Phase**
   - Analyze deployment requirements
   - Identify service dependencies
   - Generate deployment plan
   - Calculate rollback strategy

2. **Validation Phase**
   - Verify environment readiness
   - Check resource availability
   - Validate configurations
   - Run pre-deployment tests

3. **Execution Phase**
   - Deploy services in dependency order
   - Monitor health after each stage
   - Validate metrics and logs
   - Proceed to next stage on success

4. **Verification Phase**
   - Run smoke tests
   - Validate service integration
   - Check performance metrics
   - Confirm deployment success

5. **Rollback Phase** (if needed)
   - Detect failure conditions
   - Execute rollback plan
   - Restore previous state
   - Notify stakeholders

## MCP Integration

Uses multiple MCP servers:
- `kubernetes`: Deploy and manage containers
- `terraform`: Provision infrastructure
- `github-actions`: Trigger deployment pipelines

## Monitoring Integration

Integrates with monitoring tools via lib:
```javascript
const { DatadogClient } = require('${CLAUDE_PLUGIN_ROOT}/lib/integrations/datadog')
const metrics = await DatadogClient.getMetrics(service, timeRange)
```

## Notification Integration

Sends updates via Slack and PagerDuty:
```javascript
const { SlackClient } = require('${CLAUDE_PLUGIN_ROOT}/lib/integrations/slack')
await SlackClient.notify({
  channel: '#deployments',
  message: 'Deployment started',
  metadata: deploymentPlan
})
```
```

### skills/kubernetes-ops/SKILL.md

```markdown
---
name: Kubernetes Operations
description: This skill should be used when deploying to Kubernetes, managing K8s resources, troubleshooting cluster issues, configuring ingress/services, scaling deployments, or working with Kubernetes manifests. Provides comprehensive Kubernetes operational knowledge and best practices.
version: 2.0.0
---

# Kubernetes Operations

Comprehensive operational knowledge for managing Kubernetes clusters and workloads.

## Overview

Manage Kubernetes infrastructure effectively through:
- Deployment strategies and patterns
- Resource configuration and optimization
- Troubleshooting and debugging
- Security best practices
- Performance tuning

## Core Concepts

### Resource Management

**Deployments**: Use for stateless applications
- Rolling updates for zero-downtime deployments
- Rollback capabilities for failed deployments
- Replica management for scaling

**StatefulSets**: Use for stateful applications
- Stable network identities
- Persistent storage
- Ordered deployment and scaling

**DaemonSets**: Use for node-level services
- Log collectors
- Monitoring agents
- Network plugins

### Configuration

**ConfigMaps**: Store non-sensitive configuration
- Environment-specific settings
- Application configuration files
- Feature flags

**Secrets**: Store sensitive data
- API keys and tokens
- Database credentials
- TLS certificates

Use external secret management (Vault, AWS Secrets Manager) for production.

### Networking

**Services**: Expose applications internally
- ClusterIP for internal communication
- NodePort for external access (non-production)
- LoadBalancer for external access (production)

**Ingress**: HTTP/HTTPS routing
- Path-based routing
- Host-based routing
- TLS termination
- Load balancing

## Deployment Strategies

### Rolling Update

Default strategy, gradual replacement:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

**When to use**: Standard deployments, minor updates

### Recreate

Stop all pods, then create new ones:
```yaml
strategy:
  type: Recreate
```

**When to use**: Stateful apps that can't run multiple versions

### Blue-Green

Run two complete environments, switch traffic:
1. Deploy new version (green)
2. Test green environment
3. Switch traffic to green
4. Keep blue for quick rollback

**When to use**: Critical services, need instant rollback

### Canary

Gradually roll out to subset of users:
1. Deploy canary version (10% traffic)
2. Monitor metrics and errors
3. Increase traffic gradually
4. Complete rollout or rollback

**When to use**: High-risk changes, want gradual validation

## Resource Configuration

### Resource Requests and Limits

Always set for production workloads:
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**Requests**: Guaranteed resources
**Limits**: Maximum allowed resources

### Health Checks

Essential for reliability:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Liveness**: Restart unhealthy pods
**Readiness**: Remove unready pods from service

## Troubleshooting

### Common Issues

1. **Pods not starting**
   - Check: `kubectl describe pod <name>`
   - Look for: Image pull errors, resource constraints
   - Fix: Verify image name, increase resources

2. **Service not reachable**
   - Check: `kubectl get svc`, `kubectl get endpoints`
   - Look for: No endpoints, wrong selector
   - Fix: Verify pod labels match service selector

3. **High memory usage**
   - Check: `kubectl top pods`
   - Look for: Pods near memory limit
   - Fix: Increase limits, optimize application

4. **Frequent restarts**
   - Check: `kubectl get pods`, `kubectl logs <name>`
   - Look for: Liveness probe failures, OOMKilled
   - Fix: Adjust health checks, increase memory

### Debugging Commands

Get pod details:
```bash
kubectl describe pod <name>
kubectl logs <name>
kubectl logs <name> --previous  # logs from crashed container
```

Execute commands in pod:
```bash
kubectl exec -it <name> -- /bin/sh
kubectl exec <name> -- env
```

Check resource usage:
```bash
kubectl top nodes
kubectl top pods
```

## Security Best Practices

### Pod Security

- Run as non-root user
- Use read-only root filesystem
- Drop unnecessary capabilities
- Use security contexts

Example:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

### Network Policies

Restrict pod communication:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app: frontend
```

### Secrets Management

- Never commit secrets to git
- Use external secret managers
- Rotate secrets regularly
- Limit secret access with RBAC

## Performance Optimization

### Resource Tuning

1. **Start conservative**: Set low limits initially
2. **Monitor usage**: Track actual resource consumption
3. **Adjust gradually**: Increase based on metrics
4. **Set appropriate requests**: Match typical usage
5. **Set safe limits**: 2x requests for headroom

### Horizontal Pod Autoscaling

Automatically scale based on metrics:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## MCP Server Integration

This skill works with the kubernetes MCP server for operations:

**List pods**:
```javascript
const pods = await tools.k8s_list_pods({ namespace: 'default' })
```

**Get pod logs**:
```javascript
const logs = await tools.k8s_get_logs({ pod: 'api-xyz', container: 'app' })
```

**Apply manifests**:
```javascript
const result = await tools.k8s_apply_manifest({ file: 'deployment.yaml' })
```

## Detailed References

For in-depth information:
- **Deployment patterns**: `references/deployment-patterns.md`
- **Troubleshooting guide**: `references/troubleshooting.md`
- **Security hardening**: `references/security.md`

## Example Manifests

For copy-paste examples:
- **Basic deployment**: `examples/basic-deployment.yaml`
- **StatefulSet**: `examples/stateful-set.yaml`
- **Ingress config**: `examples/ingress-config.yaml`

## Validation Scripts

For manifest validation:
```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/kubernetes-ops/scripts/validate-manifest.sh deployment.yaml
```
```

### hooks/hooks.json

```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/security/scan-secrets.sh",
          "timeout": 30
        }
      ]
    },
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Evaluate if this bash command is safe for production environment. Check for destructive operations, missing safeguards, and potential security issues. Commands should be idempotent and reversible.",
          "timeout": 20
        }
      ]
    }
  ],
  "PostToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/workflow/update-status.sh",
          "timeout": 15
        }
      ]
    }
  ],
  "Stop": [
    {
      "matcher": ".*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/quality/check-config.sh",
          "timeout": 45
        },
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/workflow/notify-team.sh",
          "timeout": 30
        }
      ]
    }
  ],
  "SessionStart": [
    {
      "matcher": ".*",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/scripts/security/validate-permissions.sh",
          "timeout": 20
        }
      ]
    }
  ]
}
```

## 주요 기능 (Key Features)

### 다단계 계층형 구성 (Multi-Level Organization)

**명령어**: 기능별 구성 (CI, 모니터링, 관리)
**에이전트**: 역할 분리 (오케스트레이션 vs. 특화 에이전트)
**스킬**: 풍부한 리소스 지원 (references, examples, scripts)

### MCP 통합 (MCP Integration)

세 가지 맞춤형 MCP 서버:
- **Kubernetes**: 클러스터 오퍼레이션
- **Terraform**: 인프라 프로비저닝
- **GitHub Actions**: CI/CD 자동화

### 공유 라이브러리 (Shared Libraries)

`lib/` 하위의 재사용 가능한 코드:
- **Core**: 공통 유틸리티 (로깅, 구성 관리, 인증)
- **Integrations**: 외부 서비스 연동 (Slack, Datadog)
- **Utils**: 헬퍼 함수 (재시도, 검증)

### 구성 정보 관리 (Configuration Management)

`config/` 하위의 환경별 구성 정보:
- **Environments**: 환경별 개별 설정
- **Templates**: 재사용 가능한 배포 템플릿

### 보안 자동화 (Security Automation)

다중 보안 훅:
- 쓰기(Write) 동작 이전 비밀값 스캐닝
- 세션 시작(SessionStart) 시 권한 검증
- 완료(Stop) 시 구성 감사(Audit)

### 모니터링 통합 (Monitoring Integration)

lib 연동을 통한 내장 모니터링:
- Datadog을 통한 메트릭 조회
- PagerDuty를 통한 경보 알림
- Slack을 통한 공지 알림

## 사용 사례 (Use Cases)

1. **다중 환경 배포**: 개발/스테이징/프로덕션 환경 전반의 오케스트레이션 배포
2. **IaC(Infrastructure as Code)**: 상태 관리 기능이 포함된 Terraform 자동화
3. **CI/CD 자동화**: 빌드, 테스트 및 배포 파이프라인
4. **모니터링 및 관측 가능성**: 메트릭 및 경보 연동
5. **보안 지침 준수**: 자동 보안 스캐닝 및 검증
6. **팀 협업**: Slack 알림 및 상태 업데이트

## 이 패턴의 사용 시기 (When to Use This Pattern)

- 대규모 엔터프라이즈 환경 배포
- 다중 환경 관리
- 복잡한 CI/CD 워크플로우 운영
- 종합적인 모니터링 요구 사항이 있을 때
- 보안이 핵심적인 인프라 환경
- 긴밀한 팀 협업이 필요할 때

## 확장성 고려 사항 (Scaling Considerations)

- **성능**: 병렬 처리를 위해 독립적인 MCP 서버 구성
- **구성**: 확장성을 위해 다단계 디렉토리 레이아웃 선택
- **유지보수**: 공유 라이브러리를 통해 중복 제거
- **유연성**: 환경별 구성을 통해 개별 설정 지원
- **보안**: 계층별 보안 훅 및 세분화된 검증 적용
