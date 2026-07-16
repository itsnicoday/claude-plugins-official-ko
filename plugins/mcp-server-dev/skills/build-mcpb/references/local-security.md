# 로컬 MCP 보안 (Local MCP Security)

**MCPB는 별도의 샌드박스를 제공하지 않습니다.** 매니페스트 설정상에 `permissions` 같은 제어 블록이 없고, 파일 시스템의 가상 격리 처리나 플랫폼에 의한 네트워크 허용 리스트 강제 기능도 없습니다. 서버 프로세스는 오직 구동 중인 사용자의 계정 권한 수준 그대로 실행됩니다 — 사용자가 읽을 수 있는 모든 파일을 읽을 수 있고, 임의의 프로세스를 새로 실행할 수 있으며, 임의의 네트워크 서버 주소로 통신할 수 있습니다.

사용자는 Claude를 통해 이 프로세스를 구동합니다. 이 결합 방식이 시사하는 바는 매우 중요합니다: **도구(tool)의 모든 매개변수 입력값은 신뢰할 수 없는 정보(untrusted)**로 취급해야 합니다. 사용자가 Claude 자체를 신뢰하더라도, Claude가 접속한 외부 웹페이지에 프롬프트 인젝션(prompt-injected) 코드가 들어가 있으면 Claude는 해커의 지시대로 사용자의 로컬 컴퓨터 내 `delete_file` 도구에 임의의 의도치 않은 시스템 경로값을 집어넣어 실행시켜 버릴 수 있습니다.

이를 막을 방어책은 오직 도구 핸들러 함수 내부의 엄격한 유효성 검사뿐입니다. 아래 가이드라인은 이 보안 방어선을 구축하는 모범 사례들을 다룹니다.

---

## 경로 순회 공격 (Path traversal)

로컬 MCP 서버에서 가장 흔히 발생하는 대표적인 취약점입니다. 사용자로부터 파일 경로 파라미터를 입력받아 특정 디렉토리 루트와 결합(join)할 때는, **반드시 실제 절대 경로로 완전히 변환한 뒤 디렉토리 포함 여부를 검증(containment check)**해야 합니다.

```typescript
import { resolve, relative, isAbsolute } from "node:path";

function safeJoin(root: string, userPath: string): string {
  const full = resolve(root, userPath);
  const rel = relative(root, full);
  if (rel.startsWith("..") || isAbsolute(rel)) {
    throw new Error(`Path escapes root: ${userPath}`);
  }
  return full;
}
```

`resolve` 함수는 경로 문자열 안의 `..` 이나 심볼릭 링크 세그먼트 등을 모두 실제 주소로 단순화해 줍니다. `relative` 함수는 결합 결과 주소가 기준 루트 디렉토리를 벗어났는지 판별해 줍니다. 단순히 경로 문자열 안에 `String.includes("..")` 규칙만 걸어 필터링하려 하지 마십시오 — 특수 인코딩 방식이나 심볼릭 링크를 이용한 우회 기법을 걸러내지 못합니다.

**Python 구현 예시:**

```python
from pathlib import Path

def safe_join(root: Path, user_path: str) -> Path:
    full = (root / user_path).resolve()
    if not full.is_relative_to(root.resolve()):
        raise ValueError(f"Path escapes root: ${user_path}")
    return full
```

---

## Roots (작업 공간 기준 디렉토리) — 하드코딩 대신 호스트에 질의

소스코드 상에 환경 변수 등에서 읽어온 `ROOT` 주소를 강제로 박아버리기 전에, 호스트 환경이 `roots/list` 프로토콜을 지원하는지 먼저 확인하십시오. 이것이 사용자가 공식적으로 연동을 허가한 작업 영역의 경계를 가져오는 표준 방법입니다.

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";

const server = new McpServer({ name: "...", version: "..." });

let allowedRoots: string[] = [];
server.server.oninitialized = async () => {
  const caps = server.getClientCapabilities();
  if (caps?.roots) {
    const { roots } = await server.server.listRoots();
    allowedRoots = roots.map(r => new URL(r.uri).pathname);
  } else {
    allowedRoots = [process.env.ROOT_DIR ?? process.cwd()];
  }
};
```

```python
# fastmcp — 도구 핸들러 함수 내부
async def my_tool(ctx: Context) -> str:
    try:
        roots = await ctx.list_roots()
        allowed = [urlparse(r.uri).path for r in roots]
    except Exception:
        allowed = [os.environ.get("ROOT_DIR", os.getcwd())]
```

호스트가 제공해 주는 roots 목록이 확보된다면 이를 연동 조건으로 삼으십시오. 조회 불가 시에만 디폴트 설정값으로 fallback 처리합니다. 어떤 방식을 취하든, 사용자로부터 전달받은 모든 상대 경로값은 검증된 허용 목록 영역 내에 속하는지 매번 유효성 확인을 거쳐야 합니다.

---

## 커맨드 인젝션 취약점 방어 (Command injection)

외부 하위 프로세스를 실행하는 로직인 경우, **절대로 사용자 입력 파라미터를 쉘(shell) 문자열 구문 형태로 직접 연결해 통째로 던지지 마십시오**.

```typescript
// ❌ 매우 치명적인 취약점을 낳는 잘못된 방식
exec(`git log ${branch}`);

// ✅ 쉘을 거치지 않고, 인자를 배열 포맷으로 분리해 전송하는 방식
execFile("git", ["log", branch]);
```

외부 CLI 래퍼 도구를 설계할 때는 파라미터 argv 값들을 순수 배열 원소들로만 구성해야 합니다. 도구 설계에서 추가 커맨드 라인 옵션 지정을 굳이 지원해야 하는 환경인 경우, 인입되는 모든 플래그 옵션값들을 사전에 허가된 플래그명 화이트리스트와 꼼꼼히 대조하십시오.

---

## 기본 동작 정책은 읽기 전용 (Read-only by default)

읽기(Read)와 쓰기(Write) 기능은 반드시 서로 별개의 개별 도구로 쪼개어 등록하십시오. 사용자의 대화 흐름 중 대다수에는 읽기 전용 권한만으로도 충분합니다. 도구 자체가 읽기 전용으로 설계되어 있으면, 설령 Claude가 외부 프롬프트 공격에 오염되더라도 로컬 파일 시스템 파괴나 악의적인 쓰기 행위로 무기화되는 사태를 차단할 수 있습니다.

```
list_files   ← 안심하고 기본 연동 허용
read_file    ← 안심하고 기본 연동 허용
write_file   ← 별개 도구로 분리, 실행에 대한 삼엄한 감시 및 주의
delete_file  ← 가급적 이 도구는 기본 탑재하지 않는 것을 강력 권장
```

이 설계를 도구 주석과 연동하십시오 — 모든 읽기 도구에는 `readOnlyHint: true`를 지정하고, 삭제/변경 도구에는 `destructiveHint: true` 속성을 지정합니다. 호스트들은 이를 토대로 자동 승인 처리하거나, 위험 도구 실행 전 경고 창 팝업을 띄우는 식으로 UI를 그립니다. `../build-mcp-server/references/tool-design.md` 참고.

쓰기/삭제 도구를 보급해야 하는 경우에는, 실행 전 Elicitation API(단순 양식용)나 확인 위젯 창(MCP app용)을 띄워 사용자가 최종 서명할 때까지 임시 대기하도록 설계하여 안전장치를 마련하십시오.

---

## 리소스 과점 제한 (Resource limits)

Claude는 사용자가 원하면 악의 없이도 기꺼이 4GB 크기의 시스템 로그 파일을 읽어오려고 할 것입니다. 모든 조회성 데이터에는 단단히 제약 크기를 거십시오:

```typescript
const MAX_BYTES = 1_000_000;
const buf = await readFile(path);
if (buf.length > MAX_BYTES) {
  return {
    content: [{
      type: "text",
      text: `File is ${buf.length} bytes — too large. Showing first ${MAX_BYTES}:\n\n`
            + buf.subarray(0, MAX_BYTES).toString("utf8"),
    }],
  };
}
```

디렉토리 내 파일 목록 출력(최대 출력 개수 제한), 검색 결과 매칭(최대 매칭 행 제한) 등 외부 자원 조회를 다루는 모든 입력 엔드포인트에 상한 제약을 설계하십시오.

---

## 비밀 정보 관리 (Secrets)

- **설정 비밀 값 관리** (매니페스트 `user_config`의 `sensitive: true` 옵션): 호스트가 운영체제 보안 키체인 영역에 안전하게 숨겨 두었다가 실행 시 환경 변수로만 넘겨 줍니다. 소스 코드 안에서 이를 로그로 찍거나 도구 반환 텍스트에 포함시키지 마십시오.
- **키 파일을 평문으로 디스크에 저장하지 마십시오.** 호스트가 대리 보관해 주는 키체인 방식 외에 추가 보관이 필요한 경우에는, Node 환경인 경우 `keytar`, Python 환경인 경우 `keyring` 패키지를 이용해 운영체제 보안 보관함을 직접 연동하십시오.
- **도구가 리턴하는 반환 데이터는 대화 내역에 박제됩니다.** 즉, 연산 결과로 무심코 리턴한 데이터는 사용자의 눈과 대화 아카이브 내역에 그대로 평문 노출됩니다. 리턴 전에 민감한 정보는 별도로 마스킹(redact) 처리하십시오.

---

## 배포 제출 전 체크리스트

- [ ] 사용자 입력 파일 경로를 다루는 모든 파라미터에 대해 경로 유효 검사를 거치도록 구현했는가
- [ ] 소스코드 내에 `exec()` 또는 `shell=True` 구문이 쓰이지 않았는가 — 오직 `execFile` 및 배열 형식 매개변수 전송만 사용했는가
- [ ] 쓰기/삭제 도구가 읽기 도구와 안전하게 별개로 쪼개졌으며, `readOnlyHint`/`destructiveHint` 주석이 제대로 설정되었는가
- [ ] 파일 읽기량, 폴더 목록 출력, 검색 결과 매칭에 대해 명확한 크기 제한 정책을 적용했는가
- [ ] 비밀 크리덴셜 정보가 로그나 도구 리턴값 안에 날것으로 노출되는 흔적이 전혀 없는가
- [ ] 악의적인 인입 시나리오(`../../etc/passwd` 경로 유입, `; rm -rf ~` 쉘 인젝션 구문 유입, 10GB 용량의 가짜 파일 읽기)를 직접 모방하여 로컬 환경에서 취약점 검증 테스트를 완료했는가
