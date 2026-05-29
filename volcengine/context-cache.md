---
source: https://www.volcengine.com/docs/82379/1528789
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 上下文缓存（Context API）· /api/v3/context

> 显式上下文缓存机制：先创建缓存（拿到 `context_id`），再用「上下文缓存对话」携带 `context_id` 调用。降低重复内容的输入成本、提升响应速度。两个端点均**仅支持 Endpoint ID**（不支持直接用 Model ID）。

| 操作 | 方法 | 路径 |
| --- | --- | --- |
| 创建上下文缓存 | POST | `/api/v3/context/create` |
| 上下文缓存对话 | POST | `/api/v3/context/chat/completions` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

## 创建上下文缓存 · POST /api/v3/context/create

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | **Endpoint ID**（暂不支持 Model ID）。 |
| `messages` | object[] | ✓ | — | 希望缓存的消息列表。初始化信息（系统人设、背景）用 `system` 消息置于最前，会一直存储直到缓存到期。支持 `system` / `user` / `assistant` 消息（结构同 Chat API）。 |
| `mode` | string | ✗ | `session` | 缓存类型：`session`（Session 缓存）/ `common_prefix`（前缀缓存）。 |
| `ttl` | integer \| null | ✗ | `86400` | 过期时长（秒），范围 `[3600, 604800]`（1 小时–7 天）。创建即计时，每次使用重置为 0，超过 `ttl` 删除。 |
| `truncation_strategy` | object \| null | ✗ | `null` | 缓存窗口长度策略，仅 `mode=session` 可设。不配置时按模型自动适配。见 [`truncation_strategy`](#truncation_strategy)。 |

### `truncation_strategy`

由 `type` 区分两种模式：

#### `last_history_tokens` 模式（触发上限不重新计算）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | string | ✓ | — | 固定 `last_history_tokens`。 |
| `last_history_tokens` | integer | ✗ | `4096` | 可缓存最大 token 数，范围 `(0, 32768)`。触发上限按缓存时间长短清除（先清最早的）。 |

#### `rolling_tokens` 模式（触发上限重新计算）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | string | ✓ | — | 固定 `rolling_tokens`。 |
| `rolling_tokens` | boolean | ✗ | `true` | 历史消息接近 `max_window_tokens` 时是否按 message 粒度自动裁剪（FIFO）。`false` 时超长则停止输出（`finish_reason=length`）。 |
| `max_window_tokens` | integer \| null | ✗ | `32768` | 可存储历史消息最大值。需满足 `0 < rolling_window_tokens < max_window_tokens < 模型上下文`。 |
| `rolling_window_tokens` | integer \| null | ✗ | `4096` | 触发上限时裁剪的滚动窗口长度。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 上下文缓存 ID，后续对话需用。 |
| `model` | string | 使用的推理接入点 ID。 |
| `mode` | string | `session` / `common_prefix`。 |
| `ttl` | integer | 过期时长（秒）。 |
| `truncation_strategy` | object | Session 缓存使用的截断策略回显。 |
| `usage` | object | `prompt_tokens` / `completion_tokens` / `total_tokens` / `prompt_tokens_details.cached_tokens`。 |

## 上下文缓存对话 · POST /api/v3/context/chat/completions

大部分字段与[对话(Chat) API](./chat-completions.md) 一致，**差异**如下：

- `context_id`（string，**必填**）：指定本次请求使用的缓存 ID（创建缓存时返回）。
- `model`：暂不支持 Model ID，需用 Endpoint ID。
- `messages`：最后一个元素的 `role` 不能为 `assistant`；`mode=session` 时只需传最新一轮对话，无需历史信息。
- `tools`：不支持。
- `thinking`：不支持。
- `response_format`：不支持结构化输出参数（文档开头差异说明；参数表仍列出 `text`/`json_object`/`json_schema`）。
- `service_tier`：缓存不支持 TPM 保障包，故不支持 `auto`，默认 `default`。

其余通用字段（`stream`、`stream_options`、`max_tokens`、`stop`、`frequency_penalty`、`presence_penalty`、`temperature`、`top_p` 等）与 Chat API 相同。响应结构同 Chat API，`usage.prompt_tokens_details.cached_tokens` 反映缓存命中量。

## 示例

### 创建前缀缓存

```bash
curl https://ark.cn-beijing.volces.com/api/v3/context/create \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "messages": [{"content": "你是李雷，你只会说\"我是李雷\"", "role": "system"}],
    "mode": "common_prefix",
    "model": "ep-20250410*****-*****"
  }'
```

```json
{
  "id": "ctx-20250410175333-lkjq2",
  "model": "ep-20250410*****-*****",
  "ttl": 86400,
  "usage": { "prompt_tokens": 18, "completion_tokens": 0, "total_tokens": 18, "prompt_tokens_details": { "cached_tokens": 0 } },
  "mode": "common_prefix"
}
```

### 携带缓存对话

```bash
curl https://ark.cn-beijing.volces.com/api/v3/context/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "ep-20250410*****-*****",
    "context_id": "ctx-20250410175333-lkjq2",
    "messages": [{"role": "user", "content": "你是谁？"}]
  }'
```

## 参考

- 创建上下文缓存：https://www.volcengine.com/docs/82379/1528789
- 上下文缓存对话：https://www.volcengine.com/docs/82379/1529329
