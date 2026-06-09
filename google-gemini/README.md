---
source: https://ai.google.dev/api?hl=zh-CN
fetched_at: 2026-06-09
api_version: v1beta (兼有 v1)
last_updated: 2026-06-09
---

> 2026-06-09 更新：补齐 Batch Mode、Live API、File Search、Tuning 四篇端点文档。

# Google Gemini Developer API · 厂商概览

本目录整理 **Gemini Developer API**（`generativelanguage.googleapis.com`）的官方 HTTP 接口。不覆盖 Vertex AI Gemini API（GCP 内）。

## Base URL

| 用途 | URL |
| --- | --- |
| REST 主域 | `https://generativelanguage.googleapis.com` |
| Media 上传域 | `https://generativelanguage.googleapis.com/upload` |
| Live API（WebSocket） | `wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1beta.GenerativeService.BidiGenerateContent` |

## API 版本路径

| 版本 | 路径前缀 | 说明 |
| --- | --- | --- |
| `v1beta` | `/v1beta/...` | 主版本，绝大多数新特性（thinking、Live API、File Search、Batches、Tuning 等）仅在此提供。 |
| `v1` | `/v1/...` | 稳定子集，仅覆盖 generateContent / streamGenerateContent / countTokens / embedContent / models 列表等核心方法；新字段（如 `thinkingConfig`、`responseSchema` 的部分高级特性、`urlContext`）可能缺失。 |

> 本目录文档默认以 `/v1beta` 为准；如某端点在 `/v1` 中存在差异，会在该端点的文档内注明。

## 鉴权

两种方式二选一（HTTPS 强制）：

| 方式 | 位置 | 示例 |
| --- | --- | --- |
| HTTP header | `x-goog-api-key` | `x-goog-api-key: $GEMINI_API_KEY` |
| Query 参数 | `key` | `...?key=$GEMINI_API_KEY` |

> 官方推荐在生产环境使用 header 形式，不要把 key 放进 URL（会被网关日志、Referer 记录）。

请求体为 JSON 时必须 `Content-Type: application/json`；上传媒体时按 multipart/resumable 协议设置头部，详见 [files.md](./files.md)。

## 字段命名

REST 接口统一使用 **camelCase**（例如 `generationConfig`、`maxOutputTokens`、`responseMimeType`）。Proto/SDK 文档中常见的 snake_case（`max_output_tokens`）只是 proto 字段名；HTTP/JSON 必须用 camelCase。

## 端点索引

### Generating content（生成内容）

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `models.generateContent` | POST | `/v1beta/{model=models/*}:generateContent` | [generate-content.md](./generate-content.md) |
| `models.streamGenerateContent` | POST | `/v1beta/{model=models/*}:streamGenerateContent` | [stream-generate-content.md](./stream-generate-content.md) |

### Models（模型）

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `models.get` | GET | `/v1beta/{name=models/*}` | [models.md](./models.md) |
| `models.list` | GET | `/v1beta/models` | [models.md](./models.md) |

### Tokens

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `models.countTokens` | POST | `/v1beta/{model=models/*}:countTokens` | [count-tokens.md](./count-tokens.md) |

### Embeddings

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `models.embedContent` | POST | `/v1beta/{model=models/*}:embedContent` | [embed-content.md](./embed-content.md) |
| `models.batchEmbedContents` | POST | `/v1beta/{model=models/*}:batchEmbedContents` | [embed-content.md](./embed-content.md) |

### Files

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `media.upload` | POST | `/upload/v1beta/files`（媒体） + `/v1beta/files`（元数据） | [files.md](./files.md) |
| `files.get` | GET | `/v1beta/{name=files/*}` | [files.md](./files.md) |
| `files.list` | GET | `/v1beta/files` | [files.md](./files.md) |
| `files.delete` | DELETE | `/v1beta/{name=files/*}` | [files.md](./files.md) |

### Caching（上下文缓存）

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `cachedContents.create` | POST | `/v1beta/cachedContents` | [caching.md](./caching.md) |
| `cachedContents.list` | GET | `/v1beta/cachedContents` | [caching.md](./caching.md) |
| `cachedContents.get` | GET | `/v1beta/{name=cachedContents/*}` | [caching.md](./caching.md) |
| `cachedContents.patch` | PATCH | `/v1beta/{cachedContent.name=cachedContents/*}` | [caching.md](./caching.md) |
| `cachedContents.delete` | DELETE | `/v1beta/{name=cachedContents/*}` | [caching.md](./caching.md) |

### Batch Mode

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `models.batchGenerateContent` | POST | `/v1beta/{batch.model=models/*}:batchGenerateContent` | [batches.md](./batches.md) |
| `batches.get` | GET | `/v1beta/{name=batches/*}` | [batches.md](./batches.md) |
| `batches.list` | GET | `/v1beta/batches` | [batches.md](./batches.md) |
| `batches.cancel` | POST | `/v1beta/{name=batches/*}:cancel` | [batches.md](./batches.md) |
| `batches.delete` | DELETE | `/v1beta/{name=batches/*}` | [batches.md](./batches.md) |

### Live API（WebSocket）

| 端点 | 协议 | 文档 |
| --- | --- | --- |
| `BidiGenerateContent` | `wss://generativelanguage.googleapis.com/ws/...v1beta...BidiGenerateContent` | [live-api.md](./live-api.md) |

### File Search

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `fileSearchStores.create` | POST | `/v1beta/fileSearchStores` | [file-search.md](./file-search.md) |
| `fileSearchStores.list` / `.get` / `.delete` | GET / GET / DELETE | `/v1beta/fileSearchStores[/*]` | [file-search.md](./file-search.md) |
| `fileSearchStores.importFile` | POST | `/v1beta/{fileSearchStoreName=fileSearchStores/*}:importFile` | [file-search.md](./file-search.md) |
| `fileSearchStores.uploadToFileSearchStore` | POST | `/upload/v1beta/{fileSearchStoreName=fileSearchStores/*}:uploadToFileSearchStore` | [file-search.md](./file-search.md) |

### Tuning

| 端点 | 方法 | 路径 | 文档 |
| --- | --- | --- | --- |
| `tunedModels.create` | POST | `/v1beta/tunedModels` | [tuning.md](./tuning.md) |
| `tunedModels.get` / `.list` / `.patch` / `.delete` | GET / GET / PATCH / DELETE | `/v1beta/tunedModels[/*]` | [tuning.md](./tuning.md) |
| `tunedModels.transferOwnership` | POST | `/v1beta/{name=tunedModels/*}:transferOwnership` | [tuning.md](./tuning.md) |
| `tunedModels.generateContent` / `.streamGenerateContent` / `.countTokens` | POST | `/v1beta/{model=tunedModels/*}:...` | [tuning.md](./tuning.md) |

### 实验性

| 分类 | 入口 | 备注 |
| --- | --- | --- |
| Live Music / Interactions | `/api/live_music` / `/api/interactions-api` | 实验性，未在本目录展开。 |

### Errors

通用错误响应见 [errors.md](./errors.md)。

## SDK 一览（仅供参照，非 API 字段）

| 语言 | 包名 | 仓库 |
| --- | --- | --- |
| Python | `google-genai` | `googleapis/python-genai` |
| JavaScript / TypeScript | `@google/genai` | `googleapis/js-genai` |
| Java | `com.google.genai:google-genai` | `googleapis/java-genai` |
| Go | `google.golang.org/genai` | `googleapis/go-genai` |

> 旧 SDK `google-generativeai`（Python）/ `@google/generative-ai`（JS）已停止接收新特性，新项目应迁移到 `google-genai` / `@google/genai`。

## Developer API 与 Vertex AI Gemini API 的差异

| 维度 | Gemini Developer API（本目录） | Vertex AI Gemini API |
| --- | --- | --- |
| 域名 | `generativelanguage.googleapis.com` | `<region>-aiplatform.googleapis.com` 或 `aiplatform.googleapis.com` |
| 鉴权 | API Key（`x-goog-api-key` 或 `?key=`） | Google Cloud OAuth2 / Service Account / ADC |
| 资源 | 无项目层级，直接 `models/{model}` | 强项目层级 `projects/{p}/locations/{l}/publishers/google/models/{model}` |
| 数据驻留 | 全球，默认按 Google 规则 | 可选区域（`us-central1`、`europe-west4` 等） |
| 计费 | Google AI Studio / Gemini API 计费 | GCP 计费 |
| 适用场景 | 个人、原型、轻量产品 | 企业生产、合规、私有数据 |
| 字段差异 | 部分字段名（如 `safetySettings`、模型 ID）一致；但 Vertex 多了 `systemInstruction`-V2、`generationConfig.routingConfig`、`groundingConfig` 等 | 同左 |

> 本目录字段以 Developer API 为准。不要把 Vertex AI 独有字段写进来。

## 参考

- API 总入口：<https://ai.google.dev/api?hl=zh-CN>
- 模型介绍：<https://ai.google.dev/gemini-api/docs/models>
- SDK 总览：<https://ai.google.dev/gemini-api/docs/quickstart>
