---
name: require-tests-run
enabled: false
event: stop
action: block
conditions:
  - field: transcript
    operator: not_contains
    pattern: npm test|pytest|cargo test
---

**대화 기록에서 테스트 명령어가 감지되지 않았습니다!**

종료하기 전에 테스트를 실행하여 변경 사항이 올바르게 작동하는지 확인하십시오.

다음과 같은 테스트 명령어를 실행해 보십시오:
- `npm test`
- `pytest`
- `cargo test`

**참고:** 이 규칙은 대화 기록에 테스트 명령어가 나타나지 않으면 세션 종료를 차단합니다.
엄격한 테스트 검증이 필요할 때만 이 규칙을 활성화하십시오.
