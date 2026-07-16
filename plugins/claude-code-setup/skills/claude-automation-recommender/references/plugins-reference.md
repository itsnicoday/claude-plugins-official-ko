# 플러그인(Plugin) 추천

플러그인은 설치 가능한 skill, 명령어, 에이전트, 훅의 모음입니다. `/plugin install`을 통해 설치합니다.

**참고**: 아래는 공식 저장소의 플러그인들입니다. 추가적인 커뮤니티 플러그인을 찾으려면 웹 검색을 활용하십시오.

---

## 공식 플러그인 (Official Plugins)

### 개발 및 코드 품질 (Development & Code Quality)

| 플러그인 | 적합한 용도 | 주요 기능 |
|--------|----------|--------------|
| **plugin-dev** | Claude Code 플러그인 제작 | skill, 훅, 명령어, 에이전트 제작용 skill 제공 |
| **pr-review-toolkit** | PR 리뷰 워크플로우 | 특화된 리뷰 에이전트 제공 (코드, 테스트, 타입) |
| **code-review** | 자동 코드 리뷰 | 신뢰도 점수를 동반한 멀티 에이전트 리뷰 |
| **code-simplifier** | 코드 리팩토링 | 기능을 유지하면서 코드를 단순화 |
| **feature-dev** | 기능 개발 | 에이전트를 활용한 엔드투엔드 기능 개발 워크플로우 |

### Git & 워크플로우 (Git & Workflow)

| 플러그인 | 적합한 용도 | 주요 기능 |
|--------|----------|--------------|
| **commit-commands** | Git 워크플로우 | /commit, /commit-push-pr 명령어 제공 |
| **hookify** | 자동화 규칙 생성 | 대화 패턴에서 훅 자동 생성 |

### 프론트엔드 (Frontend)

| 플러그인 | 적합한 용도 | 주요 기능 |
|--------|----------|--------------|
| **frontend-design** | UI 개발 | 상용 수준의 UI 제작, 정형화된 디자인 탈피 |

### 학습 및 가이드 (Learning & Guidance)

| 플러그인 | 적합한 용도 | 주요 기능 |
|--------|----------|--------------|
| **explanatory-output-style** | 학습 목적 | 코드 선택에 대한 교육적인 설명 제공 |
| **learning-output-style** | 대화형 학습 | 결정 시점에 사용자의 참여 유도 |
| **security-guidance** | 보안 의식 고취 | 코드 수정 시 보안 문제에 대해 경고 |

### 언어 서버 (LSP)

| 플러그인 | 언어 |
|--------|----------|
| **typescript-lsp** | TypeScript/JavaScript |
| **pyright-lsp** | Python |
| **gopls-lsp** | Go |
| **rust-analyzer-lsp** | Rust |
| **clangd-lsp** | C/C++ |
| **jdtls-lsp** | Java |
| **kotlin-lsp** | Kotlin |
| **swift-lsp** | Swift |
| **csharp-lsp** | C# |
| **php-lsp** | PHP |
| **lua-lsp** | Lua |

---

## 빠른 참조: 코드베이스 상황 → 추천 플러그인 (Quick Reference: Codebase → Plugin)

| 코드베이스 신호 | 추천 플러그인 |
|-----------------|-------------------|
| 플러그인 제작 중 | plugin-dev |
| PR 기반 워크플로우 | pr-review-toolkit |
| Git 커밋 작업 | commit-commands |
| React/Vue/Angular 사용 | frontend-design |
| 자동화 규칙 필요 | hookify |
| TypeScript 프로젝트 | typescript-lsp |
| Python 프로젝트 | pyright-lsp |
| Go 프로젝트 | gopls-lsp |
| 보안이 중요한 코드 | security-guidance |
| 온보딩 및 학습 목적 | explanatory-output-style |

---

## 플러그인 관리 (Plugin Management)

```bash
# 플러그인 설치
/plugin install <plugin-name>

# 설치된 플러그인 목록 확인
/plugin list

# 플러그인 상세 정보 보기
/plugin info <plugin-name>
```

---

## 플러그인 추천 시점 (When to Recommend Plugins)

**다음과 같은 경우 플러그인 설치를 추천하십시오:**
- 사용자가 Anthropic 공식 저장소나 다른 공유 마켓플레이스에서 Claude Code 자동화 도구를 설치하고자 할 때
- 연관된 여러 기능이 복합적으로 필요할 때
- 팀 전체에서 워크플로우를 표준화하고자 할 때
- 최초로 Claude Code를 설정할 때
