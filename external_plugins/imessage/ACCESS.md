# Greptile

[Greptile](https://greptile.com)은 풀 리퀘스트를 자동으로 검토하는 GitHub 및 GitLab용 AI 코드 리뷰 에이전트입니다. 이 플러그인은 Claude Code를 귀하의 Greptile 계정에 연결하여, 터미널에서 직접 Greptile의 리뷰 댓글을 확인하고 해결할 수 있도록 해줍니다.

## 설정

### 1. Greptile 계정 생성

[greptile.com](https://greptile.com)에서 가입하고 GitHub 또는 GitLab 저장소를 연결합니다.

### 2. API 키 가져오기

1. [API 설정](https://app.greptile.com/settings/api)으로 이동합니다.
2. 새 API 키를 생성합니다.
3. 키를 복사합니다.

### 3. 환경 변수 설정

쉘 프로필(`.bashrc`, `.zshrc` 등)에 추가합니다:

```bash
export GREPTILE_API_KEY="your-api-key-here"
```

그런 다음 쉘을 다시 로드하거나 `source ~/.zshrc`를 실행합니다.

## 사용 가능한 도구

### 풀 리퀘스트(PR) 도구
- `list_pull_requests` - 저장소, 브랜치, 작성자 또는 상태별 선택적 필터링을 사용하여 PR 목록 조회
- `get_merge_request` - 리뷰 분석을 포함한 상세 PR 정보 가져오기
- `list_merge_request_comments` - 필터 옵션을 포함하여 PR의 모든 댓글 가져오기

### 코드 리뷰 도구
- `list_code_reviews` - 선택적 필터링을 사용하여 코드 리뷰 목록 조회
- `get_code_review` - 상세 코드 리뷰 정보 가져오기
- `trigger_code_review` - PR에 대해 새로운 Greptile 리뷰 시작하기

### 댓글 검색
- `search_greptile_comments` - 모든 Greptile 리뷰 댓글 중에서 검색

### 커스텀 컨텍스트 도구
- `list_custom_context` - 조직의 코딩 패턴 및 규칙 목록 조회
- `get_custom_context` - 특정 패턴의 상세 정보 가져오기
- `search_custom_context` - 내용으로 패턴 검색
- `create_custom_context` - 새로운 코딩 패턴 생성

## 사용 예시

Claude Code에게 다음과 같이 요청하십시오:
- "현재 PR에 달린 Greptile의 댓글을 보여주고 해결할 수 있도록 도와줘"
- "Greptile이 PR #123에서 발견한 문제는 무엇인가요?"
- "이 브랜치에 대해 Greptile 리뷰를 시작해 줘"

## 문서

자세한 내용은 [greptile.com/docs](https://greptile.com/docs)를 방문하십시오.
