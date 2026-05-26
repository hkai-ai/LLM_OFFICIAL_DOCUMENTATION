---
source: https://docs.bigmodel.cn/api-reference/文件-api/上传文件
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 BigModel Files API

## 鉴权

`Authorization: Bearer <API_KEY>`，Base `https://open.bigmodel.cn/api/paas/v4`。

## 端点速查

| 操作 | 端点 |
| --- | --- |
| 上传 | `POST /paas/v4/files` |
| 列表 | `GET /paas/v4/files` |
| 内容 | `GET /paas/v4/files/{id}/content` |
| 删除 | `DELETE /paas/v4/files/{id}` |

## `purpose` 枚举与规格

| purpose | 支持格式 | 单文件 / 数量限制 |
| --- | --- | --- |
| `batch` | `.jsonl` | 单个 ≤ 100MB，最多 1000 个 |
| `code-interpreter` | pdf / docx / doc / xls / xlsx / txt / png / jpg / jpeg / csv | 单个 ≤ 20MB，图片 ≤ 5MB，最多 100 个 |
| `agent` | 同 code-interpreter | 单个 ≤ 20MB，图片 ≤ 5MB，最多 1000 个 |
| `voice-clone-input` | mp3 / wav | 见 [audio.md §音色复刻](./audio.md)（≤ 10MB，推荐时长 3–30 秒） |
| `file-extract` | 文件解析（见 [tools.md](./tools.md)） | 由 tool_type 决定（lite / expert / prime） |

> 智谱 Files 服务不暴露独立的 `retrieve`（元数据）端点，元数据从 `GET /paas/v4/files` 列表里看；下载内容通过 `GET .../content`。

## 1. 上传 · POST /paas/v4/files

### 请求（multipart/form-data）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file` | binary | ✓ | 文件二进制 |
| `purpose` | string | ✓ | 见 §purpose 枚举 |

### 响应

```json
{
  "id": "file_xyz",
  "object": "file",
  "bytes": 5896337,
  "created_at": 1700469398,
  "filename": "data.jsonl",
  "purpose": "batch"
}
```

## 2. 列表 · GET /paas/v4/files

### 查询参数

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `purpose` | string | 按用途过滤（可选） |
| `limit` | integer | 分页大小 |
| `after` | string | 分页游标 |

### 响应

```json
{
  "object": "list",
  "data": [
    { "id": "file_xyz", "object": "file", "bytes": 5896337, "created_at": 1700469398, "filename": "data.jsonl", "purpose": "batch" }
  ]
}
```

## 3. 内容 · GET /paas/v4/files/{id}/content

下载文件原始内容（主要用于拉取 Batch 任务输出的 `output_file_id` / `error_file_id`）。

> 智谱并未给所有 purpose 文件提供下载链接；仅 batch 输出 / 工具输出类文件支持。其他场景（如 voice-clone 输出）通过专用 file_id 查询。

## 4. 删除 · DELETE /paas/v4/files/{id}

```json
{
  "id": "file_xyz",
  "object": "file",
  "deleted": true
}
```

## 参考

- 上传：https://docs.bigmodel.cn/api-reference/文件-api/上传文件
- 列表：https://docs.bigmodel.cn/api-reference/文件-api/文件列表
- 内容：https://docs.bigmodel.cn/api-reference/文件-api/文件内容
- 删除：https://docs.bigmodel.cn/api-reference/文件-api/删除文件
