---
source: https://platform.claude.com/docs/en/api/creating-message-batches
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01
---

# Message Batches · /v1/messages/batches

异步批量执行多条 `POST /v1/messages` 请求。每个 batch 最多 24 小时内完成；成功 / 失败请求按 batch 档计费（详见 [pricing.md](./pricing.md)）。本目录包含五个端点：

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/messages/batches` |
| Retrieve | GET | `/v1/messages/batches/{message_batch_id}` |
| List | GET | `/v1/messages/batches` |
| Cancel | POST | `/v1/messages/batches/{message_batch_id}/cancel` |
| Results | GET | `/v1/messages/batches/{message_batch_id}/results` |

> 请求体上限：Message Batches 端点为 **256 MB**（比单次 `/v1/messages` 的 32 MB 大）。

## 公共鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` / `Authorization` | 二选一 | API Key 或 Workload Identity Bearer Token。 |
| `anthropic-version` | ✓ | 例如 `2023-06-01`。 |
| `content-type` | Create 时 ✓ | `application/json`。 |
| `anthropic-beta` | ✗ | Batches GA 后不再强制；若需使用其他 beta 能力可叠加。 |

## Create · POST /v1/messages/batches

### 请求 Body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `requests` | `array<BatchRequest>` | ✓ | 批次内每条 Messages 请求。单批最多 **100 000** 条。 |
| `requests[].custom_id` | string | ✓ | 用户提供的请求 ID，必须在本批次内唯一；结果文件用它回查。 |
| `requests[].params` | object | ✓ | 与 `POST /v1/messages` body 等价（包含 `model` / `messages` / `max_tokens` / `system` / `tools` / `thinking` / `cache_control` 等所有字段）。 |

> `params` 内可用全部 Messages 字段；`stream` 在 batch 中无意义、`metadata.user_id`、`service_tier` 等仍可声明（实际生效以 batch 档为准）。

### 返回 · MessageBatch

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 唯一 ID，例如 `msgbatch_013Zva2CMHLNnXjNJJKqJ2EF`。 |
| `type` | string | 固定 `message_batch`。 |
| `processing_status` | enum | `in_progress` / `canceling` / `ended`。 |
| `request_counts` | object | 各状态请求数（见下）。 |
| `created_at` | string | RFC 3339。 |
| `expires_at` | string | 创建后 24 小时；超时进入 `ended` 且未完成请求计为 `expired`。 |
| `ended_at` | string | 处理结束（成功 / 失败 / 取消 / 过期任一终态）时填写。 |
| `cancel_initiated_at` | string | 触发取消时填写。 |
| `archived_at` | string | 结果归档不再可下载时填写。 |
| `results_url` | string | 处理结束后给出，指向 `.jsonl` 结果。 |

### request_counts

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `processing` | number | 处理中。**`processing_status: ended` 之前**其他字段恒为 `0`。 |
| `succeeded` | number | 成功条数。 |
| `errored` | number | 错误条数。 |
| `canceled` | number | 取消条数。 |
| `expired` | number | 24 小时未完成被丢弃的条数。 |

### 最小请求

```bash
curl https://api.anthropic.com/v1/messages/batches \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "requests": [
      {
        "custom_id": "my-custom-id-1",
        "params": {
          "model": "claude-opus-4-7",
          "max_tokens": 1024,
          "messages": [{"role": "user", "content": "Hello, world"}]
        }
      }
    ]
  }'
```

### 最小响应

```json
{
  "id": "msgbatch_013Zva2CMHLNnXjNJJKqJ2EF",
  "type": "message_batch",
  "processing_status": "in_progress",
  "request_counts": {
    "processing": 1,
    "succeeded": 0,
    "errored": 0,
    "canceled": 0,
    "expired": 0
  },
  "created_at": "2026-05-26T18:37:24.100435Z",
  "expires_at": "2026-05-27T18:37:24.100435Z",
  "ended_at": null,
  "cancel_initiated_at": null,
  "archived_at": null,
  "results_url": null
}
```

## Retrieve · GET /v1/messages/batches/{message_batch_id}

幂等查询，可用于轮询。返回结构与创建响应一致。

```bash
curl https://api.anthropic.com/v1/messages/batches/$MESSAGE_BATCH_ID \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01"
```

## List · GET /v1/messages/batches

按创建倒序返回当前 Workspace 内全部 batches。

### Query 参数

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `before_id` | string | — | 游标：返回该 ID 之前的一页。 |
| `after_id` | string | — | 游标：返回该 ID 之后的一页。 |
| `limit` | number | `20` | 范围 `1`–`1000`。 |

### 返回

```json
{
  "data": [ /* MessageBatch[] */ ],
  "first_id": "msgbatch_...",
  "last_id": "msgbatch_...",
  "has_more": true
}
```

## Cancel · POST /v1/messages/batches/{message_batch_id}/cancel

进入 `canceling` 状态；系统会完成正在执行且不可中断的请求，其余请求标记为 `canceled`。返回结构与 Retrieve 相同。

> **已完成 / 已开始的不可中断请求仍按对应档位计费**。

## Results · GET /v1/messages/batches/{message_batch_id}/results

只有 `processing_status: ended` 后可调用，返回 **`.jsonl` 流**，每行一个独立结果对象。

### 每行结构 · MessageBatchIndividualResponse

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `custom_id` | string | 对应请求中的 `custom_id`。 |
| `result` | object | 处理结果，type 字段分为四种。 |

#### result.type 枚举

| `type` | 关键字段 | 说明 |
| --- | --- | --- |
| `succeeded` | `message`（与 `/v1/messages` 响应同结构，包含 `content[]` / `model` / `stop_reason` / `usage` 等） | 请求成功 |
| `errored` | `error.error.type` / `error.error.message` / `error.request_id` | 请求失败；`error.type` 枚举见 [errors.md](./errors.md) |
| `canceled` | 仅 `type` | 由 cancel 触发 |
| `expired` | 仅 `type` | 24 小时未处理 |

> 结果顺序不保证与 `requests[]` 输入顺序一致，必须依赖 `custom_id` 回查。

### 下载示例

```bash
curl -N https://api.anthropic.com/v1/messages/batches/$MESSAGE_BATCH_ID/results \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01"
```

每行：

```jsonl
{"custom_id":"my-custom-id-1","result":{"type":"succeeded","message":{"id":"msg_...","type":"message","role":"assistant","model":"claude-opus-4-7","content":[{"type":"text","text":"..."}],"stop_reason":"end_turn","usage":{"input_tokens":11,"output_tokens":18}}}}
{"custom_id":"my-custom-id-2","result":{"type":"errored","error":{"type":"error","request_id":"req_...","error":{"type":"invalid_request_error","message":"..."}}}}
```

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | `requests[]` 为空、`custom_id` 重复、`params` 字段不合法 |
| `404` | `not_found_error` | `message_batch_id` 不存在或不在当前 Workspace |
| `413` | `request_too_large` | 请求体超 256 MB |
| `429` | `rate_limit_error` | 触发租户级 Batch RPM / token 限流 |

完整错误结构见 [errors.md](./errors.md)。

## 参考

- 概念与最佳实践：<https://platform.claude.com/docs/en/docs/build-with-claude/batch-processing>
- Create：<https://platform.claude.com/docs/en/api/creating-message-batches>
- Retrieve：<https://platform.claude.com/docs/en/api/retrieving-message-batches>
- List：<https://platform.claude.com/docs/en/api/listing-message-batches>
- Cancel：<https://platform.claude.com/docs/en/api/canceling-message-batches>
- Results：<https://platform.claude.com/docs/en/api/retrieving-message-batch-results>
