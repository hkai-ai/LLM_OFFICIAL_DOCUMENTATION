---
source: https://www.volcengine.com/docs/82379/1870405
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 文件（Files API）· /api/v3/files

> 上传文件（图片 / 视频 / PDF 等）获得 `file_id`，供 [Responses API](./responses.md) 的 `input_image` / `input_video` / `input_file` 等通过 `file_id` 引用。

| 操作 | 方法 | 路径 | 鉴权 |
| --- | --- | --- | --- |
| 上传文件 | POST | `/api/v3/files` | API Key / Access Key |
| 检索文件 | GET | `/api/v3/files/{file_id}` | API Key / Access Key |
| 查询文件列表 | GET | `/api/v3/files` | API Key / Access Key |
| 删除文件 | DELETE | `/api/v3/files/{file_id}` | API Key / Access Key |

## 上传文件 · POST /api/v3/files

请求体为 `multipart/form-data`。

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file` | file | ✓ | — | 待上传的二进制文件。 |
| `purpose` | string | ✓ | `user_data` | 文件用途。`user_data`：可灵活用于任意用途。 |
| `preprocess_configs` | object \| null | ✗ | — | 不同文件类型的预处理规则，见下表。 |
| `expire_at` | integer | ✗ | 当前+604800 | 存储有效期 UTC Unix 时间戳（秒），范围 `[当前+86400, 当前+2592000]`（1–30 天）。 |

### `preprocess_configs`

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `video.fps` | float \| null | `1` | 视频抽帧频率，范围 `[0.2, 5]`。单视频 token 用量 `[10k, 80k]`。 |
| `video.model` | string | — | 视频理解模型 ID 或 Endpoint ID，仅影响上传时抽帧策略（与推理模型不强耦合）。不传默认按 `doubao-seed-1.8` 之前模型抽帧；`doubao-seed-1.8` 及之后支持 1280 帧。 |

响应：返回一个 [file object](#file-object)。

## file object

上传 / 检索文件返回的对象。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `file`。 |
| `id` | string | 文件唯一标识。 |
| `purpose` | string | 文件用途。 |
| `filename` | string | 文件名。 |
| `bytes` | integer | 文件大小（字节），仅 `status=active` 时返回。 |
| `mime_type` | string | MIME 类型（如 `application/pdf`），仅 `status=active` 时返回。 |
| `created_at` | integer | 上传 Unix 时间戳（秒）。 |
| `expire_at` | integer | 过期 Unix 时间戳（秒）。 |
| `status` | string | `processing`（预处理中，不可用）/ `active`（可用）/ `failed`（失败，见 `error`）。 |
| `error` | object \| null | `status=failed` 时返回，含 `code`、`message`。 |
| `preprocess_configs` | object \| null | 预处理规则回显（`video.fps` / `video.model`）。 |

## 检索文件 · GET /api/v3/files/{file_id}

路径参数 `file_id`（必填）。返回对应 file object。

## 查询文件列表 · GET /api/v3/files

查询参数（Query String）：

| 参数 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` | string \| null | — | 返回该文件 ID 之后的文件。 |
| `limit` | integer | `100` | 单次最大文件数，`1~100`。 |
| `purpose` | string | — | 按用途过滤。 |
| `order` | string | `desc` | 按 `created_at` 排序：`asc` / `desc`。 |

响应：`object`=`list`、`data[]`（file object 数组）、`first_id`、`last_id`、`has_more`。

## 删除文件 · DELETE /api/v3/files/{file_id}

路径参数 `file_id`（必填）。响应：`{ "id", "object": "file", "deleted": true }`。

## 示例

### 上传（含视频抽帧配置）

```bash
curl https://ark.cn-beijing.volces.com/api/v3/files \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -F 'purpose=user_data' \
  -F 'file=@/path/ark_video.mp4' \
  -F 'preprocess_configs[video][fps]=0.3'
```

```json
{
  "object": "file",
  "id": "file-20251018114827-6zgrb",
  "purpose": "user_data",
  "filename": "demo.mp4",
  "bytes": 695110,
  "mime_type": "video/mp4",
  "created_at": 1760759307,
  "expire_at": 1761364107,
  "status": "processing",
  "preprocess_configs": { "video": { "fps": 0.3 } }
}
```

## 参考

- 上传文件：https://www.volcengine.com/docs/82379/1870405
- 检索文件：https://www.volcengine.com/docs/82379/1870406
- 查询文件列表：https://www.volcengine.com/docs/82379/1870407
- 删除文件：https://www.volcengine.com/docs/82379/1870408
- file object：https://www.volcengine.com/docs/82379/1873424
