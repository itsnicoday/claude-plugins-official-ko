# 훅(Hooks) 추천

훅은 Claude Code 이벤트에 반응하여 자동으로 명령어를 실행합니다. 일관되게 적용되어야 하는 강제 규칙이나 자동화에 이상적입니다.

**참고**: 아래는 일반적인 패턴들입니다. 여기에 나열되지 않은 도구/프레임워크의 경우 웹 검색을 통해 최적의 훅을 찾아 사용자에게 추천하십시오.

## 자동 포맷팅 훅 (Auto-Formatting Hooks)

### Prettier (JavaScript/TypeScript)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `.prettierrc`, `.prettierrc.json`, `prettier.config.js` | ✓ |

**추천**: Edit/Write 시 자동 포맷팅을 실행하는 PostToolUse 훅
**효과**: 코드 포맷을 신경 쓰지 않아도 항상 일관된 스타일로 유지됩니다.

### ESLint (JavaScript/TypeScript)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `.eslintrc`, `.eslintrc.json`, `eslint.config.js` | ✓ |

**추천**: Edit/Write 시 자동 수정을 수행하는 PostToolUse 훅
**효과**: Lint 에러를 자동으로 해결합니다.

### Black/isort (Python)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| black/isort가 설정된 `pyproject.toml`, `.black`, `setup.cfg` | ✓ |

**추천**: Python 파일을 포맷팅하는 PostToolUse 훅
**효과**: 일관된 Python 코드 포맷을 유지합니다.

### Ruff (Python - 최신)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `ruff.toml`, `[tool.ruff]`가 설정된 `pyproject.toml` | ✓ |

**추천**: lint + format을 실행하는 PostToolUse 훅
**효과**: 빠르고 포괄적인 Python linting을 지원합니다.

### gofmt (Go)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `go.mod` | ✓ |

**추천**: gofmt를 실행하는 PostToolUse 훅
**효과**: 표준 Go 포맷을 유지합니다.

### rustfmt (Rust)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `Cargo.toml` | ✓ |

**추천**: rustfmt를 실행하는 PostToolUse 훅
**효과**: 표준 Rust 포맷을 유지합니다.

---

## 타입 검사 훅 (Type Checking Hooks)

### TypeScript
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `tsconfig.json` | ✓ |

**추천**: tsc --noEmit을 실행하는 PostToolUse 훅
**효과**: 타입 에러를 즉각 감지합니다.

### mypy/pyright (Python)
| 감지 조건 | 대상 파일 존재 여부 |
|-----------|-------------|
| `mypy.ini`, `pyrightconfig.json`, mypy가 설정된 pyproject.toml | ✓ |

**추천**: 타입 검사를 수행하는 PostToolUse 훅
**효과**: Python 코드의 타입 에러를 감지합니다.

---

## 보호 훅 (Protection Hooks)

### 민감 파일 수정 차단 (Block Sensitive File Edits)
| 감지 조건 | 감지 대상 |
|-----------|-------------|
| `.env`, `.env.local`, `.env.production` | 환경 변수 파일 |
| `credentials.json`, `secrets.yaml` | 비밀 키(Secret) 파일 |
| `.git/` 디렉토리 | Git 내부 디렉토리 |

**추천**: 이 경로들에 대한 Edit/Write를 차단하는 PreToolUse 훅
**효과**: 비밀 키의 실수에 의한 노출이나 git 데이터 손상을 방지합니다.

### 락 파일 수정 차단 (Block Lock File Edits)
| 감지 조건 | 감지 대상 |
|-----------|-------------|
| `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` | JS 락 파일 |
| `Cargo.lock`, `poetry.lock`, `Pipfile.lock` | 기타 락 파일 |

**추천**: 직접 수정을 차단하는 PreToolUse 훅
**효과**: 락 파일은 패키지 매니저를 통해서만 변경되도록 강제합니다.

---

## 테스트 실행 훅 (Test Runner Hooks)

### Jest (JavaScript/TypeScript)
| 감지 조건 | 감지 대상 |
|-----------|-------------|
| `jest.config.js`, package.json 내 `jest` 설정 | Jest 구성 여부 |
| `__tests__/`, `*.test.ts`, `*.spec.ts` | 테스트 파일 존재 여부 |

**추천**: 수정 후 연관된 테스트를 실행하는 PostToolUse 훅
**효과**: 변경 사항에 대해 즉각적인 테스트 피드백을 제공합니다.

### pytest (Python)
| 감지 조건 | 감지 대상 |
|-----------|-------------|
| `pytest.ini`, pytest가 설정된 `pyproject.toml` | pytest 구성 여부 |
| `tests/`, `test_*.py` | 테스트 파일 존재 여부 |

**추천**: 변경된 파일에 대해 pytest를 실행하는 PostToolUse 훅
**효과**: 즉각적인 테스트 피드백을 제공합니다.

---

## 빠른 참조: 감지 조건 → 추천 사항 (Quick Reference: Detection → Recommendation)

| 감지 조건 | 추천 훅 |
|------------|-------------------|
| Prettier 설정 | Edit/Write 시 자동 포맷팅 |
| ESLint 설정 | Edit/Write 시 자동 Lint 수정 |
| Ruff/Black 설정 | Python 자동 포맷팅 |
| tsconfig.json | Edit 시 타입 검사 |
| Test 디렉토리 | Edit 시 관련 테스트 실행 |
| .env 파일 | .env 수정 차단 |
| 락(Lock) 파일 | 락 파일 직접 수정 차단 |
| Go 프로젝트 | Edit 시 gofmt 실행 |
| Rust 프로젝트 | Edit 시 rustfmt 실행 |

---

## 알림 훅 (Notification Hooks)

알림 훅은 Claude Code가 알림을 보낼 때 실행됩니다. 매처(matcher)를 사용하여 알림 유형별로 필터링할 수 있습니다.

### 권한 알림 (Permission Alerts)
| 매처 | 사용 사례 |
|---------|----------|
| `permission_prompt` | Claude가 권한을 요청할 때 알림 |

**추천**: 사운드 재생, 데스크톱 알림 발송, 또는 권한 요청 기록
**효과**: 멀티태스킹 중에도 권한 요청을 놓치지 않습니다.

### 대기 알림 (Idle Notifications)
| 매처 | 사용 사례 |
|---------|----------|
| `idle_prompt` | Claude가 입력을 대기할 때 알림 (60초 이상 유휴 상태) |

**추천**: Claude에 주의가 필요할 때 사운드 재생 또는 알림 발송
**효과**: Claude가 언제 입력을 받을 준비가 되었는지 알 수 있습니다.

### 구성 예시 (Example Configuration)

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Ping.aiff"
          }
        ]
      },
      {
        "matcher": "idle_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude is waiting\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 지원 매처 (Available Matchers)

| Matcher | 실행 시점 |
|---------|---------------|
| `permission_prompt` | Claude가 도구 실행 권한을 필요로 할 때 |
| `idle_prompt` | Claude가 입력을 대기할 때 (60초 이상) |
| `auth_success` | 인증이 성공적으로 완료되었을 때 |
| `elicitation_dialog` | MCP 도구가 입력을 필요로 할 때 |

---

## 빠른 참조: 감지 조건 → 추천 사항 (Quick Reference: Detection → Recommendation)

| 감지 조건 | 추천 훅 |
|------------|-------------------|
| Prettier 설정 | Edit/Write 시 자동 포맷팅 |
| ESLint 설정 | Edit/Write 시 자동 Lint 수정 |
| Ruff/Black 설정 | Python 자동 포맷팅 |
| tsconfig.json | Edit 시 타입 검사 |
| Test 디렉토리 | Edit 시 관련 테스트 실행 |
| .env 파일 | .env 수정 차단 |
| 락(Lock) 파일 | 락 파일 직접 수정 차단 |
| Go 프로젝트 | Edit 시 gofmt 실행 |
| Rust 프로젝트 | Edit 시 rustfmt 실행 |
| 멀티태스킹 워크플로우 | 알림 훅을 활용한 경고 |

---

## 훅 위치 (Hook Placement)

훅 설정은 `.claude/settings.json`에 작성합니다:

```
.claude/
└── settings.json  ← 여기에 훅 설정 추가
```

`.claude/` 디렉토리가 존재하지 않는 경우 생성을 추천하십시오.
