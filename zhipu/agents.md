---
source: https://docs.bigmodel.cn/api-reference/agent-api/%E6%99%BA%E8%83%BD%E4%BD%93%E5%AF%B9%E8%AF%9D
fetched_at: 2026-05-26
api_version: v1（注：路径在 `/api/v1/agents` 下，不在 `/paas/v4` 下）
---

# Agent API · /api/v1/agents

智谱平台预置的智能体（翻译 / 画图 / 幻灯片 / 知识助手等）的统一对话入口。**不要与已停用的旧版「智能体 / 助理 API」混淆**。

| 资源 | 路径 |
| --- | --- |
| 同步 / 流式对话 | `POST /api/v1/agents` |
| 异步结果 | `POST /api/v1/agents/async-result`（详见官方异步任务文档） |
| 对话历史 | `POST /api/v1/agents/messages`（按 `conversation_id` 拉取，详见官方） |

> Base URL：`https://open.bigmodel.cn`。Coding 子集另有 `https://open.bigmodel.cn/api/coding/v1/agents`，路径相同 schema 不变。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ZHIPU_API_KEY`（或基于 API Key 自签 JWT，详见 [README.md](./README.md) §鉴权）。 |
| `Content-Type` | ✓ | `application/json`。 |

## 1. 智能体对话 · POST /api/v1/agents

### 请求 body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `agent_id` | string | ✓ | — | 智能体 ID，例如 `general_translation`、`slides_glm_agent`、`ai_drawing_agent`、`knowledge_qa_agent` 等。 |
| `messages` | `array<Message>` | ✓ | — | 对话历史；首条通常是 `user`。 |
| `stream` | boolean | ✗ | `false` | SSE 流式。 |
| `custom_variables` | object | ✗ | — | 该 agent 专属参数；不同 agent_id 字段不同（如翻译 agent 有 `source_lang` / `target_lang` / `strategy` / `glossary`）。 |
| `request_id` | string | ✗ | — | 客户端自定义；用于排查链路。 |

### messages[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | enum | ✓ | `system` / `user` / `assistant`。 |
| `content` | `string \| object \| array` | ✓ | 纯文本、单 `{file_id\|file_url\|image_url}` 对象，或多模态 part 数组。 |

#### content 多模态 part

| `type` | 关键字段 |
| --- | --- |
| `text` | `text` |
| `image_url` | `image_url.url`（URL 或 base64） |
| `file_url` | `file_url.url` |
| `file_id` | `file_id`（来自 [files.md](./files.md)） |

### 响应（同步）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 请求 ID。 |
| `agent_id` | string | 同请求。 |
| `conversation_id` | string | 会话 ID；用于后续拉历史 / 续接。 |
| `choices` | `array<Choice>` | 至少一条。 |
| `choices[].message.role` | string | `assistant`。 |
| `choices[].message.content` | `string \| array` | 文本或多模态 part。 |
| `choices[].finish_reason` | enum | `stop` / `tool_calls` / `length` / `sensitive` / `network_error`。 |
| `usage.prompt_tokens` / `completion_tokens` / `total_tokens` | integer | — |

### 流式响应（SSE）

`stream: true` 时按 `data:` 行分发，每行：

```json
{"id":"...","conversation_id":"...","choices":[{"delta":{"content":"片段"},"finish_reason":null}]}
```

最后一行 `data: [DONE]`。

### 最小请求

```bash
curl https://open.bigmodel.cn/api/v1/agents \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "general_translation",
    "messages": [{"role":"user","content":"Hello, world."}],
    "custom_variables": { "source_lang": "en", "target_lang": "zh" }
  }'
```

## 2. 异步结果 · POST /api/v1/agents/async-result

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | ✓ | 上一次 `POST /api/v1/agents` 返回的请求 ID（当 agent 内部走异步时）。 |

返回结构同 1 节中的同步响应，并附加 `task_status`（`PROCESSING` / `SUCCESS` / `FAIL`）。

## 3. 对话历史 · POST /api/v1/agents/messages

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `conversation_id` | string | ✓ | 1 节响应里的会话 ID。 |
| `page` / `page_size` | integer | ✗ | 分页。 |

返回最近 N 条 `messages[]`。

## 已知 Agent ID 速查

| `agent_id` | 用途 | 必备 `custom_variables` |
| --- | --- | --- |
| `general_translation` | 通用翻译 | `source_lang` / `target_lang` / `strategy` / `glossary` |
| `ai_drawing_agent` | 文生图 / 风格化 | 见 [images.md](./images.md) 对照 |
| `slides_glm_agent` | 幻灯片生成 | `template_id` 等 |
| `knowledge_qa_agent` | 知识库问答 | `knowledge_id` |

> 完整 agent_id 清单与字段说明见官方 [Agent API 列表](https://docs.bigmodel.cn/api-reference/agent-api)。

## 错误码

| HTTP | code | 触发 |
| --- | --- | --- |
| `400` | `1010` | agent_id 未授权 / custom_variables 缺字段 |
| `429` | `1113` / `1114` | 触发限流 |
| `500` | `2001` | 上游异常 |

完整错误码见 [errors.md](./errors.md)。

## 参考

- 智能体对话：<https://docs.bigmodel.cn/api-reference/agent-api/智能体对话>
- Agent API 列表：<https://docs.bigmodel.cn/api-reference/agent-api>
- 已停用的旧版助理 API 见 [legacy 标记]（避免混用）。
