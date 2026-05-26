---
source: https://platform.kimi.com/docs/api/overview
fetched_at: 2026-05-26
api_version: N/A（OpenAPI 1.0；endpoint 统一带 `/v1` 前缀）
last_updated: 2026-05-26
---

# Moonshot Kimi API 概览

Moonshot（Kimi）官方 API 与 OpenAI Chat Completions 协议兼容，可直接复用 OpenAI Python / Node.js SDK 通过更换 `base_url` 接入。本仓库文档以 OpenAI 兼容形式整理。

## Base URL

| 用途 | URL |
| --- | --- |
| 全部端点 | `https://api.moonshot.cn/v1` |

`/v1` 是真实路径前缀，调用 `chat/completions` 完整路径为 `https://api.moonshot.cn/v1/chat/completions`。

> 注意：`platform.kimi.com`（开发者平台）与 `kimi.ai`（海外消费版）使用各自独立的 API Key 体系，不互通。

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${MOONSHOT_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |

API Key 在 https://platform.kimi.com 控制台创建，按组织维度隔离。

## 与 OpenAI SDK 的兼容性

- 请求 / 响应字段命名、SSE 流式格式与 OpenAI Chat Completions 一致。
- Python：`OpenAI(api_key=..., base_url="https://api.moonshot.cn/v1")`。
- Node.js：`new OpenAI({ apiKey, baseURL: "https://api.moonshot.cn/v1" })`。
- 特有字段位置：
  - `partial` 写在 `messages[]` 中**最后一条** `assistant` 消息内，不是顶层参数。
  - `thinking` 是顶层参数，但仅在 K2.5 / K2.6 系列生效；SDK 调用时通过 `extra_body={"thinking": ...}` 透传。
  - `reasoning_content` 出现在 `choices[].message` 与流式 `delta` 中，与 DeepSeek 同名同语义。

## 端点索引

| 端点 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 对话补全 | POST | `/chat/completions` | [chat-completions.md](./chat-completions.md) |
| 估算 Token 数 | POST | `/tokenizers/estimate-token-count` | [estimate-tokens.md](./estimate-tokens.md) |
| 模型列表 | GET | `/models` | [models.md](./models.md) |
| 账户余额 | GET | `/users/me/balance` | [user-balance.md](./user-balance.md) |
| 上传文件 | POST | `/files` | [files.md](./files.md) |
| 列出文件 | GET | `/files` | [files.md](./files.md) |
| 获取文件 | GET | `/files/{file_id}` | [files.md](./files.md) |
| 删除文件 | DELETE | `/files/{file_id}` | [files.md](./files.md) |
| 获取文件内容 | GET | `/files/{file_id}/content` | [files.md](./files.md) |
| 创建批处理任务 | POST | `/batches` | [batch.md](./batch.md) |
| 列出批处理任务 | GET | `/batches` | [batch.md](./batch.md) |
| 检索批处理任务 | GET | `/batches/{batch_id}` | [batch.md](./batch.md) |
| 取消批处理任务 | POST | `/batches/{batch_id}/cancel` | [batch.md](./batch.md) |

辅助主题（能力 / 指南）：

| 主题 | 文档 |
| --- | --- |
| 入门指南索引 | [guides.md](./guides.md) |
| 工具调用（function + `$web_search`） | [tool-use.md](./tool-use.md) |
| JSON Mode | [json-mode.md](./json-mode.md) |
| 视觉 / 视频多模态 | [vision.md](./vision.md) |
| 思考模式（K2 系列） | [thinking.md](./thinking.md) |
| 联网搜索 `$web_search` | [web-search.md](./web-search.md) |
| Partial Mode（前缀续写） | [partial-mode.md](./partial-mode.md) |
| 错误码 | [errors.md](./errors.md) |
| 充值与限速 Tier | [rate-limits.md](./rate-limits.md) |
| 定价 | [pricing.md](./pricing.md) |

## 模型清单

下表为当前在售模型的速查。详细字段（视觉 / 视频 / 思考支持位）来自 `GET /v1/models` 返回的 `supports_image_in` / `supports_video_in` / `supports_reasoning`。

| 模型 ID | 上下文窗口 | 输入：缓存命中 | 输入：缓存未命中 | 输出 | 视觉 / 视频 | 思考模式 |
| --- | --- | --- | --- | --- | --- | --- |
| `kimi-k2.6` | 256K | ¥1.10 / 1M | ¥6.50 / 1M | ¥27.00 / 1M | 是 | 默认开启（可关） |
| `kimi-k2.5` | 256K | ¥0.70 / 1M | ¥4.00 / 1M | ¥21.00 / 1M | 是 | 默认开启（可关） |
| `kimi-k2-0905-preview` | 256K | 详见 [chat-k2](https://platform.kimi.com/docs/pricing/chat-k2) | 同左 | 同左 | 否 | 否 |
| `kimi-k2-0711-preview` | 128K | 同上 | 同上 | 同上 | 否 | 否 |
| `kimi-k2-turbo-preview` | 256K | 同上 | 同上 | 同上 | 否 | 否 |
| `kimi-k2-thinking` | 256K | 同上 | 同上 | 同上 | 否 | 是 |
| `kimi-k2-thinking-turbo` | 256K | 同上 | 同上 | 同上 | 否 | 是 |
| `moonshot-v1-8k` | 8,192 | — | ¥2.00 / 1M | ¥10.00 / 1M | 否 | 否 |
| `moonshot-v1-32k` | 32,768 | — | ¥5.00 / 1M | ¥20.00 / 1M | 否 | 否 |
| `moonshot-v1-128k` | 131,072 | — | ¥10.00 / 1M | ¥30.00 / 1M | 否 | 否 |
| `moonshot-v1-8k-vision-preview` | 8K | — | 同 v1-8k | 同 v1-8k | 是 | 否 |
| `moonshot-v1-32k-vision-preview` | 32K | — | 同 v1-32k | 同 v1-32k | 是 | 否 |
| `moonshot-v1-128k-vision-preview` | 128K | — | 同 v1-128k | 同 v1-128k | 是 | 否 |

> `kimi-k2-*-preview` 系列计划于 2026-05-25 下线，建议迁移至 `kimi-k2.5` / `kimi-k2.6`。
>
> 价格单位 1M = 1,000,000 tokens；moonshot-v1 系列没有缓存命中定价档位，所有输入按统一价计费。

## 计费要点

- K2.5 / K2.6 区分缓存命中（`cached_tokens`）与未命中两档输入价，命中价显著低于未命中价；moonshot-v1 系列不区分。
- 流式响应需 `stream_options.include_usage: true` 才会在 `[DONE]` 前的最后一个 chunk 中返回 `usage`。
- 限速按 Tier 阶梯（充值金额）划分；详见 [rate-limits.md](./rate-limits.md)。

## 参考

- 入口：https://platform.kimi.com/docs/api/overview
- 文档索引（`llms.txt`）：https://platform.kimi.com/docs/llms.txt
- OpenAPI 规范：https://platform.kimi.com/docs/openapi.json
- 模型与价格：https://platform.kimi.com/docs/pricing/chat
- 充值与限速：https://platform.kimi.com/docs/pricing/limits
- 从 OpenAI 迁移：https://platform.kimi.com/docs/guide/migrating-from-openai-to-kimi
