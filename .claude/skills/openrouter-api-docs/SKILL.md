---
name: openrouter-api-docs
description: 需要查阅、更新或扩充本仓库 openrouter/ 目录下 OpenRouter API 中文文档时使用。覆盖 OpenRouter 聚合网关的鉴权、Chat/Completions/Responses 兼容端点、模型/额度/key 管理、Generation 元数据查询、provider/model 路由、transforms、错误与限流等。
---

# OpenRouter API 文档维护 Skill

## 文档站全貌

OpenRouter 官方文档站根域为 `https://openrouter.ai/docs`。结构分两条主线：

1. **Reference 区**（API 参考，OpenAPI 自动生成）。入口：`https://openrouter.ai/docs/api/reference/overview`。
   - `api/api-reference/<category>/<endpoint>`：端点级参数与响应详情。分类：`chat`、`completions`（部分）、`responses`、`generations`、`models`、`endpoints`、`api-keys`、`credits`、`o-auth`、`providers`。
   - `api/reference/<topic>`：跨端点主题（`authentication`、`parameters`、`limits`、`streaming`、`errors-and-debugging`、`responses/overview` 等）。
2. **Guides / Features 区**（人工撰写的特性指南）。入口：`https://openrouter.ai/docs`。
   - `guides/routing/...`：`provider-selection`、`model-fallbacks`、`routers/auto-router`、`model-variants/exacto`。
   - `guides/features/...`：`message-transforms`、`presets`、`response-caching` 等。
   - `guides/overview/...`：`auth/byok`、`auth/provisioning-api-keys`、`multimodal/overview`。
   - `use-cases/...`：`oauth-pkce`、`crypto-api` 等。

文档站对 LLM 友好：

- 任何页面 URL 末尾追加 `.md` 可直接拿到 markdown。
- 全文聚合：`https://openrouter.ai/docs/llms-full.txt`、`https://openrouter.ai/docs/llms.txt`。
- OpenAPI 规范：`https://openrouter.ai/openapi.json` / `https://openrouter.ai/openapi.yaml`。

## 抓取要点

- 多数页面 WebFetch 可直接抓取，不是 CSR。
- 注意 URL 区分：
  - Reference 入口是 `/docs/api/reference/...`（`api/reference`）。
  - 端点详情页是 `/docs/api/api-reference/...`（`api/api-reference`，多了一个 `api-`）。两个前缀都存在，**不可混用**。
  - 旧版别名 `/docs/api-reference/...`（缺前面 `api/`）已 404，不要再使用。
- Guides 区路径含 `guides/`。早期文档曾用 `features/`、`use-cases/` 等，部分老 URL 已迁移：例如 `features/provider-routing` → `guides/routing/provider-selection`，`features/model-routing` → `guides/routing/model-fallbacks`，`features/message-transforms` → `guides/features/message-transforms`。如 WebFetch 报 "This page does not exist."，先用 WebSearch（`site:openrouter.ai`）定位最新路径。
- Reference 端点的「41+ 个字段」往往在 WebFetch 摘要中被截断，需要时直接拉 `https://openrouter.ai/openapi.json` 用 jq 或本地工具核对完整 schema。
- OAuth PKCE 的端点同时存在于 `/docs/use-cases/oauth-pkce`（流程指南）与 `/docs/api/api-reference/o-auth/...`（端点字段）。

## 更新流程

1. **检查变动入口**：
   - Changelog：`https://openrouter.ai/docs/changelog`。
   - OpenAPI 版本：`https://openrouter.ai/openapi.json`（diff schema）。
   - 模型清单：`GET /api/v1/models` 的返回直接是事实来源；`https://openrouter.ai/models` 是网页清单。
   - provider 列表：`/docs/api/api-reference/providers/list-providers`。
2. **对照仓库文件**：
   - 顶层与索引：`openrouter/README.md`。
   - 单端点：按 `endpoint -> file` 表（见末尾）更新。
   - 跨主题特性：`provider-routing.md` / `model-routing.md` / `transforms.md` / `errors.md` / `rate-limits.md` / `auth.md`。
3. **统一规范**：遵守 `CONVENTIONS.md`（字段中文释义、英文标识符保留、表格列定义、不写营销文案、不加 emoji）。每个端点文件的 frontmatter `fetched_at` 更新为当天日期；`api_version` 在 OpenRouter 没有显式版本号，写 `N/A`。

## 坑点清单（本次实际踩到的）

1. **provisioning key vs 普通 API key**：`/api/v1/keys`、`/api/v1/credits` 必须用 management（provisioning）key；`/api/v1/chat/completions` 不能用 management key。两者不可互换，错用会 403。
2. **`HTTP-Referer` 与 `X-Title` 的真实必填条件**：基础 API 调用上**可选**，但要让请求计入 openrouter.ai 排行榜则 `HTTP-Referer` 必填。仅设置 `X-OpenRouter-Title` / `X-Title` 不创建 app 条目；使用 localhost URL 的应用必须**两者同时设置**才会被追踪。
3. **计费字段需显式开启**：`usage.cost`、`usage.cost_details`、`usage.prompt_tokens_details.cached_tokens` 等扩展字段只有在请求体携带 `"usage": {"include": true}` 时才返回。许多人误以为响应里默认有 `cost`。
4. **`provider` 与 `route` 容易混淆**：
   - `route: "fallback"` 控制 **多模型**回退（配合 `models` 数组）。
   - `provider.allow_fallbacks` 控制 **多 provider**回退。
   - 两者独立，可同时启用。
5. **量化字段命名**：`provider.quantizations` 取值含 `int4`/`int8`/`fp4`/`fp6`/`fp8`/`fp16`/`bf16`/`fp32`/`unknown`，**全小写**。
6. **`sort` 既能是字符串也能是对象**：字符串为简写（`"price"` / `"throughput"` / `"latency"` / `"exacto"`），对象为 `{by, partition}`。`partition: "none"` 才能跨模型全局排序。
7. **动态变体后缀**：`<model>:nitro` ≡ `sort:"throughput"`；`<model>:floor` ≡ `sort:"price"`。这是 model ID 一部分，不是单独字段。
8. **`transforms` 与 `plugins.context-compression` 是同一件事**：8k 及以下上下文长度模型**默认启用**压缩；要禁用得 `plugins: [{"id":"context-compression","enabled":false}]`，而不是把 `transforms` 留空。
9. **`rate_limit` 字段是 legacy**：`GET /api/v1/key` 返回的 `rate_limit` 固定 `-1`，不再代表真实限流上限。真实限流没有稳定的 `X-RateLimit-*` header，只能查 `Retry-After`。
10. **SSE 心跳**：流式响应里 OpenRouter 会发送 `: OPENROUTER PROCESSING` 这类 SSE 注释行防超时，按 SSE 规范忽略，不要当作数据帧解析。
11. **`/api/v1/completions` 没有独立 Reference 页面**：官方仅在概述中声明兼容 OpenAI 规范。撰写时要叠加 OpenRouter chat 扩展字段。
12. **Responses API 是 Beta**：`POST /api/v1/responses` 文档明确写 "may have breaking changes"，集成时锁定版本或加错误兜底。
13. **OpenAPI schema 字段比文档页多**：例如 `GET /api/v1/generation` 实际返回 41+ 字段，WebFetch 摘要常常只列十几个；需要权威字段表时直接拉 OpenAPI JSON。

## 关键链接表

| 主题 / 端点 | 文档 URL |
| --- | --- |
| Reference 入口 | <https://openrouter.ai/docs/api/reference/overview> |
| 鉴权概览 | <https://openrouter.ai/docs/api/reference/authentication> |
| 全部参数索引 | <https://openrouter.ai/docs/api/reference/parameters> |
| 限流 | <https://openrouter.ai/docs/api/reference/limits> |
| 流式 | <https://openrouter.ai/docs/api/reference/streaming> |
| 错误与调试 | <https://openrouter.ai/docs/api/reference/errors-and-debugging> |
| POST /chat/completions | <https://openrouter.ai/docs/api/api-reference/chat/send-chat-completion-request> |
| POST /responses（Beta） | <https://openrouter.ai/docs/api/reference/responses/overview> |
| GET /generation | <https://openrouter.ai/docs/api/api-reference/generations/get-generation> |
| GET /models | <https://openrouter.ai/docs/api/api-reference/models/get-models> |
| GET /models/{author}/{slug}/endpoints | <https://openrouter.ai/docs/api/api-reference/endpoints/list-endpoints> |
| GET /credits | <https://openrouter.ai/docs/api/api-reference/credits/get-credits> |
| GET /key | <https://openrouter.ai/docs/api/api-reference/api-keys/get-current-key> |
| GET /keys | <https://openrouter.ai/docs/api/api-reference/api-keys/list> |
| POST /keys | <https://openrouter.ai/docs/api/api-reference/api-keys/create-keys> |
| DELETE /keys/{hash} | <https://openrouter.ai/docs/api/api-reference/api-keys/delete-keys> |
| POST /auth/keys/code | <https://openrouter.ai/docs/api/api-reference/o-auth/create-auth-keys-code> |
| POST /auth/keys | <https://openrouter.ai/docs/api/api-reference/o-auth/exchange-auth-code-for-api-key> |
| OAuth PKCE 指南 | <https://openrouter.ai/docs/use-cases/oauth-pkce> |
| Provider Routing | <https://openrouter.ai/docs/guides/routing/provider-selection> |
| Model Fallbacks | <https://openrouter.ai/docs/guides/routing/model-fallbacks> |
| Auto Router | <https://openrouter.ai/docs/guides/routing/routers/auto-router> |
| Message Transforms | <https://openrouter.ai/docs/guides/features/message-transforms> |
| Provisioning Keys 指南 | <https://openrouter.ai/docs/guides/overview/auth/provisioning-api-keys> |
| Changelog | <https://openrouter.ai/docs/changelog> |
| OpenAPI JSON | <https://openrouter.ai/openapi.json> |
| 全文聚合 | <https://openrouter.ai/docs/llms-full.txt> |

## endpoint → 仓库文件

| 端点 / 主题 | 文件 |
| --- | --- |
| 概览、端点索引、headers | `openrouter/README.md` |
| `POST /chat/completions` | `openrouter/chat-completions.md` |
| `POST /completions` | `openrouter/completions.md` |
| `GET /generation` | `openrouter/generation.md` |
| `GET /models`、`/models/{a}/{s}/endpoints` | `openrouter/models.md` |
| `GET /credits` | `openrouter/credits.md` |
| `/api/v1/key`、`/api/v1/keys` 全套 | `openrouter/api-keys.md` |
| OAuth PKCE + `/auth/keys`、`/auth/keys/code` | `openrouter/auth.md` |
| Provider 路由 | `openrouter/provider-routing.md` |
| 模型回退 + Auto Router | `openrouter/model-routing.md` |
| Transforms / context-compression | `openrouter/transforms.md` |
| 错误结构与状态码 | `openrouter/errors.md` |
| 限流 | `openrouter/rate-limits.md` |
