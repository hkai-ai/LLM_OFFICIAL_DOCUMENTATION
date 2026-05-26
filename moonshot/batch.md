---
source: https://platform.kimi.com/docs/api/batch-create
fetched_at: 2026-05-26
api_version: N/A（OpenAPI 1.0；endpoint 统一 `/v1` 前缀）
---

# Moonshot Kimi · Batch API

异步批量推理。提交一份 `.jsonl` 输入文件，由平台后台执行，结果输出到 `output_file_id`。**计费为标准价的 60%**（详见 [pricing.md](./pricing.md)）。

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

Base URL：`https://api.moonshot.cn/v1`。

## 端点速查

| 操作 | 端点 |
| --- | --- |
| 创建 | `POST /v1/batches` |
| 列表 | `GET /v1/batches` |
| 检索 | `GET /v1/batches/{id}` |
| 取消 | `POST /v1/batches/{id}/cancel` |

## 1. 创建 · POST /v1/batches

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `input_file_id` | string | ✓ | — | 通过 `purpose: "batch"` 上传的 `.jsonl` 文件 ID（见 [files.md](./files.md)）。 |
| `endpoint` | string | ✓ | — | 当前仅 `/v1/chat/completions`。 |
| `completion_window` | string | ✓ | — | 任务完成时限，语义化格式：`12h` / `1d` / `3d`，**最小 12h，最大 7d**。 |
| `metadata` | object | ✗ | — | 自定义键值对，最多 16 个；key ≤ 64 字符，value ≤ 512 字符。 |

### 响应（BatchObject）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | Batch 唯一 ID。 |
| `object` | string | 固定 `batch`。 |
| `status` | string | `validating` / `failed` / `in_progress` / `finalizing` / `completed` / `expired` / `cancelling` / `cancelled` |
| `endpoint` | string | 同请求 |
| `input_file_id` | string | 同请求 |
| `output_file_id` | string \| null | 成功结果文件 ID |
| `error_file_id` | string \| null | 错误条目结果文件 ID |
| `completion_window` | string | 同请求 |
| `created_at` / `in_progress_at` / `expires_at` / `completed_at` | integer | Unix 时间戳，按状态填充 |
| `request_counts` | object | `{ completed, failed, total }` |
| `metadata` | object | 同请求 |

### 最小请求

```json
{
  "input_file_id": "file-abc123",
  "endpoint": "/v1/chat/completions",
  "completion_window": "12h"
}
```

### 最小响应

```json
{
  "id": "batch_abc123",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "input_file_id": "file-abc123",
  "completion_window": "12h",
  "status": "validating",
  "created_at": 1700000000,
  "request_counts": {"completed": 0, "failed": 0, "total": 100}
}
```

## 2. 列表 · GET /v1/batches

支持 `limit` / `after`（游标分页），返回 `{ object: "list", data: [BatchObject] }`。

## 3. 检索 · GET /v1/batches/{id}

返回 BatchObject 当前状态。

## 4. 取消 · POST /v1/batches/{id}/cancel

仅可取消 `validating` / `in_progress` / `finalizing` 状态的任务；调用后任务变为 `cancelling`，最终落到 `cancelled`。

## 输入 JSONL 格式

每行一个独立请求对象，**所有行必须使用同一模型**：

```jsonl
{"custom_id": "req-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "kimi-k2.6", "messages": [{"role":"user","content":"hi"}]}}
{"custom_id": "req-2", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "kimi-k2.6", "messages": [{"role":"user","content":"hello"}]}}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `custom_id` | ✓ | 用户自定义唯一标识，用于在输出文件中关联结果 |
| `method` | ✓ | 固定 `POST` |
| `url` | ✓ | 固定 `/v1/chat/completions` |
| `body` | ✓ | Chat Completions 请求体；字段同 [chat-completions.md](./chat-completions.md) |

## 支持模型与约束

| 项 | 说明 |
| --- | --- |
| 当前支持模型 | `kimi-k2.6` / `kimi-k2.5`（多模态请求亦支持，图 / 视频可用 base64 或 `ms://<file_id>` 引用） |
| 固定参数（不可调整） | `temperature` / `top_p` / `n` / `presence_penalty` / `frequency_penalty` |
| 输入文件 | `.jsonl`，非空，≤ 100MB，单行模型必须一致 |
| `output_file_id` 中的每行格式 | `{ id, custom_id, response: { status_code, body }, error }` |

## 输出与错误

完成后下载 `output_file_id`（通过 `GET /v1/files/{id}/content`，见 [files.md](./files.md)）。每行结构：

```json
{
  "id": "batch_req_xxx",
  "custom_id": "req-1",
  "response": {
    "status_code": 200,
    "body": { /* 完整 chat completion 响应 */ }
  },
  "error": null
}
```

失败条目写入 `error_file_id`，每行携带 `error.code` 与 `error.message`。

## 计费

- 批量推理价为同模型标准价的 **60%**（即 40% 折扣）。
- 具体单价见 [pricing.md §3](./pricing.md)。

## 完整流程

1. **构造 JSONL**：本地生成 `.jsonl` 文件（注意 LF 行结尾、UTF-8 无 BOM）。
2. **上传**：`POST /v1/files`（`purpose=batch`，见 [files.md](./files.md)）。
3. **创建 batch**：`POST /v1/batches` 携带 `input_file_id` + `endpoint` + `completion_window`。
4. **轮询**：定时调用 `GET /v1/batches/{id}` 直到 `status` 变为 `completed` / `failed` / `expired` / `cancelled`。
5. **下载**：从 `output_file_id` 读取成功结果，从 `error_file_id` 读取失败条目。

## 参考

- API：https://platform.kimi.com/docs/api/batch-create
- 列表：https://platform.kimi.com/docs/api/batch-list
- 检索：https://platform.kimi.com/docs/api/batch-retrieve
- 取消：https://platform.kimi.com/docs/api/batch-cancel
- 使用指南：https://platform.kimi.com/docs/guide/use-batch-api
- 控制台用法：https://platform.kimi.com/docs/guide/use-batch-inference
- 定价：https://platform.kimi.com/docs/pricing/batch → [pricing.md §3](./pricing.md)
