---
source: https://ai.google.dev/api/file-search?hl=zh-cn
fetched_at: 2026-05-26
api_version: v1beta
---

# File Search Store · /v1beta/fileSearchStores

Gemini 内置的托管向量检索（RAG）：把文档导入 store，模型在 `generateContent` 中通过 `tools.fileSearch` 自动检索并引用。Anthropic / OpenAI 的「内置 file search 工具」的同位能力。

| 资源 | 路径前缀 |
| --- | --- |
| FileSearchStore | `/v1beta/fileSearchStores` |
| 文档（store 内文件 chunk 化结果） | `/v1beta/{fileSearchStoreName}:importFile` / `:uploadToFileSearchStore` |

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` 或 `?key=` | ✓ | API Key。 |
| `Content-Type` | 写操作 ✓ | `application/json`；直接上传走 `/upload/...`，按 multipart/resumable 协议。 |

## FileSearchStore CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create store | POST | `/v1beta/fileSearchStores` |
| List stores | GET | `/v1beta/fileSearchStores` |
| Retrieve store | GET | `/v1beta/{name=fileSearchStores/*}` |
| Delete store | DELETE | `/v1beta/{name=fileSearchStores/*}?force=true` |

### Create 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `displayName` | string | ✗ | 人类可读名（≤512 字符）。 |
| `embeddingModel` | string | ✗ | 自定义嵌入模型；默认由 Gemini 选定（推荐留空）。 |

### FileSearchStore 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | `fileSearchStores/{id}`，不可变（≤40 字符 / `[A-Za-z0-9-]`）。 |
| `displayName` | string | — |
| `createTime` / `updateTime` | string | RFC 3339。 |
| `activeDocumentsCount` | string (int64) | 索引中可用的文档数。 |
| `pendingDocumentsCount` | string (int64) | 处理中的文档数。 |
| `failedDocumentsCount` | string (int64) | 处理失败文档数。 |
| `sizeBytes` | string (int64) | 已抽取内容总字节数。 |
| `embeddingModel` | string | 同请求。 |

### Delete 行为

`?force=true` 强制删除（即便仍含未清空的文档）；省略则非空时返回 `FAILED_PRECONDITION`。

## 导入 / 上传文件

### Import 已有 File · POST /v1beta/{fileSearchStoreName}:importFile

把通过 [Files API](./files.md) 上传得到的 `files/{file_id}` 索引到 store。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `fileName` | string | ✓ | `files/{file_id}`。 |
| `customMetadata` | object | ✗ | 自定义键值对，可用于过滤检索。 |
| `chunkingConfig` | object | ✗ | 自定义分块配置；省略走默认。 |

返回 Long-Running Operation：

```json
{
  "name": "operations/...",
  "done": false
}
```

完成后 `result.response` 为生成的 Document 对象。

### Direct upload · POST `/upload/v1beta/{fileSearchStoreName}:uploadToFileSearchStore`

直接上传二进制并立即索引，省去先调 Files API 的一步。Content-Type 与 multipart 协议同 [files.md](./files.md)。

## Document 对象（store 内文件）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | `fileSearchStores/{sid}/documents/{did}`。 |
| `displayName` | string | 原始文件名。 |
| `createTime` | string | RFC 3339。 |
| `chunks` | array | 抽取后的 chunk 列表（仅可用于检索，详情查看不返回内容）。 |
| `customMetadata` | object | 同 import 时声明。 |
| `state` | enum | `DOCUMENT_STATE_PENDING` / `DOCUMENT_STATE_ACTIVE` / `DOCUMENT_STATE_FAILED`。 |

## 在 generateContent 中引用

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      { "parts": [{ "text": "我们公司去年的差旅政策是什么？" }] }
    ],
    "tools": [{
      "fileSearch": {
        "fileSearchStoreNames": ["fileSearchStores/policy-2025"]
      }
    }]
  }'
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `tools[].fileSearch.fileSearchStoreNames` | `array<string>` | ✓ | 引用的 store 列表，可同时检索多个。 |
| `tools[].fileSearch.metadataFilter` | object | ✗ | 按 `customMetadata` 过滤。 |

模型会自动调用 file_search 子工具检索 chunk、在 `candidates[].content.parts[]` 输出文本，并在 `candidates[].groundingMetadata` 中给出引用源 chunk。

## 计费要点

- 索引 / 存储费用按 `sizeBytes` 计；详见 [pricing.md](./pricing.md) §File Search。
- 查询时检索本身的 token 计入 `cachedContentTokenCount`（按上下文缓存价计费）。
- Free Tier 提供基础免费额度。

## 限制

- Store 名最长 40 字符；displayName 最长 512 字符。
- Document 大小、并发处理任务数等以官方[配额页](https://ai.google.dev/gemini-api/docs/rate-limits?hl=zh-cn) 为准。

## 参考

- 接口 reference：<https://ai.google.dev/api/file-search?hl=zh-cn>
- 指南：<https://ai.google.dev/gemini-api/docs/file-search?hl=zh-cn>
- 配额：<https://ai.google.dev/gemini-api/docs/rate-limits?hl=zh-cn>
- Files API：[files.md](./files.md)
- Caching：[caching.md](./caching.md)
