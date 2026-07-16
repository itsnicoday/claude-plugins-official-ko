# 플러그인 매니페스트 참조 (Plugin Manifest Reference)

`plugin.json` 구성을 위한 전체 참조 가이드.

## 파일 위치 (File Location)

**필수 경로**: `.claude-plugin/plugin.json`

매니페스트 파일은 반드시 플러그인 루트의 `.claude-plugin/` 디렉토리에 위치해야 합니다. Claude Code는 이 파일이 올바른 위치에 존재하지 않으면 플러그인을 인식하지 못합니다.

## 전체 필드 참조 (Complete Field Reference)

### 핵심 필드 (Core Fields)

#### name (필수)

**타입**: String
**형식**: kebab-case
**예시**: `"test-automation-suite"`

플러그인의 고유 식별자입니다. 다음 용도로 사용됩니다:
- Claude Code에서 플러그인 식별
- 다른 플러그인과의 이름 충돌 감지
- 명령어 네임스페이스 지정 (선택 사항)

**요구 사항**:
- 설치된 모든 플러그인 중에서 유일해야 함
- 영문 소문자, 숫자, 하이픈만 사용 가능
- 공백이나 특수 문자 허용 안 됨
- 문자로 시작해야 함
- 문자나 숫자로 끝나야 함

**정규식 검증**:
```javascript
/^[a-z][a-z0-9]*(-[a-z0-9]+)*$/
```

**예시**:
- ✅ 올바름: `api-tester`, `code-review`, `git-workflow-automation`
- ❌ 잘못됨: `API Tester`, `code_review`, `-git-workflow`, `test-`

#### version

**타입**: String
**형식**: 시맨틱 버저닝 (MAJOR.MINOR.PATCH)
**예시**: `"2.1.0"`
**기본값**: 지정하지 않은 경우 `"0.1.0"`

시맨틱 버저닝 지침:
- **MAJOR**: 호환되지 않는 API 변경, 주요 변경 사항
- **MINOR**: 하위 호환되는 새로운 기능 추가
- **PATCH**: 하위 호환되는 버그 수정

**프리릴리스 버전**:
- `"1.0.0-alpha.1"` - 알파 릴리스
- `"1.0.0-beta.2"` - 베타 릴리스
- `"1.0.0-rc.1"` - 릴리스 후보(RC)

**예시**:
- `"0.1.0"` - 초기 개발 단계
- `"1.0.0"` - 첫 번째 안정 버젼 릴리스
- `"1.2.3"` - 1.2 버전의 패치 업데이트
- `"2.0.0"` - 하위 호환성이 깨지는 변경 사항이 포함된 주 버전 업데이트

#### description

**타입**: String
**길이**: 50~200자 권장
**예시**: `"Automates code review workflows with style checks and automated feedback"`

플러그인의 목적과 기능에 대한 간단한 설명입니다.

**권장 사항**:
- 플러그인이 '어떻게'가 아닌 '무엇을' 하는지에 집중
- 능동태 사용
- 주요 기능이나 이점 언급
- 마켓플레이스 화면에 맞추기 위해 200자 미만 유지

**예시**:
- ✅ "Generates comprehensive test suites from code analysis and coverage reports"
- ✅ "Integrates with Jira for automatic issue tracking and sprint management"
- ❌ "A plugin that helps you do testing stuff"
- ❌ "This is a very long description that goes on and on about every single feature..."

### 메타데이터 필드 (Metadata Fields)

#### author

**타입**: Object
**하위 필드**: name (필수), email (선택 사항), url (선택 사항)

```json
{
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com",
    "url": "https://janedeveloper.com"
  }
}
```

**대체 형식** (문자열 전용):
```json
{
  "author": "Jane Developer <jane@example.com> (https://janedeveloper.com)"
}
```

**사용 사례**:
- 크레딧 및 기여 표시
- 지원 또는 문의용 연락처
- 마켓플레이스 화면 표시
- 커뮤니티 인정

#### homepage

**타입**: String (URL)
**예시**: `"https://docs.example.com/plugins/my-plugin"`

플러그인 문서 또는 랜딩 페이지 링크입니다.

**연결 대상 권장**:
- 플러그인 문서 사이트
- 프로젝트 홈페이지
- 세부 사용 설명서
- 설치 안내서

**연결 대상 제외**:
- 소스 코드 (대신 `repository` 필드 사용)
- 이슈 트래커 (문서 내에 포함)
- 개인 웹사이트 (대신 `author.url` 사용)

#### repository

**타입**: String (URL) 또는 Object
**예시**: `"https://github.com/user/plugin-name"`

소스 코드 저장소 위치입니다.

**문자열 형식**:
```json
{
  "repository": "https://github.com/user/plugin-name"
}
```

**객체 형식** (세부 정보 포함):
```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/user/plugin-name.git",
    "directory": "packages/plugin-name"
  }
}
```

**사용 사례**:
- 소스 코드 접근
- 이슈 보고
- 커뮤니티 기여
- 투명성 및 신뢰 확보

#### license

**타입**: String
**형식**: SPDX 식별자
**예시**: `"MIT"`

소프트웨어 라이선스 식별자입니다.

**자주 사용되는 라이선스**:
- `"MIT"` - 허용 범위가 넓어 널리 쓰이는 라이선스
- `"Apache-2.0"` - 특허권 부여가 포함된 라이선스
- `"GPL-3.0"` - 카피레프트 라이선스
- `"BSD-3-Clause"` - 허용 범위가 넓은 라이선스
- `"ISC"` - MIT와 유사한 허용 라이선스
- `"UNLICENSED"` - 비공개 독점 소프트웨어

**전체 목록**: https://spdx.org/licenses/

**듀얼 라이선스 구성**:
```json
{
  "license": "(MIT OR Apache-2.0)"
}
```

#### keywords

**타입**: 문자열 배열 (Array of strings)
**예시**: `["testing", "automation", "ci-cd", "quality-assurance"]`

플러그인 검색 및 카테고리 분류를 위한 태그 목록입니다.

**권장 사항**:
- 5~10개의 키워드 지정
- 기능별 카테고리 포함
- 사용 기술 이름 추가
- 일반적인 검색 용어 사용
- 플러그인 이름과 동일한 단어는 생략

**고려할 카테고리**:
- 기능: `testing`, `debugging`, `documentation`, `deployment`
- 기술: `typescript`, `python`, `docker`, `aws`
- 워크플로우: `ci-cd`, `code-review`, `git-workflow`
- 도메인: `web-development`, `data-science`, `devops`

### 컴포넌트 경로 필드 (Component Path Fields)

#### commands

**타입**: String 또는 문자열 배열
**기본값**: `["./commands"]`
**예시**: `"./cli-commands"`

명령어 정의가 포함된 추가 디렉토리 또는 파일입니다.

**단일 경로**:
```json
{
  "commands": "./custom-commands"
}
```

**다중 경로**:
```json
{
  "commands": [
    "./commands",
    "./admin-commands",
    "./experimental-commands"
  ]
}
```

**동작 방식**: 기본 `commands/` 디렉토리에 추가되며, 이를 대체하지 않습니다.

**사용 사례**:
- 카테고리별 명령어 정리
- 안정 버전과 실험용 명령어 분리
- 공유 위치에서 명령어 로드

#### agents

**타입**: String 또는 문자열 배열
**기본값**: `["./agents"]`
**예시**: `"./specialized-agents"`

에이전트 정의가 포함된 추가 디렉토리 또는 파일입니다.

**형식**: `commands` 필드와 동일

**사용 사례**:
- 전문 영역별 에이전트 그룹화
- 범용 에이전트와 작업 전용 에이전트 분리
- 플러그인 종속성에서 에이전트 로드

#### hooks

**타입**: String (JSON 파일 경로) 또는 Object (인라인 구성)
**기본값**: `"./hooks/hooks.json"`

훅 구성 파일의 위치 또는 인라인 정의입니다.

**파일 경로**:
```json
{
  "hooks": "./config/hooks.json"
}
```

**인라인 구성**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**사용 사례**:
- 단순한 플러그인: 인라인 구성 방식 사용 (< 50행)
- 복잡한 플러그인: 외부 JSON 파일 사용
- 다중 훅 집합: 컨텍스트별로 별도의 파일 구성

#### mcpServers

**타입**: String (JSON 파일 경로) 또는 Object (인라인 구성)
**기본값**: `./.mcp.json`

MCP 서버 구성 파일의 위치 또는 인라인 정의입니다.

**파일 경로**:
```json
{
  "mcpServers": "./.mcp.json"
}
```

**인라인 구성**:
```json
{
  "mcpServers": {
    "github": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/github-mcp.js"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**사용 사례**:
- 단순한 플러그인: 단일 인라인 서버 구성 (< 20행)
- 복잡한 플러그인: 외부 `.mcp.json` 파일 사용
- 다중 서버: 항상 외부 파일 사용 권장

## 경로 해석 (Path Resolution)

### 상대 경로 규칙 (Relative Path Rules)

컴포넌트 필드에 작성되는 모든 경로는 다음 규칙을 따라야 합니다:

1. **반드시 상대 경로**: 절대 경로 불가능
2. **반드시 `./`로 시작**: 플러그인 루트에 대한 상대 경로임을 표시
3. **`../` 사용 불가능**: 상위 디렉토리 탐색 불가
4. **슬래시(`/`)만 사용**: Windows 환경에서도 슬래시 사용

**예시**:
- ✅ 올바름: `"./commands"`
- ✅ 올바름: `"./src/commands"`
- ✅ 올바름: `"./configs/hooks.json"`
- ❌ 잘못됨: `"/Users/name/plugin/commands"`
- ❌ 잘못됨: `"commands"` (`./`가 없음)
- ❌ 잘못됨: `"../shared/commands"`
- ❌ 잘못됨: `".\\commands"` (백슬래시 사용)

### 해석 순서 (Resolution Order)

Claude Code가 컴포넌트를 로드할 때:

1. **기본 디렉토리**: 표준 위치를 먼저 스캔합니다.
   - `./commands/`
   - `./agents/`
   - `./skills/`
   - `./hooks/hooks.json`
   - `./.mcp.json`

2. **맞춤 경로**: 매니페스트에 지정된 경로를 스캔합니다.
   - `commands` 필드에 지정된 경로
   - `agents` field에 지정된 경로
   - `hooks` 및 `mcpServers` 필드에 지정된 파일

3. **병합 동작**: 모든 위치에서 감지된 컴포넌트들을 로드합니다.
   - 덮어쓰지 않음
   - 발견된 모든 컴포넌트 등록
   - 이름 충돌 시 에러 발생

## 검증 (Validation)

### 매니페스트 검증 (Manifest Validation)

Claude Code는 플러그인 로드 시 매니페스트를 검증합니다:

**구문 검증**:
- 유효한 JSON 형식
- 구문 오류 없음
- 필드별 올바른 데이터 타입 사용

**필드 검증**:
- `name` 필드가 존재하며 형식이 올바름
- `version`이 시맨틱 버저닝 규격을 따름 (존재할 경우)
- 경로는 `./` 접두사를 포함하는 상대 경로 형식을 갖춤
- URL 형식이 유효함 (존재할 경우)

**컴포넌트 검증**:
- 참조된 경로가 실제로 존재함
- 훅 및 MCP 구성이 유효함
- 순환 종속성이 없음

### 일반적인 검증 오류 (Common Validation Errors)

**잘못된 이름 형식**:
```json
{
  "name": "My Plugin"  // ❌ 공백 포함
}
```
해결 방법: kebab-case 사용
```json
{
  "name": "my-plugin"  // ✅
}
```

**절대 경로 사용**:
```json
{
  "commands": "/Users/name/commands"  // ❌ 절대 경로 사용
}
```
해결 방법: 상대 경로 사용
```json
{
  "commands": "./commands"  // ✅
}
```

**./ 접두사 누락**:
```json
{
  "hooks": "hooks/hooks.json"  // ❌ ./ 접두사 누락
}
```
해결 방법: `./` 접두사 추가
```json
{
  "hooks": "./hooks/hooks.json"  // ✅
}
```

**잘못된 버전 표기**:
```json
{
  "version": "1.0"  // ❌ 시맨틱 버저닝이 아님
}
```
해결 방법: MAJOR.MINOR.PATCH 형식 사용
```json
{
  "version": "1.0.0"  // ✅
}
```

## 최소 구성 vs. 전체 구성 예시 (Minimal vs. Complete Examples)

### 최소 구성 플러그인 (Minimal Plugin)

작동 가능한 최소한의 플러그인 매니페스트:

```json
{
  "name": "hello-world"
}
```

기본 디렉토리 검색 방식에 완전히 의존합니다.

### 권장 플러그인 (Recommended Plugin)

배포 및 공유를 위해 권장되는 메타데이터 세트:

```json
{
  "name": "code-review-assistant",
  "version": "1.0.0",
  "description": "Automates code review with style checks and suggestions",
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com"
  },
  "homepage": "https://docs.example.com/code-review",
  "repository": "https://github.com/janedev/code-review-assistant",
  "license": "MIT",
  "keywords": ["code-review", "automation", "quality", "ci-cd"]
}
```

### 전체 구성 플러그인 (Complete Plugin)

모든 기능을 포함하는 완전한 구성:

```json
{
  "name": "enterprise-devops",
  "version": "2.3.1",
  "description": "Comprehensive DevOps automation for enterprise CI/CD pipelines",
  "author": {
    "name": "DevOps Team",
    "email": "devops@company.com",
    "url": "https://company.com/devops"
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
    "automation",
    "kubernetes",
    "docker",
    "deployment"
  ],
  "commands": [
    "./commands",
    "./admin-commands"
  ],
  "agents": "./specialized-agents",
  "hooks": "./config/hooks.json",
  "mcpServers": "./.mcp.json"
}
```

## 권장 사항 (Best Practices)

### 메타데이터 (Metadata)

1. **항상 버전을 포함시킬 것**: 변경 및 업데이트 기록 추적
2. **명확한 설명을 작성할 것**: 사용자가 플러그인의 목적을 이해하도록 도움
3. **연락처 정보를 제공할 것**: 사용자 지원 활성화
4. **문서 링크 제공**: 지원 부담 감소
5. **적절한 라이선스 선택**: 프로젝트 목표와 일치시킴

### 경로 (Paths)

1. **가능하면 기본 경로 사용**: 불필요한 설정 최소화
2. **논리적인 정리**: 관련된 컴포넌트끼리 그룹화
3. **맞춤 경로 문서화**: 비표준 레이아웃을 사용한 이유 기술
4. **경로 해석 테스트**: 다양한 운영체제에서 경로 해석을 검증

### 유지보수 (Maintenance)

1. **변경 시 버전 업데이트**: 시맨틱 버저닝 규칙 준수
2. **키워드 최신화**: 새로운 기능을 반영하도록 태그 수정
3. **설명 내용 최신화**: 실제 플러그인의 기능 수준과 매칭
4. **변경 이력 관리**: 버전별 히스토리 기록
5. **저장소 링크 최신화**: 저장소 URL을 현재 상태로 유지

### 배포 (Distribution)

1. **배포 전 메타데이터 작성 완료**: 모든 필수/권장 필드를 채움
2. **클린 설치 환경에서 테스트**: 개발 환경이 아닌 곳에서도 플러그인이 정상 작동하는지 검증
3. **매니페스트 검증**: 검증 도구 활용
4. **README 포함**: 설치 및 사용법 기술
5. **라이선스 파일 명시**: 플러그인 루트에 LICENSE 파일 포함
