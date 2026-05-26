---
source: https://platform.kimi.com/docs/api/chat
fetched_at: 2026-05-20
api_version: N/A
---

# 创建对话补全 · POST /v1/chat/completions

OpenAI Chat Completions 兼容协议；附加 Moonshot 特有字段：`thinking`、`partial`（在消息内）、`prompt_cache_key`、`safety_identifier`。

## 请求 Body 字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID，见 [models.md](./models.md)。 |
| `messages` | `array<Message>` | ✓ | — | 对话历史。详见 §messages。 |
| `max_tokens` | integer | ✗ | — | 已弃用，等价于 `max_completion_tokens`。 |
| `max_completion_tokens` | integer | ✗ | `1024` | 单次生成最大输出 token 数。 |
| `temperature` | number | ✗ | 见下 | `0`–`1`。K2 / K2.5 / K2.6 默认 `0.6`；moonshot-v1 默认 `0.0`；思考模式开启时被忽略。 |
| `top_p` | number | ✗ | `1.0` | `0`–`1`。思考模式开启时被忽略。 |
| `top_k` | integer | ✗ | — | 候选 token 数；moonshot-v1 / k2 系列接受，思考模式忽略。 |
| `n` | integer | ✗ | `1` | 返回候选数，`1`–`5`。 |
| `presence_penalty` | number | ✗ | `0` | `-2`–`2`。思考模式忽略。 |
| `frequency_penalty` | number | ✗ | `0` | `-2`–`2`。思考模式忽略。 |
| `stop` | `string \| array<string>` | ✗ | `null` | 最多 5 个停止序列，每个 ≤ 32 字节。 |
| `stream` | boolean | ✗ | `false` | 是否走 SSE 流式。 |
| `stream_options` | object | ✗ | — | 详见 §stream_options。 |
| `response_format` | object | ✗ | `{"type":"text"}` | 输出格式控制，详见 §response_format。 |
| `tools` | `array<Tool>` | ✗ | — | 工具定义，最多 128 项；详见 §tools。 |
| `tool_choice` | `string \| object` | ✗ | `"auto"` | `"none"` / `"auto"` / `"required"` / `{"type":"function","function":{"name":"..."}}`。 |
| `parallel_tool_calls` | boolean | ✗ | `true` | 是否允许一次响应中并行多个工具调用。 |
| `thinking` | object | ✗ | `{"type":"enabled"}`（K2.5 / K2.6） | 思考模式控制；详见 §thinking。仅在支持的模型上生效。 |
| `prompt_cache_key` | string | ✗ | — | 同一会话或同一前缀的请求使用相同值以提高缓存命中。 |
| `safety_identifier` | string | ✗ | — | 终端用户稳定标识符，用于平台合规策略。 |
| `seed` | integer | ✗ | — | 采样随机种子；不保证完全可复现。 |
| `user` | string | ✗ | — | OpenAI 兼容字段，建议改用 `safety_identifier`。 |

### messages[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `system` / `user` / `assistant` / `tool`。 |
| `content` | `string \| array<ContentPart>` | ✓ | 文本或多模态 part 数组；多模态仅在视觉 / 视频模型上有效。 |
| `name` | string | ✗ | 消息发送者标识。 |
| `tool_calls` | array | role=`assistant` 且发起 tool call 时回传 | 由模型生成，回传时原样附在历史中。 |
| `tool_call_id` | string | role=`tool` 时 ✓ | 对应上一轮 `tool_calls[i].id`。 |
| `partial` | boolean | ✗ | 仅可写在**最后一条** `assistant` 消息中，置 `true` 触发 Partial Mode；详见 [partial-mode.md](./partial-mode.md)。 |
| `reasoning_content` | string | ✗ | 思考模式下若需要在多轮中保留思维链，可回传上一轮 assistant 的 `reasoning_content`。 |

### messages[].content[]（多模态）

| `type` | 必填字段 | 说明 |
| --- | --- | --- |
| `text` | `text` (string) | 普通文本片段。 |
| `image_url` | `image_url.url` (string) | URL 可为公网链接、`data:image/...;base64,...` 或 `ms://<file_id>`（Moonshot 文件协议）。 |
| `video_url` | `video_url.url` (string) | 同上，支持 `ms://<file_id>`，仅 K2.5 / K2.6 / vision 模型。 |

### response_format

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `text` / `json_object` / `json_schema`。 |
| `json_schema` | object | type=`json_schema` 时 ✓ | 包含 `name`、`schema`，可选 `strict`（默认 `true`，严格遵循 Moonshot Functional JSON Schema / MFJS 子集）。 |

> `partial: true` 与 `response_format.type: "json_object"` 不可同时使用，组合行为未定义。

### stream_options

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `include_usage` | boolean | `false` | `true` 时在 `data: [DONE]` 之前发送一个包含 `usage` 的 chunk。 |

### tools[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `function`。 |
| `function.name` | string | ✓ | 必须匹配 `^[a-zA-Z_][a-zA-Z0-9-_]{2,63}$`。 |
| `function.description` | string | ✓ | 函数功能说明，影响模型选择行为。 |
| `function.parameters` | object | ✓ | JSON Schema 子集（MFJS）。 |
| `function.strict` | boolean | ✗ | 默认 `true`，严格校验参数。 |

### thinking

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `enabled` / `disabled`。 |
| `keep` | `"all" \| null` | ✗ | K2.6 专有；`"all"` 表示多轮中保留历史 `reasoning_content`，默认每轮重新思考。 |

> 思考模式开启时，`temperature` / `top_p` / `presence_penalty` / `frequency_penalty` 传入不会报错，但被静默忽略。模型在 `choices[].message.reasoning_content` 输出思维链，最终回答在 `content` 中；某些场景模型内部 think 但不暴露 `reasoning_content`，不要假设它一定有。

## 响应 Body 字段（非流式）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 补全唯一 ID。 |
| `object` | string | `chat.completion`。 |
| `created` | integer | Unix 秒时间戳。 |
| `model` | string | 实际使用的模型 ID。 |
| `choices` | array | 详见 §choices。 |
| `usage` | object | 详见 §usage。 |

### choices[]

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选序号。 |
| `message.role` | string | `assistant`。 |
| `message.content` | `string \| null` | 文本回复；当仅有 tool_calls 时可为 `null`。 |
| `message.tool_calls` | array | 见 §tool_calls。 |
| `message.reasoning_content` | string | 思考模式下的思维链。 |
| `finish_reason` | string | `stop` / `length` / `tool_calls` / `content_filter`。 |

### tool_calls[]

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 工具调用 ID，回传时填入 `messages[].tool_call_id`。 |
| `type` | string | `function`。 |
| `function.name` | string | 函数名。 |
| `function.arguments` | string | 参数 JSON 字符串（未解析）。 |

### usage

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 总数。 |
| `completion_tokens` | integer | 输出 token 总数（含 `reasoning_content`）。 |
| `total_tokens` | integer | = prompt + completion。 |
| `cached_tokens` | integer | 缓存命中的输入 token 数（仅 K2.5 / K2.6）。 |

## 流式响应（SSE）

事件格式：`data: <json>\n\n`，以 `data: [DONE]\n\n` 结束。

| chunk 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 同非流式。 |
| `object` | string | `chat.completion.chunk`。 |
| `created` / `model` | — | 同非流式。 |
| `choices[0].index` | integer | 始终 `0`（单候选场景）。 |
| `choices[0].delta.role` | string | 仅首 chunk 出现，值 `assistant`。 |
| `choices[0].delta.content` | string | 文本增量。 |
| `choices[0].delta.reasoning_content` | string | 思维链增量（仅思考模式）。 |
| `choices[0].delta.tool_calls` | array | tool call 增量，逐字段拼接；元素含 `index`、`id`、`type`、`function.name`、`function.arguments`（增量片段）。 |
| `choices[0].finish_reason` | `null \| string` | 中间 chunk 为 `null`，结束 chunk 同非流式。 |
| `usage` | object | 仅在 `stream_options.include_usage=true` 且为倒数第二个 chunk 时出现。 |

## 最小请求示例

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "system", "content": "你是 Kimi，由 Moonshot AI 提供的人工智能助手。"},
    {"role": "user", "content": "用一句话解释 transformer。"}
  ],
  "max_completion_tokens": 256
}
```

## 最小响应示例

```json
{
  "id": "cmpl-3a4f...",
  "object": "chat.completion",
  "created": 1747756800,
  "model": "kimi-k2.6",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Transformer 是一种基于自注意力机制的序列建模架构。",
        "reasoning_content": "用户希望简洁解释…"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 38,
    "completion_tokens": 64,
    "total_tokens": 102,
    "cached_tokens": 0
  }
}
```

## 工具调用示例

```json
{
  "model": "kimi-k2.6",
  "messages": [{"role": "user", "content": "上海今天天气如何？"}],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "查询给定城市的天气",
        "parameters": {
          "type": "object",
          "properties": {"city": {"type": "string"}},
          "required": ["city"]
        }
      }
    }
  ]
}
```

## 关闭思考模式

```json
{
  "model": "kimi-k2.6",
  "messages": [{"role": "user", "content": "..."}],
  "thinking": {"type": "disabled"}
}
```

## 参考

- 端点页面：https://platform.kimi.com/docs/api/chat
- 工具调用：https://platform.kimi.com/docs/api/tool-use
- Partial Mode：https://platform.kimi.com/docs/api/partial
- 思考模式指南：https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model
- 从 OpenAI 迁移：https://platform.kimi.com/docs/guide/migrating-from-openai-to-kimi
