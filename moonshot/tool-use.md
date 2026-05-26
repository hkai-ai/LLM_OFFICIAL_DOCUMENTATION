---
source: https://platform.kimi.com/docs/api/tool-use
fetched_at: 2026-05-26
api_version: N/A（OpenAPI 1.0）
---

# Moonshot Kimi · 工具调用（Function Calling + Built-in Tools）

工具调用走 [chat-completions.md](./chat-completions.md) 的 `tools` / `tool_choice` 字段。Kimi 支持两类工具：**用户自定义 function** 与 **平台内置工具**（如 `$web_search`）。

## 鉴权

同 chat-completions：`Authorization: Bearer <API_KEY>`，Base `https://api.moonshot.cn/v1`。

## `tools[]` 结构

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "获取城市天气",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string"}
          },
          "required": ["city"]
        },
        "strict": true
      }
    },
    {
      "type": "builtin_function",
      "function": {"name": "$web_search"}
    }
  ]
}
```

### `function` 类型

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 固定 `function` |
| `function.name` | string | 必须匹配 `^[a-zA-Z_][a-zA-Z0-9-_]{2,63}$`（长度 3–64） |
| `function.description` | string | 自然语言描述 |
| `function.parameters` | object | JSON Schema（subset） |
| `function.strict` | boolean | 默认 `true`；`true` 时模型输出严格满足 `parameters` schema。`false` 时仅需合法 JSON，内部结构不约束。 |

### `builtin_function` 类型

| 字段 | 说明 |
| --- | --- |
| `type` | 固定 `builtin_function` |
| `function.name` | 内置工具名，目前公开的有 `$web_search`（详见 [web-search.md](./web-search.md)） |

> 内置工具与自定义 function 可在同一请求内混用。

## `tool_choice` 枚举

| 取值 | 说明 |
| --- | --- |
| `"auto"`（默认） | 由模型决定是否调用工具 |
| `"required"` | 强制本轮调用至少一个工具 |
| `"none"` | 禁止调用工具 |
| `{ "type": "function", "function": { "name": "<name>" } }` | 强制调用指定 function |

## 限制

- 单次请求**最多 128 个 tools**（自定义 + 内置合计）。
- `parameters` JSON Schema 须符合 **MFJS** 子集（无 `allOf` / `oneOf` / `anyOf` / 复杂 ref 嵌套）。

## 响应 `message.tool_calls`

```json
{
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "get_weather",
        "arguments": "{\"city\":\"北京\"}"
      }
    }
  ]
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 工具调用 ID，**后续 `messages` 中 `role=tool` 的 `tool_call_id` 必须一致** |
| `type` | string | `function` / `builtin_function` |
| `function.name` | string | 函数名 |
| `function.arguments` | string | **JSON 字符串**，不是 object，需 `JSON.parse()` |

## 后续轮回传

```json
{
  "role": "tool",
  "tool_call_id": "call_abc123",
  "content": "{\"temperature\":\"22°C\"}"
}
```

`content` 通常是 `JSON.stringify(result)`，但纯文本也可。

## 流式工具调用

`stream: true` 时，工具调用在 `delta.tool_calls` 中按 chunk 渐进下发。字段与非流式一致，但 `arguments` 会分多块拼接，需在 `finish_reason=tool_calls` 出现时聚合解析。

> 流式调用不需要 `tool_stream` 字段（与智谱不同），Kimi 默认在 `stream: true` 下就支持。

## 内置工具：`$web_search`

仅声明，无需手动实现：

```json
{
  "tools": [{ "type": "builtin_function", "function": { "name": "$web_search" } }]
}
```

模型决定触发后，会返回参数到 `tool_calls`；调用方**原样回传 arguments** 即可（Kimi 服务端会代为执行检索并把结果作为下一轮上下文）。每次触发计费 **¥0.03**。详见 [web-search.md](./web-search.md)。

## 示例：自定义 function

### 请求

```json
{
  "model": "kimi-k2.6",
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

### 模型回复

```json
{
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "tool_calls": [{
        "id": "call_abc123",
        "type": "function",
        "function": {"name": "get_weather", "arguments": "{\"city\":\"北京\"}"}
      }]
    },
    "finish_reason": "tool_calls"
  }]
}
```

### 客户端回传

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "user", "content": "北京天气如何？"},
    {"role": "assistant", "tool_calls": [...]},
    {"role": "tool", "tool_call_id": "call_abc123", "content": "{\"temperature\":\"22°C\"}"}
  ]
}
```

## 参考

- 工具调用 API：https://platform.kimi.com/docs/api/tool-use
- 工具调用 Guide：https://platform.kimi.com/docs/guide/use-kimi-api-to-complete-tool-calls
- Web Search 内置工具：https://platform.kimi.com/docs/guide/use-web-search → [web-search.md](./web-search.md)
- 官方工具集合：https://platform.kimi.com/docs/guide/use-official-tools
