---
source: https://help.aliyun.com/zh/model-studio/qwen-api-via-dashscope
fetched_at: 2026-05-20
api_version: N/A
---

# DashScope 原生协议 · 文本与多模态生成

DashScope 是百炼的原生 API 协议，功能与参数最完整。请求 / 响应结构与 OpenAI 不一致，需要单独适配。

## Base URL 与路径

| 模态 | 完整路径（北京） |
| --- | --- |
| 纯文本 | `POST https://dashscope.aliyuncs.com/api/v1/services/aigc/text-generation/generation` |
| 多模态（图 / 视频 / 音） | `POST https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation` |

其他地域将 `dashscope` 替换为 `dashscope-intl` / `dashscope-us` 即可。

## 鉴权 Header

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DASHSCOPE_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |
| `X-DashScope-SSE` | 流式 ✓ | `enable` 启用 SSE；不传则走非流式。 |
| `X-DashScope-WorkSpace` | 多业务空间 ✓ | 业务空间 ID。 |

## 请求 Body 结构

```
{
  "model": "qwen3.6-plus",
  "input": {"messages": [...]},
  "parameters": {...}
}
```

| 顶层字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 模型 ID。 |
| `input.messages` | `array<Message>` | ✓ | 对话历史。 |
| `input.prompt` | string | ✗ | 与 `messages` 二选一；老式直入 prompt。 |
| `parameters` | object | ✗ | 详见 §parameters。 |

### input.messages[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `system` / `user` / `assistant` / `tool`。 |
| `content` | `string \| array<ContentPart>` | ✓ | 文本或多模态 part；多模态 part 形如 `{"text":"..."}` / `{"image":"https://..."}` / `{"video":["...","..."]}` / `{"audio":"data:audio/..."}`。 |
| `tool_calls` | array | ✗ | 模型回传的工具调用。 |
| `tool_call_id` | string | role=`tool` 时 ✓ | 对应上一轮 tool_calls。 |

> 多模态字段 key 名与 OpenAI 兼容模式（`image_url` / `input_audio` / `video_url`）**不同**。

### parameters

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `result_format` | string | `text` | `text` 直接返回字符串到 `output.text`；`message` 返回 OpenAI 风格 `output.choices[].message`。生产环境建议 `message`。 |
| `incremental_output` | boolean | `false` | SSE 时是否仅返回增量；`false` 时每个 chunk 携带累积输出。 |
| `temperature` | number | 模型相关 | `[0, 2)`。 |
| `top_p` | number | — | `(0, 1]`。 |
| `top_k` | integer | — | 候选数，`0` 禁用。 |
| `repetition_penalty` | number | — | 重复度惩罚（乘法）。 |
| `presence_penalty` | number | `0` | `[-2.0, 2.0]`。 |
| `max_tokens` | integer | 模型相关 | 输出最大 token 数。 |
| `seed` | integer | — | 范围 `[0, 2⁶³-1]`，更宽于 OpenAI 兼容模式。 |
| `stop` | `string \| array<string> \| array<int>` | — | 停止序列；DashScope 允许 token id 数组。 |
| `n` | integer | `1` | 候选数。 |
| `enable_thinking` | boolean | 模型相关 | 思考模式开关。 |
| `thinking_budget` | integer | — | 思考最大 token 数。 |
| `enable_search` | boolean | `false` | 联网搜索。 |
| `search_options` | object | — | `forced_search` / `search_strategy` / `enable_search_extension`，含义同 OpenAI 兼容模式。 |
| `tools` | array | — | Function Calling 工具列表（JSON Schema）。 |
| `tool_choice` | `string \| object` | `auto` | 同 OpenAI 兼容。 |
| `parallel_tool_calls` | boolean | — | 是否允许并行工具调用。 |
| `response_format` | object | — | `{"type":"json_object"}` 等；与 `enable_thinking: true` 互斥。 |
| `logprobs` | boolean | `false` | 是否返回 token 概率。 |
| `top_logprobs` | integer | — | 概率候选数。 |
| `translation_options` | object | — | 翻译模型源 / 目标语言。 |
| `vl_high_resolution_images` | boolean | `false` | 视觉模型高分辨率模式。 |
| `min_pixels` / `max_pixels` / `total_pixels` | integer | — | 视觉输入像素控制。 |

## 响应 Body 结构

```
{
  "output": {...},
  "usage": {...},
  "request_id": "..."
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `output.text` | string | 仅在 `result_format=text` 时出现，即模型回复文本。 |
| `output.finish_reason` | string | 仅 `result_format=text` 时；`stop` / `length` / `tool_calls`。 |
| `output.choices` | array | 仅 `result_format=message` 时；元素含 `message` 与 `finish_reason`。 |
| `output.choices[].message.role` | string | `assistant`。 |
| `output.choices[].message.content` | `string \| array` | 同上。 |
| `output.choices[].message.reasoning_content` | string | 思考模式输出。 |
| `output.choices[].message.tool_calls` | array | tool call 列表。 |
| `output.choices[].finish_reason` | string | 同 OpenAI 兼容模式。 |
| `usage.input_tokens` | integer | 输入 token 总数。 |
| `usage.output_tokens` | integer | 输出 token 总数（含 reasoning）。 |
| `usage.total_tokens` | integer | = input + output。 |
| `usage.prompt_tokens_details.cached_tokens` | integer | 缓存命中量。 |
| `usage.output_tokens_details.reasoning_tokens` | integer | 思考 token 数。 |
| `request_id` | string | 用于售后追溯。 |

> 注意 DashScope 用 `input_tokens` / `output_tokens`，OpenAI 兼容模式用 `prompt_tokens` / `completion_tokens`，字段名不同；统计接入时务必区分。

## SSE 流式

启用方式：

- 请求 header 加 `X-DashScope-SSE: enable`；
- 同时建议设置 `parameters.incremental_output: true` 走增量模式（默认 `false` 是累积模式）。

事件格式与 OpenAI 不同，每个事件包含完整 JSON 字段（不是 OpenAI 的 `chunk.delta`），形如：

```
id:1
event:result
:HTTP_STATUS/200
data:{"output":{"choices":[{"message":{"role":"assistant","content":"...增量..."},"finish_reason":"null"}]},"usage":{...},"request_id":"..."}
```

`Qwen3 商业版（思考模式）` / `Qwen3 开源版` / `QwQ` / `QVQ` 等仅支持流式，必须设置 SSE header。

## 最小请求示例

```json
{
  "model": "qwen3.6-plus",
  "input": {
    "messages": [
      {"role": "system", "content": "你是通义千问。"},
      {"role": "user", "content": "你好"}
    ]
  },
  "parameters": {"result_format": "message"}
}
```

## 多模态请求示例

```json
{
  "model": "qwen3.5-omni-plus",
  "input": {
    "messages": [
      {
        "role": "user",
        "content": [
          {"text": "描述这张图。"},
          {"image": "https://example.com/photo.jpg"}
        ]
      }
    ]
  },
  "parameters": {"result_format": "message"}
}
```

## 参考

- 端点：https://help.aliyun.com/zh/model-studio/qwen-api-via-dashscope
- 多模态生成总览：https://help.aliyun.com/zh/model-studio/multimodal-generation
- 文本生成总览：https://help.aliyun.com/zh/model-studio/text-generation
- 与 OpenAI 兼容对照：https://help.aliyun.com/zh/model-studio/compatibility-of-openai-with-dashscope
