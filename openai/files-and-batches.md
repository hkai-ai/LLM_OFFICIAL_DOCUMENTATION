---
source: https://developers.openai.com/api/reference/resources/files
fetched_at: 2026-05-19
api_version: N/A
---

# Files & Batches · /v1/files、/v1/batches

> Files 提供二进制资源管理；Batches 提供 24 小时窗口的离线批量推理。两者经常配合：批量请求 JSONL 上传到 Files，再交给 Batches 处理。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | Files 上传用 `multipart/form-data`；Batches CRUD 用 `application/json`。 |

---

## Files

### POST /v1/files（上传）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file` | file | ✓ | — | 文件二进制对象（非文件名）。 |
| `purpose` | string | ✓ | — | 用途：`assistants` / `assistants_output`（系统产物，只读） / `batch` / `batch_output`（只读） / `fine-tune` / `fine-tune-results`（只读） / `vision` / `user_data` / `evals`。 |
| `expires_after` | object | ✗ | — | `{ anchor: "created_at", seconds }`，`seconds` 范围 3600–2592000。到期自动删除。 |

### GET /v1/files

| Query | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `purpose` | string | ✗ | — | 按 purpose 过滤。 |
| `limit` | integer | ✗ | `10000` | 1–10000。 |
| `order` | string | ✗ | `desc` | `asc` / `desc`。 |
| `after` | string | ✗ | — | 分页游标。 |

### GET /v1/files/{file_id}

返回 file object。

### DELETE /v1/files/{file_id}

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | — |
| `object` | string | 固定 `file`。 |
| `deleted` | boolean | `true`。 |

### GET /v1/files/{file_id}/content

返回原始文件流。

### file object

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `file-...` |
| `object` | string | 固定 `file`。 |
| `bytes` | integer | 字节数。 |
| `created_at` | integer | Unix 秒。 |
| `expires_at` | integer | 过期 Unix 秒（可选）。 |
| `filename` | string | 原始文件名。 |
| `purpose` | string | 见上文枚举。 |
| `status` | string | **已弃用**：`uploaded` / `processed` / `error`。 |
| `status_details` | string | **已弃用**：错误详情。 |

### 示例

```bash
curl https://api.openai.com/v1/files \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F purpose="batch" \
  -F file="@requests.jsonl"
```

```json
{
  "id": "file-abc123",
  "object": "file",
  "bytes": 120000,
  "created_at": 1677610602,
  "filename": "requests.jsonl",
  "purpose": "batch"
}
```

---

## Batches

Batches 适用于可容忍延迟的大批量推理。当前仅 `24h` 完成窗口，价格相较实时调用约 5 折。

### POST /v1/batches（创建）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `input_file_id` | string | ✓ | — | 已上传的 JSONL 文件 ID（`purpose: "batch"`）。每行一条请求对象 `{ custom_id, method, url, body }`。 |
| `endpoint` | string | ✓ | — | 批量目标端点。允许值：`/v1/responses` / `/v1/chat/completions` / `/v1/embeddings` / `/v1/completions` / `/v1/moderations` / `/v1/images/generations` / `/v1/images/edits` / `/v1/videos`。 |
| `completion_window` | string | ✓ | — | 当前仅支持 `24h`。 |
| `metadata` | object | ✗ | — | 最多 16 对 K/V。 |
| `output_expires_after` | object | ✗ | — | 同 file 的 `expires_after`，作用于 output_file / error_file。 |

### GET /v1/batches/{batch_id}

返回 batch object。

### POST /v1/batches/{batch_id}/cancel

发起取消（变 `cancelling` → `cancelled`）。

### GET /v1/batches

| Query | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `limit` | integer | `20` | 1–100。 |
| `after` | string | — | 游标。 |

### batch object

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `batch_...` |
| `object` | string | 固定 `batch`。 |
| `endpoint` | string | 回显。 |
| `errors` | object \| null | `{ object: "list", data: [{ code, line, message, param }] }`。 |
| `input_file_id` | string | 回显。 |
| `completion_window` | string | 回显。 |
| `status` | string | `validating` / `failed` / `in_progress` / `finalizing` / `completed` / `expired` / `cancelling` / `cancelled`。 |
| `output_file_id` | string \| null | 成功请求结果文件。 |
| `error_file_id` | string \| null | 失败请求文件。 |
| `created_at` | integer | — |
| `in_progress_at` | integer \| null | — |
| `expires_at` | integer | — |
| `finalizing_at` | integer \| null | — |
| `completed_at` | integer \| null | — |
| `failed_at` | integer \| null | — |
| `expired_at` | integer \| null | — |
| `cancelling_at` | integer \| null | — |
| `cancelled_at` | integer \| null | — |
| `request_counts` | object | `{ total, completed, failed }`。 |
| `metadata` | object | 回显。 |
| `model` | string | 实际使用模型 ID。 |
| `usage` | object | `{ input_tokens, output_tokens, cached_tokens, reasoning_tokens, total_tokens }`。 |

### 输入 JSONL 行结构

```json
{"custom_id":"req-1","method":"POST","url":"/v1/chat/completions","body":{"model":"gpt-5","messages":[{"role":"user","content":"Hi"}]}}
```

### 输出 JSONL 行结构

```json
{
  "id": "batch_req_xxx",
  "custom_id": "req-1",
  "response": {
    "status_code": 200,
    "request_id": "req_xxx",
    "body": { "id": "chatcmpl-...", "...": "..." }
  },
  "error": null
}
```

失败行 `response` 为 null，`error` 含 `{ code, message }`。

### 示例

```json
{
  "input_file_id": "file-abc123",
  "endpoint": "/v1/chat/completions",
  "completion_window": "24h",
  "metadata": { "campaign": "summer-2026" }
}
```

```json
{
  "id": "batch_abc",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "status": "in_progress",
  "input_file_id": "file-abc123",
  "completion_window": "24h",
  "created_at": 1730000000,
  "expires_at": 1730086400,
  "request_counts": { "total": 100, "completed": 0, "failed": 0 }
}
```

## 错误码

| HTTP | error.type | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | purpose 不允许、completion_window 非法、JSONL 行格式错误。 |
| `401` | `authentication_error` | API key 无效。 |
| `404` | `not_found_error` | file_id / batch_id 不存在。 |
| `409` | `conflict` | 重复取消、batch 已终态。 |
| `429` | `rate_limit_error` | 触发并发或 quota 限制。 |

## 参考

- Files：<https://developers.openai.com/api/reference/resources/files>
- Batches：<https://developers.openai.com/api/reference/resources/batches>
- Batches 创建：<https://developers.openai.com/api/reference/resources/batches/methods/create>
