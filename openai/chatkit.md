---
source: https://developers.openai.com/api/reference/resources/beta/subresources/chatkit
fetched_at: 2026-05-26
api_version: v1（beta：`OpenAI-Beta: chatkit_beta=v1`）
---

# ChatKit · /v1/beta/chatkit（Beta）

把 OpenAI 工作流（Workflow）封装成可嵌入前端的对话组件：server 端发短期 `client_secret` → 浏览器 ChatKit SDK 直接打开 / 续接对话，自动支持文件上传、自动起标题、推理可视化等。

| 资源 | 路径前缀 |
| --- | --- |
| Sessions | `/v1/beta/chatkit/sessions` |
| Threads | `/v1/beta/chatkit/threads` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`（server 端使用）。 |
| `OpenAI-Beta` | ✓ | `chatkit_beta=v1`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 1. Sessions · /v1/beta/chatkit/sessions

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/beta/chatkit/sessions` |
| Cancel | POST | `/v1/beta/chatkit/sessions/{session_id}/cancel` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `user` | string | ✓ | 终端用户标识（≥1 字符）；用于审计与限流隔离。 |
| `workflow` | object | ✓ | 关联的 Agent Builder Workflow。 |
| `workflow.id` | string | ✓ | workflow ID。 |
| `workflow.version` | string | ✗ | 不指定走最新发布版本。 |
| `workflow.state_variables` | object | ✗ | 简单值的 KV，注入 workflow。 |
| `workflow.tracing` | object | ✗ | `{ "enabled": boolean }`。 |
| `chatkit_configuration` | object | ✗ | UI 行为开关，见下。 |
| `expires_after` | object | ✗ | `{ "anchor": "created_at", "seconds": 1-600 }`，默认较短。 |
| `rate_limits` | object | ✗ | `{ "max_requests_per_1_minute": int }`。 |

### chatkit_configuration

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `automatic_thread_titling.enabled` | boolean | `true` | 是否自动生成 thread 标题。 |
| `file_upload.enabled` | boolean | `false` | 是否允许用户上传文件。 |
| `file_upload.max_file_size` | integer | — | 单文件上限，单位 MB，`1`–`512`。 |
| `file_upload.max_files` | integer | — | 单 thread 内文件数上限。 |
| `history.enabled` | boolean | `false` | 是否在 UI 侧栏展示历史 thread。 |
| `history.recent_threads` | integer | — | 历史 thread 展示条数。 |

### ChatSession 对象（响应）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `cks_...`。 |
| `object` | string | 固定 `chatkit.session`。 |
| `client_secret` | string | **短期** token；前端 ChatKit SDK 用它鉴权。**不要长期存** —— 发完即转给前端。 |
| `expires_at` | integer | Unix 秒。 |
| `status` | enum | `active` / `expired` / `cancelled`。 |
| `user` | string | 同请求。 |
| `workflow` | object | 同请求（含解析后的版本）。 |
| `chatkit_configuration` | object | 同请求（含默认值合并）。 |
| `rate_limits` | object | 同请求。 |
| `created_at` | integer | — |

### Cancel · POST .../sessions/{session_id}/cancel

立即吊销 `client_secret`；body 为空。返回更新后的 ChatSession 对象（`status: cancelled`）。

## 2. Threads · /v1/beta/chatkit/threads

ChatKit 在对话期间自动产生 thread；server 端可读 / 删它们用作审计或自定义历史 UI。

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Retrieve | GET | `/v1/beta/chatkit/threads/{thread_id}` |
| Delete | DELETE | `/v1/beta/chatkit/threads/{thread_id}` |
| List | GET | `/v1/beta/chatkit/threads` |
| List items | GET | `/v1/beta/chatkit/threads/{thread_id}/items` |

### List query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `limit` | integer | `20` | `1`–`100`。 |
| `order` | enum | `desc` | — |
| `after` / `before` | string | — | 游标。 |
| `user` | string | — | 按终端用户过滤。 |
| `status` | enum | — | `active` / `locked` / `closed`。 |

### Thread 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `ckth_...`。 |
| `object` | string | 固定 `chatkit.thread`。 |
| `title` | string | 自动生成或用户改写。 |
| `status` | enum | `active`（默认）/ `locked` / `closed`。 |
| `user` | string | 终端用户。 |
| `workflow_id` | string | 关联的 workflow。 |
| `created_at` / `updated_at` | integer | Unix 秒。 |

### ThreadItem 对象（List items 返回）

每条 item 等价于 ChatKit UI 中的一条消息 / 工具调用 / 文件上传等：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `ckti_...`。 |
| `object` | string | 固定 `chatkit.thread.item`。 |
| `thread_id` | string | 所属 thread。 |
| `type` | enum | `user_message` / `assistant_message` / `widget` / `file_upload` / `workflow_step` / `notice` 等。 |
| `created_at` | integer | — |
| `content` | array | 与类型相关的内容载荷（文本 / widget / attachment 等）。 |

## 在前端使用

```html
<script type="module">
  import { ChatKit } from "https://cdn.openai.com/chatkit/index.js";
  ChatKit.mount({
    clientSecret: "<server 端 POST 拿到的 client_secret>",
    target: "#chat"
  });
</script>
```

短期 secret 过期后，server 端再发一份新的 session（同 `user`）以续接对话。

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | workflow.id 不存在 / `expires_after.seconds` 越界 |
| `404` | `not_found_error` | session / thread 不存在 |
| `401` | `authentication_error` | client_secret 已过期 |
| `429` | `rate_limit_error` | 触发 ChatKit 限流 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/beta/subresources/chatkit>
- 指南：<https://developers.openai.com/api/docs/guides/chatkit>
- 自定义集成：<https://developers.openai.com/api/docs/guides/custom-chatkit>
- Agent Builder Workflows：<https://platform.openai.com/agent-builder>
