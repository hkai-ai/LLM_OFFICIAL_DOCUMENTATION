---
source: https://platform.claude.com/docs/en/api/messages
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# Messages · POST /v1/messages

> 提交一组对话消息（支持文本、图像、文档、工具调用、扩展思考等内容块），模型返回下一条 assistant 消息。可用于单次 query，也可用于无状态多轮对话。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` | 二选一 | Console API Key。也可改用 `Authorization: Bearer <token>`。 |
| `anthropic-version` | ✓ | 例如 `2023-06-01`。 |
| `content-type` | ✓ | `application/json`。 |
| `anthropic-beta` | ✗ | 启用 beta 特性，多 beta 用英文逗号分隔。 |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID 或 alias，例如 `claude-opus-4-7`。可用值见 [models.md](./models.md)。 |
| `messages` | array | ✓ | — | 对话消息数组。详见 [`messages[]`](#messages)。单次请求最多 100,000 条。 |
| `max_tokens` | integer | ✓ | — | 生成的最大 token 数。模型可能在到达上限前提前停止。设为 `0` 表示仅填充 prompt 缓存而不生成。每个模型的允许上限不同。 |
| `system` | `string \| array<TextBlockParam>` | ✗ | — | system prompt。不是消息中的 role，而是顶层字段。数组形式可对单个 block 单独打 `cache_control`。 |
| `temperature` | number | ✗ | `1.0` | 采样随机度，范围 `[0.0, 1.0]`。即使 `0.0` 也并非完全确定性。 |
| `top_p` | number | ✗ | — | nucleus sampling。 |
| `top_k` | integer | ✗ | — | 仅从最可能的前 K 个 token 中采样。 |
| `stop_sequences` | array&lt;string&gt; | ✗ | — | 自定义停止序列。命中后 `stop_reason = "stop_sequence"`，`stop_sequence` 字段返回命中的串。 |
| `stream` | boolean | ✗ | `false` | 启用 SSE 增量流。事件格式见 [messages-streaming.md](./messages-streaming.md)。 |
| `metadata` | object | ✗ | — | 请求元数据。详见 [`metadata`](#metadata)。 |
| `tools` | array | ✗ | — | 可用工具定义（自定义 + 内置）。详见 [`tools[]`](#tools)。 |
| `tool_choice` | object | ✗ | `{type:"auto"}` | 工具调用策略。详见 [`tool_choice`](#tool_choice)。 |
| `thinking` | object | ✗ | — | 扩展思考配置。详见 [`thinking`](#thinking)。 |
| `service_tier` | string | ✗ | `"auto"` | 取 `"auto"` 或 `"standard_only"`。控制是否使用 Priority capacity。 |
| `cache_control` | object | ✗ | — | 顶层缓存控制：`{ "type": "ephemeral", "ttl": "5m" \| "1h" }`。自动在最末一个可缓存 block 上打缓存标记。 |
| `container` | string | ✗ | — | 用于跨请求复用容器的标识符（启用代码执行类内置工具时相关）。 |
| `mcp_servers` | array | ✗ | — | 远程 MCP server 配置（需 `mcp-client-*` beta header）。文档未在 messages 主页详细列出字段，参见专门页面。 |
| `betas` | array&lt;string&gt; | ✗ | — | 部分 SDK 在请求体里携带 beta 名（等价于 `anthropic-beta` header）。 |
| `inference_geo` | string | ✗ | — | 推理处理的地理区域，未指定则用 workspace 的 `default_inference_geo`。 |
| `output_config` | object | ✗ | — | 输出格式配置。包含 `format`（JSON schema）和 `effort`（取 `low` / `medium` / `high` / `xhigh` / `max`）。 |
| `context_management` | object | ✗ | — | 上下文管理策略（需对应 beta），文档未在 messages 主页详细展开。 |

### `messages[]`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✓ | — | 取 `"user"` 或 `"assistant"`。 |
| `content` | `string \| array<ContentBlockParam>` | ✓ | — | 字符串是单个 text block 的速记；数组则按 block 列出。 |

连续同 role 的消息会被合并。

### `messages[].content[]`

`content` 数组中每个元素都有 `type` 字段，按 `type` 区分以下结构。

#### `type: "text"`（TextBlockParam）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | `"text"` | ✓ | 固定值。 |
| `text` | string | ✓ | 文本内容。 |
| `cache_control` | object | ✗ | `{ "type": "ephemeral", "ttl": "5m" \| "1h" }`。 |
| `citations` | array | ✗ | 引用信息（如果该 text block 是 citations 的引用对象）。 |

#### `type: "image"`（ImageBlockParam）

`source` 支持两种：

```json
{ "type": "base64", "media_type": "image/jpeg" | "image/png" | "image/gif" | "image/webp", "data": "<base64>" }
```

```json
{ "type": "url", "url": "https://example.com/cat.jpg" }
```

可附 `cache_control`。

#### `type: "document"`（DocumentBlockParam）

`source` 支持四种：

```json
{ "type": "base64", "media_type": "application/pdf", "data": "<base64>" }
{ "type": "text",   "media_type": "text/plain",       "data": "<text>" }
{ "type": "url",    "url": "https://example.com/file.pdf" }
{ "type": "content","content": "<string or content blocks>" }
```

附加字段：`title`、`context`、`citations: { "enabled": true }`、`cache_control`。

#### `type: "tool_use"`（assistant 输出 / 也可作为 input 回填）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"tool_use"` | 固定值。 |
| `id` | string | 形如 `toolu_...`。 |
| `name` | string | 工具名。 |
| `input` | object | 工具入参 JSON。 |
| `cache_control` | object | 可选。 |
| `caller` | object | `{ "type": "direct" \| "code_execution_20250825" \| "code_execution_20260120" }`。 |

#### `type: "tool_result"`（user 角色回填）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"tool_result"` | 固定值。 |
| `tool_use_id` | string | 对应上一轮 `tool_use.id`。 |
| `content` | `string \| array` | 字符串或内容块数组（text / image / search_result / document / tool_reference）。 |
| `is_error` | boolean | 是否为错误结果，默认 `false`。 |
| `cache_control` | object | 可选。 |

#### `type: "thinking"`（assistant 输出，扩展思考）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"thinking"` | 固定值。 |
| `thinking` | string | 思考文本。 |
| `signature` | string | 完整性签名，用于在多轮中回填校验。 |

#### `type: "redacted_thinking"`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"redacted_thinking"` | 固定值。 |
| `data` | string | 经加密 / 脱敏的 thinking payload。 |

#### `type: "search_result"`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"search_result"` | 固定值。 |
| `title` | string | 结果标题。 |
| `source` | string | 来源标识（URL 或文档 id）。 |
| `content` | array&lt;TextBlockParam&gt; | 结果正文。 |
| `cache_control` | object | 可选。 |
| `citations` | object | `{ "enabled": boolean }`。 |

#### `type: "server_tool_use"`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"server_tool_use"` | 固定值。 |
| `id` | string | 形如 `srvtoolu_...`。 |
| `name` | string | 内置服务端工具名：`web_search` / `web_fetch` / `code_execution` / `bash_code_execution` / `text_editor_code_execution` / `tool_search_tool_regex` / `tool_search_tool_bm25`。 |
| `input` | object | 入参。 |
| `cache_control` | object | 可选。 |
| `caller` | object | 同 `tool_use.caller`。 |

#### `type: "container_upload"`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"container_upload"` | 固定值。 |
| `file_id` | string | 已通过 Files API 上传的文件 ID。 |
| `cache_control` | object | 可选。 |

### `metadata`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `user_id` | string | ✗ | 应用侧用户标识（UUID / hash / 不透明 ID）。用于 abuse detection，**不要**填入姓名、邮箱、电话等个人识别信息。 |

### `tools[]`

数组元素是 `ToolUnion`，按 `type` 区分自定义工具与内置工具。

#### 自定义工具

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 工具名。 |
| `description` | string | ✗ | 工具说明。 |
| `input_schema` | object | ✓ | JSON Schema（顶层 `type: "object"` + `properties` + `required`）。 |
| `cache_control` | object | ✗ | 可选。 |
| `strict` | boolean | ✗ | 是否启用严格 schema（部分模型支持）。 |
| `defer_loading` | boolean | ✗ | 是否延迟加载（用于 tool search）。 |
| `eager_input_streaming` | boolean | ✗ | 启用细粒度参数流式（fine-grained tool streaming）。 |

不带 `type` 字段的对象会被视为自定义工具。

#### 内置工具（部分）

| `type` | 内置工具 |
| --- | --- |
| `code_execution_20260120`、`code_execution_20250825` | Python 代码执行（`name: "code_execution"`） |
| `bash_20250124` | Bash 执行（`name: "bash"`） |
| `text_editor_20250728` | 文件编辑（`name: "str_replace_based_edit_tool"`，支持 `max_characters`） |
| `memory_20250818` | 记忆工具（`name: "memory"`） |
| `web_search_20250305` | 网络搜索（`name: "web_search"`，支持 `max_uses`） |
| `web_fetch_20250325` | 网页抓取（`name: "web_fetch"`） |
| `tool_search_tool_regex_*` / `tool_search_tool_bm25_*` | 工具检索 |

不同内置工具有各自附加字段，引入时请参考对应 capability 文档。

### `tool_choice`

| 字段 | 取值 | 说明 |
| --- | --- | --- |
| `type` | `"auto"` | 由模型决定是否调用工具（默认）。 |
| `type` | `"any"` | 必须调用任意一个工具。 |
| `type` | `"tool"` | 必须调用 `name` 指定的工具，需同时提供 `name` 字段。 |
| `type` | `"none"` | 禁止调用工具。 |
| `disable_parallel_tool_use` | boolean | 任意 `type` 均可附加。`true` 时单轮只能产生一个 `tool_use`。 |

### `thinking`

| `type` | 行为 |
| --- | --- |
| `"enabled"` | 启用扩展思考；必须填 `budget_tokens`（>= 1024，且必须 < `max_tokens`）。 |
| `"disabled"` | 关闭。 |
| `"adaptive"` | 自适应思考预算，由模型决定。 |

附加字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `budget_tokens` | integer | 用于内部推理的 token 预算，计入 `max_tokens`。仅 `enabled` 模式必填。 |
| `display` | string | `"summarized"`（默认）或 `"omitted"`。`omitted` 时流式不会发送 `thinking_delta`，只会有一个 `signature_delta`。 |

## 响应

### `Message` 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID，形如 `msg_...`。 |
| `type` | `"message"` | 固定值。 |
| `role` | `"assistant"` | 固定值。 |
| `content` | array | 内容块数组，元素类型同请求侧 `content[]`（但只会出现 assistant 侧合法的类型）。 |
| `model` | string | 实际使用的模型 ID。 |
| `stop_reason` | enum | 见下表。 |
| `stop_sequence` | `string \| null` | 命中的自定义 stop_sequence 值（如未命中为 `null`）。 |
| `usage` | object | token 使用情况，见 [`usage`](#usage)。 |
| `container` | object | 仅在使用容器类内置工具时返回，包含 container id 与到期时间（文档未在 messages 主页详细展开）。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_tokens` | integer | 输入 token。 |
| `output_tokens` | integer | 输出 token。 |
| `cache_creation_input_tokens` | integer | 本次请求写入 prompt 缓存的 token 数。 |
| `cache_read_input_tokens` | integer | 本次请求命中 prompt 缓存读取的 token 数。 |
| `server_tool_use` | object | 内置服务端工具使用次数，如 `{ "web_search_requests": 1 }`（文档未列举全部字段）。 |

### `stop_reason` 枚举

| 值 | 含义 |
| --- | --- |
| `"end_turn"` | 模型自然结束。 |
| `"max_tokens"` | 达到 `max_tokens` 上限。 |
| `"stop_sequence"` | 命中 `stop_sequences` 自定义停止串。 |
| `"tool_use"` | 模型决定调用工具，需要调用方回填 `tool_result`。 |

> 文档原文注明：枚举可能新增取值，客户端应优雅处理未知值。

## 示例

### 最小请求

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 1024,
  "messages": [
    { "role": "user", "content": "Hello, Claude" }
  ]
}
```

### 最小响应

```json
{
  "id": "msg_01A",
  "type": "message",
  "role": "assistant",
  "content": [
    { "type": "text", "text": "Hello! How can I help you today?" }
  ],
  "model": "claude-opus-4-7",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 10,
    "output_tokens": 12,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

### 工具调用请求

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 1024,
  "tools": [
    {
      "name": "get_weather",
      "description": "Get the current weather in a given location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": { "type": "string", "description": "City, State" }
        },
        "required": ["location"]
      }
    }
  ],
  "tool_choice": { "type": "auto" },
  "messages": [
    { "role": "user", "content": "What's the weather in San Francisco?" }
  ]
}
```

### 工具调用响应

```json
{
  "id": "msg_01B",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01T1x1fJ34qAmk2tNTrN7Up6",
      "name": "get_weather",
      "input": { "location": "San Francisco, CA" }
    }
  ],
  "model": "claude-opus-4-7",
  "stop_reason": "tool_use",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 472,
    "output_tokens": 89,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

### 扩展思考请求

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 20000,
  "thinking": { "type": "enabled", "budget_tokens": 10000, "display": "summarized" },
  "messages": [
    { "role": "user", "content": "What is the GCD of 1071 and 462?" }
  ]
}
```

## 错误码

详见 [errors.md](./errors.md)。常见与 messages 相关：

| HTTP | `error.type` | 触发场景 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 参数缺失、互斥参数冲突、对不支持 prefill 的模型预填 assistant 消息。 |
| `413` | `request_too_large` | 请求体超过 32 MB。 |
| `429` | `rate_limit_error` | 触发租户限流或加速限流。 |
| `529` | `overloaded_error` | API 整体过载（流式模式中也可能在 200 之后通过 `event: error` 抛出）。 |

## 注意事项

- **不支持 prefill**：Claude Mythos Preview、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 4.6 不支持在最后一条 assistant 消息上 prefill。需要结构化输出请改用 `output_config.format` 或 structured outputs。
- **长生成**：`max_tokens` 较大时建议改用 streaming 或 Message Batches API，避免空闲连接超时。
- **流式 200 后的错误**：开启 `stream: true` 后，HTTP 状态码可能是 200，但事件流中仍可能出现 `event: error`，调用方需在事件层做错误处理。
- **token 计数累积**：流式响应的 `message_delta.usage` 字段是累积值，不要再次累加。

## 参考

- 端点文档：`https://platform.claude.com/docs/en/api/messages`
- create 子页：`https://platform.claude.com/docs/en/api/messages/create`
- 模型清单：`https://platform.claude.com/docs/en/about-claude/models/overview`
