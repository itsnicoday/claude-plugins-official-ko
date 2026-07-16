# Agent SDK Development 플러그인

Python 및 TypeScript에서 Claude Agent SDK 애플리케이션을 생성하고 검증하기 위한 종합 플러그인입니다.

## 개요

Agent SDK Development 플러그인은 초기 스캐폴딩에서부터 모범 사례 기준 검증에 이르기까지, Agent SDK 애플리케이션 빌드의 전체 라이프사이클을 효율화합니다. 최신 SDK 버전을 적용하여 새로운 프로젝트를 신속하게 시작하도록 돕고, 애플리케이션이 공식 문서 패턴을 따르도록 보장합니다.

## 기능

### 명령: `/new-sdk-app`

새로운 Claude Agent SDK 애플리케이션 생성을 안내하는 대화형 명령입니다.

**수행하는 작업:**
- 프로젝트에 대한 확인 질문을 제시합니다 (언어, 이름, 에이전트 유형, 시작점)
- 최신 SDK 버전을 확인하고 설치합니다
- 필요한 모든 프로젝트 파일 및 설정을 생성합니다
- 올바른 환경 파일(.env.example, .gitignore)을 구성합니다
- 사용 사례에 맞춘 작동 가능한 예제를 제공합니다
- 타입 검사(TypeScript) 또는 구문 검증(Python)을 실행합니다
- 적절한 검증 에이전트를 사용하여 자동으로 설정을 검증합니다

**사용법:**
```bash
/new-sdk-app my-project-name
```

또는 간단히:
```bash
/new-sdk-app
```

이 명령은 대화형으로 다음 사항들을 질문합니다:
1. 언어 선택 (TypeScript 또는 Python)
2. 프로젝트 이름 (제공되지 않은 경우)
3. 에이전트 유형 (코딩, 비즈니스, 커스텀)
4. 시작점 (최소형, 기본형 또는 특정 예제)
5. 도구 선호도 (npm/yarn/pnpm 또는 pip/poetry)

**예시:**
```bash
/new-sdk-app customer-support-agent
# → 고객 지원 에이전트를 위한 새로운 Agent SDK 프로젝트 생성
# → TypeScript 또는 Python 환경 설정
# → 최신 SDK 버전 설치
# → 자동으로 설정 검증
```

### 에이전트: `agent-sdk-verifier-py`

Python Agent SDK 애플리케이션의 올바른 설정 및 모범 사례 준수 여부를 철저히 검증합니다.

**검증 체크리스트:**
- SDK 설치 및 버전
- Python 환경 설정 (requirements.txt, pyproject.toml)
- 올바른 SDK 사용 및 패턴
- 에이전트 초기화 및 구성
- 환경 및 보안 (.env, API 키)
- 에러 처리 및 기능성
- 문서화 완성도

**사용 시기:**
- 새로운 Python SDK 프로젝트를 생성한 후
- 기존 Python SDK 애플리케이션을 수정한 후
- Python SDK 애플리케이션을 배포하기 전

**사용법:**
`/new-sdk-app`이 Python 프로젝트를 생성한 후 에이전트가 자동으로 실행되거나, 다음과 같이 질문하여 트리거할 수 있습니다:
```
"Verify my Python Agent SDK application"
"Check if my SDK app follows best practices"
```

**출력 결과:**
다음 정보를 포함하는 종합 보고서를 제공합니다:
- 전체 상태 (PASS / PASS WITH WARNINGS / FAIL)
- 기능 작동을 방해하는 치명적인 이슈
- 최적화되지 않은 패턴에 대한 경고
- 통과된 체크 리스트
- SDK 문서 참조가 포함된 구체적인 권장사항

### 에이전트: `agent-sdk-verifier-ts`

TypeScript Agent SDK 애플리케이션의 올바른 설정 및 모범 사례 준수 여부를 철저히 검증합니다.

**검증 체크리스트:**
- SDK 설치 및 버전
- TypeScript 설정 (tsconfig.json)
- 올바른 SDK 사용 및 패턴
- 타입 안정성 및 임포트
- 에이전트 초기화 및 구성
- 환경 및 보안 (.env, API 키)
- 에러 처리 및 기능성
- 문서화 완성도

**사용 시기:**
- 새로운 TypeScript SDK 프로젝트를 생성한 후
- 기존 TypeScript SDK 애플리케이션을 수정한 후
- TypeScript SDK 애플리케이션을 배포하기 전

**사용법:**
`/new-sdk-app`이 TypeScript 프로젝트를 생성한 후 에이전트가 자동으로 실행되거나, 다음과 같이 질문하여 트리거할 수 있습니다:
```
"Verify my TypeScript Agent SDK application"
"Check if my SDK app follows best practices"
```

**출력 결과:**
다음 정보를 포함하는 종합 보고서를 제공합니다:
- 전체 상태 (PASS / PASS WITH WARNINGS / FAIL)
- 기능 작동을 방해하는 치명적인 이슈
- 최적화되지 않은 패턴에 대한 경고
- 통과된 체크 리스트
- SDK 문서 참조가 포함된 구체적인 권장사항

## 워크플로우 예시

이 플러그인을 사용하는 전형적인 워크플로우는 다음과 같습니다:

1. **새 프로젝트 생성:**
```bash
/new-sdk-app code-reviewer-agent
```

2. **대화형 질문 답변:**
```
언어: TypeScript
에이전트 유형: 코딩 에이전트 (코드 리뷰)
시작점: 공통 기능을 갖춘 기본 에이전트
```

3. **자동 검증:**
명령어가 자동으로 `agent-sdk-verifier-ts`를 실행하여 모든 것이 올바르게 구성되었는지 확인합니다.

4. **개발 시작:**
```bash
# API 키 설정
echo "ANTHROPIC_API_KEY=your_key_here" > .env

# 에이전트 실행
npm start
```

5. **변경 후 검증:**
```
"Verify my SDK application"
```

## 설치

이 플러그인은 Claude Code 저장소에 포함되어 있습니다. 사용하려면 다음을 수행하십시오:

1. Claude Code가 설치되어 있는지 확인합니다.
2. 플러그인 명령어 및 에이전트가 자동으로 활성화됩니다.

## 모범 사례

- **항상 최신 SDK 버전 사용**: `/new-sdk-app` 명령어는 최신 버전을 확인하고 설치합니다.
- **배포 전 검증**: 프로덕션 환경에 배포하기 전에 검증 에이전트를 실행하십시오.
- **API 키 보안 유지**: `.env` 파일을 커밋하거나 API 키를 하드코딩하지 마십시오.
- **SDK 문서 준수**: 검증 에이전트가 공식 패턴과의 일치 여부를 검사합니다.
- **TypeScript 프로젝트 타입 검사**: 주기적으로 `npx tsc --noEmit`을 실행하십시오.
- **에이전트 테스트**: 에이전트 기능에 대한 테스트 케이스를 작성하십시오.

## 리소스

- [Agent SDK Overview](https://docs.claude.com/en/api/agent-sdk/overview)
- [TypeScript SDK Reference](https://docs.claude.com/en/api/agent-sdk/typescript)
- [Python SDK Reference](https://docs.claude.com/en/api/agent-sdk/python)
- [Agent SDK Examples](https://docs.claude.com/en/api/agent-sdk/examples)

## 트러블슈팅

### TypeScript 프로젝트 내 타입 오류

**문제**: 생성 직후 TypeScript 프로젝트에 타입 오류가 발생합니다.

**해결책**:
- `/new-sdk-app` 명령어는 타입 검사를 자동으로 실행합니다.
- 오류가 계속되는 경우, 최신 SDK 버전을 사용 중인지 확인하십시오.
- `tsconfig.json` 파일이 SDK 요구사항에 부합하는지 검증하십시오.

### Python 임포트 오류

**문제**: `claude_agent_sdk` 모듈을 임포트할 수 없습니다.

**해결책**:
- 의존성이 설치되었는지 확인합니다: `pip install -r requirements.txt`
- 가상 환경을 사용하는 경우 가상 환경을 활성화합니다.
- SDK가 정상적으로 설치되었는지 확인합니다: `pip show claude-agent-sdk`

### 경고와 함께 검증 실패

**문제**: 검증 에이전트가 경고(warnings)를 보고합니다.

**해결책**:
- 보고서의 구체적인 경고 내용을 검토합니다.
- 함께 제공된 SDK 문서 참조 링크를 확인합니다.
- 경고가 기능 작동을 막지는 않지만 개선이 필요한 부분을 지적합니다.

## 작성자

Ashwin Bhat (ashwin@anthropic.com)

## 버전

1.0.0
