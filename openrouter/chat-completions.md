---
source: https://openrouter.ai/docs/api/api-reference/chat/send-chat-completion-request
fetched_at: 2026-05-19
api_version: N/A
---

# 创建对话补全 · POST /api/v1/chat/completions

> 以 OpenAI Chat Completions 兼容协议生成模型回复。OpenRouter 在标准字段之上扩展了 provider 路由、多模型回退、推理、缓存、插件等字段。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |
| `HTTP-Referer` | ✗ | 应用 URL，参与 openrouter.ai 排行榜归因。 |
| `X-Title` / `X-OpenRouter-Title` | ✗ | 应用展示名称。 |
| `X-OpenRouter-Experimental-Metadata` | ✗ | 取值 `enabled` 时返回 `openrouter_metadata` 调试信息。 |

## 请求参数

### 标准 OpenAI 兼容字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓* | — | 模型 ID。与 `models` 至少二选一。 |
| `models` | array&lt;string&gt; | ✗ | — | 候选模型 ID 数组，按优先级排序；首个失败时回退到下一个。详见 [model-routing.md](./model-routing.md)。 |
| `messages` | array&lt;object&gt; | ✓ | — | 对话消息数组，详见 `messages[]`。 |
| `max_tokens` | integer | ✗ | — | 生成的最大 token 数；部分 provider 强制最低 16。 |
| `max_completion_tokens` | integer | ✗ | — | 与 `max_tokens` 等价的新名（OpenAI 命名）。 |
| `temperature` | number | ✗ | `1.0` | 采样温度，范围 `[0, 2]`。 |
| `top_p` | number | ✗ | `1.0` | 核采样阈值，范围 `(0, 1]`。 |
| `frequency_penalty` | number | ✗ | `0.0` | 范围 `[-2, 2]`。 |
| `presence_penalty` | number | ✗ | `0.0` | 范围 `[-2, 2]`。 |
| `seed` | integer | ✗ | — | 确定性采样种子。 |
| `logit_bias` | object | ✗ | — | `{tokenId: bias}`，bias 范围 `[-100, 100]`。 |
| `logprobs` | boolean | ✗ | `false` | 是否返回每个 token 的 log probability。 |
| `top_logprobs` | integer | ✗ | — | 范围 `[0, 20]`，需开启 `logprobs`。 |
| `stop` | string \| array&lt;string&gt; | ✗ | — | 停止序列，最多 4 个。 |
| `stream` | boolean | ✗ | `false` | 是否启用 SSE 流式返回。 |
| `stream_options` | object | ✗ | — | 流式选项，如 `{"include_usage": true}` 让最后一帧带 usage。 |
| `response_format` | object | ✗ | — | 强制输出格式：`text` / `json_object` / `json_schema` / `grammar` / `python`。 |
| `tools` | array&lt;object&gt; | ✗ | — | 工具列表，详见 `tools[]`。 |
| `tool_choice` | string \| object | ✗ | — | 取 `none` / `auto` / `required` / `{"type":"function","function":{"name":"..."}}`。 |
| `parallel_tool_calls` | boolean | ✗ | `true` | 是否允许并行工具调用。 |
| `user` | string | ✗ | — | 终端用户唯一标识。 |
| `modalities` | array&lt;string&gt; | ✗ | — | 输出模态，可选 `text` / `image` / `audio`。 |
| `prediction` | object | ✗ | — | 预测内容（speculative decoding），形如 `{"type":"content","content":"..."}`。 |
| `service_tier` | string | ✗ | — | 取 `auto` / `default` / `flex` / `priority` / `scale`。 |
| `metadata` | object | ✗ | — | 自定义键值对，最多 16 条，key ≤64 字符，value ≤512 字符。 |

### OpenRouter 采样扩展

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `top_k` | integer | ✗ | `0` | ≥0；0 表示禁用。 |
| `min_p` | number | ✗ | `0.0` | 范围 `[0, 1]`，相对最可能 token 的最小阈值。 |
| `top_a` | number | ✗ | `0.0` | 范围 `[0, 1]`。 |
| `repetition_penalty` | number | ✗ | `1.0` | 范围 `[0, 2]`，对输入 token 的重复进行惩罚。 |
| `verbosity` | string | ✗ | `medium` | 输出冗长度：`low` / `medium` / `high` / `xhigh` / `max`。 |

### OpenRouter 路由扩展

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `provider` | object | ✗ | — | provider 路由偏好，详见 [provider-routing.md](./provider-routing.md)。 |
| `route` | string | ✗ | — | 取 `fallback` 启用多模型回退（配合 `models`）。 |
| `transforms` | array&lt;string&gt; | ✗ | — | 应用于 prompt 的转换链，目前主要为 `middle-out`。详见 [transforms.md](./transforms.md)。 |
| `plugins` | array&lt;object&gt; | ✗ | — | 插件列表：`context-compression` / `file-parser` / `web` / `web-fetch` / `auto-router` / `pareto-router` / `fusion` / `moderation` / `response-healing`。每项形如 `{"id":"web","enabled":true,...}`。 |
| `reasoning` | object | ✗ | — | 推理模型配置，详见 `reasoning`。 |
| `usage` | object | ✗ | — | 形如 `{"include": true}` 时响应 `usage.cost` 等扩展字段才会返回。 |
| `web_search_options` | object | ✗ | — | OpenAI 兼容的 web search 配置；OpenRouter 同时支持 `plugins: [{"id":"web"}]` 的等价用法。 |
| `cache_control` | object | ✗ | — | Anthropic 风格 prompt 缓存控制，形如 `{"type":"ephemeral","ttl":"5m"}` 或 `"1h"`。在 `messages[].content[]` 子项上设置生效。 |
| `trace` | object | ✗ | — | 可观测元数据：`trace_id` / `trace_name` / `span_name` / `generation_name` / `parent_span_id`。 |
| `session_id` | string | ✗ | — | 请求分组标识，≤256 字符。 |
| `stop_server_tools_when` | array&lt;object&gt; | ✗ | — | 代理循环中止条件：`step_count_is` / `max_tokens_used` / `max_cost` / `finish_reason_is` / `has_tool_call`。 |
| `image_config` | object | ✗ | — | 视觉模型相关配置。 |
| `debug` | object | ✗ | — | 形如 `{"echo_upstream_body": true}`，用于回显上游请求。 |

### `messages[]`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✓ | — | `system` / `user` / `assistant` / `tool` / `developer`。 |
| `content` | string \| array&lt;object&gt; | ✓ | — | 字符串或内容块数组。 |
| `name` | string | ✗ | — | 发送者标识。 |
| `tool_calls` | array&lt;object&gt; | ✗ | — | assistant 消息中携带的工具调用。 |
| `tool_call_id` | string | ✗ | — | tool 角色消息对应的调用 ID。 |

### `messages[].content[]`

多模态内容块，常见类型：`text`、`image_url`、`input_audio`、`file`。具体字段参见 OpenAI Chat Completions schema；OpenRouter 在 `image_url` 中允许 `cache_control` 子字段。

### `tools[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 通常为 `function`；OpenRouter 内置服务端工具使用 `openrouter:<name>` 前缀（`openrouter:web_search` / `openrouter:web_fetch` / `openrouter:datetime` / `openrouter:image_generation` / `openrouter:experimental__search_models`）。 |
| `function` | object | ✓* | 函数声明（`name` / `description` / `parameters` JSON Schema），`type=function` 时必填。 |

### `provider`

详见 [provider-routing.md](./provider-routing.md)。主要子字段：`order` / `allow_fallbacks` / `require_parameters` / `data_collection` / `only` / `ignore` / `quantizations` / `sort` / `max_price` / `preferred_max_latency` / `preferred_min_throughput` / `zdr` / `enforce_distillable_text`。

### `reasoning`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `effort` | string | ✗ | `xhigh` / `high` / `medium` / `low` / `minimal` / `none`。 |
| `summary` | string | ✗ | 推理摘要冗长度：`auto` / `concise` / `detailed`。 |
| `enabled` | boolean | ✗ | 显式开关。 |
| `exclude` | boolean | ✗ | 在响应中排除推理段。 |
| `max_tokens` | integer | ✗ | 推理 token 上限。 |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `object` | string | 固定 `chat.completion`。 |
| `created` | integer | Unix 秒级时间戳。 |
| `model` | string | 实际使用的模型 ID（在 `models` 回退或 `openrouter/auto` 情况下为最终选择）。 |
| `provider` | string | OpenRouter 扩展，实际服务该请求的 provider 名称。 |
| `choices` | array&lt;object&gt; | 候选数组。 |
| `usage` | object | token 与费用统计，详见下。 |
| `service_tier` | string | 实际使用的服务等级。 |
| `system_fingerprint` | string | 系统指纹。 |
| `openrouter_metadata` | object | 仅当 `X-OpenRouter-Experimental-Metadata: enabled` 时返回，含路由调试信息。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选序号。 |
| `message` | object | assistant 回复：`role` / `content` / `tool_calls` / `reasoning` / `refusal` / `audio` / `images`。 |
| `finish_reason` | string | `stop` / `length` / `tool_calls` / `content_filter` / `error`。 |
| `logprobs` | object | 仅在 `logprobs=true` 时返回。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数（OpenRouter 归一化值）。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 总和。 |
| `cost` | number | 该请求总费用（USD）。仅在请求设置 `usage: {"include": true}` 时返回。 |
| `is_byok` | boolean | 是否走 BYOK key。 |
| `prompt_tokens_details` | object | `cached_tokens` / `cache_write_tokens` / `audio_tokens` / `video_tokens`。 |
| `completion_tokens_details` | object | `reasoning_tokens` / `audio_tokens` / `accepted_prediction_tokens` / `rejected_prediction_tokens`。 |
| `cost_details` | object | `upstream_inference_prompt_cost` / `upstream_inference_completions_cost` / `upstream_inference_cost`。 |

### `openrouter_metadata`（实验）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `strategy` | string | `direct` / `auto` / `free` / `latest` / `alias` / `fallback` / `pareto` / `bodybuilder` / `fusion`。 |
| `requested` | string | 客户端最初请求的 model 字段。 |
| `summary` | string | 路由概要。 |
| `attempt` | integer | 当前尝试序号。 |
| `attempts` | array | 历史尝试。 |
| `endpoints` | object | 候选 endpoint 元数据。 |
| `region` | string | 处理区域。 |
| `is_byok` | boolean | BYOK 状态。 |
| `params` | object | 路由参数：`quality_floor` / `throughput_floor` / `version_group` 等。 |
| `pipeline` | array | 处理阶段：`type`（`guardrail` / `plugin` / `server_tools` / `response_healing` / `context_compression`）/ `name` / `summary` / `cost_usd` / `guardrail_id` / `guardrail_scope`。 |

## 流式响应

设置 `stream: true` 后，响应为 SSE。事件结构与 OpenAI 一致，每行以 `data: ` 开头：

- 中间事件：`{"id":"...","object":"chat.completion.chunk","choices":[{"delta":{"content":"..."}}]}`
- 终止：`data: [DONE]`
- 心跳：`: OPENROUTER PROCESSING`（SSE 注释，按规范忽略即可）
- 末帧（`stream_options.include_usage: true` 时）：携带 `usage`。
- 流中错误：作为 SSE 事件下发，HTTP 状态仍为 200，`finish_reason: "error"`。

## 示例

### 最小请求

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {"role": "user", "content": "Hello"}
  ]
}
```

### 多 provider 路由 + 回退

```json
{
  "models": ["anthropic/claude-3.5-sonnet", "openai/gpt-4o"],
  "route": "fallback",
  "provider": {
    "order": ["Anthropic", "OpenAI"],
    "allow_fallbacks": true,
    "require_parameters": true,
    "data_collection": "deny",
    "quantizations": ["fp16", "bf16"],
    "sort": "price"
  },
  "transforms": ["middle-out"],
  "usage": {"include": true},
  "messages": [{"role": "user", "content": "Summarize this..."}]
}
```

### 最小响应

```json
{
  "id": "gen-...",
  "object": "chat.completion",
  "created": 1715000000,
  "model": "openai/gpt-4o-mini",
  "provider": "OpenAI",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "Hi!"},
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 8,
    "completion_tokens": 2,
    "total_tokens": 10,
    "cost": 0.000012,
    "prompt_tokens_details": {"cached_tokens": 0}
  }
}
```

## 错误码

见 [errors.md](./errors.md)。

## 参考

- 端点文档：<https://openrouter.ai/docs/api/api-reference/chat/send-chat-completion-request>
- 参数索引：<https://openrouter.ai/docs/api/reference/parameters>
- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
