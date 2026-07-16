# 서버 기능 (Server capabilities) — 스펙 표준의 나머지 기능

서버의 3대 핵심 요소를 제외한 추가 기능들입니다. 대부분 선택 사항이지만, 몇 가지는 거의 공짜로 구현할 수 있는 매우 유용한 기능입니다.

---

## `instructions` — 시스템 프롬프트 주입

설정 한 줄만 추가하면 Claude의 시스템 프롬프트에 직접 반영됩니다. 각 도구 설명에 담기 어려운 도구 사용 팁이나 힌트를 작성할 때 유용합니다.

```typescript
const server = new McpServer(
  { name: "my-server", version: "1.0.0" },
  { instructions: "Always call search_items before get_item — IDs aren't guessable." },
);
```

```python
mcp = FastMCP("my-server", instructions="Always call search_items before get_item — IDs aren't guessable.")
```

이것은 스펙상 가장 효율적인 한 줄 설정입니다. Claude가 도구를 자꾸 엉뚱하게 사용한다면, 이곳에 지침을 추가하여 해결할 수 있습니다.

---

## Sampling (샘플링) — LLM 호출을 호스트에 위임

도구 내부 로직에 LLM 추론(요약, 분류, 생성 등)이 필요할 때, 직접 모델 클라이언트를 패키징해 넣을 필요가 없습니다. 호스트에 실행을 요청하십시오.

```typescript
// Inside a tool handler
const result = await extra.sendRequest({
  method: "sampling/createMessage",
  params: {
    messages: [{ role: "user", content: { type: "text", text: `Summarize: ${doc}` } }],
    maxTokens: 500,
  },
}, CreateMessageResultSchema);
```

```python
# fastmcp
response = await ctx.sample("Summarize this document", context=doc)
```

**클라이언트 측 지원이 필요합니다** — 먼저 `clientCapabilities.sampling`을 확인하십시오. 모델 선호도 힌트는 부분 문자열 매칭으로 판단됩니다 (`"claude-3-5"`는 모든 Claude 3.5 모델군과 매칭됨).

---

## Roots (루트 디렉토리) — 작업 공간 경계 쿼리

루트 디렉토리를 소스 코드에 하드코딩하는 대신, 호스트에 사용자가 승인한 디렉토리가 어디인지 물어보십시오.

```typescript
const caps = server.getClientCapabilities();
if (caps?.roots) {
  const { roots } = await server.server.listRoots();
  // roots: [{ uri: "file:///home/user/project", name: "My Project" }]
}
```

```python
roots = await ctx.list_roots()
```

특히 MCPB 로컬 서버에서 중요하게 작용합니다 — `build-mcpb/references/local-security.md`를 참고하십시오.

---

## Logging (로그) — 정형화되고 레벨 인식이 가능한 로그

원격 서버에서는 stderr보다 이 방식이 훨씬 우수합니다. 클라이언트에서 중요도 레벨에 따라 로그를 필터링할 수 있습니다.

```typescript
// In a tool handler
await extra.sendNotification({
  method: "notifications/message",
  params: { level: "info", logger: "my-tool", data: { msg: "Processing", count: 42 } },
});
```

```python
await ctx.info("Processing", count=42)   # 지원: ctx.debug, ctx.warning, ctx.error
```

로그 레벨은 syslog 표준을 따릅니다: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`. 클라이언트는 `logging/setLevel`을 통해 로그 출력 최소 레벨을 설정합니다.

---

## Progress (진행 상태) — 시간이 오래 걸리는 도구용

클라이언트는 요청 `_meta` 필드에 `progressToken`을 실어 보냅니다. 서버는 이에 대한 응답으로 진행 상태 알림을 발행합니다.

```typescript
async (args, extra) => {
  const token = extra._meta?.progressToken;
  for (let i = 0; i < 100; i++) {
    if (token !== undefined) {
      await extra.sendNotification({
        method: "notifications/progress",
        params: { progressToken: token, progress: i, total: 100, message: `Step ${i}` },
      });
    }
    await doStep(i);
  }
  return { content: [{ type: "text", text: "Done" }] };
}
```

```python
async def long_task(ctx: Context) -> str:
    for i in range(100):
        await ctx.report_progress(progress=i, total=100, message=f"Step {i}")
        await do_step(i)
    return "Done"
```

---

## Cancellation (취소 처리) — 중단 신호 준수

시간이 오래 걸리는 도구는 SDK에서 제공하는 `AbortSignal`을 확인해야 합니다:

```typescript
async (args, extra) => {
  for (const item of items) {
    if (extra.signal.aborted) throw new Error("Cancelled");
    await process(item);
  }
}
```

fastmcp는 asyncio 취소 매커니즘을 통해 이를 자동으로 처리하므로, 핸들러가 온전히 비동기로 구현되어 있다면 명시적인 체크 로직이 필요하지 않습니다.

---

## Completion (자동완성) — 프롬프트 인자 자동완성

인자가 있는 프롬프트나 리소스 템플릿을 등록했다면, 다음과 같이 자동완성 기능을 제공할 수 있습니다:

```typescript
server.registerPrompt("query", {
  argsSchema: {
    table: completable(z.string(), async (partial) => tables.filter(t => t.startsWith(partial))),
  },
}, ...);
```

프롬프트에 입력 가능한 후보 값이 아주 많은 경우가 아니라면 우선순위가 낮습니다.

---

## 어떤 기능이 클라이언트의 지원을 필요로 하나요?

| 기능 | 서버 측 선언 여부 | 클라이언트 필수 지원 여부 | 미지원 시 대체 동작 |
|---|---|---|---|
| `instructions` | 암시적 지원 | — | — (항상 작동함) |
| Logging | `logging: {}` | — | stderr로 출력 |
| Progress | — | `progressToken` 전송 여부 | 자동으로 무시됨 |
| Sampling | — | `sampling: {}` | 자체 LLM 모듈 직접 구현 및 탑재 |
| Elicitation | — | `elicitation: {}` | 텍스트 반환하여 Claude가 질문을 전달하도록 요청 |
| Roots | — | `roots: {}` | 환경 변수 설정으로 대체 |

하단의 3개 기능을 사용하기 전에는 `server.getClientCapabilities()` (TS) 또는 `ctx.session.client_params.capabilities` (fastmcp)를 통해 클라이언트 지원 여부를 반드시 체크하십시오.
