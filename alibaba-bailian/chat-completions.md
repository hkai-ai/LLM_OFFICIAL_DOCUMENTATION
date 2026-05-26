---
source: https://help.aliyun.com/zh/model-studio/qwen-api-via-openai-chat-completions
fetched_at: 2026-05-20
api_version: N/A
---

# 创建对话补全（OpenAI 兼容） · POST /compatible-mode/v1/chat/completions

OpenAI Chat Completions 协议；附加百炼特有字段：`enable_thinking` / `thinking_budget` / `enable_search` / `search_options` / `translation_options` / `cache_control` / 多模态像素控制等。

## 请求 Body 字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID；可为 Qwen 系列、`deepseek-*`、`kimi-*`、`glm-*` 等百炼托管 ID。 |
| `messages` | `array<Message>` | ✓ | — | 对话历史，详见 §messages。 |
| `stream` | boolean | ✗ | `false` | 是否走 SSE 流式；`qwen3.6` 思考模式、qwq、qvq、qwen3 开源等**仅支持** `true`。 |
| `stream_options` | object | ✗ | — | `{"include_usage": true}` 时最后 chunk 携带 `usage`。 |
| `temperature` | number | ✗ | 模型相关 | `[0, 2)`。 |
| `top_p` | number | ✗ | — | `(0, 1]`。 |
| `top_k` | integer | ✗ | — | 候选 token 数；`0` 表示禁用。 |
| `max_tokens` | integer | ✗ | 模型相关 | 输出最大 token 数（含 `reasoning_tokens`）。 |
| `repetition_penalty` | number | ✗ | — | 重复度惩罚；与 OpenAI `frequency_penalty` 不同语义，倍率乘法。 |
| `presence_penalty` | number | ✗ | `0` | `[-2.0, 2.0]`。 |
| `frequency_penalty` | number | ✗ | `0` | `[-2.0, 2.0]`。 |
| `n` | integer | ✗ | `1` | 候选数 `1`–`4`。 |
| `seed` | integer | ✗ | — | OpenAI 兼容模式取值范围 `[0, 2³¹-1]`；DashScope 原生为 `[0, 2⁶³-1]`。 |
| `stop` | `string \| array` | ✗ | — | 停止序列。 |
| `response_format` | object | ✗ | `{"type":"text"}` | `text` / `json_object`；与 `enable_thinking: true` 同用时不能选 `json_object`（会报错）。 |
| `tools` | array | ✗ | — | Function Calling 工具列表。 |
| `tool_choice` | `string \| object` | ✗ | `auto` | `none` / `auto` / `required` / `{"type":"function","function":{"name":"..."}}`。 |
| `parallel_tool_calls` | boolean | ✗ | — | 是否允许一次响应并行多个工具调用。 |
| `logprobs` | boolean | ✗ | `false` | 是否返回 token 概率。 |
| `top_logprobs` | integer | ✗ | — | 每个 token 返回的候选概率数量。 |
| `enable_thinking` | boolean | ✗ | 模型相关 | 显式开 / 关思考模式。 |
| `thinking_budget` | integer | ✗ | — | 思考过程最大 token 上限。 |
| `enable_search` | boolean | ✗ | `false` | 开启联网搜索；详见 §search_options。 |
| `search_options` | object | ✗ | — | 搜索策略，详见 §search_options。 |
| `translation_options` | object | ✗ | — | 翻译类模型的源语言 / 目标语言设置（仅 `qwen-mt-*`）。 |
| `cache_control` | object | ✗ | — | 显式缓存；与 Anthropic 风格一致：`{"type":"ephemeral","ttl":"5m"}`。命中后 `usage.prompt_tokens_details.cached_tokens` 显示命中数。 |
| `modalities` | `array<string>` | ✗ | `["text"]` | 输出模态，例如 `["text","audio"]`（Qwen-Omni）。 |
| `audio` | object | ✗ | — | 音频输出配置（voice / format），仅在 `modalities` 含 `"audio"` 时生效。 |
| `min_pixels` / `max_pixels` / `total_pixels` | integer | ✗ | — | 图像输入像素下 / 上界，影响视觉编码长度。 |
| `vl_high_resolution_images` | boolean | ✗ | `false` | 视觉模型高分辨率模式。 |
| `enable_code_interpreter` | boolean | ✗ | `false` | 启用内置代码解释器工具。 |

### messages[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `system` / `user` / `assistant` / `tool`。 |
| `content` | `string \| array<ContentPart>` | ✓ | 文本或多模态 part 数组。 |
| `name` | string | ✗ | 消息发送者名。 |
| `tool_calls` | array | assistant 发起 tool call 时由模型回传 | 多轮中需原样保留。 |
| `tool_call_id` | string | role=`tool` 时 ✓ | 对应上一轮 tool_calls 元素 ID。 |
| `reasoning_content` | string | ✗ | 多轮中保留上一轮思维链（仅部分模型要求）。 |

### messages[].content[]（多模态）

| `type` | 必填字段 | 说明 |
| --- | --- | --- |
| `text` | `text` (string) | 普通文本。 |
| `image_url` | `image_url.url` (string) | HTTP URL 或 `data:image/...;base64,...`；也支持 `fileid://...` 引用百炼平台文件。 |
| `input_audio` | `input_audio.data` (base64) + `input_audio.format` | Qwen-Omni 音频输入。 |
| `video` | `video` (`array<url>`) | 多张图像帧组成的视频。 |
| `video_url` | `video_url.url` (string) | 视频 URL；可附 `fps` / `min_pixels` / `max_pixels`。 |

### search_options

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `forced_search` | boolean | `false` | 强制必须执行搜索（即使模型判断无需）。 |
| `search_strategy` | string | `turbo` | `turbo` / `max` / `agent` / `agent_max`；不同档位影响搜索深度与计费。 |
| `enable_search_extension` | boolean | `false` | 启用搜索结果扩展（更多来源召回）。 |

> 联网搜索按 `search_strategy` 单独计费，详见模型广场。

## 响应 Body 字段（非流式）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 补全 ID，形如 `chatcmpl-...`。 |
| `object` | string | `chat.completion`。 |
| `created` | integer | Unix 秒。 |
| `model` | string | 实际使用的模型 ID。 |
| `choices` | array | 详见 §choices。 |
| `usage` | object | 详见 §usage。 |

### choices[]

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选序号。 |
| `message.role` | string | `assistant`。 |
| `message.content` | `string \| null` | 文本回复；纯 tool_call 时可为 `null`。 |
| `message.reasoning_content` | string | 思考模式下的思维链。 |
| `message.tool_calls` | array | tool call 列表，结构与 OpenAI 一致。 |
| `logprobs` | object | 当 `logprobs: true` 时返回。 |
| `finish_reason` | string | `stop` / `length` / `tool_calls` / `content_filter`。 |

### usage

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入 token 总数。 |
| `completion_tokens` | integer | 输出 token 总数（含 `reasoning_tokens`）。 |
| `total_tokens` | integer | = prompt + completion。 |
| `prompt_tokens_details.cached_tokens` | integer | 命中显式缓存的输入 token。 |
| `prompt_tokens_details.text_tokens` / `image_tokens` / `video_tokens` / `audio_tokens` | integer | 按模态拆分的输入 token。 |
| `completion_tokens_details.text_tokens` | integer | 输出中文本部分。 |
| `completion_tokens_details.audio_tokens` | `integer \| null` | Omni 模型音频输出 token。 |
| `completion_tokens_details.reasoning_tokens` | `integer \| null` | 思考链 token 数。 |
| `cache_creation.ephemeral_5m_input_tokens` | integer | 本次首次写入显式缓存的 token 数（仅 `cache_control` 触发时）。 |

## 流式响应（SSE）

`stream: true` 时返回 OpenAI 风格 SSE，每条 `data: <json>\n\n`，以 `data: [DONE]\n\n` 结束。

| chunk 字段 | 说明 |
| --- | --- |
| `id` / `object`(`chat.completion.chunk`) / `created` / `model` | 同非流式。 |
| `choices[0].delta.role` | 仅首 chunk 出现 `assistant`。 |
| `choices[0].delta.content` | 文本增量。 |
| `choices[0].delta.reasoning_content` | 思维链增量。 |
| `choices[0].delta.tool_calls[i]` | tool call 增量，含 `index` / `id` / `type` / `function.name` / `function.arguments`（分片字符串）。 |
| `choices[0].finish_reason` | 中间为 `null`，结束 chunk 为 `stop` 等。 |
| `usage` | 仅当 `stream_options.include_usage: true` 时在尾部 chunk 携带。 |

> Qwen3 思考模式、qwq、qvq、qwen3 开源等模型**仅支持流式**调用，非流式直接拒绝。

## 最小请求示例

```json
{
  "model": "qwen3.6-plus",
  "messages": [
    {"role": "system", "content": "你是通义千问，由阿里云开发。"},
    {"role": "user", "content": "用一句话解释 attention。"}
  ]
}
```

## 开启思考 + 限制思考预算

```json
{
  "model": "qwen3.6-max-preview",
  "messages": [{"role": "user", "content": "证明 sqrt(2) 是无理数。"}],
  "stream": true,
  "enable_thinking": true,
  "thinking_budget": 4096
}
```

## 开启联网搜索

```json
{
  "model": "qwen3.6-plus",
  "messages": [{"role": "user", "content": "今天上海的天气？"}],
  "enable_search": true,
  "search_options": {
    "forced_search": true,
    "search_strategy": "max",
    "enable_search_extension": true
  }
}
```

## 显式缓存

```json
{
  "model": "qwen3.6-plus",
  "messages": [
    {
      "role": "system",
      "content": "<长系统提示...>",
      "cache_control": {"type": "ephemeral", "ttl": "5m"}
    },
    {"role": "user", "content": "..."}
  ]
}
```

命中时响应 `usage.prompt_tokens_details.cached_tokens > 0`；首次写入时 `usage.cache_creation.ephemeral_5m_input_tokens` 反映写入量。

## 参考

- 端点：https://help.aliyun.com/zh/model-studio/qwen-api-via-openai-chat-completions
- 与 OpenAI 兼容总览：https://help.aliyun.com/zh/model-studio/compatibility-of-openai-with-dashscope
- Responses API：https://help.aliyun.com/zh/model-studio/compatibility-with-openai-responses-api
- 首次调用千问：https://help.aliyun.com/zh/model-studio/first-api-call-to-qwen
- 文本生成总览：https://help.aliyun.com/zh/model-studio/text-generation
