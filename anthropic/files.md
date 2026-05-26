---
source: https://platform.claude.com/docs/en/api/files-create
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01；anthropic-beta: files-api-2025-04-14
---

# Files API · /v1/files（Beta）

把 PDF / 图像 / 文本等文件预先上传到 Anthropic 侧，Messages 请求只引用 `file_id` 即可，无需每次都重传 base64。**当前仍是 beta**：每个请求必须带 header `anthropic-beta: files-api-2025-04-14`。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Upload | POST | `/v1/files` |
| List | GET | `/v1/files` |
| Retrieve metadata | GET | `/v1/files/{file_id}` |
| Download content | GET | `/v1/files/{file_id}/content` |
| Delete | DELETE | `/v1/files/{file_id}` |

> 请求体上限：Files 端点为 **500 MB**。

## 公共鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` / `Authorization` | 二选一 | API Key 或 Workload Identity Bearer。 |
| `anthropic-version` | ✓ | 例如 `2023-06-01`。 |
| `anthropic-beta` | ✓ | 必填 `files-api-2025-04-14`；如需叠加其他 beta，用逗号分隔。 |
| `content-type` | Upload 时 ✓ | `multipart/form-data`。 |

## File 对象

所有读取类端点返回相同结构：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 唯一 ID，例如 `file_011CNha8iCJcU1wXNR6q4V8w`。 |
| `type` | string | 固定 `file`。 |
| `created_at` | string | RFC 3339。 |
| `filename` | string | 上传时的原始文件名。 |
| `mime_type` | string | 服务端识别到的 MIME，如 `application/pdf`、`image/png`、`text/plain`。 |
| `size_bytes` | number | 字节数。 |
| `downloadable` | boolean | 是否允许通过 `GET .../content` 下载（部分场景生成的文件不可重新下载）。 |
| `scope` | object | 可选；该文件所属的作用域，例如 `{ "id": "session_...", "type": "session" }`。无 scope 表示账号级文件。 |

## Upload · POST /v1/files

`multipart/form-data` 表单：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `file` | ✓ | 文件字节流；常用类型见 [Working with files](https://platform.claude.com/docs/en/docs/build-with-claude/files)。 |

### 示例

```bash
curl https://api.anthropic.com/v1/files \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: files-api-2025-04-14" \
  -F "file=@/path/to/document.pdf"
```

```json
{
  "id": "file_011CNha8iCJcU1wXNR6q4V8w",
  "type": "file",
  "created_at": "2026-05-26T18:37:24.100435Z",
  "filename": "document.pdf",
  "mime_type": "application/pdf",
  "size_bytes": 102400,
  "downloadable": false,
  "scope": null
}
```

## List · GET /v1/files

### Query 参数

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `before_id` | string | — | 游标。 |
| `after_id` | string | — | 游标。 |
| `limit` | number | `20` | 范围 `1`–`1000`。 |
| `scope_id` | string | — | 只返回某个作用域（如 session）下的文件。 |

### 响应

```json
{
  "data": [
    {
      "id": "file_011CNha8iCJcU1wXNR6q4V8w",
      "type": "file",
      "created_at": "2026-05-26T18:37:24.100435Z",
      "filename": "document.pdf",
      "mime_type": "application/pdf",
      "size_bytes": 102400,
      "downloadable": false
    }
  ],
  "first_id": "file_011CNha8iCJcU1wXNR6q4V8w",
  "last_id": "file_013Zva2CMHLNnXjNJJKqJ2EF",
  "has_more": true
}
```

## Retrieve metadata · GET /v1/files/{file_id}

返回单个 File 对象。

```bash
curl https://api.anthropic.com/v1/files/$FILE_ID \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: files-api-2025-04-14"
```

## Download content · GET /v1/files/{file_id}/content

返回原始文件字节流（`Content-Type` 与 `mime_type` 一致）；`downloadable: false` 的文件调用此端点会得到 `400` 或 `404`。

```bash
curl https://api.anthropic.com/v1/files/$FILE_ID/content \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: files-api-2025-04-14" \
  -o downloaded.pdf
```

## Delete · DELETE /v1/files/{file_id}

```bash
curl -X DELETE https://api.anthropic.com/v1/files/$FILE_ID \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: files-api-2025-04-14"
```

返回：

```json
{ "id": "file_011CNha8iCJcU1wXNR6q4V8w", "type": "file_deleted" }
```

## 在 Messages 中引用文件

上传后可在 `messages[].content[]` 中作为不同 block 引用：

| 用途 | content block |
| --- | --- |
| PDF / 文本文档 | `{"type":"document","source":{"type":"file","file_id":"file_..."}}` |
| 图像 | `{"type":"image","source":{"type":"file","file_id":"file_..."}}` |
| 服务端容器读取（code execution） | `container_upload` block：`{"type":"container_upload","file_id":"file_..."}` |

> 文件引用支持配合 `cache_control: { type: "ephemeral" }` 做缓存。

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 文件类型不支持 / `downloadable: false` 还想下载 |
| `404` | `not_found_error` | `file_id` 不存在或不在当前 Workspace |
| `413` | `request_too_large` | 文件超 500 MB |
| `429` | `rate_limit_error` | 触发 Files 配额 |

## 参考

- Upload：<https://platform.claude.com/docs/en/api/files-create>
- List：<https://platform.claude.com/docs/en/api/files-list>
- Metadata：<https://platform.claude.com/docs/en/api/files-metadata>
- Content：<https://platform.claude.com/docs/en/api/files-content>
- Delete：<https://platform.claude.com/docs/en/api/files-delete>
- Working with files：<https://platform.claude.com/docs/en/docs/build-with-claude/files>
