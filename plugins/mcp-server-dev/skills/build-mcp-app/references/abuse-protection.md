# 인증 없는 호스팅 서버의 남용 방지 대책 (Abuse protection for authless hosted servers)

인증 단계가 없는 StreamableHTTP 서버는 인터넷 상의 누구에게나 열려 있습니다.
보호해야 할 자원은 다음 세 가지입니다: 호스트 서버의 컴퓨팅 파워, 도구가 소비하는 제3자 API 할당량(quota), 그리고 대용량 `callServerTool` 페이로드가 소비하는 송신(egress) 대역폭.

## 사용자별 개별 식별자의 부재

인증이 없는 모드에서는 토큰(token)이 존재하지 않고 무상태 트랜스포트로 동작하므로 세션 ID도 제공되지 않습니다. claude.ai로부터 유입되는 모든 트래픽은 Anthropic의 Egress 게이트웨이를 거쳐 프록시되므로, 모든 웹 사용자들의 접근은 다음의 소수 IP 대역에서 오는 것으로 인식됩니다:

```
160.79.104.0/21
2607:6bc0::/48
```

(자세한 정보는 https://platform.claude.com/docs/en/api/ip-addresses 참고.)

반면 Claude Desktop, Claude Code 및 기타 호스트 환경은 **사용자의 로컬 머신에서 직접 접속**하므로 개별적인 사용자별 IP 식별이 가능합니다. 따라서 로컬 호스트 연결 클라이언트에 대해서는 IP별 처리가 가능하지만, claude.ai 연결에 대해서는 Anthropic 공통 IP 풀을 전체 통틀어 제한할 수밖에 없습니다. 사용자 단위의 엄밀한 사용량 한계 제어가 반드시 필요하다면, OAuth 인증을 도입해야 합니다.

## 계층형 토큰 버킷 알고리즘 (Tiered token-bucket)

```ts
const ANTHROPIC_CIDRS = ["160.79.104.0/21", "2607:6bc0::/48"];
const TIERS = {
  anthropic: { capacity: 600, refillPerSec: 100 }, // 공유 풀
  other:     { capacity: 30,  refillPerSec: 2   }, // IP별 풀
};
```

`req.ip` 값을 CIDR 대역 정보와 매칭하여 처리할 버킷 유형(`"anthropic"` 또는 `"ip:<addr>"`)을 지정합니다. 버킷 소진 시 429 에러 코드와 `Retry-After` 헤더를 내려보냅니다. 이는 개별 인스턴스 단위의 최소한의 방어선입니다 — 다중 인스턴스 환경 전반에 걸친 방어 제어는 인프라 앞단(Cloudflare, Cloud Armor 등)에서 처리하여 컨테이너를 무상태(stateless) 구조로 가볍게 유지하십시오.

## 실제 토폴로지에 부합하는 `trust proxy` 설정

Express 환경 등에서 `req.ip`가 `X-Forwarded-For` 헤더를 정상적으로 반영하게 하려면 `app.set('trust proxy', N)` 설정이 명시되어야 합니다. 이를 단순히 `true`로 지정하면 거쳐온 모든 경로를 맹신하게 되므로, 악의적인 클라이언트가 헤더에 `X-Forwarded-For: 160.79.108.42`와 같이 위조된 정보를 실어 보내 Anthropic IP 등급인 것처럼 우회할 수 있습니다. 프록시 설정을 신뢰할 수 있는 정확한 프록시 경유 단계 개수(예: 단일 로드밸런서(LB) 뒷단이면 `1`, Cloudflare 뒷단에서 로드밸런서를 경유해 원본 서버로 오면 `2`)로 한정해 구성해야 하며, **운영 환경에서는 절대로 `true`로 설정하지 마십시오**.

## Anthropic IP의 하드 허용(allowlisting) 여부 판단

`160.79.104.0/21` 대역 이외의 모든 IP 트래픽을 원천 차단해 버리면 Claude Desktop이나 Claude Code 등 다른 호스트 환경 사용자들의 유입이 모두 차단됩니다. 단지 claude.ai 환경 사용자만을 대상으로 한정하려는 목적이 아니라면, CIDR 매칭 정보는 접근 제어가 아닌 레이트 리밋(rate limit)의 **계층(tier) 분류용**으로만 활용하십시오.

## 업스트림 API 응답 캐싱 처리

제3자 API를 래핑하는 도구의 경우, 정규화된 쿼리 문자열을 키로 하고 수 시간의 만료 시간(TTL)을 가지는 캐시(키 값에 비밀정보가 노출되지 않도록 처리)를 프로세스 내부 메모리에 유지하는 것이 가장 일차적이고 강력한 비용 관리 수단이 됩니다 — 중복 호출에 대해 무료로 즉시 응답할 수 있어 부하 급증(thundering-herd) 상황을 무난히 흡수할 수 있습니다. 레이트 리밋 제어는 어디까지나 안전망일 뿐이며, 첫 번째 방어선은 캐싱이 되어야 합니다.
