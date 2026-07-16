# ruby-lsp

코드 인텔리전스 및 분석 기능을 제공하는 Claude Code용 Ruby 언어 서버(Language Server)입니다.

## Supported Extensions (지원되는 확장자)
`.rb`, `.rake`, `.gemspec`, `.ru`, `.erb`

## Installation (설치 방법)

### gem을 통한 설치 (권장)
```bash
gem install ruby-lsp
```

### Bundler를 통한 설치
Gemfile에 다음을 추가합니다:
```ruby
gem 'ruby-lsp', group: :development
```

그 다음 실행합니다:
```bash
bundle install
```

## Requirements (요구사항)
- Ruby 3.0 이상

## More Information (추가 정보)
- [Ruby LSP Website](https://shopify.github.io/ruby-lsp/)
- [GitHub Repository](https://github.com/Shopify/ruby-lsp)
