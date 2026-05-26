---
source: https://platform.claude.com/docs/en/api/overview
fetched_at: 2026-05-26
api_version: 2023-06-01（最新 anthropic-version）
last_updated: 2026-05-26
---

# Anthropic Claude API

> 官方文档现位于 `https://platform.claude.com/docs/en/api/`。旧域名 `https://docs.anthropic.com/en/api/` 已 301 永久重定向至上述新域名。

## 基本信息

| 项 | 值 |
| --- | --- |
| Base URL | `https://api.anthropic.com` |
| 文档站 | `https://platform.claude.com/docs/en/api/overview` |
| 风格 | RESTful + JSON；POST 接口的 `Content-Type` 固定为 `application/json` |
| 流式响应 | Server-Sent Events（SSE，命名事件）；通过请求体 `stream: true` 启用 |

## 鉴权与必备请求头

所有请求必须二选一提供身份凭证：

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` | 二选一 | Console 生成的 API Key。两者均可，二者只需其一。 |
| `Authorization` | 二选一 | 形如 `Bearer <token>`，token 由 Workload Identity Federation 的 `POST /v1/oauth/token` 颁发，短期有效。 |
| `anthropic-version` | ✓ | API 版本号，例如 `2023-06-01`。详见 [versioning.md](./versioning.md)。 |
| `content-type` | ✓ | `application/json`。 |
| `anthropic-beta` | ✗ | 启用 beta 能力，多 beta 用英文逗号分隔（如 `feature1,feature2`）。详见 [versioning.md](./versioning.md)。 |

响应头：

- `request-id`：全局请求 ID，用于支持工单。
- `anthropic-organization-id`：API key 所属组织 ID。

## 请求体积上限

| 端点 | 上限 |
| --- | --- |
| Messages、Token Counting | `32 MB` |
| Message Batches | `256 MB` |
| Files | `500 MB` |
| Sessions / Agents / Environments | `32 MB` |

超过上限会触发 `413 request_too_large`。

## SDK

| 语言 | 包名 / 安装 | 仓库 |
| --- | --- | --- |
| Python | `pip install anthropic`（3.9+） | `anthropics/anthropic-sdk-python` |
| TypeScript | `npm install @anthropic-ai/sdk`（Node 20+，TS 4.9+） | `anthropics/anthropic-sdk-typescript` |
| Java | `com.anthropic:anthropic-java`（Java 8+） | `anthropics/anthropic-sdk-java` |
| Go | `go get github.com/anthropics/anthropic-sdk-go`（Go 1.23+） | `anthropics/anthropic-sdk-go` |
| C# | `dotnet add package Anthropic`（.NET Standard 2.0+） | `anthropics/anthropic-sdk-csharp` |
| Ruby | `bundle add anthropic`（Ruby 3.2.0+） | `anthropics/anthropic-sdk-ruby` |
| PHP | `composer require anthropic-ai/sdk`（PHP 8.1+） | `anthropics/anthropic-sdk-php` |
| CLI | `brew install anthropics/tap/ant` | — |

SDK 自动注入 `x-api-key`、`anthropic-version`、`content-type` 三个 header。

## 端点索引

| 类别 | METHOD | PATH | 文档 |
| --- | --- | --- | --- |
| Messages | `POST` | `/v1/messages` | [messages.md](./messages.md) |
| Messages 流式 | `POST` | `/v1/messages`（`stream: true`） | [messages-streaming.md](./messages-streaming.md) |
| Token 计数 | `POST` | `/v1/messages/count_tokens` | [count-tokens.md](./count-tokens.md) |
| Models 列表 | `GET` | `/v1/models` | [models.md](./models.md) |
| Models 详情 | `GET` | `/v1/models/{model_id}` | [models.md](./models.md) |
| 错误 | — | — | [errors.md](./errors.md) |
| 版本 / beta | — | — | [versioning.md](./versioning.md) |

仓库中暂未单独整理但官方提供的端点（GA / Beta）：

| 端点 | 状态 | 官方 URL |
| --- | --- | --- |
| `POST /v1/messages/batches` | GA | `https://platform.claude.com/docs/en/api/creating-message-batches` |
| `POST /v1/files` / `GET /v1/files` | Beta | `https://platform.claude.com/docs/en/api/files-create` |
| `POST /v1/skills` / `GET /v1/skills` | Beta | `https://platform.claude.com/docs/en/api/skills/create-skill` |
| `POST /v1/agents` / `GET /v1/agents` | Beta（Managed Agents） | `https://platform.claude.com/docs/en/managed-agents/agent-setup` |
| `POST /v1/sessions` / `GET /v1/sessions/{id}/stream` | Beta（Managed Agents） | `https://platform.claude.com/docs/en/managed-agents/sessions` |
| `POST /v1/environments` / `GET /v1/environments` | Beta（Managed Agents） | `https://platform.claude.com/docs/en/managed-agents/environments` |

## 官方文档顶级目录

文档站侧边栏的顶层组织（截至 `fetched_at`）：

- **API reference**：`overview`、`messages` 系列（create / streaming / count_tokens）、`models` 系列、`message-batches`、`files`、`skills`、`managed-agents`
- **Forming requests**：`client-sdks`、`authentication`、`workload-identity-federation`
- **Handling responses**：`errors`、`rate-limits`、`service-tiers`
- **Versioning**：`versioning`、`beta-headers`
- **About Claude / Models**：`models/overview`、`models/model-ids-and-versions`、`model-deprecations`
- **Capabilities / Build with Claude**：streaming、batch-processing、tool use、extended thinking、structured outputs、vision、pdf、prompt caching 等

## 参考

- API overview：`https://platform.claude.com/docs/en/api/overview`
- Client SDKs：`https://platform.claude.com/docs/en/api/client-sdks`
- Models 概览：`https://platform.claude.com/docs/en/about-claude/models/overview`
