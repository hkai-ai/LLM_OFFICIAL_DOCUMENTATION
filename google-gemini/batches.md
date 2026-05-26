---
source: https://ai.google.dev/api/batch-mode?hl=zh-cn
fetched_at: 2026-05-26
api_version: v1beta
---

# Batch Mode · /v1beta/{model=models/*}:batchGenerateContent

把大量 `GenerateContentRequest`（或 `EmbedContentRequest`）打包到一份输入源，由 Gemini 异步执行；按对应模型实时单价的 **50%** 计费，但**完成 SLA 最长 24 小时**，不保证更快返回。

> Batch 资源名形如 `batches/{batch_id}`。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` 或 `?key=` | ✓ | API Key。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 端点索引

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create batch | POST | `/v1beta/{batch.model=models/*}:batchGenerateContent` |
| Retrieve batch | GET | `/v1beta/{name=batches/*}` |
| List batches | GET | `/v1beta/batches` |
| Cancel batch | POST | `/v1beta/{name=batches/*}:cancel` |
| Delete batch | DELETE | `/v1beta/{name=batches/*}` |

> Embeddings 路径为 `/v1beta/{batch.model=models/*}:batchEmbedContents`，结构同步。

## Create batch · POST /v1beta/{batch.model=models/*}:batchGenerateContent

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `batch.displayName` | string | ✗ | 任意用户标签。 |
| `batch.inputConfig` | object | ✓ | 输入源，二选一：`fileName` 或 `requests`。 |
| `batch.inputConfig.fileName` | string | 二选一 | 之前通过 Files API 上传的 `files/{file_id}`；文件内容为 JSONL，每行一个 `GenerateContentRequest`。 |
| `batch.inputConfig.requests` | object | 二选一 | 内联请求：`{"requests":[{"request":{"contents":[...]},"metadata":{"key":"val"}}, ...]}`。 |
| `batch.priority` | int64 | ✗ | 默认 `0`；同一帐户内大值优先调度。 |

### 响应 · Batch 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | `batches/{batch_id}`。 |
| `displayName` | string | 同请求。 |
| `model` | string | `models/{model_id}`。 |
| `state` | enum | `BATCH_STATE_UNSPECIFIED` / `BATCH_STATE_PENDING` / `BATCH_STATE_RUNNING` / `BATCH_STATE_SUCCEEDED` / `BATCH_STATE_FAILED` / `BATCH_STATE_CANCELLED` / `BATCH_STATE_EXPIRED`。 |
| `createTime` / `updateTime` / `endTime` | string | RFC 3339。 |
| `batchStats.requestCount` | string (int64) | 总请求数。 |
| `batchStats.successfulRequestCount` | string (int64) | 成功数。 |
| `batchStats.failedRequestCount` | string (int64) | 失败数。 |
| `batchStats.pendingRequestCount` | string (int64) | 等待中。 |
| `output.responsesFile` | string | 完成后给出，`files/{file_id}`，内容为 JSONL；每行包含 `metadata`（回显请求时附带的）、`response`（`GenerateContentResponse`） 或 `error`（Status）。 |
| `output.inlinedResponses` | object | 当输入用 `inputConfig.requests` 时直接返回 `{"inlinedResponses":[{"response":..., "metadata":...}]}`。 |
| `priority` | int64 | 同请求。 |

### 最小请求（文件输入）

```bash
# 1. 上传 JSONL 到 Files API
curl -X POST "https://generativelanguage.googleapis.com/upload/v1beta/files?key=$GEMINI_API_KEY" \
  -F "file=@input.jsonl;type=application/jsonl"

# 2. 创建 batch
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:batchGenerateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "batch": {
      "displayName": "evening-eval",
      "inputConfig": { "fileName": "files/abc123" }
    }
  }'
```

### 最小响应

```json
{
  "name": "batches/abc",
  "displayName": "evening-eval",
  "model": "models/gemini-2.5-flash",
  "state": "BATCH_STATE_PENDING",
  "createTime": "2026-05-26T18:37:24.100435Z",
  "updateTime": "2026-05-26T18:37:24.100435Z",
  "batchStats": {
    "requestCount": "1000",
    "successfulRequestCount": "0",
    "failedRequestCount": "0",
    "pendingRequestCount": "1000"
  }
}
```

### JSONL 输入示例（每行一条 GenerateContentRequest）

```jsonl
{"request":{"contents":[{"parts":[{"text":"Translate to French: hello"}]}]},"metadata":{"id":"r-1"}}
{"request":{"contents":[{"parts":[{"text":"Translate to German: hello"}]}]},"metadata":{"id":"r-2"}}
```

> `request` 字段结构与 [generate-content.md](./generate-content.md) 中同名字段完全一致。

## Retrieve batch · GET /v1beta/{name=batches/*}

返回 Batch 对象；可用于轮询。

## List batches · GET /v1beta/batches

### Query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `pageSize` | int32 | — | 单页条数。 |
| `pageToken` | string | — | 翻页 token。 |
| `filter` | string | — | 过滤表达式，例如 `state=BATCH_STATE_SUCCEEDED`。 |

### 响应

```json
{
  "batches": [ /* Batch[] */ ],
  "nextPageToken": "..."
}
```

## Cancel batch · POST /v1beta/{name=batches/*}:cancel

进入 `BATCH_STATE_CANCELLED`；已完成请求仍计费。空 body，返回 `{}`。

## Delete batch · DELETE /v1beta/{name=batches/*}

删除资源记录（结果文件归 Files API 单独管理）。

## 计费

成功请求按对应模型实时定价的 **50%** 收取；详细单价见 [pricing.md](./pricing.md) §Batch。

## 限制与配额

- 输入文件大小、单批请求数、并发任务数等以官方[配额页](https://ai.google.dev/gemini-api/docs/rate-limits?hl=zh-cn) 为准。
- `BATCH_STATE_EXPIRED` 表示 24 小时窗口内未完成；剩余请求不计费但已完成请求按 batch 价计费。

## 参考

- 接口 reference：<https://ai.google.dev/api/batch-mode?hl=zh-cn>
- 使用指南：<https://ai.google.dev/gemini-api/docs/batch-mode?hl=zh-cn>
- Files API：[files.md](./files.md)
- 定价：[pricing.md](./pricing.md)
