---
description: 새로운 Claude Agent SDK 애플리케이션 생성 및 설정
argument-hint: [project-name]
---

귀하는 사용자가 새로운 Claude Agent SDK 애플리케이션을 생성하는 것을 돕는 역할을 맡았습니다. 다음 단계를 주의 깊게 따르십시오:

## 참조 문서

시작하기 전에 공식 문서를 검토하여 정확하고 최신의 가이드를 제공할 수 있도록 하십시오. WebFetch를 사용하여 다음 페이지를 확인하십시오:

1. **개요부터 시작**: https://docs.claude.com/en/api/agent-sdk/overview
2. **사용자가 선택한 언어에 따른 SDK 참조 문서 확인**:
   - TypeScript: https://docs.claude.com/en/api/agent-sdk/typescript
   - Python: https://docs.claude.com/en/api/agent-sdk/python
3. **개요에 언급된 주요 가이드 확인**:
   - 스트리밍 vs 단일 모드 (Streaming vs Single Mode)
   - 권한 설정 (Permissions)
   - 커스텀 도구 (Custom Tools)
   - MCP 연동 (MCP integration)
   - 서브 에이전트 (Subagents)
   - 세션 처리 (Sessions)
   - 사용자 요구사항에 부합하는 기타 관련 가이드

**중요**: 항상 최신 패키지 버전을 확인하고 사용하십시오. 설치하기 전에 WebSearch 또는 WebFetch를 통해 현재 버전을 검증하십시오.

## 요구사항 수집

중요: 아래 질문은 한 번에 하나씩만 하십시오. 질문을 던진 후 사용자의 답변을 기다렸다가 다음 질문을 하십시오. 이를 통해 사용자가 보다 수월하게 응답할 수 있습니다.

질문은 다음 순서대로 진행합니다 (사용자가 인자를 통해 이미 제공한 정보는 건너뜁니다):

1. **언어** (첫 번째 질문): "TypeScript와 Python 중 어떤 언어를 사용하시겠습니까?"
   - 계속 진행하기 전에 응답을 기다립니다.

2. **프로젝트 이름** (두 번째 질문): "프로젝트 이름을 무엇으로 지정하시겠습니까?"
   - 만약 `$ARGUMENTS`가 제공된 경우 이를 프로젝트 이름으로 사용하고 이 질문은 건너뜁니다.
   - 계속 진행하기 전에 응답을 기다립니다.

3. **에이전트 유형** (세 번째 질문. 단, 2번 질문에 대한 답변이 충분히 상세했던 경우 건너뜁니다): "어떤 종류의 에이전트를 빌드하려고 하십니까? 예시:
   - 코딩 에이전트 (SRE, 보안 검토, 코드 리뷰)
   - 비즈니스 에이전트 (고객 지원, 콘텐츠 작성)
   - 커스텀 에이전트 (사용 사례를 직접 설명)"
   - 계속 진행하기 전에 응답을 기다립니다.

4. **시작점** (네 번째 질문): "다음 중 무엇을 원하십니까?
   - 개발 시작을 위한 최소형 'Hello World' 예제
   - 일반적인 기능이 포함된 기본 에이전트
   - 귀하의 사용 사례에 특화된 특정 예제"
   - 계속 진행하기 전에 응답을 기다립니다.

5. **도구 선택** (다섯 번째 질문): 사용할 도구들을 사용자에게 안내하고, 해당 도구들이 사용자가 원하는 것이 맞는지 확인합니다 (예: npm 대신 pnpm이나 bun을 선호할 수 있습니다). 요구사항을 구현할 때 사용자의 기본 설정을 최대한 반영하십시오.

모든 질문에 대한 답변이 수집되면, 설정 계획(setup plan) 수립을 진행합니다.

## 설정 계획

사용자의 답변을 기반으로 다음 사항들을 포함하는 계획을 수립합니다:

1. **프로젝트 초기화**:
   - 프로젝트 디렉터리 생성 (존재하지 않는 경우)
   - 패키지 매니저 초기화:
     - TypeScript: `npm init -y`를 실행하고 package.json을 `type: "module"` 및 적절한 scripts(typecheck 스크립트 포함)로 구성
     - Python: `requirements.txt`를 생성하거나 `poetry init` 사용
   - 필요한 설정 파일 생성:
     - TypeScript: SDK에 부합하는 적절한 설정이 포함된 `tsconfig.json` 생성
     - Python: 필요에 따라 부가적인 설정 파일 구성

2. **최신 버전 확인**:
   - 설치하기 **전에** WebSearch를 실행하거나 npm/PyPI를 체크하여 최신 버전을 확인합니다.
   - TypeScript의 경우: https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk 확인
   - Python의 경우: https://pypi.org/project/claude-agent-sdk/ 확인
   - 설치할 버전을 사용자에게 안내합니다.

3. **SDK 설치**:
   - TypeScript: `npm install @anthropic-ai/claude-agent-sdk@latest` (또는 특정 최신 버전 지정)
   - Python: `pip install claude-agent-sdk` (pip는 기본적으로 최신 버전을 설치함)
   - 설치 후 실제로 설치된 버전을 검증합니다:
     - TypeScript: package.json 확인 또는 `npm list @anthropic-ai/claude-agent-sdk` 실행
     - Python: `pip show claude-agent-sdk` 실행

4. **스타터 파일 생성**:
   - TypeScript: 기본적인 쿼리 예제를 포함하는 `index.ts` 또는 `src/index.ts` 생성
   - Python: 기본적인 쿼리 예제를 포함하는 `main.py` 생성
   - 올바른 임포트 구문 및 기본적인 에러 처리 포함
   - 최신 SDK 버전의 모던하고 규격화된 문법 및 패턴 적용

5. **환경 설정**:
   - `ANTHROPIC_API_KEY=your_api_key_here`를 포함하는 `.env.example` 파일 생성
   - `.env` 파일을 `.gitignore`에 등록
   - https://console.anthropic.com/ 에서 API 키를 획득하는 방법 안내

6. **선택 사항: .claude 디렉터리 구조 생성**:
   - 에이전트, 명령, 설정을 관리할 `.claude/` 디렉터리 생성 여부 제안
   - 예제 서브 에이전트나 슬래시 명령어 추가를 원하는지 문의

## 구현

요구사항 수집을 마치고 계획에 대한 사용자의 승인을 얻은 후 다음을 실행합니다:

1. WebSearch 또는 WebFetch를 사용하여 최신 패키지 버전을 확인합니다.
2. 설정 단계를 순서대로 수행합니다.
3. 필요한 모든 파일을 생성합니다.
4. 의존성을 설치합니다 (항상 최신 안정 버전을 사용하십시오).
5. 설치된 버전을 확인하여 사용자에게 안내합니다.
6. 사용자가 구상하는 에이전트 유형에 맞춘 작동 가능한 예제를 생성합니다.
7. 각 코드 파트의 역할을 설명하는 유용한 주석을 추가합니다.
8. **종료하기 전에 코드가 정상 작동하는지 검증하십시오**:
   - TypeScript의 경우:
     - `npx tsc --noEmit`을 실행하여 타입 에러 유무 확인
     - 타입 검사가 완전히 통과할 때까지 모든 타입 에러를 수정
     - 임포트 및 타입의 유효성 보장
     - 타입 검사가 에러 없이 통과한 경우에만 계속 진행
   - Python의 경우:
     - 임포트의 유효성 검증
     - 기본적인 구문 오류 확인
   - **코드가 완벽하게 검증되기 전까지는 설정을 마친 것으로 간주하지 마십시오.**

## 검증

모든 파일 생성과 의존성 설치가 끝나면, 해당하는 검증 에이전트를 실행하여 Agent SDK 애플리케이션이 올바르게 구성되었고 사용 준비가 되었는지 검증합니다:

1. **TypeScript 프로젝트의 경우**: **agent-sdk-verifier-ts** 에이전트를 실행하여 설정을 검증합니다.
2. **Python 프로젝트의 경우**: **agent-sdk-verifier-py** 에이전트를 실행하여 설정을 검증합니다.
3. 에이전트는 SDK 사용법, 구성, 기능 및 공식 문서 준수 여부를 검사합니다.
4. 검증 보고서를 검토하고 발견된 문제를 해결합니다.

## 시작 가이드

설정 및 검증이 모두 끝나면 사용자에게 다음을 제공합니다:

1. **후속 단계**:
   - API 키를 설정하는 방법
   - 에이전트를 실행하는 방법:
     - TypeScript: `npm start` 또는 `node --loader ts-node/esm index.ts`
     - Python: `python main.py`

2. **유용한 리소스**:
   - TypeScript SDK 참조 문서 링크: https://docs.claude.com/en/api/agent-sdk/typescript
   - Python SDK 참조 문서 링크: https://docs.claude.com/en/api/agent-sdk/python
   - 핵심 개념 설명: 시스템 프롬프트, 권한 설정, 도구, MCP 서버

3. **기타 일반적인 후속 조치**:
   - 시스템 프롬프트를 커스터마이징하는 방법
   - MCP를 통해 커스텀 도구를 추가하는 방법
   - 권한을 구성하는 방법
   - 서브 에이전트를 생성하는 방법

## 중요 주의사항

- **항상 최신 버전 사용**: 패키지를 설치하기 전에 항상 WebSearch를 사용하거나 npm/PyPI를 직접 조회하여 최신 버전을 확인하십시오.
- **코드의 정상 작동 여부 검증**:
  - TypeScript의 경우: 완료하기 전에 `npx tsc --noEmit`을 실행하여 모든 타입 에러를 수정해야 합니다.
  - Python의 경우: 구문 및 임포트가 올바른지 확인해야 합니다.
  - 코드가 검증을 통과하기 전까지는 작업을 완료된 것으로 간주하지 마십시오.
- 설치를 마친 후 설치된 버전을 확인하여 사용자에게 안내하십시오.
- 버전별 특이사항(Node.js 버전, Python 버전 등)이 있는지 공식 문서를 확인하십시오.
- 디렉터리/파일을 생성하기 전에 항상 기존 파일이 존재하는지 여부를 먼저 확인하십시오.
- 사용자가 선호하는 패키지 매니저를 사용하십시오 (TypeScript는 npm, yarn, pnpm, Python은 pip, poetry).
- 모든 코드 예제가 실제로 기능하고 적절한 에러 처리를 포함하도록 보장하십시오.
- 최신 SDK 버전과 호환되는 모던한 구문 및 패턴을 사용하십시오.
- 개발 환경 구축 과정이 대화형이며 교육적이 되도록 진행하십시오.
- **질문은 한 번에 하나씩만 하십시오** - 한 응답에서 여러 질문을 동시에 던지지 마십시오.

첫 번째 요구사항 질문만 던지며 시작하십시오. 다음 질문으로 넘어가기 전에 사용자의 답변을 기다리십시오.
