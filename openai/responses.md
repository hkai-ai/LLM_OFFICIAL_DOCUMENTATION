---
source: https://developers.openai.com/api/reference/resources/responses/methods/create
fetched_at: 2026-05-19
api_version: N/A
---

# Responses API · POST /v1/responses

> 新一代主推端点。支持有状态对话、内置工具（web 搜索 / 文件搜索 / 计算机操作 / 代码解释器 / 图像生成 / MCP）、推理摘要、后台模式、上下文压缩等。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | `application/json` |
| `OpenAI-Organization` | ✗ | 多 organization 计费指定。 |
| `OpenAI-Project` | ✗ | project 隔离指定。 |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。见 [models.md](./models.md)。 |
| `input` | string \| array | ✓ | — | 字符串或 item 数组，详见 [`input[]`](#input)。 |
| `instructions` | string | ✗ | — | 系统级指令；和 `input` 中的 system/developer message 等效但不会被持久化进 conversation。 |
| `previous_response_id` | string | ✗ | — | 引用先前 response，使其历史作为上下文。需 `store: true`。 |
| `conversation` | string \| object | ✗ | — | 关联 conversation ID（`/v1/conversations`），由服务端维护多轮上下文。 |
| `tools` | array | ✗ | — | 函数 + 内置工具，详见 [`tools[]`](#tools)。 |
| `tool_choice` | string \| object | ✗ | `auto` | `none` / `auto` / `required` 或显式指定某工具：`{ "type": "file_search" }`、`{ "type": "function", "name": "..." }` 等。 |
| `include` | array&lt;string&gt; | ✗ | — | 让响应附带特定输出，常见枚举：`web_search_call.action.sources` / `file_search_call.results` / `message.input_image.image_url` / `reasoning.encrypted_content`。 |
| `max_output_tokens` | integer | ✗ | — | 可见输出 + 推理 token 上限。 |
| `max_tool_calls` | integer | ✗ | — | 一次 response 内允许的工具调用上限。 |
| `metadata` | object | ✗ | — | 最多 16 对 K/V，键 ≤ 64 字符，值 ≤ 512 字符。 |
| `parallel_tool_calls` | boolean | ✗ | `true` | 是否并行调用工具。 |
| `reasoning` | object | ✗ | — | 推理控制，见 [`reasoning`](#reasoning)。 |
| `temperature` | number | ✗ | `1` | `[0, 2]`。 |
| `top_p` | number | ✗ | `1` | nucleus sampling。 |
| `top_logprobs` | integer | ✗ | — | 每位置返回 top-N 候选（需要 include `message.output_text.logprobs`）。 |
| `truncation` | string | ✗ | `disabled` | `auto` / `disabled`。`auto` 时若超长则自动截断早期 message。 |
| `store` | boolean | ✗ | `true` | 是否持久化 response，支持后续 retrieve / 用作 `previous_response_id`。 |
| `stream` | boolean | ✗ | `false` | SSE 流式。 |
| `background` | boolean | ✗ | `false` | 后台异步执行；客户端可断开，稍后通过 retrieve 拉取。 |
| `text` | object | ✗ | — | 文本输出格式，见 [`text`](#text)。 |
| `user` | string | ✗ | — | 遗留字段，等同 `safety_identifier`。 |
| `prompt` | object | ✗ | — | Prompt template 引用：`{ id, version, variables }`。 |
| `prompt_cache_key` | string | ✗ | — | 提示缓存路由 key。 |
| `safety_identifier` | string | ✗ | — | 终端用户稳定哈希标识。 |
| `service_tier` | string | ✗ | `auto` | `auto` / `default` / `flex` / `priority`。 |
| `context_management` | array | ✗ | — | 上下文压缩策略，见 [`context_management`](#context_management)。 |

### `input[]`

`input` 可以是字符串（等价于一条 `user` message），也可以是 item 数组。常见 item 类型：

| `type` | 角色场景 | 关键字段 |
| --- | --- | --- |
| `message` | 任意 role | `role` (`system` / `developer` / `user` / `assistant`)、`content[]` |
| `function_call` | 调用方注入历史的工具调用 | `id`、`call_id`、`name`、`arguments` |
| `function_call_output` | 工具回填结果 | `call_id`、`output`（字符串） |
| `reasoning` | 注入或回填推理项 | `id`、`summary[]`、`encrypted_content` |
| `file_search_call` | 历史文件检索 | `id`、`queries`、`results` |
| `web_search_call` | 历史 web 搜索 | `id`、`action`（`{ type, query, sources }`） |
| `computer_call` | computer use 工具调用 | `call_id`、`action`、`pending_safety_checks` |
| `computer_call_output` | computer use 回填 | `call_id`、`output`（截图等） |
| `code_interpreter_call` | code interpreter 调用 | `id`、`code`、`results` |
| `image_generation_call` | image_generation 工具调用 | `id`、`result`（base64 图像） |
| `mcp_call` / `mcp_list_tools` / `mcp_approval_request` / `mcp_approval_response` | MCP 工具相关 | 见各 type 对应字段 |
| `local_shell_call` / `local_shell_call_output` | 本地 shell（host-side 工具） | — |

#### `message.content[]`（输入侧）

| `type` | 字段 | 说明 |
| --- | --- | --- |
| `input_text` | `text` | 文本片段。 |
| `input_image` | `image_url`、`file_id`、`detail` | URL/base64/已上传文件；`detail` 取 `low` / `high` / `auto` / `original`。 |
| `input_file` | `file_id` 或 `file_url` 或 `file_data` + `filename` | 引用文档。 |
| `input_audio` | `input_audio.data`、`input_audio.format` | base64 音频（限 audio-preview 模型）。 |

#### `message.content[]`（assistant 输出侧）

| `type` | 字段 | 说明 |
| --- | --- | --- |
| `output_text` | `text`、`annotations[]`、`logprobs` | 文本输出，annotations 含 `url_citation` / `file_citation` / `file_path` 等。 |
| `refusal` | `refusal` | 拒答。 |

### `tools[]`

`tools[]` 支持函数工具与多种内置工具。每个 item 至少含 `type`。

| `type` | 关键字段 | 说明 |
| --- | --- | --- |
| `function` | `name`、`description`、`parameters`（JSON Schema）、`strict` | 自定义函数。 |
| `file_search` | `vector_store_ids[]`、`max_num_results`、`filters`、`ranking_options` | 调用 Vector Stores 进行 RAG。 |
| `web_search_preview` / `web_search` | `user_location`、`search_context_size` | 内置 web 搜索（preview 与 GA 名共存，依赖模型）。 |
| `computer_use_preview` | `display_width`、`display_height`、`environment` (`browser` / `mac` / `windows` / `ubuntu`) | 计算机操作。 |
| `code_interpreter` | `container`（`auto` 或 `{ type: "auto", file_ids }`） | 代码执行沙箱。 |
| `image_generation` | `background`、`input_fidelity`、`model`、`moderation`、`output_compression`、`output_format`、`partial_images`、`quality`、`size` | 内置图像生成。 |
| `mcp` | `server_label`、`server_url` 或 `connector_id`、`allowed_tools`、`require_approval` (`always` / `never` / 细粒度对象)、`headers` | MCP 远端工具集。 |
| `local_shell` | — | host-side 本地 shell（需客户端实现）。 |
| `custom` | `name`、`description`、`format` | grammar-constrained 自定义工具（如 lark）。 |

### `reasoning`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `effort` | string | `minimal` / `low` / `medium` / `high`。 |
| `summary` | string | `auto` / `concise` / `detailed`。控制推理摘要风格。 |
| `generate_summary` | boolean | **已弃用**别名，等价于 `summary != null`。 |

### `text`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `format.type` | string | `text` / `json_schema`。 |
| `format.name` | string | 仅 `json_schema`：schema 名。 |
| `format.schema` | object | 仅 `json_schema`：JSON Schema。 |
| `format.strict` | boolean | 仅 `json_schema`：严格模式。 |
| `format.description` | string | 仅 `json_schema`：描述。 |
| `verbosity` | string | `low` / `medium` / `high`（仅 GPT-5 系列）。 |

### `context_management`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 固定 `compaction`。 |
| `compact_threshold` | integer | 输入 token 触发压缩的阈值。 |

## 响应

### 顶层对象（`object: "response"`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `resp_...` |
| `object` | string | 固定 `response`。 |
| `created_at` | integer | Unix 时间戳。 |
| `status` | string | `queued` / `in_progress` / `completed` / `incomplete` / `cancelled` / `failed`。 |
| `error` | object \| null | `{ code, message }`，失败时填充。 |
| `incomplete_details` | object \| null | `{ reason }`，常见 `max_output_tokens` / `content_filter`。 |
| `instructions` | string \| null | 回显请求字段。 |
| `max_output_tokens` | integer \| null | 回显。 |
| `model` | string | 实际模型 ID。 |
| `output` | array | 输出 item 数组，见 [`output[]`](#output)。 |
| `output_text` | string | 便利字段：所有 `message` item 中 `output_text` 拼接。 |
| `parallel_tool_calls` | boolean | 回显。 |
| `previous_response_id` | string \| null | 回显。 |
| `reasoning` | object \| null | 回显 `{ effort, summary }`。 |
| `store` | boolean | 回显。 |
| `temperature` | number | 回显。 |
| `text` | object | 回显 `{ format, verbosity }`。 |
| `tool_choice` | string \| object | 回显。 |
| `tools` | array | 回显。 |
| `top_p` | number | 回显。 |
| `truncation` | string | 回显。 |
| `usage` | object | 见 [`usage`](#usage)。 |
| `user` | string \| null | 回显。 |
| `metadata` | object | 回显。 |
| `background` | boolean | 回显。 |
| `conversation` | object \| null | `{ id }`，若关联了 conversation。 |
| `safety_identifier` | string \| null | 回显。 |
| `prompt_cache_key` | string \| null | 回显。 |
| `service_tier` | string | 实际命中层级。 |

### `output[]`

| `type` | 关键字段 | 说明 |
| --- | --- | --- |
| `message` | `id`、`role: "assistant"`、`status`、`content[]`（`output_text` / `refusal`） | 文本输出。 |
| `reasoning` | `id`、`summary[]`（含 `type: "summary_text"`、`text`）、`encrypted_content` | 推理摘要 / 加密推理。 |
| `function_call` | `id`、`call_id`、`name`、`arguments`、`status` | 自定义函数调用。 |
| `file_search_call` | `id`、`queries[]`、`results[]` | RAG 检索。 |
| `web_search_call` | `id`、`action`（`{ type: "search" \| "open_page", query, sources }`） | web 搜索。 |
| `code_interpreter_call` | `id`、`container_id`、`code`、`results[]`、`status` | 代码执行。 |
| `computer_call` | `id`、`call_id`、`action`、`pending_safety_checks[]`、`status` | 计算机操作。 |
| `image_generation_call` | `id`、`status`、`result`（base64）、`output_format` | 内置图像生成。 |
| `mcp_call` / `mcp_list_tools` / `mcp_approval_request` | 见对应 type 字段 | MCP 相关。 |
| `local_shell_call` | `id`、`call_id`、`action` | 本地 shell 调用。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_tokens` | integer | 输入 token 数。 |
| `output_tokens` | integer | 输出 token 数。 |
| `total_tokens` | integer | 合计。 |
| `input_tokens_details.cached_tokens` | integer | 命中提示缓存。 |
| `output_tokens_details.reasoning_tokens` | integer | 推理 token。 |

## 流式响应

`stream: true` 时返回多个事件，事件类型形如 `response.created` / `response.in_progress` / `response.output_item.added` / `response.content_part.added` / `response.output_text.delta` / `response.output_text.done` / `response.function_call_arguments.delta` / `response.reasoning_summary_text.delta` / `response.completed` / `response.failed` / `response.incomplete` / `response.error` 等。每个事件结构：

```
event: response.output_text.delta
data: { "type": "response.output_text.delta", "response_id": "resp_...", "item_id": "msg_...", "output_index": 0, "content_index": 0, "delta": "Hello" }
```

## 后台模式

`background: true` 直接返回 `status: "queued"` 的 response 对象。客户端可：

1. 轮询 `GET /v1/responses/{id}` 查看 `status`；
2. 通过 `POST /v1/responses/{id}/cancel` 取消；
3. 通过 webhook（`response.completed` / `response.failed`）异步通知。

## 示例

### 最小请求

```json
{
  "model": "gpt-5",
  "input": "Summarize the OpenAI Responses API in one sentence."
}
```

### 最小响应

```json
{
  "id": "resp_abc",
  "object": "response",
  "created_at": 1730000000,
  "status": "completed",
  "model": "gpt-5-2026-01-15",
  "output": [
    {
      "id": "msg_1",
      "type": "message",
      "role": "assistant",
      "status": "completed",
      "content": [
        { "type": "output_text", "text": "Responses is OpenAI's unified API ...", "annotations": [] }
      ]
    }
  ],
  "output_text": "Responses is OpenAI's unified API ...",
  "usage": {
    "input_tokens": 18,
    "output_tokens": 24,
    "total_tokens": 42,
    "input_tokens_details": { "cached_tokens": 0 },
    "output_tokens_details": { "reasoning_tokens": 0 }
  }
}
```

### 带内置工具

```json
{
  "model": "gpt-5",
  "input": "What did OpenAI announce yesterday?",
  "tools": [{ "type": "web_search" }],
  "include": ["web_search_call.action.sources"]
}
```

## 错误码

| HTTP | error.code | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 参数错误。 |
| `401` | `authentication_error` | API key 无效。 |
| `403` | `permission_error` | 区域 / 资源不允许。 |
| `404` | `not_found_error` | model 或 response_id 不存在。 |
| `429` | `rate_limit_error` / `insufficient_quota` | 限流 / 余额耗尽。 |
| `500` | `server_error` | 服务端异常。 |
| `503` | `engine_overloaded` | 过载。 |

## 参考

- 端点文档：<https://developers.openai.com/api/reference/resources/responses/methods/create>
- Streaming events：<https://developers.openai.com/api/reference/resources/responses/streaming>
- Conversations：<https://developers.openai.com/api/reference/resources/conversations>
