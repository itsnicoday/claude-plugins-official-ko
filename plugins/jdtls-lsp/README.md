# jdtls-lsp

Claude Code를 위한 Java 언어 서버(Eclipse JDT.LS)로, 코드 인텔리전스 및 리팩터링 기능을 제공합니다.

## 지원하는 확장자
`.java`

## 설치

### Homebrew를 통한 설치 (macOS)
```bash
brew install jdtls
```

### 패키지 관리자를 통한 설치 (Linux)
```bash
# Arch Linux (AUR)
yay -S jdtls

# 기타 배포판: 수동 설치 필요
```

### 수동 설치
1. [Eclipse JDT.LS 릴리스](https://download.eclipse.org/jdtls/snapshots/)에서 다운로드합니다.
2. 특정 디렉터리에 압축을 풉니다 (예: `~/.local/share/jdtls`).
3. PATH에 포함된 경로에 `jdtls`라는 이름의 래퍼 스크립트를 생성합니다.

## 요구 사항
- Java 17 이상 (JRE가 아닌 JDK 필수)

## 추가 정보
- [Eclipse JDT.LS GitHub](https://github.com/eclipse-jdtls/eclipse.jdt.ls)
- [VSCode Java Extension](https://github.com/redhat-developer/vscode-java) (JDT.LS 사용)
