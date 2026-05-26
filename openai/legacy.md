---
source: https://developers.openai.com/api/reference/overview
fetched_at: 2026-05-26
api_version: v1
---

# Legacy APIs

OpenAI 标记为 **legacy** 但仍在文档中保留可调用的端点。新项目优先使用 Responses + Conversations（替代 Assistants v2）、Chat Completions（替代 Completions）以及新版 Realtime（替代 Realtime Beta）。

| 子集 | 状态 | 替代方案 |
| --- | --- | --- |
| Assistants v2（Beta） | 仍在维护 | Responses + Conversations + Vector Stores |
| Threads / Runs / Messages | 同上 | 同上 |
| Realtime Beta（sessions / transcription_sessions） | 老 endpoint | 新版 [realtime.md](./realtime.md)（`/v1/realtime/client_secrets`） |
| Completions (Legacy) | 老 endpoint | [chat-completions.md](./chat-completions.md) 或 [responses.md](./responses.md) |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `OpenAI-Beta` | Assistants 必填 | `assistants=v2`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

---

## 1. Assistants v2（Beta）

### 1.1 Assistants 资源

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/beta/assistants` |
| Retrieve | GET | `/v1/beta/assistants/{assistant_id}` |
| Update | POST | `/v1/beta/assistants/{assistant_id}` |
| Delete | DELETE | `/v1/beta/assistants/{assistant_id}` |
| List | GET | `/v1/beta/assistants` |

Create body 主要字段：`name` / `description` / `model` / `instructions` / `tools[]`（含 `code_interpreter` / `file_search` / `function`） / `tool_resources` / `metadata` / `temperature` / `top_p` / `response_format` 等。

### 1.2 Threads · `/v1/beta/threads`

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/beta/threads` |
| Create and run | POST | `/v1/beta/threads/{thread_id}/create_and_run` |
| Retrieve | GET | `/v1/beta/threads/{thread_id}` |
| Update | POST | `/v1/beta/threads/{thread_id}` |
| Delete | DELETE | `/v1/beta/threads/{thread_id}` |

### 1.3 Runs · `/v1/beta/threads/{thread_id}/runs`

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/beta/threads/{thread_id}/runs` |
| Retrieve | GET | `/v1/beta/threads/{thread_id}/runs/{run_id}` |
| Update | POST | `/v1/beta/threads/{thread_id}/runs/{run_id}` |
| List | GET | `/v1/beta/threads/{thread_id}/runs` |
| Cancel | POST | `/v1/beta/threads/{thread_id}/runs/{run_id}/cancel` |
| Submit tool outputs | POST | `/v1/beta/threads/{thread_id}/runs/{run_id}/submit_tool_outputs` |

#### Steps 子资源

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Retrieve | GET | `/v1/beta/threads/{thread_id}/runs/{run_id}/steps/{step_id}` |
| List | GET | `/v1/beta/threads/{thread_id}/runs/{run_id}/steps` |

### 1.4 Messages · `/v1/beta/threads/{thread_id}/messages`

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/beta/threads/{thread_id}/messages` |
| Retrieve | GET | `/v1/beta/threads/{thread_id}/messages/{message_id}` |
| Update | POST | `/v1/beta/threads/{thread_id}/messages/{message_id}` |
| Delete | DELETE | `/v1/beta/threads/{thread_id}/messages/{message_id}` |
| List | GET | `/v1/beta/threads/{thread_id}/messages` |

### 1.5 Run streaming events

Run 可通过 `stream: true` 返回 SSE 事件，事件类型包括：

| 事件 | 含义 |
| --- | --- |
| `thread.run.created` / `.queued` / `.in_progress` / `.requires_action` / `.completed` / `.failed` / `.cancelled` / `.expired` | Run 状态变化。 |
| `thread.run.step.created` / `.in_progress` / `.completed` / `.failed` / `.cancelled` / `.expired` | Step 状态变化。 |
| `thread.message.created` / `.in_progress` / `.delta` / `.completed` / `.incomplete` | Message 输出。 |
| `thread.run.step.delta` | Step 输出增量。 |
| `error` / `done` | 流终止信号。 |

> Assistants 体系**强烈建议迁移**到 Responses + Conversations，可获得新模型 / 新工具 / 更低延迟。

---

## 2. Realtime Beta · /v1/realtime

老路径 `/v1/realtime/sessions` 与 `/v1/realtime/transcription_sessions` 已被新版 `/v1/realtime/client_secrets`（[realtime.md](./realtime.md)）取代，但仍可调用。

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create session | POST | `/v1/realtime/sessions` |
| Create transcription session | POST | `/v1/realtime/transcription_sessions` |

### 请求 body

字段语义与新版 `client_secrets` 的 `session` 子字段一致，但响应返回的是 `RealtimeSession` 顶层对象（含 `client_secret.value` 与 `client_secret.expires_at`）。

> 新代码请直接用 `/v1/realtime/client_secrets`；事件协议本身仍兼容。

---

## 3. Completions (Legacy) · POST /v1/completions

文本补全的老接口，仅 `gpt-3.5-turbo-instruct` / `babbage-002` / `davinci-002` 等少数 base 模型仍可用。

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 仅支持上述少数模型。 |
| `prompt` | `string \| array<string> \| array<integer> \| array<array<integer>>` | ✓ | 支持单 / 多 prompt、字符串或 token ID。 |
| `max_tokens` | integer | ✗ | 默认 `16`。 |
| `temperature` | number | ✗ | `0`–`2`。 |
| `top_p` | number | ✗ | — |
| `n` | integer | ✗ | 候选数。 |
| `stream` | boolean | ✗ | SSE。 |
| `logprobs` | integer | ✗ | `0`–`5`。 |
| `echo` | boolean | ✗ | 是否回显 prompt。 |
| `stop` | `string \| array<string>` | ✗ | 停止序列。 |
| `presence_penalty` / `frequency_penalty` | number | ✗ | — |
| `best_of` | integer | ✗ | 服务器端候选采样。 |
| `logit_bias` | object | ✗ | token 偏置。 |
| `user` | string | ✗ | 终端用户标识。 |
| `suffix` | string | ✗ | （`gpt-3.5-turbo-instruct` 上 FIM 续写）。 |
| `seed` | integer | ✗ | 可复现。 |

### 响应

```json
{
  "id": "cmpl-...",
  "object": "text_completion",
  "created": 1716700000,
  "model": "gpt-3.5-turbo-instruct",
  "choices": [
    { "text": "...", "index": 0, "logprobs": null, "finish_reason": "stop" }
  ],
  "usage": { "prompt_tokens": 5, "completion_tokens": 7, "total_tokens": 12 }
}
```

> Chat 模型 ID（如 `gpt-4o`）**不支持** Completions 接口；请用 Chat Completions 或 Responses。

---

## 参考

- 总览：<https://developers.openai.com/api/reference/overview>
- Assistants：<https://developers.openai.com/api/reference/resources/beta/subresources/assistants>
- Threads：<https://developers.openai.com/api/reference/resources/beta/subresources/threads>
- Runs：<https://developers.openai.com/api/reference/resources/beta/subresources/threads/subresources/runs>
- Realtime Beta：<https://developers.openai.com/api/reference/resources/realtime/subresources/sessions>
- Completions：<https://developers.openai.com/api/reference/resources/completions>
- 迁移到 Responses 指南：<https://developers.openai.com/api/docs/guides/migrating-to-responses>
