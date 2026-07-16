---
name: warn-sensitive-files
enabled: true
event: file
action: warn
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.env$|\.env\.|credentials|secrets
---

🔐 **민감한 파일 감지됨**

민감한 데이터를 포함할 가능성이 있는 파일을 수정하고 있습니다:
- 자격 증명이 하드코딩되지 않았는지 확인하십시오
- 비밀 정보는 환경 변수를 사용하십시오
- 이 파일이 .gitignore에 포함되어 있는지 확인하십시오
- 비밀 정보 관리자(secrets manager) 사용을 고려해 보십시오
