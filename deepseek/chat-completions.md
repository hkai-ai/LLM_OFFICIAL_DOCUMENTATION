---
source: https://api-docs.deepseek.com/zh-cn/api/create-chat-completion
fetched_at: 2026-05-19
api_version: N/A
---

# 对话补全 · POST /chat/completions

> 根据消息列表生成模型回复，OpenAI Chat Completions 协议兼容。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DEEPSEEK_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |
| `Accept` | ✗ | 流式建议 `text/event-stream` |

Base URL：`https://api.deepseek.com`（或 `https://api.deepseek.com/v1`）。如需 `prefix` 续写，请改用 `https://api.deepseek.com/beta`。

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID，取值见 [models.md](./models.md)，例如 `deepseek-v4-pro` / `deepseek-v4-flash` / `deepseek-chat` / `deepseek-reasoner`。 |
| `messages` | array | ✓ | — | 对话消息列表，至少 1 条；按 role 区分子结构，详见 [`messages[]`](#messages)。 |
| `max_tokens` | integer | ✗ | — | 单次回复最大生成 token 数；上限以模型最大输出为准。 |
| `temperature` | number | ✗ | `1` | 取值 `[0, 2]`。思考模式下被忽略。 |
| `top_p` | number | ✗ | `1` | 取值 `(0, 1]`，与 `temperature` 二选一使用。思考模式下被忽略。 |
| `frequency_penalty` | number | ✗ | `0` | 取值 `[-2, 2]`，对高频 token 进行惩罚。思考模式下被忽略。 |
| `presence_penalty` | number | ✗ | `0` | 取值 `[-2, 2]`，对已出现 token 进行惩罚。思考模式下被忽略。 |
| `response_format` | object | ✗ | `{ "type": "text" }` | 输出格式控制，详见 [`response_format`](#response_format)。 |
| `stop` | string \| array<string> | ✗ | — | 停止序列，最多 16 个。匹配到的内容不会出现在 `content` 中。 |
| `stream` | boolean | ✗ | `false` | 是否以 SSE 流式返回。 |
| `stream_options` | object | ✗ | — | 流式额外选项，详见 [`stream_options`](#stream_options)。 |
| `tools` | array | ✗ | — | 可调用的工具列表，最多 128 项，详见 [`tools[]`](#tools)。 |
| `tool_choice` | string \| object | ✗ | `auto` | `none` / `auto` / `required` 或 `{ "type": "function", "function": { "name": "<fn>" } }`。 |
| `logprobs` | boolean | ✗ | `false` | 是否返回每个输出 token 的对数概率。 |
| `top_logprobs` | integer | ✗ | — | `[0, 20]`，每个位置返回最可能的 N 个候选；需同时设 `logprobs=true`。 |
| `thinking` | object | ✗ | `{ "type": "enabled" }`（V4 系列） | 思考模式开关与强度，详见 [`thinking`](#thinking)。 |
| `user_id` | string | ✗ | — | 自定义用户标识，正则 `[a-zA-Z0-9\-_]`，长度 ≤ 512。 |

> DeepSeek 文档将 `frequency_penalty` / `presence_penalty` 标注为兼容性参数：思考模式下不生效，非思考模式下行为与 OpenAI 一致。

### `messages[]`

按 `role` 不同，可携带字段不同。

#### `role = "system"`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `system`。 |
| `content` | string | ✓ | 系统提示词。 |
| `name` | string | ✗ | 发送者标识。 |

#### `role = "user"`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `user`。 |
| `content` | string | ✓ | 用户输入。 |
| `name` | string | ✗ | 发送者标识。 |

#### `role = "assistant"`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `assistant`。 |
| `content` | string \| null | ✓ | 模型回复内容。当包含 `tool_calls` 时可为 `null`。 |
| `name` | string | ✗ | 发送者标识。 |
| `tool_calls` | array | ✗ | 上一轮模型产生的工具调用，结构同响应中的 `tool_calls`。 |
| `reasoning_content` | string \| null | ✗ | 思考模式下回传上一轮思维链。多轮中若发生 tool call，必须回传；否则可省略。 |
| `prefix` | boolean | ✗ | Beta，强制模型以该 `content` 作为回答前缀续写。只能用于消息数组中最后一条消息，且该消息 `role` 必须为 `assistant`。需 `base_url = https://api.deepseek.com/beta`。 |

#### `role = "tool"`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `tool`。 |
| `content` | string | ✓ | 工具执行结果。 |
| `tool_call_id` | string | ✓ | 对应 assistant 上一轮 `tool_calls[].id`。 |

### `response_format`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `text` 或 `json_object`。设为 `json_object` 时必须在 prompt 中引导模型输出 JSON，否则可能产生空字符串或超长输出。 |

### `stream_options`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `include_usage` | boolean | ✗ | 为 `true` 时，流式响应在最终 `[DONE]` 之前会额外发送一个包含完整 `usage` 的 chunk。 |

### `tools[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 当前仅支持 `function`。 |
| `function.name` | string | ✓ | 函数名。 |
| `function.description` | string | ✗ | 描述供模型理解。 |
| `function.parameters` | object | ✗ | JSON Schema 描述参数。 |

### `thinking`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✗ | `enabled` / `disabled`。V4 系列默认 `enabled`。 |
| `reasoning_effort` | string | ✗ | `high` / `max`。普通请求默认 `high`；Agent 复杂任务自动 `max`。 |

> 思考模式开启时，`temperature` / `top_p` / `frequency_penalty` / `presence_penalty` 不生效。

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 本次响应的唯一标识。 |
| `object` | string | 固定 `chat.completion`（流式分片为 `chat.completion.chunk`）。 |
| `created` | integer | Unix 时间戳（秒）。 |
| `model` | string | 实际使用的模型 ID。 |
| `system_fingerprint` | string | 后端配置指纹。 |
| `choices` | array | 候选回复列表，详见 [`choices[]`](#choices)。 |
| `usage` | object | token 使用统计，详见 [`usage`](#usage)。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 序号，从 `0` 开始。 |
| `message` | object | 模型消息，详见 [`choices[].message`](#choicesmessage)。 |
| `finish_reason` | string | `stop` / `length` / `content_filter` / `tool_calls` / `insufficient_system_resource`。 |
| `logprobs` | object \| null | 对数概率信息（`logprobs=true` 时返回）。 |

### `choices[].message`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `role` | string | 固定 `assistant`。 |
| `content` | string \| null | 最终回答内容；触发 `tool_calls` 时可能为 `null`。 |
| `reasoning_content` | string \| null | 思考模式下的思维链内容，与 `content` 同级。 |
| `tool_calls` | array | 工具调用列表，每项含 `id` / `type` / `function.name` / `function.arguments`。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 数（已含缓存命中部分）。 |
| `completion_tokens` | integer | 输出 token 数（含 `reasoning_tokens`）。 |
| `total_tokens` | integer | 输入 + 输出之和。 |
| `prompt_cache_hit_tokens` | integer | 输入中命中上下文硬盘缓存的 token 数。 |
| `prompt_cache_miss_tokens` | integer | 输入中未命中缓存的 token 数。 |
| `completion_tokens_details` | object | 输出 token 细分，含 `reasoning_tokens`（思维链 token 数）。 |

## 流式响应

`stream=true` 时返回 SSE，每个事件 `data:` 行后跟一个 JSON：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` / `object` / `created` / `model` | — | 同非流式，`object = chat.completion.chunk`。 |
| `choices[].delta.role` | string | 仅首个 chunk 出现。 |
| `choices[].delta.content` | string | 增量文本片段。 |
| `choices[].delta.reasoning_content` | string | 思考模式下的增量思维链片段，发生在 `content` 之前。 |
| `choices[].delta.tool_calls` | array | 增量的工具调用结构（`function.arguments` 也是增量拼接）。 |
| `choices[].finish_reason` | string \| null | 仅最后一个 chunk 给出。 |
| `usage` | object | 仅在 `stream_options.include_usage = true` 时，在 `[DONE]` 之前的最后一个数据 chunk 返回。 |

数据流以 `data: [DONE]` 结束。等待推理期间服务器可能发送 `: keep-alive` 注释行（流式）或空行（非流式）；如果 10 分钟内仍未开始推理，服务器会关闭连接。

## 示例

### 最小请求

```json
{
  "model": "deepseek-chat",
  "messages": [
    {"role": "user", "content": "你好"}
  ]
}
```

### 最小响应

```json
{
  "id": "chatcmpl-xxxxxxxx",
  "object": "chat.completion",
  "created": 1747641600,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！有什么我可以帮你的吗？"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21,
    "prompt_cache_hit_tokens": 0,
    "prompt_cache_miss_tokens": 9
  }
}
```

### 思考模式响应（节选）

```json
{
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "reasoning_content": "用户问的是 …… 我先 ……",
      "content": "答案是 42。"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 32,
    "completion_tokens": 480,
    "total_tokens": 512,
    "prompt_cache_hit_tokens": 0,
    "prompt_cache_miss_tokens": 32,
    "completion_tokens_details": { "reasoning_tokens": 430 }
  }
}
```

### `prefix` 续写请求

```json
{
  "model": "deepseek-chat",
  "messages": [
    {"role": "user", "content": "写一个 Python 快排"},
    {"role": "assistant", "content": "```python\n", "prefix": true}
  ],
  "stop": ["```"]
}
```

> 该请求必须把 base URL 改为 `https://api.deepseek.com/beta`。

## 错误码

| HTTP | 含义 | 触发原因 |
| --- | --- | --- |
| `400` | invalid_request_error | 请求体格式错误。 |
| `401` | authentication_error | API key 错误。 |
| `402` | insufficient_balance | 账户余额不足。 |
| `422` | invalid_parameter | 参数取值不合法。 |
| `429` | rate_limit | 触发限速（TPM / RPM / 并发）。 |
| `500` | server_error | 服务端故障。 |
| `503` | service_unavailable | 服务过载。 |

详见 [errors.md](./errors.md)。

## 参考

- 端点文档：https://api-docs.deepseek.com/zh-cn/api/create-chat-completion
- 思考模式：https://api-docs.deepseek.com/zh-cn/guides/thinking_mode
- 工具调用：https://api-docs.deepseek.com/zh-cn/guides/tool_calls
- JSON 输出：https://api-docs.deepseek.com/zh-cn/guides/json_mode
- 多轮对话：https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat
- prefix 续写：https://api-docs.deepseek.com/zh-cn/guides/chat_prefix_completion
