---
source: https://ai.google.dev/api/generate-content?hl=zh-CN#method:-models.streamgeneratecontent
fetched_at: 2026-05-19
api_version: v1beta
---

# 流式生成内容 · POST /v1beta/{model=models/*}:streamGenerateContent

> 与 [generateContent](./generate-content.md) **请求字段完全一致**，但响应以增量分块的形式连续推送，便于实现打字机效果与早响应。

## 请求

请求体、Path 参数、Header 与 `:generateContent` 一致，参见 [generate-content.md](./generate-content.md)。

额外可用的 query 参数：

| Query | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `alt` | string | `json` | 取 `json`（默认，按 JSON 数组分块）或 `sse`（Server-Sent Events 形式，每行 `data: <json>`）。 |

## 响应格式

### 1. JSON 数组分块（默认，`alt=json`）

服务端用 chunked transfer 持续推送一个 **JSON 数组**，但每条 `GenerateContentResponse` 是数组的一个元素，按需 flush。SDK 通常按 `,\n` 边界增量解析。示意：

```
[
{"candidates":[{"content":{"role":"model","parts":[{"text":"量子"}]},"index":0}],"modelVersion":"gemini-2.5-flash-002","responseId":"..."}
,
{"candidates":[{"content":{"role":"model","parts":[{"text":"纠缠"}]},"index":0}],"modelVersion":"..."}
,
...
,
{"candidates":[{"content":{"role":"model","parts":[{"text":"。"}]},"finishReason":"STOP","index":0}],"usageMetadata":{"promptTokenCount":12,"candidatesTokenCount":38,"totalTokenCount":50},"modelVersion":"..."}
]
```

### 2. SSE 流（`?alt=sse`）

`Content-Type: text/event-stream`，每个事件由一行 `data: <json>` + 空行组成；服务端不会发送显式的结束事件，连接关闭即流结束。示意：

```
data: {"candidates":[{"content":{"role":"model","parts":[{"text":"量子"}]},"index":0}],"modelVersion":"..."}

data: {"candidates":[{"content":{"role":"model","parts":[{"text":"纠缠"}]},"index":0}],"modelVersion":"..."}

data: {"candidates":[{"content":{"role":"model","parts":[{"text":"。"}]},"finishReason":"STOP","index":0}],"usageMetadata":{...},"modelVersion":"..."}
```

## 单个分块结构

每个分块都是一个完整的 `GenerateContentResponse`（结构同 [generate-content.md](./generate-content.md#响应generatecontentresponse)），但常见字段是**增量**而非累计：

| 字段 | 出现时机 | 增量含义 |
| --- | --- | --- |
| `candidates[].content.parts[].text` | 每个分块 | 仅本块新增文本，调用方需自行拼接。 |
| `candidates[].content.parts[].functionCall` | 通常在工具触发时整段返回 | 多数情况下整个 `functionCall` 在单个分块内一次性出现，args 不再切片；少数模型/长 args 会分块。 |
| `candidates[].content.parts[].executableCode` / `codeExecutionResult` | 触发代码执行时 | 整段返回。 |
| `candidates[].finishReason` | 仅最后一个分块 | 出现即流即将结束。 |
| `candidates[].safetyRatings` | 通常在末块 | 累计评级。 |
| `candidates[].groundingMetadata` / `urlContextMetadata` | 末块 | 累计 grounding 信息。 |
| `usageMetadata` | 末块 | 整次请求的累计 token 用量。 |
| `promptFeedback` | 任意分块（一般首块/末块） | 输入侧反馈，被拦截时只会有该字段，无 candidates。 |
| `modelVersion`、`responseId` | 通常每块都带 | 同一请求内一致。 |

> 与 OpenAI 的 `delta` 字段不同，Gemini 没有单独的 delta 包裹：每个 chunk 直接是一个 `GenerateContentResponse`，`parts[].text` 字段本身就是该块的增量。

## 终止与错误

- **正常结束**：连接关闭，且末块带 `finishReason ∈ {STOP, MAX_TOKENS, ...}`。
- **安全拦截**：可能只发一个含 `promptFeedback.blockReason` 的分块后结束；或在中途返回带 `finishReason=SAFETY` 的末块。
- **HTTP 4xx/5xx**：响应头阶段直接返回错误 JSON（结构见 [errors.md](./errors.md)），不进入流式状态。
- **流中错误**：极少数情况下 chunk 中会出现含 `error` 字段的对象，调用方需做兼容。

## 示例（SSE）

请求：

```
POST /v1beta/models/gemini-2.5-flash:streamGenerateContent?alt=sse
x-goog-api-key: $GEMINI_API_KEY
Content-Type: application/json

{
  "contents": [{ "role": "user", "parts": [{ "text": "用一句话介绍量子纠缠。" }] }]
}
```

响应（节选）：

```
data: {"candidates":[{"content":{"role":"model","parts":[{"text":"量子纠缠是指"}]},"index":0}],"modelVersion":"gemini-2.5-flash-002","responseId":"abc"}

data: {"candidates":[{"content":{"role":"model","parts":[{"text":"两个或多个粒子的状态相互关联"}]},"index":0}],"modelVersion":"..."}

data: {"candidates":[{"content":{"role":"model","parts":[{"text":"。"}]},"finishReason":"STOP","index":0}],"usageMetadata":{"promptTokenCount":12,"candidatesTokenCount":38,"totalTokenCount":50},"modelVersion":"..."}
```

## 参考

- 端点文档：<https://ai.google.dev/api/generate-content?hl=zh-CN#method:-models.streamgeneratecontent>
- 同步端点：[generate-content.md](./generate-content.md)
