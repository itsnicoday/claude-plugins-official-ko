# clangd-lsp

코드 인텔리전스, 진단(diagnostics) 및 포맷팅을 제공하는 Claude Code용 C/C++ 언어 서버(clangd)입니다.

## 지원되는 확장자 (Supported Extensions)
`.c`, `.h`, `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hxx`, `.C`, `.H`

## 설치 (Installation)

### Homebrew 사용 (macOS) (Via Homebrew)
```bash
brew install llvm
# Add to PATH: export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```

### 패키지 관리자 사용 (Linux) (Via package manager)
```bash
# Ubuntu/Debian
sudo apt install clangd

# Fedora
sudo dnf install clang-tools-extra

# Arch Linux
sudo pacman -S clang
```

### Windows
[LLVM 릴리스](https://github.com/llvm/llvm-project/releases)에서 다운로드하거나 다음을 통해 설치하십시오:
```bash
winget install LLVM.LLVM
```

## 추가 정보 (More Information)
- [clangd 웹사이트](https://clangd.llvm.org/)
- [시작 가이드](https://clangd.llvm.org/installation)
