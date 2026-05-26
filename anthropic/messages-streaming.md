---
source: https://platform.claude.com/docs/en/api/messages-streaming
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# Messages 流式响应 · POST /v1/messages（`stream: true`）

> 在 `POST /v1/messages` 请求体中设置 `"stream": true`，接口以 Server-Sent Events（SSE，命名事件）形式增量返回 `Message` 对象的各组成部分。

## 鉴权与请求头

同 [messages.md](./messages.md)。响应 `Content-Type` 为 `text/event-stream`。

## SSE 线格式

每条事件由 SSE 命名事件块组成：

```
event: <event_name>
data: <json>

```

事件之间用空行分隔。`data` 中的 JSON 总是带有与 `event` 同名的 `type` 字段。版本 `2023-06-01` 起：

- 增量是「真正的增量」，例如 `" Hello"`, `" my"`, `" name"`（不再是累计字符串）。
- 所有事件都是命名事件，已移除 `data: [DONE]` 哨兵。

## 事件流顺序

一次完整的流式响应按下面的顺序：

1. `message_start`：返回 `Message` 框架（`content` 为空数组）。
2. 一段或多段 content block，每段依次：
   - `content_block_start`：声明这一段的 `index` 与 block 类型。
   - 若干 `content_block_delta`：增量更新这一 block。
   - `content_block_stop`：宣告此 block 结束。
3. 一或多个 `message_delta`：更新顶层 `Message`（如 `stop_reason`、累积 `usage`）。
4. 一个 `message_stop`：流结束。

`ping` 事件可在任意位置出现任意次。`error` 事件可在 200 之后的任何时刻出现。

> 警告：`message_delta.usage` 中的 token 计数是**累积值**，不要重复加和。

## 事件类型

### `message_start`

载荷的 `message` 字段是初始 `Message`（与非流式响应结构相同，但 `content` 为空）。

```sse
event: message_start
data: {"type": "message_start", "message": {"id": "msg_...", "type": "message", "role": "assistant", "content": [], "model": "claude-opus-4-7", "stop_reason": null, "stop_sequence": null, "usage": {"input_tokens": 25, "output_tokens": 1}}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"message_start"` | 固定值。 |
| `message` | object | 初始 `Message` 对象。 |

### `content_block_start`

宣告新 block 开始。`content_block` 与最终 block 同 `type`，但增量字段为空（text 的 `text: ""`、tool_use 的 `input: {}`、thinking 的 `thinking: ""` / `signature: ""`、server_tool_use 的 `input: {}` 等）。

```sse
event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"content_block_start"` | 固定值。 |
| `index` | integer | 本 block 在最终 `content[]` 中的下标。 |
| `content_block` | object | 此 block 的初始壳。 |

### `content_block_delta`

按 `delta.type` 更新指定 `index` 上的 block。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"content_block_delta"` | 固定值。 |
| `index` | integer | 对应的 block 下标。 |
| `delta` | object | 增量内容，见下方 [delta 子类型](#delta-子类型)。 |

### `content_block_stop`

```sse
event: content_block_stop
data: {"type": "content_block_stop", "index": 0}
```

### `message_delta`

更新顶层 `Message` 字段（典型字段：`stop_reason`、`stop_sequence`、累积 `usage`）。

```sse
event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn","stop_sequence":null},"usage":{"output_tokens":15}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"message_delta"` | 固定值。 |
| `delta` | object | 顶层字段的增量（`stop_reason`、`stop_sequence`）。 |
| `usage` | object | 累积 token 计数；可能包含 `input_tokens` / `output_tokens` / `cache_creation_input_tokens` / `cache_read_input_tokens` / `server_tool_use`。 |

### `message_stop`

```sse
event: message_stop
data: {"type": "message_stop"}
```

### `ping`

任意位置可能出现，无业务字段：

```sse
event: ping
data: {"type": "ping"}
```

### `error`

200 之后流中也可能出现错误事件，载荷与非流式错误一致：

```sse
event: error
data: {"type": "error", "error": {"type": "overloaded_error", "message": "Overloaded"}}
```

未知 `event` 类型可能随版本新增，客户端应优雅忽略。

## delta 子类型

`content_block_delta.delta.type` 的取值：

### `text_delta`

文本 block 的字符增量。

```sse
event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"ello frien"}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"text_delta"` | 固定值。 |
| `text` | string | 本次增量文本。 |

### `input_json_delta`

`tool_use` / `server_tool_use` block 的 `input` 字段增量，**是部分 JSON 串**（而最终 `input` 是 object）。需要累积所有 partial_json 后在 `content_block_stop` 时解析；也可用 Pydantic 等做部分 JSON 解析。

```sse
event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"input_json_delta","partial_json":"{\"location\": \"San Fra"}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"input_json_delta"` | 固定值。 |
| `partial_json` | string | 部分 JSON 字符串。 |

注：当前模型每次只发射一个完整 key+value，可能在多个 delta 之间存在停顿。

### `thinking_delta`

`thinking` block 的内容增量。仅在 `thinking.type` 为 `enabled` / `adaptive` 且 `display != "omitted"` 时出现。

```sse
event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"thinking_delta","thinking":"I need to find the GCD..."}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"thinking_delta"` | 固定值。 |
| `thinking` | string | 本次思考增量。 |

### `signature_delta`

紧跟在 `content_block_stop` 之前发出一次，用于校验整个 thinking block 的完整性。

```sse
event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"signature_delta","signature":"EqQB..."}}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"signature_delta"` | 固定值。 |
| `signature` | string | 签名字符串，需在多轮中回填到 thinking block。 |

> `display: "omitted"` 时，thinking block 仅会有 `content_block_start` → 一次 `signature_delta` → `content_block_stop`，不会有 `thinking_delta`。

### `citations_delta`

当响应启用 citations 时，引用信息以增量形式追加到对应 text block 上。文档列出该 delta 类型，具体字段参见 citations 专门文档。

## 完整示例

### 基本文本流

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_1nZdL29xx5MUA1yADyHTEsnR8uuvGzszyY","type":"message","role":"assistant","content":[],"model":"claude-opus-4-7","stop_reason":null,"stop_sequence":null,"usage":{"input_tokens":25,"output_tokens":1}}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: ping
data: {"type":"ping"}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"!"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn","stop_sequence":null},"usage":{"output_tokens":15}}

event: message_stop
data: {"type":"message_stop"}
```

### 工具调用流

包含一个 text block（前缀解释）+ 一个 tool_use block（参数以 `input_json_delta` 形式发送）：

```
event: content_block_start
data: {"type":"content_block_start","index":1,"content_block":{"type":"tool_use","id":"toolu_01T1x1fJ34qAmk2tNTrN7Up6","name":"get_weather","input":{}}}

event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"input_json_delta","partial_json":"{\"location\":"}}

event: content_block_delta
data: {"type":"content_block_delta","index":1,"delta":{"type":"input_json_delta","partial_json":" \"San Francisco, CA\"}"}}

event: content_block_stop
data: {"type":"content_block_stop","index":1}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"tool_use","stop_sequence":null},"usage":{"output_tokens":89}}

event: message_stop
data: {"type":"message_stop"}
```

> 提示：在自定义工具上设置 `eager_input_streaming: true` 可启用 fine-grained tool streaming，参数会更细粒度地分块到达。

### 扩展思考流

```
event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"thinking","thinking":"","signature":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"thinking_delta","thinking":"I need to find the GCD of 1071 and 462..."}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"signature_delta","signature":"EqQBCgIYAhIM..."}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}
```

随后会出现常规 text block 段落。

### 内置 web_search 流

`server_tool_use` block 的 `input` 也使用 `input_json_delta`；工具结果以独立 block（`type: "web_search_tool_result"`）发送，结构在 `content_block_start.content_block` 中给出，本身不再用 delta 发送。

## 错误恢复

- Claude 4.5 及更早：可以把已接收到的 partial 文本作为 assistant 消息塞回 `messages` 并重新发起请求继续。
- Claude 4.6 及更新：改为发一条 user 消息，告知模型先前响应被中断、要求从中断点继续，例如：
  > Your previous response was interrupted and ended with [previous_response]. Continue from where you left off.
- `tool_use` 与 `thinking` block 不能部分恢复；只能从最近一个完整 text block 重新续写。

## 参考

- 端点文档：`https://platform.claude.com/docs/en/api/messages-streaming`
- 上级目录：`https://platform.claude.com/docs/en/api/overview`
