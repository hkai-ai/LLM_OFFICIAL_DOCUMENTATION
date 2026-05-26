---
source: https://openrouter.ai/docs/api/api-reference/models/get-models
fetched_at: 2026-05-19
api_version: N/A
---

# 模型相关端点

## GET /api/v1/models

> 列出 OpenRouter 全部可用模型及其元数据（架构、定价、上下文长度、支持参数等）。

### 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |

### 请求参数（query string）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `category` | string | ✗ | — | 按用途类别筛选（如编程、角色扮演、营销等 12 类）。 |
| `supported_parameters` | string | ✗ | — | 逗号分隔的参数名，仅返回支持这些参数的模型。 |
| `output_modalities` | string | ✗ | — | 按输出模态筛选（`text` / `image` / `audio` / `embedding` 等）。 |
| `use_rss` | boolean | ✗ | `false` | 以 RSS 源形式返回。 |
| `use_rss_chat_links` | boolean | ✗ | `false` | RSS 项使用 chat 链接。 |

### 响应

返回 `{"data": [Model, ...]}`，每个 `Model` 包含：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 模型 ID，如 `anthropic/claude-3.5-sonnet`。 |
| `name` | string | 显示名称。 |
| `canonical_slug` | string | 标准 slug。 |
| `created` | integer | Unix 时间戳。 |
| `description` | string | 模型描述。 |
| `context_length` | integer | 最大上下文 token 数。 |
| `knowledge_cutoff` | string | 训练数据截止日期。 |
| `architecture` | object | 架构信息，详见下方。 |
| `pricing` | object | 定价（单位：USD/token 或 USD/次），详见下方。 |
| `top_provider` | object | 首选 provider 元信息，详见下方。 |
| `per_request_limits` | object | 单请求限制（如 max prompt tokens 等）。 |
| `supported_parameters` | array&lt;string&gt; | 支持的参数名列表，如 `temperature` / `top_p` / `tools` / `response_format`。 |
| `default_parameters` | object | 默认参数值。 |
| `links` | object | 相关 endpoint URL。 |
| `expiration_date` | string \| null | 下线日期。 |
| `hugging_face_id` | string \| null | HuggingFace 上的模型 ID。 |
| `supported_voices` | array&lt;string&gt; | TTS 模型可用音色。 |

### `architecture`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_modalities` | array&lt;string&gt; | 输入模态，如 `text` / `image` / `audio` / `file`。 |
| `output_modalities` | array&lt;string&gt; | 输出模态。 |
| `tokenizer` | string | tokenizer 标识，如 `GPT` / `Claude` / `Llama3`。 |
| `instruct_type` | string \| null | 指令格式标识。 |

### `pricing`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt` | string | 每输入 token 的价格（USD）。 |
| `completion` | string | 每输出 token 的价格（USD）。 |
| `request` | string | 每请求的固定费用（USD）。 |
| `image` | string | 每图像输入的费用（USD）。 |
| `internal_reasoning` | string | 推理 token 的费用（USD）。 |
| `web_search` | string | 启用 web search 时的附加费用。 |
| `input_cache_read` | string | 命中缓存时的输入 token 费用。 |
| `input_cache_write` | string | 写入缓存时的输入 token 费用。 |

### `top_provider`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `context_length` | integer | 该 provider 上的上下文长度。 |
| `max_completion_tokens` | integer \| null | 最大完成 token 数。 |
| `is_moderated` | boolean | 是否启用 moderation。 |

### 示例响应

```json
{
  "data": [
    {
      "id": "anthropic/claude-3.5-sonnet",
      "name": "Anthropic: Claude 3.5 Sonnet",
      "created": 1718841600,
      "description": "...",
      "context_length": 200000,
      "architecture": {
        "input_modalities": ["text", "image"],
        "output_modalities": ["text"],
        "tokenizer": "Claude",
        "instruct_type": null
      },
      "pricing": {
        "prompt": "0.000003",
        "completion": "0.000015",
        "request": "0",
        "image": "0.0048",
        "input_cache_read": "0.0000003",
        "input_cache_write": "0.00000375"
      },
      "top_provider": {
        "context_length": 200000,
        "max_completion_tokens": 8192,
        "is_moderated": false
      },
      "per_request_limits": null,
      "supported_parameters": ["temperature", "top_p", "tools", "tool_choice", "response_format", "stop", "stream"]
    }
  ]
}
```

## GET /api/v1/models/{author}/{slug}/endpoints

> 列出某个模型在所有 provider 上的具体 endpoint，含每个 provider 的量化等级、价格、吞吐、延迟、可用参数等。

### 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |

### 路径参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `author` | string | ✓ | 模型作者/组织，如 `anthropic`。 |
| `slug` | string | ✓ | 模型 slug，如 `claude-3.5-sonnet`。 |

### 响应

返回 `{"data": {...}}`，`data` 包含：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 模型 ID。 |
| `name` | string | 显示名。 |
| `created` | integer | Unix 时间戳。 |
| `description` | string | 描述。 |
| `architecture` | object | 同 `/models` 中结构。 |
| `endpoints` | array&lt;object&gt; | provider 端点数组，每项见下。 |

### `endpoints[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `provider_name` | string | provider 名称（枚举）。 |
| `tag` | string | endpoint 标签。 |
| `quantization` | string \| null | 量化等级（`int4`/`int8`/`fp4`/`fp6`/`fp8`/`fp16`/`bf16`/`fp32`/`unknown`）。 |
| `context_length` | integer | 该 endpoint 支持的上下文长度。 |
| `max_completion_tokens` | integer \| null | 最大完成 token 数。 |
| `max_prompt_tokens` | integer \| null | 最大提示 token 数。 |
| `supported_parameters` | array&lt;string&gt; | 该 endpoint 支持的参数。 |
| `status` | string | endpoint 状态（枚举）。 |
| `uptime_last_30m` | number \| null | 过去 30 分钟可用率（百分比）。 |
| `latency_last_30m` | object | 延迟分位数（p50/p75/p90/p99）。 |
| `throughput_last_30m` | object | 吞吐分位数。 |
| `pricing` | object | 同 `/models` 中 `pricing` 结构。 |

### 错误码

| HTTP | 含义 |
| --- | --- |
| `200` | 成功。 |
| `404` | 模型不存在。 |
| `500` | 服务器错误。 |

## 参考

- `/models`：<https://openrouter.ai/docs/api/api-reference/models/get-models>
- `/models/{author}/{slug}/endpoints`：<https://openrouter.ai/docs/api/api-reference/endpoints/list-endpoints>
- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
