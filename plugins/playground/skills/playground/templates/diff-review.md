# Diff 리뷰 템플릿 (Diff Review Template)

git 커밋, 풀 리퀘스트(PR), 코드 변경 사항 등 코드 diff를 검토하고 피드백을 위해 대화형으로 한 줄씩 댓글을 달 수 있는 플레이그라운드를 빌드할 때 이 템플릿을 사용하십시오.

## 레이아웃

```
+-------------------+----------------------------------+
|                   |                                  |
|  커밋 헤더:       |  Diff 콘텐츠                     |
|  • 해시(Hash)     |  (hunk가 포함된 파일들)          |
|  • 메시지         |  라인 번호 및                    |
|  • 작성자/날짜    |  +/- 표시 포함                   |
|                   |                                  |
+-------------------+----------------------------------+
|  프롬프트 출력 패널 (우측 하단 고정)                  |
|  [ 전체 복사 ]                                       |
|  프롬프트 양식으로 포맷된 모든 댓글 표시             |
+------------------------------------------------------+
```

Diff 리뷰 플레이그라운드는 구문 강조가 적용된 git diff를 표시합니다. 사용자가 라인을 클릭하여 댓글을 추가하면, 해당 댓글들이 코드 리뷰 피드백용 프롬프트 생성 시 포함됩니다.

## Diff 리뷰용 컨트롤 유형

| 기능 | 컨트롤 유형 | 동작 |
|---|---|---|
| 라인별 댓글 추가 | 임의의 diff 라인 클릭 | 해당 라인 아래에 입력창(textarea) 노출 |
| 댓글 표시기 | 댓글이 작성된 라인의 배지 | 피드백이 있는 라인 표시 |
| 저장/취소 | 댓글 상자 내 버튼 | 댓글 저장 또는 파기 |
| 프롬프트 복사 | 프롬프트 패널 내 버튼 | 모든 댓글을 클립보드에 복사 |

## Diff 렌더링

렌더링을 위해 diff 데이터를 다음과 같이 구조화된 포맷으로 파싱합니다:

```javascript
const diffData = [
  {
    file: "path/to/file.py",
    hunks: [
      {
        header: "@@ -41,13 +41,13 @@ function context",
        lines: [
          { type: "context", oldNum: 41, newNum: 41, content: "unchanged line" },
          { type: "deletion", oldNum: 42, newNum: null, content: "removed line" },
          { type: "addition", oldNum: null, newNum: 42, content: "added line" },
        ]
      }
    ]
  }
];
```

## 라인 유형별 스타일링

| 유형 | 배경색 | 글자색 | 접두사 |
|---|---|---|---|
| `context` | 투명 (transparent) | 기본값 | ` ` (공백) |
| `addition` | 녹색 계열 (#dafbe1 light / rgba(46,160,67,0.15) dark) | 녹색 (#1a7f37 light / #7ee787 dark) | `+` |
| `deletion` | 적색 계열 (#ffebe9 light / rgba(248,81,73,0.15) dark) | 적색 (#cf222e light / #f85149 dark) | `-` |
| `hunk-header` | 청색 계열 (#ddf4ff light) | 청색 (#0969da light) | `@@` |

## 댓글 시스템

댓글 추적을 위해 각 diff 라인에 고유 식별자를 부여합니다:

```javascript
const comments = {}; // { lineId: commentText }

function selectLine(lineId, lineEl) {
  // Deselect previous
  document.querySelectorAll('.diff-line.selected').forEach(el =>
    el.classList.remove('selected'));
  document.querySelectorAll('.comment-box.active').forEach(el =>
    el.classList.remove('active'));

  // Select new
  lineEl.classList.add('selected');
  document.getElementById(`comment-box-${lineId}`).classList.add('active');
}

function saveComment(lineId) {
  const textarea = document.getElementById(`textarea-${lineId}`);
  const comment = textarea.value.trim();

  if (comment) {
    comments[lineId] = comment;
  } else {
    delete comments[lineId];
  }

  renderDiff(); // Re-render to show comment indicator
  updatePromptOutput();
}
```

## 프롬프트 출력 포맷

구조화된 코드 리뷰 형식을 생성합니다:

```javascript
function updatePromptOutput() {
  const commentKeys = Object.keys(comments);

  if (commentKeys.length === 0) {
    promptContent.innerHTML = '<span class="no-comments">임의의 라인을 클릭하여 댓글을 추가하세요...</span>';
    return;
  }

  let output = '코드 리뷰 댓글:\n\n';

  commentKeys.forEach(lineId => {
    const lineEl = document.querySelector(`[data-line-id="${lineId}"]`);
    const file = lineEl.dataset.file;
    const lineNum = lineEl.dataset.lineNum;
    const content = lineEl.dataset.content;

    output += `📍 ${file}:${lineNum}\n`;
    output += `   코드: ${content.trim()}\n`;
    output += `   댓글: ${comments[lineId]}\n\n`;
  });

  promptContent.textContent = output;
}
```

## 라인 요소의 데이터 속성(Data attributes)

프롬프트 생성을 위해 각 라인 요소에 메타데이터를 저장합니다:

```html
<div class="diff-line addition"
     data-line-id="0-1-5"
     data-file="src/utils/handler.py"
     data-line-num="45"
     data-content="subagent_id = tracker.register()">
```

## 실제 데이터 채우기

특정 커밋에 대한 diff 뷰어를 만들기 위해:

1. `git show <commit> --format="%H%n%s%n%an%n%ad" -p` 명령어를 실행합니다.
2. 출력을 `diffData` 구조로 파싱합니다.
3. 헤더 섹션에 커밋 메타데이터를 포함시킵니다.

## 테마 지원

라이트 모드와 다크 모드를 모두 지원합니다:

```css
/* Light mode */
body { background: #f6f8fa; color: #1f2328; }
.file-card { background: #ffffff; border: 1px solid #d0d7de; }
.diff-line.addition { background: #dafbe1; }
.diff-line.deletion { background: #ffebe9; }

/* Dark mode */
body { background: #0d1117; color: #c9d1d9; }
.file-card { background: #161b22; border: 1px solid #30363d; }
.diff-line.addition { background: rgba(46, 160, 67, 0.15); }
.diff-line.deletion { background: rgba(248, 81, 73, 0.15); }
```

## 대화형 기능

- **호버 힌트:** 라인에 마우스를 올렸을 때 "클릭하여 댓글 작성" 툴팁 표시
- **댓글 표시기:** 댓글이 저장된 라인에 배지(💬) 표시
- **토스트 알림:** 복사 시 "클립보드에 복사되었습니다!" 피드백 제공
- **기존 댓글 수정:** 이전에 저장한 댓글을 수정할 수 있도록 허용

## 예시 주제

- Git 커밋 리뷰 (라인별 댓글 작성이 포함된 단일 커밋 diff)
- 풀 리퀘스트(PR) 리뷰 (여러 커밋, 파일 수준 및 라인 수준 댓글)
- 코드 diff 비교 (리팩터링 전/후 비교)
- 병합 충돌 해결 (주석과 함께 양쪽 버전 모두 표시)
- 코드 감사 (라인별 분석 결과가 포함된 보안 검토)
