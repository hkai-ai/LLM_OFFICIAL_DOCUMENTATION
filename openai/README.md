---
source: https://developers.openai.com/api/reference/overview
fetched_at: 2026-05-26
api_version: N/A
last_updated: 2026-05-26
---

> 2026-05-26 更新：补齐 moderations / fine-tuning / vector-stores / realtime / containers / conversations / evals / uploads / webhooks / skills / videos / chatkit / admin / legacy 共 14 篇端点文档。

# OpenAI API · 厂商概览

## 基础信息

| 项 | 值 |
| --- | --- |
| Base URL | `https://api.openai.com/v1` |
| 协议 | HTTPS |
| 请求体 | 默认 `application/json`；上传文件、音频、图片用 `multipart/form-data` |
| SSE 流式 | 所有支持 `stream: true` 的端点采用 Server-Sent Events |
| 官方文档站 | `https://developers.openai.com/api/reference`（HTML，可抓取）；`https://platform.openai.com/docs/api-reference`（CSR + 反爬，WebFetch 通常 403） |

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | 值为 `Bearer <OPENAI_API_KEY>`。API key 在 OpenAI Platform 控制台生成。 |
| `OpenAI-Organization` | ✗ | 当账户归属多个 organization 时显式指定计费 organization ID。 |
| `OpenAI-Project` | ✗ | 显式指定 project ID。新版 project-scoped key 默认带绑定，可省略；遗留 user key 在跨 project 调用时需要。 |
| `OpenAI-Beta` | ✗ | 启用尚处 Beta 的特性。历史值如 `assistants=v2`。 |
| `Content-Type` | ✓ | 通常 `application/json`，上传二进制改 `multipart/form-data`。 |

> 「API keys should be provided via HTTP Bearer authentication. If you belong to multiple organizations or access projects through a legacy user API key, pass a header to specify which organization and project to use.」

## 官方 SDK

| 语言 | 包名 | 仓库 |
| --- | --- | --- |
| Python | `openai` | github.com/openai/openai-python |
| JavaScript / TypeScript | `openai` | github.com/openai/openai-node |
| Go | `github.com/openai/openai-go` | github.com/openai/openai-go |
| Java | `com.openai:openai-java` | github.com/openai/openai-java |
| .NET | `OpenAI` | github.com/openai/openai-dotnet |
| Ruby | `openai` | github.com/openai/openai-ruby |
| PHP | 暂无官方包，社区维护为主 | — |

> 本仓库只整理 HTTP API 字段，SDK 调用语法不作为文档对象。

## 端点索引

下表覆盖 `developers.openai.com/api/reference` 一级分类（截至 fetched_at）。

### Responses（主推）

| 端点 | METHOD | PATH | 说明 |
| --- | --- | --- | --- |
| Create a model response | POST | `/v1/responses` | 新一代统一响应端点，详见 [responses.md](./responses.md)。 |
| Retrieve a model response | GET | `/v1/responses/{response_id}` | 拉取已存储的 response。 |
| Delete a model response | DELETE | `/v1/responses/{response_id}` | 删除已存储 response。 |
| List input items | GET | `/v1/responses/{response_id}/input_items` | 列出输入项。 |
| Cancel a response | POST | `/v1/responses/{response_id}/cancel` | 取消后台 / 长任务 response。 |
| Count tokens | POST | `/v1/responses/{response_id}/tokens` | 估算 token。 |
| Compact a response | POST | `/v1/responses/{response_id}/compact` | 压缩长上下文。 |

附属资源：Conversations（`/v1/conversations`）见 [conversations.md](./conversations.md)；Streaming events 与 Responses 文档合并。

### Chat Completions（仍支持）

| 端点 | METHOD | PATH | 说明 |
| --- | --- | --- | --- |
| Create chat completion | POST | `/v1/chat/completions` | 详见 [chat-completions.md](./chat-completions.md)。 |
| Retrieve | GET | `/v1/chat/completions/{completion_id}` | 仅当 `store: true` 才可拉取。 |
| Update | POST | `/v1/chat/completions/{completion_id}` | 更新已存 chat completion 的 metadata。 |
| Delete | DELETE | `/v1/chat/completions/{completion_id}` | — |
| List | GET | `/v1/chat/completions` | 列出已存 completions。 |
| List messages | GET | `/v1/chat/completions/{completion_id}/messages` | — |

### Embeddings

详见 [embeddings.md](./embeddings.md)：POST `/v1/embeddings`。

### Audio

详见 [audio.md](./audio.md)：

- POST `/v1/audio/speech`（TTS）
- POST `/v1/audio/transcriptions`（STT）
- POST `/v1/audio/translations`
- POST `/v1/audio/voices`
- `/v1/audio/voice_consents`（CRUD + list）

### Images

详见 [images.md](./images.md)：

- POST `/v1/images/generations`
- POST `/v1/images/edits`
- POST `/v1/images/variations`

### Files & Batches

详见 [files-and-batches.md](./files-and-batches.md)：

- Files：`/v1/files`（list/upload/retrieve/delete/content）
- Uploads：`/v1/uploads`（分片上传 create/parts/complete/cancel）
- Batches：`/v1/batches`（create/retrieve/list/cancel）

### Models

详见 [models.md](./models.md)：`/v1/models`、`/v1/models/{id}`（GET / DELETE）。

### Moderations

POST `/v1/moderations`：检查内容是否违反 usage policy。详见 [moderations.md](./moderations.md)。

### Fine-tuning

`/v1/fine_tuning/jobs`（CRUD + list/cancel/pause/resume/events）；checkpoints、checkpoint permissions、alpha graders 为子资源。详见 [fine-tuning.md](./fine-tuning.md)。

### Vector Stores

`/v1/vector_stores`（CRUD/search），子资源 files、file_batches。Responses 与 Assistants 的 `file_search` 工具依赖。详见 [vector-stores.md](./vector-stores.md)。

### Realtime

`/v1/realtime/*`：client_secrets、calls（accept/hangup/refer/reject）+ WebSocket 协议。详见 [realtime.md](./realtime.md)。

### Webhooks

server 端 webhook 接收 batch / fine-tuning / eval / background response / realtime call 等异步事件。详见 [webhooks.md](./webhooks.md)。

### Containers

`/v1/containers`（CRUD），子资源 files。Code Interpreter 的运行容器。详见 [containers.md](./containers.md)。

### Conversations

`/v1/conversations`：Responses 的服务端会话存储；items 子资源。详见 [conversations.md](./conversations.md)。

### Uploads

`/v1/uploads`：大文件分片上传协议（≤8 GB），配合 Files 使用。详见 [uploads.md](./uploads.md)。

### Evals

`/v1/evals`：评估任务 CRUD、runs、output items。详见 [evals.md](./evals.md)。

### Skills

`/v1/skills`：可上传 SKILL 包，模型在 Responses + code_interpreter 容器内按需加载。详见 [skills.md](./skills.md)。

### Videos

`/v1/videos`：Sora 2 视频生成（generations / extends / edits / remix / characters）。详见 [videos.md](./videos.md)。

### ChatKit（Beta）

`/v1/beta/chatkit/*`：sessions、threads（前端 SDK 配套）。详见 [chatkit.md](./chatkit.md)。

### Administration（admin API key 限定）

`/v1/organization/*`：audit_logs、admin API keys、usage 指标、invites、users、roles、groups、data retention、spend alerts、certificates、projects（含 nested users / service_accounts / api_keys / rate_limits / permissions / groups / roles）。详见 [admin.md](./admin.md)。

### Legacy

| 端点 | PATH | 说明 |
| --- | --- | --- |
| Completions（legacy） | POST `/v1/completions` | 仅 `gpt-3.5-turbo-instruct` 等少量模型 |
| Assistants v2 | `/v1/beta/assistants`、`/v1/beta/threads`、`/v1/beta/threads/{id}/runs`、`/v1/beta/threads/{id}/messages` | 官方建议迁移到 Responses + Conversations |
| Realtime Beta | `/v1/realtime/sessions`、`/v1/realtime/transcription_sessions` | 旧版 Realtime |

全部 legacy 子集详见 [legacy.md](./legacy.md)。

## Chat Completions vs Responses API

OpenAI 在 2024 年底推出 Responses API 作为新一代主推接口，与传统 Chat Completions 并存。Chat Completions 不会很快下线，但新特性（计算机操作、内置 web 搜索、文件搜索、推理摘要、后台模式、长上下文 compaction 等）优先在 Responses 上线。

| 维度 | `/v1/chat/completions` | `/v1/responses` |
| --- | --- | --- |
| 输入字段 | `messages` 数组（强制 role + content） | `input`：字符串或 item 数组（item 类型多样，可混合 message / function_call / reasoning / tool_call_output 等） |
| 多轮上下文 | 调用方维护历史，每次重传 `messages` | 可由服务端维护，`previous_response_id` 或 `conversation` 引用；可启用 `store: true` 持久化 |
| 推理摘要 | 不暴露 | `reasoning.summary` + `reasoning.effort`（reasoning 模型） |
| 内置工具 | 仅 `web_search_options` | `tools` 支持 `file_search` / `web_search` / `web_search_preview` / `code_interpreter` / `computer_use_preview` / `image_generation` / `mcp` / `local_shell` 等内置类型 |
| 输出 | `choices[].message.content` 字符串 / `tool_calls` | `output` item 数组（message / reasoning / function_call / file_search_call / web_search_call / code_interpreter_call / computer_call / image_generation_call / mcp_call 等），并提供便捷字段 `output_text` |
| 结构化输出 | `response_format`：text / json_object / json_schema | `text.format`：text / json_schema，含 `verbosity` |
| token 上限字段 | `max_completion_tokens`（`max_tokens` 已弃用，对 reasoning 模型无效） | `max_output_tokens` |
| 推理强度 | `reasoning_effort`、`verbosity` 顶层字段 | `reasoning.effort`、`text.format.verbosity` |
| 后台模式 | 不支持 | `background: true` |
| 计费缓存键 | `prompt_cache_key` | `prompt_cache_key` |
| 风控标识 | `safety_identifier` | `safety_identifier` |
| 上下文压缩 | 不支持 | `context_management.compaction` |

> 「Responses is the new aggregated endpoint. Chat Completions remains supported and is the right choice when you need a stateless drop-in for the openai-compatible ecosystem.」

## 参考

- API 总览：<https://developers.openai.com/api/reference/overview>
- 模型清单：<https://developers.openai.com/api/docs/models>
- 错误码指南：<https://developers.openai.com/api/docs/guides/error-codes>
- platform 站镜像（CSR，可能 403）：<https://platform.openai.com/docs/api-reference>
