---
source: https://api-docs.deepseek.com/zh-cn/
fetched_at: 2026-05-26
api_version: N/A
---

# DeepSeek 使用指南汇总

整理官方左侧导航中「API 指南」类的关键点：思考模式、多轮对话、Function Calling、JSON Output、对话前缀续写（Beta）、Token 用量。与 [chat-completions.md](./chat-completions.md) 互补，本页只列**约束 / 注意事项**与实操要点，不重复参数表。

---

## 1. 思考模式（thinking）

| 维度 | 说明 |
| --- | --- |
| 支持模型 | `deepseek-v4-pro`、`deepseek-reasoner`（其余模型见官方[模型列表](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)） |
| 启用 | 请求体 `thinking.type = "enabled"`，OpenAI SDK 通过 `extra_body={"thinking":{"type":"enabled"}}` 透传 |
| 关闭 | `thinking.type = "disabled"` |
| 推理力度 | `reasoning_effort: "high" \| "max"`；普通调用默认 `high`；Agent 类客户端（如 Claude Code 模式）自动 `max` |
| 思维链字段 | 响应 `choices[].message.reasoning_content`，与 `content` 并列 |
| 失效采样参数 | 思考模式下 `temperature` / `top_p` / `presence_penalty` / `frequency_penalty` 全部**不生效**（设了不报错） |

### 多轮中的 reasoning_content

| 场景 | 是否需要回传 |
| --- | --- |
| 上一轮**未**触发 tool call | 不需要回传；API 忽略。 |
| 上一轮**触发了** tool call | **必须**把 `assistant` 消息（含 `reasoning_content` + `tool_calls`）原样回传，否则后续 `tool` 消息会 `400`。 |

---

## 2. 多轮对话

| 规则 | 说明 |
| --- | --- |
| 状态 | API **无状态**；每次请求必须把完整 `messages[]` 历史拼好发送。 |
| 顺序 | `[system?, user, assistant, user, assistant, ...]`。 |
| 工具消息 | 在 `assistant` 携带 `tool_calls` 之后必须紧跟 `role: "tool"` 消息（带 `tool_call_id`）才能继续 user 轮。 |
| Token 上限 | 超过模型上下文窗口的历史会被服务端拒绝，由客户端自行裁剪。 |

---

## 3. Function Calling（tool_calls）

### 请求 tools 元素

```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "获取实时天气",
    "parameters": {
      "type": "object",
      "properties": { "city": { "type": "string" } },
      "required": ["city"],
      "additionalProperties": false
    }
  }
}
```

### 响应 tool_calls 元素

```json
{
  "id": "call_abc",
  "type": "function",
  "function": { "name": "get_weather", "arguments": "{\"city\":\"Shanghai\"}" }
}
```

### 多轮回传

```json
[
  {"role":"user","content":"上海天气"},
  {"role":"assistant","content":null,"tool_calls":[{...}]},
  {"role":"tool","tool_call_id":"call_abc","content":"{\"temp\":21}"}
]
```

### 与思考模式

DeepSeek-V3.2 起，思考模式同时支持 tool_calls。但思考模式下采样参数（temperature 等）不生效，函数描述本身仍可指导工具选择。

### strict 模式

`function.parameters` 必须满足：所有字段在 `required` 中、`additionalProperties: false`，否则触发严格校验错误。`parallel_tool_calls` 字段 DeepSeek 文档未明示支持，建议保守串行。

---

## 4. JSON Output

| 字段 | 取值 | 说明 |
| --- | --- | --- |
| `response_format` | `{"type":"json_object"}` | 启用 JSON Output。 |

要求：

1. system 或 user prompt 中必须**包含字面量 `json`** 字符串，否则可能返回普通文本。
2. prompt 中需给出**期望的 JSON 结构示例**，否则模型可能生成不一致字段。
3. 设置足够大的 `max_tokens` 防止 JSON 被截断。
4. 偶尔会返回空 `content`，自行重试。

> DeepSeek 暂不支持 OpenAI 风格的 `response_format.type = "json_schema"` 严格 schema 模式。

---

## 5. 对话前缀续写（Chat Prefix Completion，Beta）

强制模型从开发者预设的「assistant 前缀」继续生成，适合代码补全、固定模板、JSON 续写等。

| 维度 | 说明 |
| --- | --- |
| Base URL | `https://api.deepseek.com/beta`（与正式接口域名不同） |
| 用法 | `messages[]` 最后一条必须 `role: "assistant"` + `prefix: true`；`content` 为强制前缀字符串 |
| 配合 `stop` | 常用 `stop` 终止符（如 ` ``` `）防止生成多余内容 |
| 与思考模式 | 不可与 `thinking.type: "enabled"` 同时使用 |

### 示例

```python
messages = [
  {"role": "user", "content": "Please write quick sort code"},
  {"role": "assistant", "content": "```python\n", "prefix": True}
]
```

---

## 6. Token 用量计算

### usage 字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入总 token。 |
| `completion_tokens` | integer | 输出 token（**含 `reasoning_tokens`**）。 |
| `total_tokens` | integer | 二者之和。 |
| `prompt_cache_hit_tokens` | integer | 命中硬盘缓存的输入 token。 |
| `prompt_cache_miss_tokens` | integer | 未命中输入 token。 |
| `reasoning_tokens` | integer | 思考模式产生的思维链 token。 |
| `prompt_tokens_details` | object | 输入细分（如 `cached_tokens`）。 |

### 经验换算

| 输入 | 大致 token |
| --- | --- |
| 1 个英文字符 | `≈ 0.3` |
| 1 个中文字符 | `≈ 0.6` |

> 实际计费以 API 响应 `usage` 字段为准。

### 离线 tokenizer

DeepSeek 提供 [`deepseek_tokenizer.zip`](https://api-docs.deepseek.com/zh-cn/quick_start/token_usage)，可在本地按相同算法预估 token，避免预扣费 / 限流前查询。

---

## 参考

- 思考模式：<https://api-docs.deepseek.com/zh-cn/guides/thinking_mode>
- 多轮对话：<https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat>
- Function Calling：<https://api-docs.deepseek.com/zh-cn/guides/tool_calls>
- JSON Output：<https://api-docs.deepseek.com/zh-cn/guides/json_mode>
- Chat Prefix Completion：<https://api-docs.deepseek.com/zh-cn/guides/chat_prefix_completion>
- Token 计算：<https://api-docs.deepseek.com/zh-cn/quick_start/token_usage>
