# 디자인 플레이그라운드 템플릿 (Design Playground Template)

컴포넌트, 레이아웃, 여백, 색상, 타이포그래피, 애니메이션, 반응형 레이아웃 등 시각적 디자인 결정 사항과 관련된 플레이그라운드를 빌드할 때 이 템플릿을 사용하십시오.

## 레이아웃

```
+-------------------+----------------------+
|                   |                      |
|  컨트롤 영역       |  실시간 컴포넌트/    |
|  그룹화 기준:     |  레이아웃 미리보기   |
|  • 여백           |  (가상 페이지 또는   |
|  • 색상           |   단독 카드로        |
|  • 타이포그래피   |   렌더링)            |
|  • 그림자/테두리  |                      |
|  • 상호작용       |                      |
|                   +----------------------+
|                   |  프롬프트 출력       |
|                   |  [ 프롬프트 복사 ]   |
+-------------------+----------------------+
```

## 결정 사항에 따른 컨트롤 유형

| 결정 사항 | 컨트롤 유형 | 예시 |
|---|---|---|
| 크기, 여백, 반경 | 슬라이더 | 테두리 반경 0–24px |
| 기능 On/Off | 토글 | 테두리 표시 여부, 호버 효과 |
| 세트 내 선택 | 드롭다운 | 글꼴 계열, 감쇠 곡선(easing curve) |
| 색상 | 색상(Hue)+채도(Saturation)+명도(Lightness) 슬라이더 | 그림자 색상, 강조색 |
| 레이아웃 구조 | 클릭 가능한 카드 | sidebar-left / top-nav / no-nav |
| 반응형 레이아웃 | 뷰포트 너비 슬라이더 | 중단점(breakpoints)에 따른 그리드 배치 재정렬 확인 |

## 미리보기 렌더링

상태 값을 미리보기 요소의 인라인 스타일(inline style)에 직접 반영합니다:

```javascript
function renderPreview() {
  const el = document.getElementById('preview');
  el.style.borderRadius = state.radius + 'px';
  el.style.padding = state.padding + 'px';
  el.style.boxShadow = state.shadow
    ? `0 ${state.shadowY}px ${state.shadowBlur}px rgba(0,0,0,${state.shadowOpacity})`
    : 'none';
}
```

해당 사항이 있는 경우 미리보기를 라이트 배경과 다크 배경 모두에서 보여주십시오. 컨텍스트 토글 버튼을 포함하십시오.

## 디자인 프롬프트 출력

단순한 스펙 시트 나열이 아닌 개발자에게 내리는 지시문 형태로 작성하십시오:

> "Update the card to feel soft and elevated: 12px border-radius, 24px horizontal padding, a medium box-shadow (0 4px 12px rgba(0,0,0,0.1)). On hover, lift it with translateY(-1px) and deepen the shadow slightly."

사용자가 Tailwind를 사용하는 경우 Tailwind 클래스를 제안하십시오. 순수 CSS를 사용하는 경우 CSS 속성명을 제안하십시오.

## 예시 주제

- 버튼 스타일 탐색기 (반경, 여백, 가중치, 호버/액티브 상태)
- 카드 컴포넌트 (그림자 깊이, 반경, 콘텐츠 레이아웃, 이미지)
- 레이아웃 빌더 (사이드바 너비, 콘텐츠 최대 너비, 헤더 높이, 그리드)
- 타이포그래피 스케일 (h1-body-caption에 걸친 기준 크기, 비율, 줄 높이)
- 색상 팔레트 생성기 (기본 색상, 보조/강조/표면 색상 도출)
- 대시보드 밀도 (여유로움 → 조밀함 슬라이더로 모든 요소를 비율에 맞게 스케일 조정)
- 모달/다이얼로그 (너비, 오버레이 불투명도, 진입 애니메이션, 테두리 반경)
