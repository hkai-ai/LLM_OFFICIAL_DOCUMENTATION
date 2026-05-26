---
source: https://platform.kimi.com/docs/api/files
fetched_at: 2026-05-26
api_version: N/A（OpenAPI 1.0）
---

# Moonshot Kimi · Files API

文件上传服务，用于：内容抽取（文件问答 RAG）、图片 / 视频引用、Batch API 输入。

## 鉴权

`Authorization: Bearer <API_KEY>`，Base `https://api.moonshot.cn/v1`。

## 端点速查

| 操作 | 端点 |
| --- | --- |
| 上传 | `POST /v1/files`（multipart/form-data） |
| 列表 | `GET /v1/files` |
| 检索（元数据） | `GET /v1/files/{id}` |
| 内容（抽取后文本） | `GET /v1/files/{id}/content` |
| 删除 | `DELETE /v1/files/{id}` |

## `purpose` 枚举

| 值 | 用途 |
| --- | --- |
| `file-extract` | 内容抽取（文件问答 / RAG）。上传后服务端会抽取文本，可通过 `/content` 拉取 |
| `image` | 多模态请求中通过 `ms://<file_id>` 引用 |
| `video` | 多模态请求中通过 `ms://<file_id>` 引用 |
| `batch` | Batch API 输入 `.jsonl` 文件，详见 [batch.md](./batch.md) |

## 1. 上传 · POST /v1/files

### 请求（multipart/form-data）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file` | binary | ✓ | 文件二进制 |
| `purpose` | string | ✓ | 见上 |

### 文件限制

- 单文件 ≤ **100MB**
- 单用户最多 **1000** 个文件
- 总容量 ≤ **10GB**

### 支持格式

> 由 `purpose` 决定具体可用集合：

| purpose | 推荐格式 |
| --- | --- |
| `file-extract` | `.pdf` / `.txt` / `.csv` / `.doc` / `.docx` / `.xls` / `.xlsx` / `.ppt` / `.pptx` / `.md` / `.html` / `.json` / `.java` / `.js` / `.py` / `.yaml` / `.yml` 及 `.jpeg` / `.png` / `.bmp` / `.gif` / `.svg` |
| `image` | PNG / JPEG / WebP / GIF |
| `video` | MP4 / MPEG / MOV / AVI / FLV / MPG / WebM / WMV / 3GPP |
| `batch` | `.jsonl` |

### 响应（FileObject）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 文件唯一 ID |
| `object` | string | 固定 `file` |
| `bytes` | integer | 文件大小（字节） |
| `created_at` | integer | Unix 时间戳 |
| `filename` | string | 原始文件名 |
| `purpose` | string | 同请求 |
| `status` | string | 处理状态（如 `ready`） |
| `status_details` | string | 失败 / 警告详情 |

### 最小响应

```json
{
  "id": "file-abc123",
  "object": "file",
  "bytes": 1024,
  "created_at": 1700000000,
  "filename": "doc.pdf",
  "purpose": "file-extract",
  "status": "ready"
}
```

## 2. 列表 · GET /v1/files

返回 `{ object: "list", data: [FileObject] }`。可附查询参数过滤。

## 3. 检索 · GET /v1/files/{id}

返回单个 FileObject。

## 4. 获取内容 · GET /v1/files/{id}/content

- 对 `purpose=file-extract` 文件：返回抽取后的文本内容。
- 对 `purpose=batch` 输出文件（`output_file_id` / `error_file_id`）：返回 `.jsonl` 原始内容，每行一个结果。

## 5. 删除 · DELETE /v1/files/{id}

```json
{
  "id": "file-abc123",
  "object": "file",
  "deleted": true
}
```

## 多模态引用：`ms://<file_id>` 协议

`image` / `video` purpose 上传后，可在 chat completions 的 `image_url.url` / `video_url.url` 字段直接传入 `ms://file-abc123` 引用文件。详见 [vision.md](./vision.md)。

## 参考

- Files API 总览：https://platform.kimi.com/docs/api/files
- 上传：https://platform.kimi.com/docs/api/files-upload
- 列表：https://platform.kimi.com/docs/api/files-list
- 检索：https://platform.kimi.com/docs/api/files-retrieve
- 内容：https://platform.kimi.com/docs/api/files-content
- 删除：https://platform.kimi.com/docs/api/files-delete
- 文件问答 Guide：https://platform.kimi.com/docs/guide/use-kimi-api-for-file-based-qa
