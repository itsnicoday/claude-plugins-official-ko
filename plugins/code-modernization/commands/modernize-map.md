---
description: Dependency & topology mapping — call graphs, data lineage, batch flows, rendered as navigable diagrams
argument-hint: <system-dir>
---

`legacy/$1`의 **의존성 및 토폴로지 맵(dependency and topology map)**을 구축하고 이를 시각적으로 렌더링합니다.

평가(assessment) 단계에서 도메인을 파악했습니다. 이제 한 단계 더 깊이 들어가서 각 *컴포넌트*들이 어떻게 연결되어 있는지 파악할 차례입니다. 이는 엔지니어가 코드를 수정하기 전에 반드시 확인해야 하는 지도입니다.

## 결과물 목록

`legacy/$1` 아래의 소스 코드를 구문 분석하고 아래의 4가지 데이터 세트를 추출하는 단발성 분석 스크립트(Python 또는 Shell 중 선택)를 작성하십시오. 토폴로지 구축 시 다음의 세 가지 원칙이 적용됩니다. 이를 잘못 처리하면 오해의 소지가 있는 맵이 생성됩니다:

1. **에지는 두 곳에 존재합니다** — 소스 코드에 명시된 직접 호출(direct calls), *그리고* 대상이 변수인 디스패처/라우터 호출(설정 테이블, 라우팅 맵, 의존성 주입, 동적 디스패치). 에지를 분석 불가능으로 선언하기 전에 설정 파일과 변수 값을 대조하여 분석하십시오.
2. **코드↔스토리지 연결은 주로 외부 설정 파일에 정의됩니다** — 소스 코드가 아니라 작업/배포 기술자(descriptors)가 논리적 이름을 물리적 저장소에 매핑합니다.
3. **진입점(Entry points)은 주로 배포 설정에 정의됩니다** — 소스 코드가 아닙니다. 이를 파싱하지 않으면 모든 최상위 모듈이 도달 불가능한 것처럼 보입니다.

다음 항목들을 추출하십시오:

- **프로그램/모듈 호출 그래프 (Program/module call graph)** — 직접 호출 (`CALL`, 메서드 호출, `import`/`require`) *및* 디스패처 호출 (`EXEC CICS LINK/XCTL`, DI 컨테이너 구성, 프레임워크 라우팅, 리플렉션/팩토리). 라우팅 테이블, 카피북(copybooks), 설정 파일 또는 상수 풀(constant pools)과 대조하여 동적 호출 대상을 해석하십시오.
- **데이터 의존성 그래프 (Data dependency graph)** — 어떤 모듈이 어떤 데이터 저장소를 읽고 쓰는지 관련 설정을 통해 연결합니다: `SELECT…ASSIGN TO` ↔ JCL `DD` (배치 COBOL), `EXEC CICS READ/WRITE…FILE()` ↔ CSD `DEFINE FILE` (CICS 온라인), `EXEC SQL` 테이블 참조 (임베디드 SQL), ORM 어노테이션/매핑 (Java/.NET), 모델 파일 (Node/Python/Ruby). UI/화면 바인딩 (BMS 맵, JSP, 템플릿)도 의존성에 포함되므로 함께 추출하십시오.
- **진입점 (Entry points)** — 스택의 가장 바깥쪽 호출자가 정의된 위치에서 읽어옵니다: JCL `EXEC PGM=` 및 CICS CSD `DEFINE TRANSACTION` (메인프레임), `web.xml`/라우팅 어노테이션/라우팅 파일 (웹), `main()`/argv 파싱 (CLI), 큐/스케줄러 구독 (이벤트 기반).
- **데드엔드 후보 (Dead-end candidates)** — 인바운드 에지(inbound edges)가 없는 모듈. **위의 모든 진입점 및 호출 에지 유형이 그래프에 반영된 후에만 의미가 있습니다.** 해석되지 않은 동적 호출의 대상이 될 수 있는 모듈에 대해서는 데드엔드 판단을 보류(suppress)하십시오. grep에만 의존한 그래프는 디스패처 기반 모듈(CICS 프로그램, Spring 컨트롤러, ORM 바인딩 DAO 등)이 실제로는 작동 중임에도 데드엔드로 잘못 표시할 가능성이 높습니다.

소스 코드가 고정 컬럼 형식(COBOL 8~72열, RPG 등)인 경우, 정규식 매칭을 수행하기 전에 코드 영역을 슬라이스하고 주석 라인을 제거하십시오. 그렇지 않으면 일련번호나 주석 처리된 코드가 매칭될 수 있습니다.

스크립트를 `analysis/$1/extract_topology.py` (또는 `.sh`)로 저장하여 나중에 다시 실행하고 감사할 수 있도록 하십시오. 스크립트가 기계가 읽을 수 있는 `analysis/$1/topology.json`을 작성하고 사람이 읽을 수 있는 요약 정보를 출력하도록 만드십시오. 스크립트를 실행한 후 요약 정보를 보여주십시오 (규모가 매우 큰 경우에는 약 200라인으로 제한).

`topology.json`은 대화형 뷰어에 제공되므로 반드시 다음 스키마를 따라야 합니다:

```json
{
  "system": "<display name>",
  "root": {
    "id": "sys", "name": "<system>", "kind": "system",
    "children": [
      { "id": "dom:<domain>", "name": "<Domain>", "kind": "domain",
        "children": [
          { "id": "<MODULE>", "name": "<MODULE>", "kind": "module",
            "language": "cobol", "loc": 1234, "file": "src/MODULE.cbl" }
        ] },
      { "id": "dom:data", "name": "Data stores", "kind": "domain",
        "children": [
          { "id": "ds:<NAME>", "name": "<NAME>", "kind": "datastore" }
        ] }
    ]
  },
  "edges": [
    { "source": "<id>", "target": "<id>", "kind": "call" }
  ],
  "entryPoints": ["<id>", "..."],
  "deadEnds": ["<id>", "..."],
  "observations": ["<architect observation>", "..."],
  "flows": [
    { "name": "<business flow>", "persona": "<who experiences it>",
      "description": "<one sentence, plain language>",
      "steps": [
        { "label": "<business-language step>", "nodes": ["<id>", "<id>"] }
      ] }
  ]
}
```

- 리프(leaf) 모듈들을 `domain` 컨테이너 아래로 그룹화합니다 (사용 가능한 경우 `/modernize-assess`에서 도출된 도메인을 사용하십시오). 리프 종류: `module`, `datastore`, `job`, `screen`. `loc`는 시각화 시 원의 크기를 결정하므로 모듈에 반드시 포함시키십시오.
- 에지 종류: `call` (직접), `dispatch` (동적/라우터), `read`, `write`. 모든 에지의 엔드포인트는 트리 구조 내에 존재하는 리프 id여야 합니다.
- `deadEnds`: 추출 과정에서 도출된 데드엔드 후보들로, 뷰어에서는 점선 테두리로 표시됩니다. 위의 보류(suppression) 규칙을 적용하십시오. 해결되지 않은 동적 호출의 대상이 될 수 있는 모듈은 여기에 포함시키지 말고, 대신 그 불확실성을 `observations`에 기록하십시오.
- **데이터 저장소 id와 이름은 반드시 논리적 식별자여야 합니다** — DD 이름, 데이터 세트 이름, 테이블/스키마 이름 또는 최대 host:port 형식. 해석된 설정 값이 URL 또는 DSN인 경우, topology.json에 기록하기 전에 사용자 정보 및 자격 증명 쿼리 파라미터를 제거하십시오. 이 파일은 커밋되고 뷰어에 이름이 그대로 표시되기 때문입니다. 원시(raw) 설정 값을 `observations`에 그대로 복사하지 마십시오.
- `observations`: 3~7개의 아키텍트 관점의 관찰 내용 — 긴밀하게 결합된 클러스터, 단일 장애점(single points of failure), 서비스 추출 후보, 너무 많은 모듈이 접근하는 데이터 저장소, 추출기에서 해결하지 못한 동적 디스패치 대상 등.
- `flows`: 사용자 시나리오 흐름 섹션입니다. 아래 내용을 참고하십시오.

## 사용자 시나리오 흐름 (Persona flows)

시스템을 유지관리하는 사람이 아니라 시스템을 실제로 사용하는 사람(persona) 중심의 **엔드투엔드 비즈니스 흐름 2~4개**를 추적합니다 (예: 복지 시스템의 경우: 신청자, 사회복지사, 감사관 / 빌링 시스템의 경우: 고객, 빌링 담당자). 각 흐름에 대해 다음을 기술하십시오:

- `name` + 쉬운 비즈니스 용어로 작성된 한 문장짜리 `description` — 데이터 흐름 레이블("CLM batch ingest")이 아니라 의사 결정권자가 이해할 수 있는 비즈니스 언어("신청자가 주간 지원금을 신청함")로 작성되어야 합니다.
- `steps`: 3~8개 단계로 구성되며, 각 단계는 비즈니스 용어 `label`과 해당 단계를 구현하는 실행 순서대로 나열된 `nodes` (프로그램 + 데이터 저장소) 목록을 가집니다.

이는 기술 지도와 비기술적 이해관계자를 잇는 다리 역할을 합니다. 동일한 다이어그램이 엔지니어에게는 "어떤 프로그램이 X를 처리하는가"에 대한 답을 주고, 다른 모든 이들에게는 "누군가 지원금을 신청하면 어떤 일이 일어나는가"에 대한 답을 제공합니다.

## 시각화 렌더링 (Render)

`analysis/$1/TOPOLOGY.html`은 **대화형 맵**입니다. 전체 시스템의 줌 인/아웃이 가능한 서클 팩(containers로서의 도메인, LOC 크기별 모듈)을 렌더링하며, 의존성 에지, 검색, 노드별 상세 사이드바, 에지 종류 토글, 그리고 각 시나리오 흐름을 번호가 매겨진 경로로 보여주는 시나리오 단계별 실행 모드를 제공합니다. 이 뷰어는 직접 작성하지 말고, 플러그인에 내장된 템플릿을 사용하여 빌드하십시오:

```bash
python3 - "${CLAUDE_PLUGIN_ROOT}/assets/topology-viewer.html" analysis/$1 <<'EOF'
import json, sys
tpl_path, out_dir = sys.argv[1], sys.argv[2]
tpl = open(tpl_path).read()
marker = "/*__TOPOLOGY_DATA__*/ null"
assert marker in tpl, f"injection marker not found in {tpl_path}"
data = json.dumps(json.load(open(f"{out_dir}/topology.json")))
# topology.json은 신뢰할 수 없는(UNTRUSTED) 소스에서 파생된 데이터입니다. (노드 이름은 파일명에서 오고,
# observations/flows는 분석된 코드에서 유도됨). 이 데이터는 <script> 블록에 직접 주입되는데, HTML 파서가 JS 문자열 컨텍스트와
# 상관없이 "</script>" 바이트를 만나면 즉시 <script> 블록을 닫아버립니다. 따라서 노드 이름이 "x</script><script>…"처럼 되어 있으면
# 원치 않는 스크립트가 실행될 수 있습니다. json.dumps는 기본적으로 "<"를 이스케이프하지 않으므로, 이러한 브레이크아웃 공격을 차단하기 위해
# 이스케이프(JSON 안전 형식)를 적용합니다.
data = data.replace("<", "\\u003c").replace(">", "\\u003e").replace("&", "\\u0026")
open(f"{out_dir}/TOPOLOGY.html", "w").write(
    tpl.replace(marker, "/*__TOPOLOGY_DATA__*/ " + data))
print(f"wrote {out_dir}/TOPOLOGY.html")
EOF
```

뷰어는 필요한 d3 라이브러리가 템플릿 내에 인라인화되어 완전히 독립적으로 작동하므로, 오프라인 및 외부망이 차단된 폐쇄망에서도 잘 작동합니다. 만약 `python3` 실행 시 템플릿을 찾지 못하면 `${CLAUDE_PLUGIN_ROOT}` 치환이 되지 않은 것이므로, 직접 뷰어를 작성하지 말고 이 오류를 보고하십시오.

Mermaid는 **작고 내보내기 쉬운** 다이어그램을 작성할 때만 사용합니다. 문서화 및 PR에서 재사용할 수 있도록 독립형 `.mmd` 파일을 생성하되, 각 파일의 에지 수는 40개 이하로 유지하십시오. 전체 그래프가 이보다 크다면 도메인 수준으로 축소하여 표현하십시오 (복잡한 Mermaid 그래프는 가독성이 떨어지며, 대화형 맵이 존재하는 이유가 바로 그 때문입니다):

- `analysis/$1/call-graph.mmd` — 진입점이 하이라이트된 도메인 수준의 `graph TD`
- `analysis/$1/data-lineage.mmd` — 프로그램 → 데이터 저장소 및 읽기/쓰기가 표시된 `graph LR`
- `analysis/$1/critical-path.mmd` — `flows`의 주 흐름을 시각화한 `flowchart TD` (사용 가능한 텔레메트리 지표가 있는 경우 p50/p99 소요 시간 기록, `/modernize-assess` 4단계 참고)

## 결과 제시 (Present)

사용자에게 브라우저에서 `analysis/$1/TOPOLOGY.html` 파일을 열고, 모듈을 검색하고, 클릭하여 연결 관계를 확인하며, 드롭다운에서 비즈니스 시나리오 흐름을 선택하여 플레이해 보도록 안내하십시오.
