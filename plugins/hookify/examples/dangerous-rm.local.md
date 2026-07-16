---
name: block-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

⚠️ **위험한 rm 명령어가 감지되었습니다!**

이 명령어는 중요한 파일을 삭제할 수 있습니다. 다음을 수행하십시오:
- 경로가 올바른지 확인하십시오
- 보다 안전한 접근 방식을 사용하는 것을 고려해 보십시오
- 백업이 있는지 확인하십시오
