---
source: https://help.aliyun.com/zh/model-studio/model-api-reference/
fetched_at: 2026-11-09
api_version: N/A（Anthropic 兼容，SDK 在 Base URL 后追加 /v1/messages）
---

# Anthropic Messages 兼容（Claude Code 代理） · POST /v1/messages

> 通过百炼 Claude Code 代理，以 Anthropic Messages 协议调用百炼模型。

## Base URL（按地域）

| 地域 | Base URL |
| --- | --- |
| 华北 2（北京） | `https://dashscope.aliyuncs.com/api/v2/apps/claude-code-proxy` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/api/v2/apps/claude-code-proxy` |
| 美西（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/api/v2/apps/claude-code-proxy` |
| 德国（法兰克福） | 文档未说明 |

Anthropic SDK 会在 Base URL 后追加 `/v1/messages`，因此北京地域的完整端点为：

```
POST https://dashscope.aliyuncs.com/api/v2/apps/claude-code-proxy/v1/messages
```

> 注意：`/anthropic` 不是该代理的 Base URL（实测返回 `404`）；`coding.dashscope.aliyuncs.com` 也不提供该代理。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DASHSCOPE_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |
| `x-api-key` | ✗ | 不支持；实测被忽略 |
| `anthropic-version` | ✗ | 不需要 |

> 该代理使用 DashScope 的 Bearer 鉴权，而不是 Anthropic 原生的 `x-api-key`。在 Claude Code 中应使用 `ANTHROPIC_AUTH_TOKEN`（对应 `Authorization: Bearer`），而不是只发送 `x-api-key` 的 `ANTHROPIC_API_KEY`。

## 请求参数

请求体与 Anthropic Messages API 兼容，公共字段（`model` / `messages` / `max_tokens` / `stream` / `system` / `tools` 等）定义见 [anthropic/messages.md](../anthropic/messages.md)。

`model` 使用百炼侧的模型 ID；具体可用模型以百炼控制台与 [models.md](./models.md) 为准。本仓库未单独整理该代理的差异化字段，官方未明示的字段一律以「文档未说明」为准。

## 示例

### 最小请求

```bash
curl https://dashscope.aliyuncs.com/api/v2/apps/claude-code-proxy/v1/messages \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "<BAILIAN_MODEL_ID>",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### 最小响应

```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [{"type": "text", "text": "..."}],
  "model": "<BAILIAN_MODEL_ID>",
  "stop_reason": "end_turn",
  "usage": {"input_tokens": 1, "output_tokens": 1}
}
```

## 错误

| HTTP | 响应 | 含义 |
| --- | --- | --- |
| `401` | `{"code":"InvalidApiKey","message":"No API-key provided."}` | 未提供 `Authorization` |
| `401` | `{"code":"InvalidApiKey","message":"Invalid API-key provided."}` | `Authorization` 中的 Key 无效 |
| `404` | 空响应 | Base URL 或路径错误（例如误用 `/anthropic`） |

## 验证记录

2026-11-09 实测（未使用真实密钥）：

- `POST /anthropic/v1/messages`、`/api/v1/anthropic/v1/messages`、`/compatible-mode/anthropic/v1/messages` 等旧推测路径均返回 `404`。
- `POST https://dashscope.aliyuncs.com/api/v2/apps/claude-code-proxy/v1/messages` 无鉴权返回 `401 No API-key provided.`，确认端点存在。
- 仅携带 `Authorization: Bearer invalid-test-key` 返回 `401 Invalid API-key provided.`；仅携带 `x-api-key` 仍返回 `No API-key provided.`，确认鉴权使用 Bearer。
- 不携带 `anthropic-version` 也能进入鉴权判断，确认该 header 非必需。
- 新加坡、美西地域的 `/api/v2/apps/claude-code-proxy/v1/messages` 同样返回 `401 No API-key provided.`。

## 参考

- 阿里云百炼 API 参考：https://help.aliyun.com/zh/model-studio/model-api-reference/
- Anthropic Messages：../anthropic/messages.md
