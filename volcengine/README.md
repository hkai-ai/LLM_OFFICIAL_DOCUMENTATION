---
source: https://www.volcengine.com/docs/82379/1099455
fetched_at: 2026-05-28
api_version: v3（数据面路径前缀 /api/v3）
last_updated: 2026-05-28
---

# 火山方舟（Volcengine Ark）API 概览

火山方舟（Volcengine Ark）是火山引擎旗下的大模型服务平台，提供豆包（Doubao）系列模型及平台托管的 GLM、DeepSeek 等第三方模型，覆盖文本生成、多模态理解、图片生成、视频生成、向量化、工具调用、上下文缓存、批量推理等能力。数据面 API 与 OpenAI Chat Completions / Responses、Anthropic 等协议高度兼容，可复用主流 SDK 更换 `base_url` 接入。

## Base URL

| 接口类型 | Base URL | 说明 |
| --- | --- | --- |
| 数据面 API（模型调用） | `https://ark.cn-beijing.volces.com/api/v3` | Chat / Responses / Embeddings / Images / Video / Batch / Files 等。 |
| 管控面 API（资源管控） | `https://ark.cn-beijing.volcengineapi.com/` | 管理 API Key、推理接入点（Endpoint）等，火山引擎 OpenAPI 风格（`?Action=...&Version=...`）。 |

> Coding Plan 用户的 Base URL 与上表不同，请以官方 Coding Plan 文档为准，避免地址错误产生额外费用。

## 鉴权

数据面 API 支持两种鉴权方式：

### API Key 鉴权（推荐，简单）

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

API Key 在[控制台 API Key 管理](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey)创建。`model` 字段可填 Model ID 或 Endpoint ID。

### Access Key 鉴权（企业精细化管理）

使用 Access Key（AK/SK）做 HMAC-SHA256 签名，可按资源组 / 云产品等维度管理权限。签名相关字段：`Service=ark`、`Region=cn-beijing`。通过 Access Key 鉴权时 `model` 字段**须填 Endpoint ID**。建议用 IAM 用户的 Access Key 而非主账号。管控面 API 仅支持 Access Key 鉴权。

## 端点索引（数据面 API）

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| 对话(Chat) API | POST | `/api/v3/chat/completions` | [chat-completions.md](./chat-completions.md) |
| 创建模型响应（Responses） | POST | `/api/v3/responses` | [responses.md](./responses.md) |
| 查询 / 删除 / 上下文（Responses） | GET/DELETE | `/api/v3/responses/{id}` 等 | [responses.md](./responses.md) |
| 向量化（多模态 Embeddings） | POST | `/api/v3/embeddings/multimodal` | [embeddings.md](./embeddings.md) |
| 图片生成 | POST | `/api/v3/images/generations` | [images.md](./images.md) |
| 视频生成任务 | POST/GET/DELETE | `/api/v3/contents/generations/tasks` | [video.md](./video.md) |
| 上下文缓存 | POST | `/api/v3/context/create`、`/api/v3/context/chat/completions` | [context-cache.md](./context-cache.md) |
| 批量(Chat) API | POST | `/api/v3/batch/chat/completions` | [batch.md](./batch.md) |
| 文件（File API） | POST/GET/DELETE | `/api/v3/files` 等 | [files.md](./files.md) |
| 分词 | POST | `/api/v3/tokenization` | [tokenization.md](./tokenization.md) |
| 应用(bot) API | POST | `/api/v3/bots/chat/completions` | [bot.md](./bot.md) |

辅助主题：

| 主题 | 文档 |
| --- | --- |
| 模型清单 | [models.md](./models.md) |
| 定价 | [pricing.md](./pricing.md) |
| 错误码 | [errors.md](./errors.md) |
| 兼容 OpenAI SDK | [openai-compat.md](./openai-compat.md) |

## 模型清单概要

完整清单见 [models.md](./models.md)。在售主力：

- **Doubao Seed 2.0（旗舰）**：`doubao-seed-2-0-pro-260215`、`doubao-seed-2-0-lite-260215/260428`、`doubao-seed-2-0-mini-260215/260428`、`doubao-seed-2-0-code-preview-260215`。256k 上下文，支持深度思考 / 多模态 / 工具调用。
- **往期**：`doubao-seed-1-8-251228`、`doubao-seed-1-6` 系列（含 flash / vision）、`doubao-1-5` 系列、专用模型 `doubao-seed-character`（角色扮演）/ `doubao-seed-translation`（翻译）。
- **平台托管第三方**：`glm-4-7-251222`、`deepseek-v4-pro-260425`、`deepseek-v4-flash-260425`、`deepseek-v3-2-251201`。
- **视频生成**：`doubao-seedance-2-0-260128`、`doubao-seedance-2-0-fast-260128`（Seedance）。
- **图片生成**：`doubao-seedream-5-0-260128` 等（Seedream）。
- **向量化**：`doubao-embedding-vision`。

## 关键约定

- **分段计费**：多数大语言模型按单次请求输入长度区间（`[0,32]` / `(32,128]` / `(128,256]`，单位 k token），部分还按输出长度区间，采用不同单价。详见 [pricing.md](./pricing.md)。
- **在线推理模式**：通过 `service_tier`（`fast` 低延迟 / `auto` TPM 保障包优先 / `default` 常规）选择服务等级。
- **深度思考**：`thinking.type`（`enabled` / `disabled` / `auto`）与 `reasoning_effort`（`minimal`~`max`）控制，响应含 `reasoning_content`。
- **货币**：全部人民币（元）计价。

## 参考

- 产品文档首页：https://www.volcengine.com/docs/82379/1099455
- 快速入门：https://www.volcengine.com/docs/82379/1399008
- API 参考（对话 Chat API）：https://www.volcengine.com/docs/82379/1494384
- Base URL 及鉴权：https://www.volcengine.com/docs/82379/1298459
