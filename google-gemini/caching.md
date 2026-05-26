---
source: https://ai.google.dev/api/caching?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 上下文缓存 · /v1beta/cachedContents

> 把要在多次请求中复用的固定上下文（长文档、视频、系统指令、工具声明等）预先入库，后续 `generateContent` 用 `cachedContent` 字段引用，命中部分按更低的缓存价计费且减少传输。仅特定模型支持（例如 `gemini-2.5-pro`、`gemini-2.5-flash`、`gemini-1.5-pro`、`gemini-1.5-flash`），最低 token 数有限制（一般为 4096，模型不同）。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` | ✓ | API Key。 |
| `Content-Type` | ✓ | `application/json`。 |

## CachedContent 对象

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `name` | string | ✗ | — | output-only，资源名 `cachedContents/{id}`。 |
| `displayName` | string | ✗ | — | 人类可读名，最多 128 字符。 |
| `model` | string | ✓ | — | 缓存绑定的模型，格式 `models/{model}`，必须与未来引用方一致。 |
| `systemInstruction` | Content | ✗ | — | 系统指令，结构同 [generate-content.md](./generate-content.md#contents)。 |
| `contents` | array&lt;Content&gt; | ✗ | — | 要缓存的会话内容。 |
| `tools` | array&lt;Tool&gt; | ✗ | — | 要缓存的工具声明。 |
| `toolConfig` | ToolConfig | ✗ | — | 工具配置。 |
| `expireTime` | string (RFC3339) | ✗ | — | 绝对过期时间；与 `ttl` 二选一。 |
| `ttl` | string (Duration，如 `"3600s"`) | ✗ | — | 创建后相对过期，与 `expireTime` 二选一；不设默认 1 小时。 |
| `createTime` | string (RFC3339) | — | — | output-only。 |
| `updateTime` | string (RFC3339) | — | — | output-only。 |
| `usageMetadata.totalTokenCount` | integer | — | — | output-only，缓存的总 token 数。 |

## 端点一：cachedContents.create

| 维度 | 值 |
| --- | --- |
| 方法 | POST |
| 路径 | `/v1beta/cachedContents` |
| Body | `CachedContent`（必填 `model`，至少含 `contents` / `systemInstruction` / `tools` 之一） |
| 响应 | 创建后的 `CachedContent`（含 `name`、`createTime`、`expireTime`、`usageMetadata`） |

最小请求：

```json
{
  "model": "models/gemini-2.5-flash",
  "displayName": "doc-cache-001",
  "systemInstruction": { "parts": [{ "text": "你是一个论文助手。" }] },
  "contents": [
    {
      "role": "user",
      "parts": [
        { "fileData": { "mimeType": "application/pdf", "fileUri": "https://generativelanguage.googleapis.com/v1beta/files/abc123" } }
      ]
    }
  ],
  "ttl": "3600s"
}
```

## 端点二：cachedContents.list

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/cachedContents` |

Query 参数：

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `pageSize` | integer | ✗ | `1000`（文档实际呈现的默认值） | 单页最大数量。 |
| `pageToken` | string | ✗ | — | 上一页返回的 `nextPageToken`。 |

响应：

```json
{
  "cachedContents": [ /* CachedContent[] */ ],
  "nextPageToken": "..."
}
```

## 端点三：cachedContents.get

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/{name=cachedContents/*}` |
| Path 参数 | `name`（必填） |
| 响应 | 单个 `CachedContent`（不含 `contents` / `tools` 等大字段，仅元数据） |

## 端点四：cachedContents.patch

| 维度 | 值 |
| --- | --- |
| 方法 | PATCH |
| 路径 | `/v1beta/{cachedContent.name=cachedContents/*}` |
| Path 参数 | `cachedContent.name`（必填） |
| Query 参数 | `updateMask`（FieldMask，必填）。仅 `ttl` 与 `expireTime` 可被更新。 |
| Body | 仅含要更新字段的 `CachedContent` |
| 响应 | 更新后的 `CachedContent` |

```
PATCH /v1beta/cachedContents/abc123?updateMask=ttl
Content-Type: application/json

{ "ttl": "7200s" }
```

## 端点五：cachedContents.delete

| 维度 | 值 |
| --- | --- |
| 方法 | DELETE |
| 路径 | `/v1beta/{name=cachedContents/*}` |
| Path 参数 | `name`（必填） |
| 响应 | 空 JSON `{}` |

## 在 generateContent 中引用

```json
{
  "contents": [
    { "role": "user", "parts": [{ "text": "用 50 字总结附件文档第 3 章。" }] }
  ],
  "cachedContent": "cachedContents/abc123"
}
```

> 引用时**不要再重复**把缓存里的 `systemInstruction` / `tools` / 长文档塞回 `contents`。模型实际看到的是 `cachedContent` + 本次 `contents` 拼接。

响应 `usageMetadata.cachedContentTokenCount` 会显示命中的缓存 token 数；命中部分按缓存价计费，未命中（如新增的 `contents`）按正常价计费。

## 错误

| HTTP | `status` | 触发原因 |
| --- | --- | --- |
| 400 | `INVALID_ARGUMENT` | 缓存 token 数低于模型最低门槛（多数为 4096）、`model` 不支持缓存、`updateMask` 指向不可变字段。 |
| 404 | `NOT_FOUND` | `name` 不存在或已过期。 |
| 409 | `ABORTED` | 并发更新冲突。 |

详细错误结构见 [errors.md](./errors.md)。

## 参考

- 端点文档：<https://ai.google.dev/api/caching?hl=zh-CN>
- 缓存指南：<https://ai.google.dev/gemini-api/docs/caching>
