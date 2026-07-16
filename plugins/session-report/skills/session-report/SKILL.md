---
name: session-report
description: ~/.claude/projects 트랜스크립트로부터 Claude Code 세션 사용량(토큰, 캐시, 서브에이전트, 스킬, 비용이 큰 프롬프트)에 대한 탐색 가능한 HTML 보고서를 생성합니다.
---

# 세션 보고서 (Session Report)

Claude Code 사용량에 대한 자체 완결형 HTML 보고서를 생성하고 현재 작업 디렉토리에 저장합니다.

## 단계 (Steps)

1. **데이터 가져오기.** 번들된 분석기(기본 범위: 최근 7일. 사용자가 `24h`, `30d`, `all` 등 다른 범위를 전달한 경우 해당 범위를 반영)를 실행합니다. `analyze-sessions.mjs` 스크립트는 이 SKILL.md와 같은 디렉토리에 존재합니다. 절대 경로를 사용하십시오:
   ```sh
   node <skill-dir>/analyze-sessions.mjs --json --since 7d > /tmp/session-report.json
   ```
   전체 기간 데이터를 보려면 `--since` 옵션을 생략하십시오.

2. `/tmp/session-report.json` 파일을 **읽습니다.** `overall`, `by_project`, `by_subagent_type`, `by_skill`, `cache_breaks`, `top_prompts` 항목들을 대략적으로 확인합니다.

3. 이 SKILL.md와 함께 번들 제공되는 **템플릿을 복사**하여 현재 작업 디렉토리의 출력 경로에 붙여넣습니다:
   ```sh
   cp <skill-dir>/template.html ./session-report-$(date +%Y%m%d-%H%M).html
   ```

4. **출력 파일을 편집합니다** (Write가 아닌 Edit 도구를 사용하여 템플릿의 JS/CSS를 그대로 유지하십시오):
   - `<script id="report-data" type="application/json">` 의 내용을 1단계에서 얻은 전체 JSON 데이터로 교체합니다. 페이지 내의 JS가 이 데이터 블록으로부터 종합 토탈(hero total), 모든 테이블, 막대 차트 및 상세 분석(drill-downs)을 자동으로 렌더링합니다.
   - `<!-- AGENT: anomalies -->` 블록을 **3~5개의 한 줄짜리 주요 발견 사항**으로 채웁니다. 수치는 가능하면 **전체 토큰 대비 백분율(%)**로 표현하십시오 (전체 = `overall.input_tokens.total + overall.output_tokens`). 발견 사항당 한 줄씩 작성하되, 마크업 구조를 정확히 지키십시오:
     ```html
     <div class="take bad"><div class="fig">41.2%</div><div class="txt"><b>cc-monitor</b> consumed 41% of the week across just 3 sessions</div></div>
     ```
     클래스 구분: 자원 낭비/이상 징후는 `.take bad` (빨간색), 정상 신호는 `.take good` (초록색), 중립적인 정보는 `.take info` (파란색)를 사용하십시오. `.fig` 영역에는 하나의 짧은 숫자(백분율 %, 개수, 또는 `12×` 같은 배수)를 넣습니다. `.txt` 영역에는 프로젝트/스킬/프롬프트명을 포함한 한 문장의 설명을 작성하고, 주요 주체는 `<b>` 태그로 감싸십시오. 다음 특징들을 포착하십시오: 과도한 토큰 비율을 점유하는 특정 프로젝트나 스킬, 캐시 히트율(cache-hit) 85% 미만, 전체 사용량의 2%를 초과하는 단일 프롬프트, 호출당 평균 1M 토큰 이상을 소모하는 서브에이전트 유형, 특정 시점에 몰려 있는 캐시 브레이크 등.
   - 페이지 **하단**의 `<!-- AGENT: optimizations -->` 블록을 특정 항목과 연계된 1~4개의 `<div class="callout">` 최적화 제안으로 채웁니다 (예: "`/weekly-status` spawned 7 subagents for 8.1% of total — scope it to fewer parallel agents").
   - 기존 섹션들의 구조를 변경하지 마십시오.

5. 저장된 파일 경로를 사용자에게 **보고합니다.** 파일을 직접 열거나 렌더링하지 마십시오.

## 유의 사항 (Notes)

- 템플릿 파일이 대화형 동작(정렬, 펼치기/접기, 블록 문자 바 차트 등)의 원천입니다. 귀하의 역할은 마크업 빌드가 아닌 데이터 및 분석을 제공하는 것입니다.
- 주석 및 설명은 간결하고 구체적이어야 합니다 — JSON 파일에 명시된 실제 프로젝트명, 수치, 타임스탬프를 언급하십시오.
- `top_prompts` 에는 이미 서브에이전트 소모 토큰이 산정되어 있으며, 태스크 알림 후속 연산들도 원래의 프롬프트 사용량으로 환산되어 포함되어 있습니다.
- 만약 JSON 파일 크기가 2MB를 초과하면, 데이터를 삽입하기 전에 `top_prompts` 목록과 `cache_breaks` 목록을 각각 상위 100개씩만 남기도록 다듬으십시오 (기본적으로 한도가 이미 걸려 있을 것입니다).
