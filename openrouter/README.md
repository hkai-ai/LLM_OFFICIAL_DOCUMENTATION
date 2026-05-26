---
source: https://openrouter.ai/docs/api/reference/overview
fetched_at: 2026-05-19
api_version: N/A
last_updated: 2026-05-19
---

# OpenRouter API 概览

OpenRouter 是一个 LLM 聚合网关，将上百家模型供应商（OpenAI、Anthropic、Google、Meta、Mistral 等）统一为 OpenAI 兼容协议暴露，对外提供单一的 API key、统一计费、跨 provider 的回退与路由能力。

## Base URL

```
https://openrouter.ai/api/v1
```

所有端点均以该前缀开头（除 OAuth 授权 URL 外）。

## 鉴权

| 方式 | 说明 |
| --- | --- |
| `Authorization: Bearer <API_KEY>` | 主流方式。普通 API key 用于推理；management key（也称 provisioning key）用于密钥与额度管理。两者不可互换。 |
| OAuth PKCE | 客户端应用一键集成用户的 OpenRouter 账户。流程见 [auth.md](./auth.md)。 |

management key 不能调用 `/chat/completions` 等推理端点；普通 API key 不能调用 `/keys`、`/credits` 等管理端点。

## 推荐与必填的请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json`（POST 时） |
| `HTTP-Referer` | ✗ | 你的站点 URL；用于 openrouter.ai 排行榜归因。**没有此 header 不会生成 app 页面，使用量不会出现在排行榜。** |
| `X-Title` | ✗ | 应用展示名称。与 `X-OpenRouter-Title` 等价。 |
| `X-OpenRouter-Title` | ✗ | 同上。仅设置 title 而不带 `HTTP-Referer` 不会创建 app 条目。使用 localhost URL 的应用必须配合此 header 才能被追踪。 |

## 与官方厂商直连的区别

| 维度 | 直连官方 | OpenRouter |
| --- | --- | --- |
| 协议 | 各家不同 | 统一 OpenAI Chat Completions（另有 Responses Beta、Anthropic Messages 兼容） |
| 鉴权 | 各家 API key | 单一 OpenRouter key |
| 计费 | 各家独立结算 | OpenRouter credits 统一结算（也支持 BYOK 走自己的 key） |
| 路由 | 单一 provider | `provider`/`models`/`route` 控制 provider 顺序、量化等级、回退、价格上限等 |
| 模型透明度 | 固定 | 可指定多模型回退；响应 `model` 字段返回实际使用的模型 |
| 上下文压缩 | 由调用方处理 | `transforms` / `plugins.context-compression` 自动 middle-out |

## 端点索引

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| Chat Completions | `POST` | `/api/v1/chat/completions` | [chat-completions.md](./chat-completions.md) |
| Text Completions（OpenAI 兼容） | `POST` | `/api/v1/completions` | [completions.md](./completions.md) |
| Responses API（Beta，OpenAI 兼容） | `POST` | `/api/v1/responses` | 见下方说明 |
| Generation 元数据 | `GET` | `/api/v1/generation` | [generation.md](./generation.md) |
| 列出模型 | `GET` | `/api/v1/models` | [models.md](./models.md) |
| 列出模型的 endpoint | `GET` | `/api/v1/models/{author}/{slug}/endpoints` | [models.md](./models.md) |
| 当前 key 信息 | `GET` | `/api/v1/key` | [api-keys.md](./api-keys.md) |
| 列出 API keys | `GET` | `/api/v1/keys` | [api-keys.md](./api-keys.md) |
| 创建 API key | `POST` | `/api/v1/keys` | [api-keys.md](./api-keys.md) |
| 删除 API key | `DELETE` | `/api/v1/keys/{hash}` | [api-keys.md](./api-keys.md) |
| 查询额度 | `GET` | `/api/v1/credits` | [credits.md](./credits.md) |
| 创建 OAuth 授权码 | `POST` | `/api/v1/auth/keys/code` | [auth.md](./auth.md) |
| 用授权码换 API key | `POST` | `/api/v1/auth/keys` | [auth.md](./auth.md) |

## OpenAI / Anthropic 兼容协议

- **OpenAI 兼容**：`/api/v1/chat/completions`、`/api/v1/completions`、`/api/v1/models` 直接复用 OpenAI 请求/响应 schema。可将 OpenAI SDK 的 `base_url` 改为 `https://openrouter.ai/api/v1` 直接使用。
- **Responses API（Beta）**：`POST /api/v1/responses`，定位为 OpenAI Responses API 的 drop-in replacement，处于 beta，可能有 breaking change。
- **Anthropic 兼容**：OpenRouter 在 reference 中以 OpenAI 协议为主，Anthropic Messages 风格的字段（如 `cache_control`、`reasoning`、`prediction`）以扩展形式合并到 Chat Completions 请求体内；OpenRouter 本身不暴露独立的 `/v1/messages` 端点（如有需要直接在 Chat Completions 中使用 anthropic 模型并配合 `cache_control`）。

## 路由、回退与转换

- Provider 路由（量化等级、地区、隐私、价格上限等）：见 [provider-routing.md](./provider-routing.md)。
- 多模型回退、`openrouter/auto` 元模型：见 [model-routing.md](./model-routing.md)。
- 上下文压缩（`transforms` / `context-compression` 插件）：见 [transforms.md](./transforms.md)。
- 错误结构与状态码：见 [errors.md](./errors.md)。
- 限流策略：见 [rate-limits.md](./rate-limits.md)。

## 参考

- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
- 完整 OpenAPI 规范：<https://openrouter.ai/openapi.json>（YAML：<https://openrouter.ai/openapi.yaml>）
- 文档站全文聚合（便于 LLM 抓取）：<https://openrouter.ai/docs/llms-full.txt>
