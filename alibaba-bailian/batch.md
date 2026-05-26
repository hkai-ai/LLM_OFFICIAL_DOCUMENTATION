---
source: https://help.aliyun.com/zh/model-studio/batch-interfaces-compatible-with-openai
fetched_at: 2026-05-26
api_version: N/A
---

# Batch 异步批量推理（OpenAI 兼容） · /compatible-mode/v1/batches

将多条 chat / embeddings 请求打包到一份 JSONL 输入文件，由百炼异步执行；**成功计费按对应模型实时价 50% 收取**，失败 / 校验未通过的请求不计费。

> Batch 专用 Base URL（华北 2 / 北京）：`https://batch.dashscope.aliyuncs.com/compatible-mode/v1` — 与实时 endpoint 不同；其他地域用各自 base URL。

## 流程概览

1. `POST /v1/files`（`purpose: "batch"`）上传 JSONL 输入文件，得 `file-batch-xxx`。
2. `POST /v1/batches` 创建批次。
3. 轮询 `GET /v1/batches/{batch_id}` 直至 `status: completed`。
4. `GET /v1/files/{output_file_id}/content` 下载结果 JSONL；失败行去 `error_file_id`。

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DASHSCOPE_API_KEY}` |
| `Content-Type` | ✓ | JSON 接口为 `application/json`；上传文件为 `multipart/form-data` |

## 输入文件格式（JSONL）

每行一条独立请求；同一文件内 `url` 必须与创建批次时的 `endpoint` 一致。

```json
{"custom_id":"req-1","method":"POST","url":"/v1/chat/completions","body":{"model":"qwen-plus","messages":[{"role":"user","content":"hi"}]}}
{"custom_id":"req-2","method":"POST","url":"/v1/chat/completions","body":{"model":"qwen-plus","messages":[{"role":"user","content":"hello"}]}}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `custom_id` | string | ✓ | 用户自定义 ID，用于结果回查（结果顺序不保证）。 |
| `method` | string | ✓ | 固定 `POST`。 |
| `url` | string | ✓ | 对应实时 endpoint 路径，目前支持 `/v1/chat/completions`、`/v1/embeddings`、`/v1/chat/ds-test`（调试用）。 |
| `body` | object | ✓ | 与对应实时 endpoint 完全一致的请求体。 |

文件限制：

| 维度 | 限制 |
| --- | --- |
| 单文件大小 | `≤ 500 MB` |
| 单文件请求数 | `≤ 50000` |
| 单 batch 任务总 token | `qwen3.x` 系列最大输入 `256K`；其余按模型实时上限 |

## 上传输入文件 · POST /v1/files

```bash
curl https://dashscope.aliyuncs.com/compatible-mode/v1/files \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -F "purpose=batch" \
  -F "file=@input.jsonl"
```

| 表单字段 | 必填 | 说明 |
| --- | --- | --- |
| `purpose` | ✓ | 固定 `batch`。 |
| `file` | ✓ | JSONL 文件二进制。 |

响应：

```json
{
  "id": "file-batch-xxx",
  "object": "file",
  "bytes": 12345,
  "filename": "input.jsonl",
  "purpose": "batch",
  "status": "processed",
  "created_at": 1716700000
}
```

## 创建批次 · POST /v1/batches

### 请求 Body 字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `input_file_id` | string | ✓ | — | 上一步返回的 `file-batch-xxx`；也可直接传 OSS 路径 `oss://{bucket}/{key}` 或带签名的 OSS URL。 |
| `endpoint` | string | ✓ | — | `/v1/chat/completions` / `/v1/embeddings` / `/v1/chat/ds-test`，必须与 JSONL 内的 `url` 一致。 |
| `completion_window` | string | ✓ | — | 最长等待时间，取值 `24h`–`336h`（最长 14 天）。逾期未完成则进入 `expired`。 |
| `metadata` | `map<string,string>` | ✗ | — | 自定义元数据；常用键：`ds_name`（≤100 字符）、`ds_description`（≤200 字符）、`ds_batch_finish_callback`（完成回调 URL）。 |

### 响应（batch 对象）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 批次 ID。 |
| `object` | string | 固定 `batch`。 |
| `endpoint` | string | 同请求。 |
| `errors` | object \| null | 创建期校验错误；正常为 `null`。 |
| `input_file_id` | string | 同请求。 |
| `completion_window` | string | 同请求。 |
| `status` | string | 见下方枚举。 |
| `output_file_id` | string | 处理完成后生成；存放成功请求结果。 |
| `error_file_id` | string | 存放失败请求；任何请求失败时生成。 |
| `created_at` | integer | Unix 秒。 |
| `in_progress_at` | integer | 进入 `in_progress` 的时间。 |
| `expires_at` | integer | 任务到期时间。 |
| `finalizing_at` | integer | 进入 `finalizing` 的时间。 |
| `completed_at` | integer | 完成时间。 |
| `failed_at` | integer | 失败时间。 |
| `expired_at` | integer | 过期时间。 |
| `cancelling_at` / `cancelled_at` | integer | 取消相关时间。 |
| `request_counts.total` | integer | 请求总数。 |
| `request_counts.completed` | integer | 已完成数。 |
| `request_counts.failed` | integer | 失败数。 |
| `metadata` | object | 同请求。 |

`status` 枚举：

```
validating → in_progress → finalizing → completed
                                      ↘ failed
                                      ↘ expired
            ↘ cancelling → cancelled
```

### 最小请求

```bash
curl https://dashscope.aliyuncs.com/compatible-mode/v1/batches \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "input_file_id": "file-batch-xxx",
    "endpoint": "/v1/chat/completions",
    "completion_window": "24h",
    "metadata": { "ds_name": "demo-batch" }
  }'
```

## 查询单个批次 · GET /v1/batches/{batch_id}

返回结构同创建响应。轮询频率建议 ≥ 10 秒一次。

## 列表批次 · GET /v1/batches

### Query 参数

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` | string | — | 游标 ID。 |
| `limit` | integer | `20` | `1`–`100`。 |
| `ds_name` | string | — | 按 `metadata.ds_name` 模糊匹配。 |
| `input_file_ids` | string | — | 逗号分隔，最多 `20`。 |
| `status` | string | — | 逗号分隔的状态过滤。 |
| `create_after` | string | — | `yyyyMMddHHmmss`。 |
| `create_before` | string | — | `yyyyMMddHHmmss`。 |

### 响应

```json
{
  "object": "list",
  "data": [ /* batch[] */ ],
  "first_id": "batch_xxx",
  "last_id": "batch_yyy",
  "has_more": true
}
```

## 取消批次 · POST /v1/batches/{batch_id}/cancel

仅 `validating` / `in_progress` 状态可取消；进入 `cancelling` 后等待清理，最终 `cancelled`。**已完成请求仍计费**。

返回结构同 batch 对象。

## 下载结果 · GET /v1/files/{file_id}/content

输出 JSONL；每行：

```json
{
  "id": "batch_req_xxx",
  "custom_id": "req-1",
  "response": {
    "status_code": 200,
    "request_id": "...",
    "body": { /* 与实时 endpoint 同结构 */ }
  },
  "error": null
}
```

失败行的 `response.body` 为 `null`，`error` 形如 `{ "code": "...", "message": "..." }`。

## 计费规则

- 成功请求按对应模型实时单价的 **50%** 计费。
- 校验失败 / 模型侧 `failed` / `expired` / `cancelled` 但未生效的请求不计费。
- 取消任务中**已经完成**的请求仍正常计费。

## 错误码

参见 [errors.md](./errors.md)。常见：

| HTTP | code | 触发 |
| --- | --- | --- |
| `400` | `InvalidParameter` | `completion_window` 超出范围 / `endpoint` 与 JSONL 内 `url` 不一致 |
| `400` | `InvalidFile` | 上传文件未通过 JSONL 校验 |
| `404` | `BatchNotFound` | `batch_id` 不存在或不属于当前 API Key |
| `409` | `InvalidState` | 对非可取消状态调用 cancel |

## 参考

- Batch（OpenAI 兼容）：<https://help.aliyun.com/zh/model-studio/batch-interfaces-compatible-with-openai>
- Files 上传：<https://help.aliyun.com/zh/model-studio/upload-files-via-openai-compatible-api>
- 错误码：<https://help.aliyun.com/zh/model-studio/error-code>
