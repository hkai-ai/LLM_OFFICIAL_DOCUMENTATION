---
source: https://developers.openai.com/api/reference/resources/vector_stores/methods/create
fetched_at: 2026-05-26
api_version: v1
---

# Vector Stores · /v1/vector_stores

托管向量数据库，与 `file_search` 工具（Responses / Assistants v2）配套；先建 store，再把 Files API 上传的 file 通过 `vector_stores.files` 或 `vector_stores.file_batches` 索引进来，随后在 Responses / Search 端点引用。

| 资源 | 路径前缀 |
| --- | --- |
| Vector Stores | `/v1/vector_stores` |
| Files（store 内文件） | `/v1/vector_stores/{vector_store_id}/files` |
| File Batches（批量入库） | `/v1/vector_stores/{vector_store_id}/file_batches` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 1. Vector Store CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/vector_stores` |
| Retrieve | GET | `/v1/vector_stores/{vector_store_id}` |
| Update | POST | `/v1/vector_stores/{vector_store_id}` |
| Delete | DELETE | `/v1/vector_stores/{vector_store_id}` |
| List | GET | `/v1/vector_stores` |
| Search | POST | `/v1/vector_stores/{vector_store_id}/search` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✗ | 用户标签。 |
| `description` | string | ✗ | 描述。 |
| `file_ids` | `array<string>` | ✗ | 创建时直接挂入的 file ID 列表（来自 Files API）。 |
| `chunking_strategy` | object | ✗ | 切分策略，见下。 |
| `expires_after` | object | ✗ | `{ "anchor": "last_active_at", "days": 7 }`，过期自动删 store。 |
| `metadata` | object | ✗ | 自定义 KV。 |

### chunking_strategy

| `type` | 说明 |
| --- | --- |
| `auto` | OpenAI 自动决定（默认）。 |
| `static` | `{"type":"static","static":{"max_chunk_size_tokens":int,"chunk_overlap_tokens":int}}`。 |

### VectorStore 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `vs_...`。 |
| `object` | string | 固定 `vector_store`。 |
| `name` / `description` | string | 同请求。 |
| `status` | enum | `expired` / `in_progress` / `completed`。 |
| `file_counts` | object | `{ in_progress, completed, failed, cancelled, total }`。 |
| `bytes` | integer | 已使用字节（计费基础）。 |
| `usage_bytes` | integer | 同 `bytes`，新版字段。 |
| `expires_after` | object | 同请求。 |
| `expires_at` | integer | Unix 秒，过期时刻。 |
| `created_at` | integer | — |
| `last_active_at` | integer | 最近一次被读 / 写时间。 |
| `metadata` | object | 同请求。 |

### Update body

可改字段：`name` / `description` / `expires_after` / `metadata`。其余字段不可修改。

### List query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` / `before` | string | — | 游标。 |
| `limit` | integer | `20` | `1`–`100`。 |
| `order` | enum | `desc` | `asc` / `desc`，按 `created_at`。 |

### Search · POST /v1/vector_stores/{id}/search

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `query` | `string \| array<string>` | ✓ | 查询文本。 |
| `max_num_results` | integer | ✗ | 默认 `10`，最大 `50`。 |
| `filters` | object | ✗ | 按 `attributes` 做相等 / 比较过滤；语法同 Responses file_search 工具。 |
| `ranking_options` | object | ✗ | `{ "score_threshold": number, "ranker": "auto" \| "default-2024-11-15" }`。 |
| `rewrite_query` | boolean | ✗ | 是否让模型先改写查询。 |

返回：

```json
{
  "object": "vector_store.search_results.page",
  "search_query": "...",
  "data": [
    {
      "file_id": "file-...",
      "filename": "...",
      "score": 0.812,
      "attributes": { "...": "..." },
      "content": [ { "type": "text", "text": "..." } ]
    }
  ],
  "has_more": false,
  "next_page": null
}
```

## 2. Vector Store Files · /v1/vector_stores/{id}/files

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/vector_stores/{id}/files` |
| Retrieve | GET | `/v1/vector_stores/{id}/files/{file_id}` |
| Update | POST | `/v1/vector_stores/{id}/files/{file_id}` |
| Delete | DELETE | `/v1/vector_stores/{id}/files/{file_id}` |
| List | GET | `/v1/vector_stores/{id}/files` |
| Retrieve content | GET | `/v1/vector_stores/{id}/files/{file_id}/content` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file_id` | string | ✓ | Files API 上传的 file ID。 |
| `chunking_strategy` | object | ✗ | 覆盖 store 默认策略。 |
| `attributes` | object | ✗ | KV，可在 search `filters` 中引用。 |

### VectorStoreFile 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `vsf_...`。 |
| `object` | string | 固定 `vector_store.file`。 |
| `vector_store_id` | string | 所属 store。 |
| `status` | enum | `in_progress` / `completed` / `cancelled` / `failed`。 |
| `chunking_strategy` | object | 实际生效（解析 `auto` 后）。 |
| `last_error` | `object \| null` | `{ "code", "message" }`。 |
| `created_at` | integer | — |
| `updated_at` | integer | — |
| `usage_bytes` | integer | 索引占用字节。 |
| `attributes` | object | 同请求。 |

### Content 接口

`GET .../content` 返回 chunk 列表：

```json
{
  "object": "list",
  "data": [ { "type": "text", "text": "...chunk content..." } ],
  "has_more": false,
  "next_page": null
}
```

## 3. File Batches · /v1/vector_stores/{id}/file_batches

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create batch | POST | `/v1/vector_stores/{id}/file_batches` |
| Retrieve batch | GET | `/v1/vector_stores/{id}/file_batches/{batch_id}` |
| List files in batch | GET | `/v1/vector_stores/{id}/file_batches/{batch_id}/files` |
| Cancel batch | POST | `/v1/vector_stores/{id}/file_batches/{batch_id}/cancel` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file_ids` | `array<string>` | ✓ | 单批最多 500 个 file ID。 |
| `chunking_strategy` | object | ✗ | 同上。 |
| `attributes` | object | ✗ | 所有文件共享的 attributes。 |

### VectorStoreFileBatch 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `vsfb_...`。 |
| `object` | string | 固定 `vector_store.files_batch`。 |
| `vector_store_id` | string | — |
| `status` | enum | `in_progress` / `completed` / `cancelled` / `failed`。 |
| `file_counts` | object | `{ in_progress, completed, failed, cancelled, total }`。 |
| `created_at` | integer | — |

## 在 Responses 中引用

```bash
curl https://api.openai.com/v1/responses \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4.1",
    "tools": [{
      "type": "file_search",
      "vector_store_ids": ["vs_abc"]
    }],
    "input": "公司差旅政策？"
  }'
```

## 计费

按 store 已索引内容大小（`bytes` / `usage_bytes`）日计费；详见 [pricing.md](./pricing.md) §Vector stores。Search 请求本身按 token 计费（查询会被 rewrite）。

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | file_ids 重复、chunking_strategy 越界 |
| `404` | `not_found_error` | store / file 不存在 |
| `409` | `invalid_request_error` | 对 `expired` store 写入 |
| `429` | `rate_limit_error` | 索引并发上限 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/vector_stores>
- 创建：<https://developers.openai.com/api/reference/resources/vector_stores/methods/create>
- File Search 工具指南：<https://developers.openai.com/api/docs/guides/tools-file-search>
- Files API：[files-and-batches.md](./files-and-batches.md)
