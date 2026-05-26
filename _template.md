---
source: <官方文档 URL>
fetched_at: YYYY-MM-DD
api_version: <版本号 / 日期版本 / N/A>
---

# <端点中文名> · POST /v1/<path>

> 一句话说明端点用途。例如：「根据消息列表生成模型回复。」

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |
| `<vendor-specific>` | ✗ | 例如 `anthropic-version`、`anthropic-beta` |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。允许值见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组，详见下方子结构。 |
| `max_tokens` | integer | ✗ | — | 生成的最大 token 数。 |
| `temperature` | number | ✗ | `1.0` | 采样温度，范围 `[0, 2]`。 |
| `stream` | boolean | ✗ | `false` | 是否启用 SSE 流式返回。 |

### `messages[]`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✓ | — | 取值：`system` / `user` / `assistant` / `tool`。 |
| `content` | string \| array | ✓ | — | 字符串或内容块数组（见 `messages[].content[]`）。 |
| `name` | string | ✗ | — | 发送者标识。 |

### `messages[].content[]`

…按需展开…

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `object` | string | 固定值，如 `chat.completion`。 |
| `model` | string | 实际使用的模型 ID。 |
| `choices` | array | 候选结果数组。 |
| `usage` | object | token 使用统计。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 总 token 数。 |

## 流式响应（如适用）

事件类型与各事件载荷结构。

## 示例

### 最小请求

```json
{
  "model": "<model-id>",
  "messages": [
    {"role": "user", "content": "Hello"}
  ]
}
```

### 最小响应

```json
{
  "id": "...",
  "object": "chat.completion",
  "model": "<model-id>",
  "choices": [...],
  "usage": {...}
}
```

## 错误码

| HTTP | 含义 | 触发原因 |
| --- | --- | --- |
| `400` | invalid_request_error | 参数缺失或格式错误。 |
| `401` | authentication_error | API key 无效。 |
| `429` | rate_limit_error | 触发限流。 |

## 参考

- 端点文档：<URL>
- 上级目录：<URL>
