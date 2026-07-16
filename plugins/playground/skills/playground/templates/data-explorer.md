# 데이터 탐색기 템플릿 (Data Explorer Template)

SQL 빌더, API 디자이너, 정규표현식(regex) 빌더, 파이프라인 시각화, 크론(cron) 스케줄러 등 데이터 쿼리, API, 파이프라인 또는 구조화된 설정과 관련된 플레이그라운드를 빌드할 때 이 템플릿을 사용하십시오.

## 레이아웃

```
+-------------------+----------------------+
|                   |                      |
|  컨트롤 영역       |  포맷된 출력         |
|  그룹화 기준:     |  (구문 강조된 코드   |
|  • 소스/테이블    |   또는 시각적        |
|  • 컬럼/필드      |   다이어그램)        |
|  • 필터           |                      |
|  • 그룹화         |                      |
|  • 정렬           |                      |
|  • 제한           |                      |
|                   +----------------------+
|                   |  프롬프트 출력       |
|                   |  [ 프롬프트 복사 ]   |
| +-----------------+----------------------+
```

## 결정 사항에 따른 컨트롤 유형

| 결정 사항 | 컨트롤 유형 | 예시 |
|---|---|---|
| 사용 가능한 항목 중 선택 | 클릭 가능한 카드/칩 | 테이블명, 컬럼, HTTP 메서드 |
| 필터/조건 행 추가 | 추가 버튼 → 드롭다운 + 입력란 행 | WHERE 컬럼 연산자 값 |
| 조인 유형 또는 집계 | 행당 드롭다운 | INNER/LEFT/RIGHT, COUNT/SUM/AVG |
| 제한(Limit)/오프셋(Offset) | 슬라이더 | 결과 개수 1–500 |
| 정렬 | 드롭다운 + ASC/DESC 토글 | 컬럼 기준 정렬 |
| 기능 On/Off | 토글 | 설명 표시 여부, 헤더 포함 여부 |

## 미리보기 렌더링

색상 클래스가 적용된 `<span>` 태그를 사용하여 구문 강조(syntax-highlight)된 출력을 렌더링합니다:

```javascript
function renderPreview() {
  const el = document.getElementById('preview');
  // Color-code by token type
  el.innerHTML = sql
    .replace(/\b(SELECT|FROM|WHERE|JOIN|ON|GROUP BY|ORDER BY|LIMIT)\b/g, '<span class="kw">$1</span>')
    .replace(/\b(users|orders|products)\b/g, '<span class="tbl">$1</span>')
    .replace(/'[^']*'/g, '<span class="str">$&</span>');
}
```

파이프라인 스타일의 플레이그라운드인 경우, 화살표 연결자가 있는 배치된 div들을 사용하여 가로 또는 세로 흐름 다이어그램을 렌더링하십시오.

## 데이터 프롬프트 출력

생 쿼리 자체를 그대로 출력하기보다는, 빌드해야 할 명세서(specification) 형태로 구성하십시오:

> "Write a SQL query that joins orders to users on user_id, filters for orders after 2024-01-01 with total > $50, groups by user, and returns the top 10 users by order count."

프롬프트만으로 자립할 수 있도록 스키마 컨텍스트(테이블명, 컬럼 타입 등)를 포함하십시오.

## 예시 주제

- SQL 쿼리 빌더 (테이블, 조인, 필터, group by, order by, limit)
- API 엔드포인트 디자이너 (라우트, 메서드, 요청/응답 필드 빌더)
- 데이터 변환 파이프라인 (소스 → 필터 → 맵 → 집계 → 출력)
- 정규표현식 빌더 (샘플 문자열, 매치 그룹, 실시간 하이라이트)
- 크론(cron) 스케줄러 빌더 (시각적 타임라인, 간격, 요일 토글)
- GraphQL 쿼리 빌더 (타입 선택, 필드 피커, 중첩 리졸버)
