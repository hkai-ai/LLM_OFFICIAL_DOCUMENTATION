---
source: https://docs.bigmodel.cn/api-reference/知识库-api
fetched_at: 2026-05-26
api_version: v1（注：路径在 `/llm-application/open/...` 与 `/zrag/...` 下，不在 `/paas/v4`）
---

# 知识库 API

智谱托管的 RAG 知识库：建知识库 → 上传 / URL / 解析文档 → 自动切片 + 向量化 → 通过检索接口召回，或在 Agent / Chat 中通过 `tools.retrieval` 引用。

| 资源 | 路径前缀 |
| --- | --- |
| 知识库 | `/llm-application/open/knowledge` |
| 文档 | `/llm-application/open/document` |
| 检索 | `/zrag/retrieval/*` |
| 问答 Agent（流式） | `/llm-application/open/v3/application/invoke` |

> Base URL：`https://open.bigmodel.cn/api`。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ZHIPU_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`，文件上传走 `multipart/form-data`。 |

## 1. 知识库 CRUD · /llm-application/open/knowledge

### 端点

| 动作 | METHOD | PATH | 文档 |
| --- | --- | --- | --- |
| 创建 | POST | `/llm-application/open/knowledge` | 创建知识库 |
| 列表 | GET | `/llm-application/open/knowledge` | 知识库列表 |
| 详情 | GET | `/llm-application/open/knowledge/{knowledge_id}` | 知识库详情 |
| 编辑 | PUT | `/llm-application/open/knowledge/{knowledge_id}` | 编辑知识库 |
| 删除 | DELETE | `/llm-application/open/knowledge/{knowledge_id}` | 删除知识库 |
| 使用量 | GET | `/llm-application/open/knowledge/used` | 知识库使用量 |

### 创建 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `embedding_id` | integer | ✓ | 向量模型 ID：`3`（embedding-3）/ `11`（text-embedding-v4 等）/ `12`。 |
| `embedding_model` | string | ✗ | 模型字符串名（与 id 二选一即可）。 |
| `name` | string | ✓ | 知识库名。 |
| `description` | string | ✗ | 描述。 |
| `background` | string | ✗ | 背景色选项。 |
| `icon` | string | ✗ | 图标选项。 |
| `contextual` | integer | ✗ | 上下文增强开关：`0` 关 / `1` 开。 |

### 通用响应

```json
{
  "code": 200,
  "message": "success",
  "timestamp": 1716700000,
  "data": { /* 端点相关 */ }
}
```

创建后 `data.id` 即 `knowledge_id`。

### 使用量响应

```json
{
  "data": {
    "used": 12345678,
    "total": 1073741824,
    "expire_time": 1716700000
  }
}
```

## 2. 文档管理 · /llm-application/open/document

### 端点

| 动作 | METHOD | PATH | 备注 |
| --- | --- | --- | --- |
| 上传文件文档 | POST | `/llm-application/open/document/upload_document/{knowledge_id}` | `multipart/form-data` 表单 `file=@<path>`。 |
| 上传 URL 文档 | POST | `/llm-application/open/document/upload_url/{knowledge_id}` | body：`{ "url": "https://...", "knowledge_type": 1 }`。 |
| 解析文档图片 | POST | `/llm-application/open/document/parse-image` | 把文档内图片单独抽出 OCR。 |
| 列表 | GET | `/llm-application/open/document` | query：`knowledge_id` / `page` / `size` / `purpose`。 |
| 详情 | GET | `/llm-application/open/document/{document_id}` | — |
| 删除 | DELETE | `/llm-application/open/document/{document_id}` | — |
| 重新向量化 | POST | `/llm-application/open/document/embedding/{document_id}` | 切片或参数变更后重建索引。 |

### 文档对象字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 文档 ID。 |
| `custom_separator` | array | 自定义切片分隔符。 |
| `sentence_size` | integer | 切片大小。 |
| `knowledge_type` | integer | `1` 文本 / `2` URL / `3` 多模态。 |
| `file_name` | string | — |
| `file_size` | integer | 字节。 |
| `failed_info` | object \| null | 失败原因。 |
| `status` | integer | `1` 处理中 / `2` 完成 / `3` 失败 / `4` 未启用。 |
| `embedding_stat` | integer | 向量化进度。 |
| `word_num` | integer | 字符数。 |

## 3. 检索 · /zrag/retrieval

### 知识库检索（传统）· POST /zrag/retrieval/retrieve

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `knows` | `array<{knowledge_id, doc_ids?}>` | ✓ | 范围。 |
| `query` | string | ✓ | 查询文本。 |
| `top_k` | integer | ✗ | 最终召回条数。 |
| `recall_method` | enum | ✗ | `embedding` / `keyword` / `mixed`。 |
| `enable_rerank` | boolean | ✗ | 重排。 |
| `enable_rewrite` | boolean | ✗ | 查询重写。 |
| `enable_expansion` | boolean | ✗ | 查询扩展。 |
| `messages` | array | ✗ | 多轮对话上下文。 |

### 全模态检索 · POST /zrag/retrieval/retrieve

与传统检索复用同一端点，但带：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `multimodal` | boolean | 启用多模态检索。 |
| `multimodal_parts` | `array<{image_url}>` | 以图搜文 / 图。 |

### 响应

```json
{
  "code": 200,
  "data": {
    "rewritten_query": "...",
    "contents": [
      {
        "text": "...",
        "image_url": "...",
        "video_url": "...",
        "score": 0.83,
        "metadata": { "knowledge_id": "...", "document_id": "...", "chunk_id": "..." }
      }
    ]
  }
}
```

## 4. 问答 Agent 对话（流式）

为「知识库 + LLM」一体化的对话场景提供的包装端点。

| 端点 | METHOD | PATH |
| --- | --- | --- |
| 流式问答 | POST | `/llm-application/open/v3/application/invoke` |

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `app_id` | string | ✓ | 应用 ID（在 Console 配置好知识库 + LLM 后给出）。 |
| `messages` | array | ✓ | 对话历史。 |
| `stream` | boolean | ✗ | 通常设 `true`。 |
| `custom_variables` | object | ✗ | 应用变量。 |

### 流式事件

```text
data: {"id":"...","choices":[{"delta":{"content":"片段"}}]}
data: {"id":"...","choices":[{"delta":{"content":""},"finish_reason":"stop"}],"references":[{...}]}
data: [DONE]
```

`references[]` 为知识库引用，含 `document_id` / `chunk_id` / `score` / `text`。

## 错误码

| HTTP | code | 触发 |
| --- | --- | --- |
| `400` | `1011` | knowledge_id 无效 / embedding_id 不在允许集 |
| `404` | `1014` | document 不存在 |
| `409` | `1015` | 重复向量化（任务正在跑） |
| `413` | `1016` | 单文件超大 |
| `429` | `1113` | 触发知识库 API 限流 |

完整错误码见 [errors.md](./errors.md)。

## 参考

- 知识库 API 列表：<https://docs.bigmodel.cn/api-reference/知识库-api>
- 创建知识库：<https://docs.bigmodel.cn/api-reference/知识库-api/创建知识库>
- 全模态检索：<https://docs.bigmodel.cn/api-reference/知识库-api/全模态知识库检索>
- 问答 Agent（流式）：<https://docs.bigmodel.cn/api-reference/agent-api/问答-agent-对话（流式）>
- 文件 API：[files.md](./files.md)
- Embeddings：[embeddings.md](./embeddings.md)
