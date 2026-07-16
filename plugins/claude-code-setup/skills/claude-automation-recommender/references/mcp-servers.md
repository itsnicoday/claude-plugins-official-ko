# MCP 서버 추천

MCP(Model Context Protocol) 서버는 외부 도구 및 서비스에 연결하여 Claude의 기능을 확장합니다.

**참고**: 아래는 일반적인 MCP 서버들입니다. 코드베이스의 서비스 및 연동 환경에 특화된 MCP 서버를 찾으려면 웹 검색을 활용하십시오.

## 설정 및 팀 공유 (Setup & Team Sharing)

**연결 방법:**
1. **프로젝트 설정** (`.mcp.json`) - 해당 디렉토리에서만 사용 가능
2. **글로벌 설정** (`~/.claude.json`) - 모든 프로젝트에서 사용 가능
3. **Git에 커밋된 `.mcp.json`** - 전체 팀원이 함께 사용 가능 (추천!)

**팁**: `.mcp.json`을 Git에 커밋하여 전체 팀원이 동일한 MCP 서버 설정을 공유하도록 하십시오.

**디버깅**: `claude --mcp-debug` 명령어를 사용하여 설정 문제를 식별하십시오.

## 문서 및 지식 (Documentation & Knowledge)

### context7
**적합한 용도**: 대중적인 라이브러리나 SDK를 사용하는 프로젝트에서 Claude가 최신 문서를 참조하여 코딩하기를 원할 때

| 추천 시점 | 예시 |
|----------------|----------|
| React, Vue, Angular 사용 시 | 프론트엔드 프레임워크 |
| Express, FastAPI, Django 사용 시 | 백엔드 프레임워크 |
| Prisma, Drizzle 사용 시 | ORM |
| Stripe, Twilio, SendGrid 사용 시 | 제3자(Third-party) API |
| AWS SDK, Google Cloud 사용 시 | 클라우드 SDK |
| LangChain, OpenAI SDK 사용 시 | AI/ML 라이브러리 |

**효과**: 학습 데이터에만 의존하지 않고 실시간 최신 문서를 가져옴으로써, 존재하지 않는(hallucinated) API나 오래된 코드 패턴 작성을 방지합니다.

---

## 브라우저 및 프론트엔드 (Browser & Frontend)

### Playwright MCP
**적합한 용도**: 브라우저 자동화, 테스트, 스크린샷 캡처 등이 필요한 프론트엔드 프로젝트

| 추천 시점 | 예시 |
|----------------|----------|
| React/Vue/Angular 앱 | UI 컴포넌트 테스트 |
| E2E 테스트 필요 시 | 사용자 흐름 검증 |
| 시각적 회귀 테스트 필요 시 | 스크린샷 비교 |
| UI 이슈 디버깅 | 사용자 화면 확인 |
| 폼(Form) 입력 테스트 | 다단계 워크플로우 테스트 |

**효과**: Claude가 실행 중인 앱을 조작하고 스크린샷을 찍거나, 폼을 채우고 UI 동작을 직접 검증할 수 있습니다.

### Puppeteer MCP
**적합한 용도**: 헤드리스(Headless) 브라우저 자동화, 웹 스크래핑

| 추천 시점 | 예시 |
|----------------|----------|
| HTML 기반 PDF 생성 | 리포트 생성 |
| 웹 스크래핑 작업 | 데이터 추출 |
| 헤드리스 테스트 | CI 환경 |

---

## 데이터베이스 (Databases)

### Supabase MCP
**적합한 용도**: 백엔드 또는 데이터베이스로 Supabase를 사용하는 프로젝트

| 추천 시점 | 예시 |
|----------------|----------|
| Supabase 프로젝트 감지 시 | 종속성 패키지에 `@supabase/supabase-js` 포함 |
| 인증 및 데이터베이스 필요 시 | 사용자 관리 앱 |
| 실시간(Real-time) 기능 필요 시 | 실시간 데이터 동기화 |

**효과**: Claude가 테이블을 조회하고 인증을 관리하며, Supabase 스토리지와 직접 연동할 수 있습니다.

### Convex MCP
**적합한 용도**: 백엔드로 Convex를 사용하는 프로젝트 (반응형 DB, 서버 함수, 인증, 스토리지를 단일 플랫폼에서 제공)

| 추천 시점 | 예시 |
|----------------|----------|
| Convex 프로젝트 감지 시 | 종속성 패키지에 `convex` 포함, `convex/` 디렉토리 존재, 루트에 `convex.json` 존재 |
| 실시간 / 반응형 UI | `convex/react`에서 `useQuery` / `useMutation` / `useAction` 사용 |
| 모바일 + Convex | 종속성 패키지에 `convex/react-native` 포함 |
| Convex 기반 AI / 채팅 / 에이전트 기능 | 종속성 패키지에 `@convex-dev/agent` 포함 |

**효과**: Claude가 live 배포 환경(테이블, 함수 스펙, 환경 변수, 로그)을 성찰하고, `tables`, `function-spec`, `data`, `run-once-query`, `logs`, `env list/set/get` 등의 도구를 통해 쿼리나 뮤테이션을 실행할 수 있습니다. `npx convex mcp start`를 통해 실행합니다.

### PostgreSQL MCP
**적합한 용도**: PostgreSQL 데이터베이스에 직접 연결

| 추천 시점 | 예시 |
|----------------|----------|
| 순수 PostgreSQL 사용 시 | ORM 레이어 없음 |
| DB 마이그레이션 | 스키마 관리 |
| 데이터 분석 작업 | 복잡한 쿼리 조회 |
| 데이터 이슈 디버깅 | 실제 데이터 확인 |

### Neon MCP
**적합한 용도**: Neon 서버리스 Postgres 사용자

### Turso MCP
**적합한 용도**: Turso/libSQL edge 데이터베이스 사용자

---

## 버전 관리 및 DevOps (Version Control & DevOps)

### GitHub MCP
**적합한 용도**: GitHub 호스팅 저장소의 이슈 및 PR 연동

| 추천 시점 | 예시 |
|----------------|----------|
| GitHub 저장소 | GitHub 원격 저장소가 설정된 `.git` |
| 이슈 기반 개발 | 커밋 시 이슈 번호 참조 |
| PR 워크플로우 | 리뷰 및 머지(Merge) 작업 |
| GitHub Actions | CI/CD 파이프라인 접근 |
| 릴리스 관리 | 태그 및 릴리스 자동화 |

**효과**: Claude가 이슈 생성, PR 리뷰, 워크플로우 실행 확인, 릴리스 관리를 수행할 수 있습니다.

### GitLab MCP
**적합한 용도**: GitLab 호스팅 저장소

### Linear MCP
**적합한 용도**: Linear로 이슈를 추적하는 팀

| 추천 시점 | 예시 |
|----------------|----------|
| Linear 워크스페이스 | `ABC-123` 형태의 이슈 번호 참조 |
| 스프린트 계획 | 백로그 관리 |
| 코드에서 이슈 생성 | TODO 주석을 분석하여 이슈 자동 생성 |

---

## 클라우드 인프라 (Cloud Infrastructure)

### AWS MCP
**적합한 용도**: AWS 인프라 관리

| 추천 시점 | 예시 |
|----------------|----------|
| 종속성 패키지에 AWS SDK 포함 | `@aws-sdk/*` 패키 |
| 코드형 인프라 (IaC) | Terraform, CDK, SAM |
| Lambda 개발 | 서버리스 함수 |
| S3, DynamoDB 사용 | 클라우드 데이터 서비스 |

### Cloudflare MCP
**적합한 용도**: Cloudflare Workers, Pages, R2, D1

| 추천 시점 | 예시 |
|----------------|----------|
| Cloudflare Workers | 에지(Edge) 함수 |
| Pages 배포 | 정적 사이트 호스팅 |
| R2 스토리지 | 객체 스토리지 |
| D1 데이터베이스 | 에지 SQL 데이터베이스 |

### Vercel MCP
**적합한 용도**: Vercel 배포 및 설정

---

## 모니터링 및 관측성 (Monitoring & Observability)

### Sentry MCP
**적합한 용도**: 에러 추적 및 디버깅

| 추천 시점 | 예시 |
|----------------|----------|
| Sentry 설정 시 | 종속성 패키지에 `@sentry/*` 포함 |
| 운영 환경 디버깅 | 에러 분석 |
| 에러 패턴 분석 | 유사 에러 그룹화 |
| 릴리스 추적 | 배포와 에러 발생 시점 연계 |

**효과**: Claude가 Sentry 이슈를 조사하고, 근본 원인을 찾아 해결책을 제시할 수 있습니다.

### Datadog MCP
**적합한 용도**: APM, 로그, 메트릭 모니터링

---

## 커뮤니케이션 (Communication)

### Slack MCP
**적합한 용도**: Slack 워크스페이스 연동

| 추천 시점 | 예시 |
|----------------|----------|
| Slack 사용 팀 | 알림 발송 |
| 배포 알림 | 알림 채널 구성 |
| 인시던트 대응 | 업데이트 정보 공유 |

### Notion MCP
**적합한 용도**: Notion 워크스페이스 기반 문서 관리

| 추천 시점 | 예시 |
|----------------|----------|
| Notion을 문서 도구로 사용 | 페이지 읽기/수정 |
| 기술 지식 보관소 | 문서 검색 |
| 회의록 작성 | 회의록 요약본 생성 |

---

## 파일 및 데이터 (File & Data)

### Filesystem MCP
**적합한 용도**: 내장 도구 이상의 향상된 파일 조작 기능 제공

| 추천 시점 | 예시 |
|----------------|----------|
| 복잡한 파일 작업 | 일괄 처리(Batch processing) |
| 파일 모니터링 | 변경 사항 감시 |
| 고급 검색 | 커스텀 검색 패턴 적용 |

### Memory MCP
**적합한 용도**: 세션 간 유지되는 영구 메모리 제공

| 추천 시점 | 예시 |
|----------------|----------|
| 장기 프로젝트 | 컨텍스트 기억 |
| 사용자 선호도 설정 | 환경 설정 저장 |
| 학습 패턴 누적 | 기술 지식 축적 |

**효과**: Claude가 여러 대화에 걸쳐 프로젝트 컨텍스트, 결정 사항, 코딩 패턴을 기억합니다.

---

## 컨테이너 및 DevOps (Containers & DevOps)

### Docker MCP
**적합한 용도**: 컨테이너 관리

| 추천 시점 | 예시 |
|----------------|----------|
| Docker Compose 파일 존재 | 컨테이너 오케스트레이션 |
| Dockerfile 존재 | 이미지 빌드 |
| 컨테이너 디버깅 | 로그 확인, exec 명령 실행 |

### Kubernetes MCP
**적합한 용도**: Kubernetes 클러스터 관리

| 추천 시점 | 예시 |
|----------------|----------|
| K8s manifest 파일 | 파드(Pod) 배포 및 스케일링 |
| Helm 차트 | 패키지 관리 |
| 클러스터 디버깅 | 파드 로그, 상태 정보 확인 |

---

## AI & ML

### Exa MCP
**적합한 용도**: 웹 검색 및 조사

| 추천 시점 | 예시 |
|----------------|----------|
| 정보 조사 작업 | 최신 정보 검색 |
| 경쟁 분석 | 시장 조사 |
| 부족한 기술 문서 보완 | 코드 예제 검색 |

---

## 빠른 참조: 감지 패턴 (Quick Reference: Detection Patterns)

| 감지 조건 | 추천 MCP 서버 |
|----------|-------------------|
| 대중적인 npm 패키지 | context7 |
| React/Vue/Next.js | Playwright MCP |
| `@supabase/supabase-js` | Supabase MCP |
| 종속성 패키지에 `convex`, `convex/` 디렉토리, 또는 `convex.json` | Convex MCP |
| `pg` 또는 `postgres` | PostgreSQL MCP |
| GitHub 원격 저장소 | GitHub MCP |
| `.linear` 또는 Linear 관련 코드 | Linear MCP |
| `@aws-sdk/*` | AWS MCP |
| `@sentry/*` | Sentry MCP |
| `docker-compose.yml` | Docker MCP |
| Slack 웹훅 URL | Slack MCP |
| `@anthropic-ai/sdk` | Anthropic 문서용 context7 |
