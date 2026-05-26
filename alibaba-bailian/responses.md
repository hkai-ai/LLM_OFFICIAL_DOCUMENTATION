---
source: https://help.aliyun.com/zh/model-studio/compatibility-with-openai-responses-api
fetched_at: 2026-05-20
api_version: N/A
---

# 创建响应（OpenAI 兼容 Responses） · POST /compatible-mode/v1/responses

百炼对 OpenAI Responses API 的兼容实现。相对于 Chat Completions：

- 接受**字符串直入**或**消息数组**两种输入；
- 通过 `previous_response_id` 链接历史，无需客户端维护 `messages[]`；
- 内置工具：`web_search`、`code_interpreter`、`web_extract`（网页抓取）、`image_search` 等，无需自定义实现；
- 通过 `reasoning.effort` 控制思考深度档位：`none` / `minimal` / `low` / `medium` / `high`；
- 可加 `x-dashscope-session-cache: enable` header 自动缓存会话上下文。

## Base URL

| 地域 | URL |
| --- | --- |
| 华北 2（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1/responses` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1/responses` |
| 美西（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1/responses` |

## 关键字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 模型 ID。 |
| `input` | `string \| array<Message>` | ✓ | 字符串或 Chat 格式消息数组。 |
| `previous_response_id` | string | ✗ | 上一轮响应的 `id`，自动续接上下文；有效期 7 天。 |
| `instructions` | string | ✗ | 等价 system prompt。 |
| `tools` | array | ✗ | 自定义 function tool 或启用内置工具，例如 `{"type":"web_search"}` / `{"type":"code_interpreter"}` / `{"type":"web_extract"}`。 |
| `tool_choice` | `string \| object` | ✗ | 同 Chat Completions。 |
| `reasoning.effort` | string | ✗ | `none` / `minimal` / `low` / `medium` / `high`。 |
| `stream` | boolean | ✗ | 是否走 SSE。 |
| `max_output_tokens` | integer | ✗ | 输出 token 上限。 |
| `temperature` / `top_p` / `top_k` | number | ✗ | 采样参数，同 Chat Completions。 |
| `response_format` | object | ✗ | `text` / `json_object`，与 thinking 互斥。 |
| `metadata` | object | ✗ | 任意自定义键值，用于业务侧追踪。 |

### 内置工具

| `type` | 说明 |
| --- | --- |
| `web_search` | 联网搜索，按 strategy 计费。 |
| `code_interpreter` | 沙盒代码执行。 |
| `web_extract` | 给定 URL，抓取并解析正文。 |
| `image_search` | 图像检索。 |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 响应 ID；可作为下一轮 `previous_response_id`。 |
| `object` | string | `response`。 |
| `created` | integer | Unix 秒。 |
| `model` | string | 实际模型 ID。 |
| `output` | array | 多种事件块的有序数组；每个元素 `type` 取值如 `message` / `tool_call` / `web_search_result` 等。 |
| `output[].type` | string | 事件类型。 |
| `output[].content` | array | `{"type":"output_text","text":"..."}` 等。 |
| `usage.input_tokens` / `output_tokens` / `total_tokens` | integer | Token 统计。 |
| `usage.input_tokens_details.cached_tokens` | integer | 会话缓存命中。 |
| `usage.output_tokens_details.reasoning_tokens` | integer | 思考 token。 |
| `status` | string | `completed` / `incomplete` / `in_progress`。 |

## 会话缓存

请求加 header `x-dashscope-session-cache: enable`，结合 `previous_response_id` 使用；服务端自动缓存上下文，降低多轮延迟与费用。缓存命中量见 `usage.input_tokens_details.cached_tokens`。

## 最小请求示例

```json
{
  "model": "qwen3.6-plus",
  "input": "用一句话解释 tokenizer。"
}
```

## 启用内置 web_search

```json
{
  "model": "qwen3.6-plus",
  "input": "Anthropic 最近发布了什么模型？",
  "tools": [{"type": "web_search"}]
}
```

## 多轮续接

```json
{
  "model": "qwen3.6-plus",
  "input": "继续上面的话题。",
  "previous_response_id": "resp_abc123"
}
```

## 参考

- 端点：https://help.aliyun.com/zh/model-studio/compatibility-with-openai-responses-api
- 总览：https://help.aliyun.com/zh/model-studio/qwen-api-reference/
