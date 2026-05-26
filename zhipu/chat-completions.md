---
source: https://docs.bigmodel.cn/api-reference/模型-api/对话补全
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM Chat Completions · POST /paas/v4/chat/completions

> 与指定模型对话交互。同步 / 流式 / 异步三套调用方式共享同一字段集，仅 endpoint 与响应包装不同。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |

完整 URL：
- 同步 / 流式：`POST https://open.bigmodel.cn/api/paas/v4/chat/completions`
- 异步：`POST https://open.bigmodel.cn/api/paas/v4/async/chat/completions`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组，详见 `messages[]`。 |
| `stream` | boolean | ✗ | `false` | SSE 流式输出。 |
| `temperature` | number | ✗ | `1.0`（因系列而异） | 范围 `[0.0, 1.0]`，限两位小数。 |
| `top_p` | number | ✗ | `0.95`（因系列而异） | 范围 `[0.01, 1.0]`，限两位小数。**不建议与 `temperature` 同时调整**。 |
| `max_tokens` | integer | ✗ | — | 输出上限，范围 `[1, 131072]`。 |
| `do_sample` | boolean | ✗ | `true` | `false` 时启用贪心解码，`temperature` / `top_p` 被忽略。 |
| `tools` | array | ✗ | — | 工具列表，最多 128 个；类型支持 `function` / `retrieval` / `web_search` / `mcp`。 |
| `tool_choice` | string | ✗ | `auto` | **当前仅支持 `auto`**。 |
| `tool_stream` | boolean | ✗ | `false` | 流式返回工具调用。仅 GLM-5.1 / 5 / 5-Turbo / 4.7 / 4.6 支持。 |
| `stop` | array | ✗ | — | 停止词，最多 4 个。 |
| `response_format` | object | ✗ | `{type: "text"}` | 取值 `{type: "text"}` 或 `{type: "json_object"}`。 |
| `thinking` | object | ✗ | — | 思考模式开关，结构见下。GLM-4.5+ 支持。 |
| `request_id` | string | ✗ | 自动生成 | UUID 格式，6–64 字符。 |
| `user_id` | string | ✗ | — | 终端用户唯一标识，6–128 字符（敏感词风控关联）。 |

### `messages[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `system` / `user` / `assistant` / `tool` |
| `content` | string \| array | ✓ | 字符串或多模态内容块数组（见 `messages[].content[]`） |
| `tool_calls` | array | ✗ | `role=assistant` 时携带工具调用（结构同响应） |
| `tool_call_id` | string | 当 `role=tool` 时 ✓ | 关联的工具调用 ID |

### `messages[].content[]`（多模态）

| 块 type | 字段 | 说明 |
| --- | --- | --- |
| `text` | `text: string` | 纯文本 |
| `image_url` | `image_url.url: string` | 公网 URL 或 base64；单张 ≤ 5MB，分辨率 ≤ 6000×6000 |
| `video_url` | `video_url.url: string` | ≤ 200MB；旧 GLM-4V-Plus ≤ 20MB / ≤ 30s |
| `file_url` | `file_url.url: string` | 文件 URL |
| `input_audio` | `input_audio.data: string`（base64）+ `input_audio.format: "wav"\|"mp3"` | 仅 `glm-4-voice`，≤ 10 分钟，1 秒 ≈ 12.5 tokens（向上取整） |

### `thinking`

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `type` | string | `enabled` | `enabled` / `disabled` |
| `clear_thinking` | boolean | `true` | 是否清除历史思考内容（Preserved Thinking） |

> GLM-5.1 / 5 / 4.7 / 4.5v 强制思考；其他支持模型由 `type` 控制（auto-decide）。

### `tools[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | `function` / `retrieval` / `web_search` / `mcp` |
| `function.name` | string | 函数名 |
| `function.description` | string | 描述 |
| `function.parameters` | object | JSON Schema |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID。 |
| `request_id` | string | 请求标识。 |
| `created` | integer | Unix 时间戳。 |
| `model` | string | 实际模型 ID。 |
| `choices` | array | 候选结果数组。 |
| `usage` | object | token 使用统计。 |
| `content_filter` | array | 内容安全审核结果（`role` / `level`）。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 候选索引。 |
| `message.role` | string | 通常 `assistant`。 |
| `message.content` | string \| array | 输出文本或多模态。 |
| `message.reasoning_content` | string | 思考过程（思考模式开启时）。 |
| `message.tool_calls` | array | 工具调用，见下。 |
| `finish_reason` | string | `stop` / `length` / `tool_calls` / `sensitive` / `network_error` / `model_context_window_exceeded` |

### `message.tool_calls[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 工具调用 ID（后续轮 `tool_call_id` 必须一致）。 |
| `type` | string | `function` / `web_search` / `retrieval` |
| `function.name` | string | 函数名。 |
| `function.arguments` | string | **JSON 字符串**，不是 object。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | — |
| `completion_tokens` | integer | 含 `reasoning_content` token。 |
| `total_tokens` | integer | — |
| `prompt_tokens_details.cached_tokens` | integer | 缓存命中部分（见 §上下文缓存）。 |

### `web_search`（响应顶层数组，使用内置 `web_search` 工具时）

| 字段 | 说明 |
| --- | --- |
| `icon` | 网站图标 |
| `title` | 网页标题 |
| `link` | URL |
| `media` | 网站名 |
| `publish_date` | 发布时间 |
| `content` | 内容摘要 |
| `refer` | 角标序号 |

## 流式响应（SSE）

`stream: true` 时按 OpenAI 风格分块下发：

```json
{
  "id": "...",
  "created": 1704067200,
  "model": "glm-5.1",
  "choices": [
    {
      "index": 0,
      "delta": {
        "role": "assistant",
        "content": "增量文本",
        "reasoning_content": "思考增量",
        "tool_calls": [...]
      },
      "finish_reason": null
    }
  ],
  "usage": { /* 最后一块带 usage */ }
}
```

流末标志：`data: [DONE]`。

`tool_stream: true` 时，工具调用也按 chunk 渐进下发到 `delta.tool_calls`。

## 异步对话补全

`POST /paas/v4/async/chat/completions`，**立即返回任务 ID**，不在响应中包含最终结果：

```json
{
  "model": "glm-5.1",
  "id": "8a31e5f6-8f8b-4c3d-9e2a-7f1b3c5d7e9a",
  "request_id": "req-1234567890",
  "task_status": "PROCESSING"
}
```

`task_status`：`PROCESSING` / `SUCCESS` / `FAIL`。

### 查询异步结果

`GET https://open.bigmodel.cn/api/paas/v4/async-result/{id}`

任务完成后返回结构与同步响应一致，外层附 `task_status`。建议 `max_tokens ≥ 1024`，否则可能立即返回不完整结果。

## 上下文缓存（隐式）

- **默认自动开启**，无需 `cache_control` 字段。
- 命中体现在 `usage.prompt_tokens_details.cached_tokens`。
- 缓存命中价约为标准 input 价的 50%（具体折扣以 [pricing.md](./pricing.md) / 官方为准）。
- TTL 未公开；连续相同前缀的请求命中率最高。
- 文档未规定最小命中阈值。
- 详见 https://docs.bigmodel.cn/cn/guide/capabilities/cache。

## 示例

### 最小请求

```json
{
  "model": "glm-5.1",
  "messages": [
    {"role": "user", "content": "你好"}
  ]
}
```

### 思考模式 + 流式

```json
{
  "model": "glm-5.1",
  "messages": [{"role": "user", "content": "解决这个复杂问题..."}],
  "thinking": {"type": "enabled"},
  "stream": true
}
```

### 工具调用

```json
{
  "model": "glm-5.1",
  "messages": [{"role": "user", "content": "北京天气如何？"}],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "获取城市天气",
        "parameters": {
          "type": "object",
          "properties": {"city": {"type": "string"}},
          "required": ["city"]
        }
      }
    }
  ],
  "tool_choice": "auto"
}
```

### 视觉

```json
{
  "model": "glm-5v-turbo",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "image_url", "image_url": {"url": "https://..."}},
        {"type": "text", "text": "这是什么？"}
      ]
    }
  ]
}
```

### 最小响应

```json
{
  "id": "8884d9f8-7d55-4f5c-a524-94c8eb5f889a",
  "request_id": "req_12345",
  "created": 1704067200,
  "model": "glm-5.1",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "你好！我是智谱AI助手。"},
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30,
    "prompt_tokens_details": {"cached_tokens": 0}
  }
}
```

## 参考

- 同步对话：https://docs.bigmodel.cn/api-reference/模型-api/对话补全
- 异步对话：https://docs.bigmodel.cn/api-reference/模型-api/对话补全异步
- 查询异步结果：https://docs.bigmodel.cn/api-reference/模型-api/查询异步结果
- 工具调用：https://docs.bigmodel.cn/cn/guide/capabilities/function-calling
- 工具流式：https://docs.bigmodel.cn/cn/guide/capabilities/stream-tool
- 思考模式：https://docs.bigmodel.cn/cn/guide/capabilities/thinking
- 上下文缓存：https://docs.bigmodel.cn/cn/guide/capabilities/cache
- 结构化输出：https://docs.bigmodel.cn/cn/guide/capabilities/struct-output
- 流式：https://docs.bigmodel.cn/cn/guide/capabilities/streaming
