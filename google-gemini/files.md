---
source: https://ai.google.dev/api/files?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# Files API · 上传与管理媒体

> 用于把图片、音频、视频、PDF、纯文本独立上传到 Gemini 后端，得到 `files/{id}` URI，再在 `generateContent.contents[].parts[].fileData.fileUri` 中引用，避免每次请求重复编码大体积内联数据。文件默认保留 **48 小时**，过期自动删除；每个项目存储上限约 **20 GB**，单文件上限约 **2 GB**（以官方实时说明为准）。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` | ✓ | API Key。 |
| `Content-Type` | ✓ | 元数据 JSON 用 `application/json`；resumable 上传按下文协议设置。 |

## 端点一：media.upload

> 媒体与元数据**走不同路径**：上传用 `/upload/v1beta/files`，纯元数据/列表/删除用 `/v1beta/files`。支持三种协议：`media`（简单上传）、`multipart`、`resumable`（可断点续传）。

### 协议参数

| Query | 取值 | 说明 |
| --- | --- | --- |
| `uploadType` | `media` / `multipart` / `resumable` | `media`：仅文件字节，无元数据。`multipart`：单请求同时携带 JSON 元数据与字节。`resumable`：先发起会话获 `X-Goog-Upload-URL`，后续 PUT 字节，可断点续传，适合大文件。 |

### Resumable 上传（推荐）

1. **发起会话**

   ```
   POST /upload/v1beta/files?uploadType=resumable
   x-goog-api-key: $GEMINI_API_KEY
   X-Goog-Upload-Protocol: resumable
   X-Goog-Upload-Command: start
   X-Goog-Upload-Header-Content-Length: <文件字节数>
   X-Goog-Upload-Header-Content-Type: <MIME>
   Content-Type: application/json

   { "file": { "displayName": "demo.mp4" } }
   ```

   响应头返回 `X-Goog-Upload-URL`，body 为空。

2. **上传字节**

   ```
   POST <X-Goog-Upload-URL>
   X-Goog-Upload-Command: upload, finalize
   X-Goog-Upload-Offset: 0
   Content-Length: <文件字节数>

   <raw bytes>
   ```

   成功后返回 `CreateFileResponse`：

   ```json
   { "file": { "name": "files/abc123", "uri": "https://generativelanguage.googleapis.com/v1beta/files/abc123", "state": "PROCESSING", "...": "..." } }
   ```

### File 上传请求体

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file.displayName` | string | ✗ | — | 人类可读名，最多 512 字符。 |

> `mimeType`、`sizeBytes`、`sha256Hash`、`uri`、`downloadUri`、`state`、`createTime`、`updateTime`、`expirationTime`、`videoMetadata` 均为 output-only，调用方不传。

## 端点二：files.get

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/{name=files/*}` |
| Path 参数 | `name`（必填，格式 `files/{id}`） |
| 响应 | 单个 `File` 对象 |

## 端点三：files.list

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/files` |

Query 参数：

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `pageSize` | integer | ✗ | `10` | 最大 `100`。 |
| `pageToken` | string | ✗ | — | 上一页返回的 `nextPageToken`。 |

响应：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `files` | array&lt;File&gt; | 当前页文件。 |
| `nextPageToken` | string | 下一页 token；为空表示终页。 |

## 端点四：files.delete

| 维度 | 值 |
| --- | --- |
| 方法 | DELETE |
| 路径 | `/v1beta/{name=files/*}` |
| Path 参数 | `name`（必填） |
| 响应 | 空 JSON `{}` |

## File 资源字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | 资源名，格式 `files/{id}`；输出字段，由服务端生成（也可在 upload 时指定，名称需匹配 `^[a-z0-9]([a-z0-9-]{0,38}[a-z0-9])?$`，最长 40）。 |
| `displayName` | string | 调用方可写，最多 512 字符。 |
| `mimeType` | string | output-only。 |
| `sizeBytes` | string (int64) | output-only。 |
| `createTime` | string (RFC3339) | output-only。 |
| `updateTime` | string (RFC3339) | output-only。 |
| `expirationTime` | string (RFC3339) | output-only；默认创建后约 48 小时过期。 |
| `sha256Hash` | string (bytes, base64) | output-only。 |
| `uri` | string | output-only，供 `fileData.fileUri` 引用。 |
| `downloadUri` | string | output-only，可下载链接。 |
| `state` | State | output-only。 |
| `source` | Source | output-only。 |
| `error` | google.rpc.Status | 处理失败时存在，结构同 [errors.md](./errors.md) 的 `error` 主体（含 `code` / `message` / `details`）。 |
| `videoMetadata.videoDuration` | string (Duration) | output-only，仅视频文件存在。 |

### State 枚举

| 值 | 说明 |
| --- | --- |
| `STATE_UNSPECIFIED` | 默认占位。 |
| `PROCESSING` | 上传完成但仍在异步处理，不能用于推理。 |
| `ACTIVE` | 可用。 |
| `FAILED` | 处理失败，详见 `error`。 |

> 视频与较大音频需要在 `PROCESSING -> ACTIVE` 之间轮询 `files.get`，状态变为 `ACTIVE` 后再发起 `generateContent`。

### Source 枚举

| 值 | 说明 |
| --- | --- |
| `SOURCE_UNSPECIFIED` | 未指定。 |
| `UPLOADED` | 由用户通过 Files API 上传。 |
| `GENERATED` | 由 Gemini 模型生成（如图像/音频输出）。 |
| `REGISTERED` | 引用自 Google Cloud Storage 的注册文件。 |

## 在 generateContent 中引用

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        { "fileData": { "mimeType": "video/mp4", "fileUri": "https://generativelanguage.googleapis.com/v1beta/files/abc123" } },
        { "text": "总结这段视频。" }
      ]
    }
  ]
}
```

## 错误

参见 [errors.md](./errors.md)。常见：

| HTTP | `status` | 触发原因 |
| --- | --- | --- |
| 400 | `INVALID_ARGUMENT` | MIME 与文件内容不一致、超出大小限制。 |
| 404 | `NOT_FOUND` | `name` 不存在或已过期被回收。 |
| 429 | `RESOURCE_EXHAUSTED` | 项目存储超额或调用频率超限。 |

## 参考

- 端点文档：<https://ai.google.dev/api/files?hl=zh-CN>
- 文件 prompt 指南：<https://ai.google.dev/gemini-api/docs/prompting_with_media>
