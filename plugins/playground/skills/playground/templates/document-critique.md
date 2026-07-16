# 문서 검토 템플릿 (Document Critique Template)

SKILL.md 파일, README, 스펙 사양서, 제안서 등 구조화된 피드백 및 승인/거부/의견 작성 워크플로우를 통해 문서의 리뷰 및 검토를 돕는 플레이그라운드를 빌드할 때 이 템플릿을 사용하십시오.

## 레이아웃

```
+---------------------------+--------------------+
|                           |                    |
|  문서 콘텐츠              |  제안 사항 패널    |
|  (라인 번호 표시 및       |  (필터링 가능 목록)|
|   제안 사항 하이라이트    |  • 승인            |
|   포함)                   |  • 거부            |
|                           |  • 의견            |
|                           |                    |
+---------------------------+--------------------+
|  프롬프트 출력 (승인 및 의견이 입력된 항목들)          |
|  [ 프롬프트 복사 ]                                 |
+----------------------------------------------------+
```

## 주요 컴포넌트

### 문서 패널 (좌측)
- 라인 번호가 표시된 전체 문서 출력
- 제안 사항이 있는 라인의 좌측 경계선에 색상 하이라이트 표시
- 상태별 색상 구분: 대기(황색), 승인(녹색), 거부(반투명 적색)
- 제안 카드를 클릭하면 해당하는 라인으로 스크롤 이동

### 제안 사항 패널 (우측)
- 필터 탭: 전체 / 대기 / 승인 / 거부
- 각 상태별 개수를 보여주는 헤더 통계
- 각 제안 카드 표시 항목:
  - 라인 참조 (예: "Line 3" 또는 "Lines 17-24")
  - 제안 내용 텍스트
  - 동작 버튼: 승인 / 거부 / 의견 (이미 결정된 경우 초기화)
  - 사용자 의견 작성을 위한 선택적 입력란 (textarea)

### 프롬프트 출력 (하단)
- 승인된 제안 및 사용자 의견만을 바탕으로 프롬프트 생성
- 그룹화 기준: 승인된 개선 사항, 추가 피드백, 거부된 항목 (컨텍스트 제공용)
- 클릭 시 "복사됨!" 피드백을 제공하는 복사 버튼

## 상태(State) 구조

```javascript
const suggestions = [
  {
    id: 1,
    lineRef: "Line 3",
    targetText: "description: Creates interactive...",
    suggestion: "The description is too long. Consider shortening.",
    category: "clarity",  // clarity(명확성), completeness(완전성), performance(성능), accessibility(접근성), ux
    status: "pending",    // pending(대기), approved(승인), rejected(거부)
    userComment: ""
  },
  // ... 더 많은 제안 사항들
];

let state = {
  suggestions: [...],
  activeFilter: "all",
  activeSuggestionId: null
};
```

## 라인별 제안 매칭

`lineRef`를 파싱하여 제안을 문서 라인에 매칭합니다:

```javascript
const suggestion = state.suggestions.find(s => {
  const match = s.lineRef.match(/Line[s]?\s*(\d+)/);
  if (match) {
    const targetLine = parseInt(match[1]);
    return Math.abs(targetLine - lineNum) <= 2; // 인근 라인과 퍼지 매칭
  }
  return false;
});
```

## 문서 렌더링

인라인 마크다운 스타일 포맷팅을 처리합니다:

```javascript
// ``` 라인은 건너뛰고, 본문 내용은 code-block-wrapper로 감싸기
if (line.startsWith('```')) {
  inCodeBlock = !inCodeBlock;
  // wrapper div 열기 또는 닫기
}

// 제목(Headers)
if (line.startsWith('# ')) renderedLine = `<h1>...</h1>`;
if (line.startsWith('## ')) renderedLine = `<h2>...</h2>`;

// 인라인 포맷팅 (코드 블록 밖에서 적용)
renderedLine = renderedLine.replace(/`([^`]+)`/g, '<code>$1</code>');
renderedLine = renderedLine.replace(/\*\*([^*]+)\*\*/g, '<strong>$1</strong>');
```

## 프롬프트 출력 생성

즉시 반영 가능한 조치 항목만 포함합니다:

```javascript
function updatePrompt() {
  const approved = state.suggestions.filter(s => s.status === 'approved');
  const withComments = state.suggestions.filter(s => s.userComment?.trim());

  if (approved.length === 0 && withComments.length === 0) {
    // 플레이스홀더 표시
    return;
  }

  let prompt = 'Please update [DOCUMENT] with the following changes:\n\n';

  if (approved.length > 0) {
    prompt += '## Approved Improvements\n\n';
    for (const s of approved) {
      prompt += `**${s.lineRef}:** ${s.suggestion}`;
      if (s.userComment?.trim()) {
        prompt += `\n  → User note: ${s.userComment.trim()}`;
      }
      prompt += '\n\n';
    }
  }

  // 추가 의견이나 거부된 항목을 컨텍스트용으로 하단에 기재
}
```

## 스타일링 하이라이트

```css
.doc-line.has-suggestion {
  border-left: 3px solid #bf8700;  /* 대기 상태는 황색 */
  background: rgba(191, 135, 0, 0.08);
}

.doc-line.approved {
  border-left-color: #1a7f37;  /* 승인은 녹색 */
  background: rgba(26, 127, 55, 0.08);
}

.doc-line.rejected {
  border-left-color: #cf222e;  /* 거부는 적색 */
  background: rgba(207, 34, 46, 0.08);
  opacity: 0.6;
}
```

## 제안 항목 미리 생성

특정 문서에 대한 검토 플레이그라운드를 빌드할 때:

1. 문서 콘텐츠를 읽습니다.
2. 다음을 포함하는 제안을 분석 및 생성합니다:
   - 구체적인 라인 참조
   - 명확하고 실행 가능한 제안 텍스트
   - 카테고리 태그 (clarity, completeness, performance, accessibility, ux)
3. HTML에 문서 콘텐츠와 제안 배열을 모두 포함시킵니다.

## 예시 사용 사례

- SKILL.md 검토 (스킬 정의 품질, 완전성, 명확성)
- README 검토 (문서화 품질, 누락된 섹션, 모호한 설명)
- 스펙 사양서 검토 (요구사항 명확성, 예외 상황 누락, 모호성)
- 제안서 피드백 (구조, 논리 구성, 누락된 컨텍스트)
- 코드 주석 검토 (docstring 품질, 인라인 주석 유용성)
