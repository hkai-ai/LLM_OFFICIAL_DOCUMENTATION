---
source: https://developers.openai.com/api/reference/resources/conversations
fetched_at: 2026-05-26
api_version: v1
---

# Conversations · /v1/conversations

Responses API 的长期会话存储：把若干消息 / 工具调用 / 工具结果作为「item」串成 conversation，再在 Responses 创建时通过 `conversation` 字段引用即可自动续接历史，无需手动维护 `previous_response_id` 链。

| 资源 | 路径前缀 |
| --- | --- |
| Conversations | `/v1/conversations` |
| Items（会话中的 message / tool call 等） | `/v1/conversations/{conversation_id}/items` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 1. Conversation CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/conversations` |
| Retrieve | GET | `/v1/conversations/{conversation_id}` |
| Update | POST | `/v1/conversations/{conversation_id}` |
| Delete | DELETE | `/v1/conversations/{conversation_id}` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `items` | `array<ConversationItem>` | ✗ | 初始 items；省略表示空会话。 |
| `metadata` | object | ✗ | 至多 16 个 key-value，key ≤64 字符、value ≤512 字符。 |

### Conversation 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `conv_...`。 |
| `object` | string | 固定 `conversation`。 |
| `created_at` | integer | Unix 秒。 |
| `metadata` | object | 同请求。 |

### Update body

只允许 `metadata`；按 key 合并，传 `""` 删除该 key。

### Delete 响应

```json
{ "id": "conv_...", "object": "conversation.deleted", "deleted": true }
```

## 2. Items 子资源

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/conversations/{conversation_id}/items` |
| Retrieve | GET | `/v1/conversations/{conversation_id}/items/{item_id}` |
| List | GET | `/v1/conversations/{conversation_id}/items` |
| Delete | DELETE | `/v1/conversations/{conversation_id}/items/{item_id}` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `items` | `array<ConversationItem>` | ✓ | 一次追加多条；按数组顺序入库。 |

### ConversationItem 公共字段

每个 item 类型与 Responses API 的 `output` / `input` item 完全一致（共享同一套 schema），常见 `type`：

| `type` | 关键字段 |
| --- | --- |
| `message` | `id`, `role`（`user` / `assistant` / `system` / `developer` / `tool`）, `content[]`（`text` / `input_text` / `output_text` / `input_image` / `input_audio` / `output_audio` / `refusal` 等 part）, `status`（`in_progress` / `completed` / `incomplete`）, `phase`。 |
| `function_call` | `name`, `arguments`, `call_id`, `status`。 |
| `function_call_output` | `call_id`, `output`, `status`。 |
| `file_search_call` / `web_search_call` / `code_interpreter_call` / `computer_call` / `image_generation_call` / `mcp_call` | 各自工具相关字段。 |
| `reasoning` | `summary[]` 等。 |
| `image_generation_output` | 输出图字段。 |

### List query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` / `before` | string | — | 游标。 |
| `limit` | integer | `20` | `1`–`100`。 |
| `order` | enum | `desc` | — |
| `include` | `array<string>` | — | 例如 `"step_details.tool_calls[*].search_results"` 等增强字段。 |

### 响应

```json
{ "object": "list", "data": [ /* ConversationItem[] */ ], "first_id": "...", "last_id": "...", "has_more": true }
```

### Delete

```json
{ "id": "msg_...", "object": "conversation.item.deleted", "deleted": true }
```

## 在 Responses 中引用

```bash
curl https://api.openai.com/v1/responses \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4.1",
    "conversation": "conv_abc",
    "input": "继续上次的话题"
  }'
```

Responses 执行完毕后会**自动**把本轮新生成的 message / tool call / tool result 追加到该 conversation；不需再 `POST /items`。

## 与 previous_response_id 的关系

| 方式 | 适用场景 |
| --- | --- |
| `conversation` | 多客户端共享、需服务端持久化历史的场景。 |
| `previous_response_id` | 简单线性接续，无需共享。 |

二者**不可同时使用**。

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | items 数组中存在不合法 type / 引用不存在的 call_id |
| `404` | `not_found_error` | conversation / item 不存在 |
| `429` | `rate_limit_error` | 触发 Responses 限流 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/conversations>
- 指南：<https://developers.openai.com/api/docs/guides/conversation-state>
- Responses API：[responses.md](./responses.md)
