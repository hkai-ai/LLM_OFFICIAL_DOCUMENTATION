---
source: https://openrouter.ai/docs/api/reference/errors-and-debugging
fetched_at: 2026-05-19
api_version: N/A
---

# 错误处理

> OpenRouter 全部错误响应使用统一 JSON 结构。请求被上游 provider 拒绝时，会把 provider 的原始错误透传到 `error.metadata.raw`，但 HTTP 状态码与顶层 `error.code` 由 OpenRouter 归一化。

## 错误响应结构

```ts
{
  error: {
    code: number;        // HTTP 状态码
    message: string;     // 错误描述
    metadata?: object;   // 上下文信息，结构依错误类型而异
  }
}
```

## HTTP 状态码

| 状态码 | 含义 |
| --- | --- |
| `400` | Bad Request：参数缺失或非法，或 CORS 失败。 |
| `401` | 凭证无效：OAuth session 过期、API key 被禁用或无效。 |
| `402` | 账户/API key 额度不足。 |
| `403` | 拒绝访问：权限不足、guardrail 拦截或 moderation 命中。 |
| `408` | 请求超时。 |
| `429` | 触发限流。 |
| `502` | 选定模型故障或上游返回非法响应。 |
| `503` | 没有可用 provider 满足你的路由要求。 |

## Moderation 错误的 `metadata`

当输入被审核拦截时（通常 `403`）：

```ts
{
  reasons: string[];        // 命中的违规原因
  flagged_input: string;    // 被标记的文本片段（≤100 字符）
  provider_name: string;    // 执行 moderation 的 provider 名称
  model_slug: string;       // 模型标识
}
```

## Provider 错误的 `metadata`

上游 provider 报错时：

```ts
{
  provider_name: string;    // 报错的 provider 名称
  raw: unknown;             // provider 上游的原始错误体（透传）
}
```

## `Retry-After` Header

429 和 503 响应可能携带标准 HTTP `Retry-After`，单位为秒：

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

调用方应当遵守该值再发起重试。

## 流式（SSE）中的错误

- **请求建立前**出错：返回标准 JSON 错误体，HTTP 状态见上表。
- **流中**出错：HTTP 已 200，错误以 SSE 事件下发，`choices[].finish_reason` 为 `"error"`，事件内含 `error` 对象。

## 示例

### 401

```json
{
  "error": {
    "code": 401,
    "message": "No auth credentials found"
  }
}
```

### 403（moderation）

```json
{
  "error": {
    "code": 403,
    "message": "Input flagged by moderation",
    "metadata": {
      "reasons": ["hate"],
      "flagged_input": "...",
      "provider_name": "OpenAI",
      "model_slug": "openai/gpt-4o"
    }
  }
}
```

### 502（provider 错误）

```json
{
  "error": {
    "code": 502,
    "message": "Bad upstream response",
    "metadata": {
      "provider_name": "Together",
      "raw": {"error": "Internal Server Error"}
    }
  }
}
```

## 参考

- 错误与调试：<https://openrouter.ai/docs/api/reference/errors-and-debugging>
- Responses API Beta 错误处理：<https://openrouter.ai/docs/api/reference/responses/error-handling>
