---
source: https://docs.bigmodel.cn/api-reference/批处理-api/创建批处理任务
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 BigModel Batch API

异步批量处理。提交一个 `.jsonl` 文件，由平台批量执行，结果输出到一个新 file_id。**目前仅支持 `/v4/chat/completions` 端点**。

## 鉴权

`Authorization: Bearer <API_KEY>`，Base `https://open.bigmodel.cn/api/paas/v4`。

## 端点

| 操作 | 端点 |
| --- | --- |
| 创建 | `POST /paas/v4/batches` |
| 列表 | `GET /paas/v4/batches` |
| 检索 | `GET /paas/v4/batches/{id}` |
| 取消 | `POST /paas/v4/batches/{id}/cancel` |

## 1. 创建任务

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `input_file_id` | string | ✓ | — | 已上传的 `.jsonl` 文件 ID（`purpose=batch`） |
| `endpoint` | string | ✓ | — | 当前仅 `/v4/chat/completions` |
| `auto_delete_input_file` | boolean | ✗ | `true` | 是否自动删除原始文件 |
| `metadata` | object | ✗ | — | 键值对，最多 16 个；每键 ≤ 64 字符，每值 ≤ 512 字符 |

### 响应（Batch 对象）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | Batch ID |
| `object` | string | 固定 `batch` |
| `status` | string | `queued` / `validating` / `running` / `completed` / `cancelling` / `cancelled` / `failed` / `expired` |
| `endpoint` | string | 同请求 |
| `input_file_id` | string | 同请求 |
| `output_file_id` | string | 成功结果输出 file_id（任务完成时填充） |
| `error_file_id` | string | 错误结果 file_id（任务完成时填充） |
| `created_at` | integer | Unix 时间戳 |
| `completed_at` | integer | 完成时间戳 |
| `completed` | integer | 已完成请求数 |
| `failed` | integer | 失败请求数 |
| `total` | integer | 请求总数 |
| `metadata` | object | 同请求 |

### 最小请求

```bash
curl -X POST https://open.bigmodel.cn/api/paas/v4/batches \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "input_file_id": "file_123", "endpoint": "/v4/chat/completions" }'
```

### 输入 JSONL 格式

每行一个独立请求，与 Chat Completions 一致：

```jsonl
{"custom_id": "req-1", "method": "POST", "url": "/v4/chat/completions", "body": {"model": "glm-4.7", "messages": [{"role":"user","content":"hi"}]}}
{"custom_id": "req-2", "method": "POST", "url": "/v4/chat/completions", "body": {"model": "glm-4.7", "messages": [{"role":"user","content":"hello"}]}}
```

## 2. 列出 / 检索 / 取消

详细字段同 OpenAI Batches API 兼容设计；详见官方各子页。

## 限制与计费

- 单个 input 文件 ≤ 100MB，单账户最多 1000 个 batch 文件。
- 单个 batch 任务 `metadata` ≤ 16 项。
- **`completion_window`：智谱当前未在请求中暴露此字段**（OpenAI 兼容写法可能不生效），完成时限以服务端默认值为准。
- 批量价具体折扣以 [pricing.md](./pricing.md) 与官方[计费说明](https://www.bigmodel.cn/pricing)为准。

## 参考

- 创建批处理：https://docs.bigmodel.cn/api-reference/批处理-api/创建批处理任务
- 列出批处理：https://docs.bigmodel.cn/api-reference/批处理-api/列出批处理任务
- 检索批处理：https://docs.bigmodel.cn/api-reference/批处理-api/检索批处理任务
- 取消批处理：https://docs.bigmodel.cn/api-reference/批处理-api/取消批处理任务
- Batch Guide：https://docs.bigmodel.cn/cn/guide/tools/batch
- Batch FAQ：https://docs.bigmodel.cn/cn/faq/batch-api-issues
- Batch Prompt 最佳实践：https://docs.bigmodel.cn/cn/best-practice/prompt/batch-prompt
