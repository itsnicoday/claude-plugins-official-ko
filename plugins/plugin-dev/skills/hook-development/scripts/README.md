# 훅 개발 유틸리티 스크립트 (Hook Development Utility Scripts)

이 스크립트들은 배포하기 전에 훅 구현 내용을 검증, 테스트 및 린트(lint)하는 데 도움을 줍니다.

## validate-hook-schema.sh

`hooks.json` 구성 파일의 구조적 유효성 및 공통적인 문제를 검증합니다.

**사용법:**
```bash
./validate-hook-schema.sh path/to/hooks.json
```

**검사 항목:**
- JSON 구문 오류 여부
- 필수 필드 존재 여부
- 유효한 훅 이벤트 이름 검사
- 올바른 훅 타입 지정 여부 (command/prompt)
- 허용 범위 내의 제한 시간(timeout) 값 설정 여부
- 하드코딩된 경로 감지
- 프롬프트 훅과 이벤트 호환성 검사

**예시:**
```bash
cd my-plugin
./validate-hook-schema.sh hooks/hooks.json
```

## test-hook.sh

Claude Code에 배포하기 전에 샘플 입력을 기반으로 개별 훅 스크립트를 테스트합니다.

**사용법:**
```bash
./test-hook.sh [options] <hook-script> <test-input.json>
```

**옵션:**
- `-v, --verbose` - 세부 실행 정보 출력
- `-t, --timeout N` - 제한 시간(초) 설정 (기본값: 60)
- `--create-sample <event-type>` - 샘플 테스트 입력 JSON 생성

**예시:**
```bash
# 샘플 테스트 입력 생성
./test-hook.sh --create-sample PreToolUse > test-input.json

# 훅 스크립트 테스트
./test-hook.sh my-hook.sh test-input.json

# 세부 출력 및 커스텀 제한 시간으로 테스트
./test-hook.sh -v -t 30 my-hook.sh test-input.json
```

**기능:**
- 필수 환경 변수(CLAUDE_PROJECT_DIR, CLAUDE_PLUGIN_ROOT) 구성
- 실행 시간 측정
- 출력 JSON 구조 검증
- 종료 코드 및 의미 표시
- 환경 파일 출력값 확인

## hook-linter.sh

훅 스크립트 내의 일반적인 문제 및 모범 사례 위반 여부를 검사합니다.

**사용법:**
```bash
./hook-linter.sh <hook-script.sh> [hook-script2.sh ...]
```

**검사 항목:**
- 셰뱅(shebang) 존재 여부
- `set -euo pipefail` 사용 여부
- 표준 입력(stdin) 읽기 여부
- 적절한 에러 처리 로직
- 변수 따옴표 감싸기 여부 (인젝션 방지)
- 올바른 종료 코드 사용 여부
- 하드코딩된 경로 감지
- 오랜 시간이 소요되는 연산 감지
- 에러 정보의 표준 에러(stderr) 출력 여부
- 입력 값 검증 여부

**예시:**
```bash
# 단일 스크립트 린트
./hook-linter.sh ../examples/validate-write.sh

# 다중 스크립트 린트
./hook-linter.sh ../examples/*.sh
```

## 일반적인 작업 흐름 (Typical Workflow)

1. **훅 스크립트 작성**
   ```bash
   vim my-plugin/scripts/my-hook.sh
   ```

2. **스크립트 린트 실행**
   ```bash
   ./hook-linter.sh my-plugin/scripts/my-hook.sh
   ```

3. **테스트 입력 생성**
   ```bash
   ./test-hook.sh --create-sample PreToolUse > test-input.json
   # 필요에 따라 test-input.json 파일 편집
   ```

4. **훅 테스트 실행**
   ```bash
   ./test-hook.sh -v my-plugin/scripts/my-hook.sh test-input.json
   ```

5. **hooks.json에 추가**
   ```bash
   # my-plugin/hooks/hooks.json 파일 편집
   ```

6. **구성 정보 검증**
   ```bash
   ./validate-hook-schema.sh my-plugin/hooks/hooks.json
   ```

7. **Claude Code에서 테스트**
   ```bash
   claude --debug
   ```

## 팁 (Tips)

- 사용자의 워크플로우를 방해하지 않도록 배포하기 전에 반드시 훅을 먼저 테스트하십시오.
- 훅 동작을 디버깅하려면 상세 모드(`-v`)를 사용하십시오.
- 보안 위험 및 모범 사례 위반 여부를 잡기 위해 린터 출력을 꼼꼼히 확인하십시오.
- 변경 사항이 생긴 후에는 항상 `hooks.json` 파일을 검증하십시오.
- 다양한 상황(안전한 작업, 위험한 작업, 예외 상황 등)에 대해 개별 테스트 입력 파일을 구성해 사용하십시오.

## 일반적인 문제 해결 (Common Issues)

### 훅이 실행되지 않는 경우 (Hook doesn't execute)

확인할 사항:
- 스크립트에 셰뱅이 정의되어 있는지 확인 (`#!/bin/bash`)
- 스크립트에 실행 권한이 있는지 확인 (`chmod +x`)
- `hooks.json`에 정의된 경로가 올바른지 확인 (반드시 `${CLAUDE_PLUGIN_ROOT}` 사용)

### 훅 제한 시간 초과 (Hook times out)

- `hooks.json`에서 timeout 임계치를 줄입니다.
- 훅 스크립트 성능을 최적화합니다.
- 시간이 오래 걸리는 무거운 연산을 제거합니다.

### 훅이 에러 메시지 없이 실패하는 경우 (Hook fails silently)

- 종료 코드를 확인합니다 (반드시 0 또는 2여야 함).
- 에러 정보가 표준 에러(stderr, `>&2`)로 정상 출력되는지 확인합니다.
- 출력 JSON 구조의 유효성을 검증합니다.

### 인젝션 취약점 (Injection vulnerabilities)

- 항상 변수를 따옴표로 감싸십시오: `"$variable"`
- `set -euo pipefail` 구문을 사용하십시오.
- 모든 입력 필드 값을 사전에 검증하십시오.
- 취약점 예방을 위해 린터를 지속적으로 가동하십시오.
