---
source: https://platform.claude.com/docs/en/api/messages-count-tokens
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# Token 计数 · POST /v1/messages/count_tokens

> 在不真正发起一次生成的前提下，计算给定 `messages` / `system` / `tools` 等输入会消耗多少输入 token。常用于成本预估、限流前检查、prompt 设计。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` | 二选一 | API Key。 |
| `anthropic-version` | ✓ | 例如 `2023-06-01`。 |
| `content-type` | ✓ | `application/json`。 |
| `anthropic-beta` | ✗ | 启用 beta 特性。 |

请求体积上限 32 MB。

## 请求参数

参数语义与 `POST /v1/messages` 保持一致，但**不会**真正生成响应，因此与生成行为相关的字段（`max_tokens`、`temperature`、`top_p`、`top_k`、`stop_sequences`、`stream`、`metadata`、`service_tier`）不影响计数。文档列出的可用参数：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 用于计数的模型 ID（不同模型 tokenizer 可能不同）。 |
| `messages` | array | ✓ | 对话消息数组。结构与 [messages.md](./messages.md) 中 `messages[]` 一致。 |
| `system` | `string \| array<TextBlockParam>` | ✗ | system prompt。 |
| `tools` | array | ✗ | 工具定义（自定义 + 内置）。 |
| `tool_choice` | object | ✗ | 工具调用策略。 |
| `thinking` | object | ✗ | 扩展思考配置（影响计数）。 |
| `mcp_servers` | array | ✗ | 远程 MCP server 配置。 |
| `container` | string | ✗ | 容器 ID。 |
| `cache_control` | object | ✗ | 顶层 ephemeral 缓存控制。 |
| `output_config` | object | ✗ | 输出格式配置（JSON schema 等）。 |
| `context_management` | object | ✗ | 上下文管理策略。 |
| `betas` | array&lt;string&gt; | ✗ | 等价于 `anthropic-beta` header。 |

> 文档原文未在该端点页详尽展开每个嵌套字段的 schema，按规范默认与 `/v1/messages` 同名字段保持一致。

## 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_tokens` | integer | 该请求会消耗的输入 token 总数（包含 `messages`、`system`、`tools`、`thinking` 等占用）。 |

> 文档未明确指出该端点是否同时返回 `cache_creation_input_tokens` / `cache_read_input_tokens`。`/v1/messages` 的 `usage` 中包含这两项，但 count_tokens 仅承诺 `input_tokens`。文档未明确。

## 示例

### 请求

```json
{
  "model": "claude-opus-4-7",
  "messages": [
    { "role": "user", "content": "Hello, Claude" }
  ]
}
```

```bash
curl https://api.anthropic.com/v1/messages/count_tokens \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-4-7",
    "messages": [{"role": "user", "content": "Hello, Claude"}]
  }'
```

### 响应

```json
{
  "input_tokens": 10
}
```

## 错误码

| HTTP | `error.type` | 触发场景 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 字段缺失或不合法。 |
| `413` | `request_too_large` | 请求体超过 32 MB。 |
| `429` | `rate_limit_error` | 触发限流（count_tokens 也受租户限流约束）。 |

完整错误列表见 [errors.md](./errors.md)。

## 参考

- 端点文档：`https://platform.claude.com/docs/en/api/messages-count-tokens`
- 上级目录：`https://platform.claude.com/docs/en/api/overview`
