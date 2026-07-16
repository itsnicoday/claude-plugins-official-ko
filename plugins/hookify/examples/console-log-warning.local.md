---
name: warn-console-log
enabled: true
event: file
pattern: console\.log\(
action: warn
---

🔍 **console.log 감지됨**

console.log 구문을 추가하고 있습니다. 다음을 고려해 보십시오:
- 단순 디버깅용입니까, 아니면 정식 로그를 남겨야 합니까?
- 이것이 프로덕션 환경에 배포됩니까?
- 대신 로깅 라이브러리를 사용해야 합니까?
