---
source: https://www.volcengine.com/docs/82379/1526787
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 应用(bot) API · POST /api/v3/bots/chat/completions

> 调用在方舟控制台创建的应用 / 智能体（Bot），可自带联网内容插件、私域知识库插件、群聊角色等能力。请求 / 响应在 [Chat API](./chat-completions.md) 基础上扩展。

完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/bots/chat/completions`

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

## 与 Chat API 的差异

- `model`：填**应用 ID（Bot ID）**，而非 Model ID。
- 新增 `metadata` 字段（群聊配置、联网地点信息等）。
- 响应 `usage` 改为 `bot_usage`（按各 Endpoint 拆分用量），并可包含联网 / 知识库插件的引用信息。

## 请求参数（差异与扩展）

通用字段 `messages`、`thinking`、`stream`、`stream_options`、`max_tokens`、`stop`、`frequency_penalty`、`presence_penalty`、`temperature`、`top_p`、`logprobs`、`top_logprobs`、`logit_bias`、`tools`（仅 `function`）与 [Chat API](./chat-completions.md) 一致。`model` 填 Bot ID。新增：

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `metadata` | object \| null | ✗ | `null` | 额外参数，见下表。 |

### `metadata`

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `group_chat_config` | object | — | 运行时动态群聊配置。 |
| `group_chat_config.characters` | object[] | — | 群聊角色列表，每项含 `name`、`system_prompt`、`model_desc.endpoint_id`。 |
| `group_chat_config.description` | string \| null | `null` | 群聊场景描述（主题、时间地点、用户角色等）。 |
| `group_chat_config.user_name` | string \| null | `用户` | “我”所扮演的角色名称。 |
| `user_info` | string \| null | `null` | 联网智能体使用（如天气场景）。须为可反序列化的 JSON 字符串，含 `city` 与 `district` 字段。 |
| `emit_intention_signal_extra` | string \| null | `"false"` | `"true"` 时中途返回 intention 状态「正在搜索」。 |
| `target_character_name` | string \| null | `null` | 群聊 Bot 对话时指定本次发言角色名（须存在于 `characters`）。 |

## 响应

顶层 `id` / `model` / `created` / `object`（`chat.completion`）/ `choices` 与 [Chat API 响应](./chat-completions.md#响应) 一致（`choices[].message` 含 `content` / `reasoning_content` / `tool_calls`，`choices[].logprobs`、`choices[].moderation_hit_type` 同 Chat API）。token 用量字段不同：

### `bot_usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `model_usage` | object[] | 各 Endpoint 的 token 消耗。 |
| `model_usage[].name` | string | 调用的模型 ID。 |
| `model_usage[].prompt_tokens` | integer | 输入 token（含用户提示词与插件返回信息）。 |
| `model_usage[].completion_tokens` | integer | 输出 token。 |
| `model_usage[].total_tokens` | integer | 该 Endpoint 总 token。 |

> Bot 响应还可包含联网内容插件、私域知识库插件的引用 / 检索结果数据结构，详见下方参考链接。

## 参考

- 应用(bot) API：https://www.volcengine.com/docs/82379/1526787
- 获取 Bot ID：https://www.volcengine.com/docs/82379/1526787
- 联网插件数据结构：https://www.volcengine.com/docs/82379/1285209
- 知识库插件数据结构：https://www.volcengine.com/docs/82379/1285210
