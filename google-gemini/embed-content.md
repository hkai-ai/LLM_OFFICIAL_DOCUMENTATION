---
source: https://ai.google.dev/api/embeddings?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 文本嵌入 · POST /v1beta/{model=models/*}:embedContent / :batchEmbedContents

> 将输入内容编码为向量。常用模型 ID：`gemini-embedding-001`、`text-embedding-004`、`embedding-001`。

## 端点一：models.embedContent

### Path 参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 资源名，格式 `models/{model}`。 |

### 请求 Body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `content` | Content | ✓ | — | 待嵌入的内容；多数模型只接受单条 `parts[].text`。 |
| `taskType` | string | ✗ | 模型默认 | 见下方枚举。 |
| `title` | string | ✗ | — | 仅在 `taskType=RETRIEVAL_DOCUMENT` 时生效，作为文档标题参与编码。 |
| `outputDimensionality` | integer | ✗ | 模型默认 | 截断到指定维度（仅 `text-embedding-004` 及之后的模型支持，常见取 `768` / `1536` / `3072`）。 |

### `taskType` 全部枚举

| 值 | 说明 |
| --- | --- |
| `TASK_TYPE_UNSPECIFIED` | 默认占位。 |
| `RETRIEVAL_QUERY` | 检索查询侧。 |
| `RETRIEVAL_DOCUMENT` | 检索文档侧；可配合 `title`。 |
| `SEMANTIC_SIMILARITY` | 语义相似度。 |
| `CLASSIFICATION` | 分类任务。 |
| `CLUSTERING` | 聚类任务。 |
| `QUESTION_ANSWERING` | 问答匹配。 |
| `FACT_VERIFICATION` | 事实校验。 |
| `CODE_RETRIEVAL_QUERY` | 代码检索查询。 |

### 响应（EmbedContentResponse）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `embedding.values` | array&lt;number&gt; | 嵌入向量，长度为 `outputDimensionality` 或模型默认维度。 |
| `embedding.statistics` | object | 部分模型返回，含 `truncated`、`tokenCount` 等元信息（文档未在所有模型上承诺该字段）。 |

## 端点二：models.batchEmbedContents

### Path 参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 资源名，所有子请求需使用同一模型。 |

### 请求 Body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `requests` | array&lt;EmbedContentRequest&gt; | ✓ | — | 子请求列表；每个元素结构与 `embedContent` 的 body 一致，但**必须**额外包含 `model` 字段（值与 path 中一致）。 |

### 响应（BatchEmbedContentsResponse）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `embeddings` | array&lt;ContentEmbedding&gt; | 与 `requests[]` 顺序一一对应。 |

## 示例

### 单条嵌入请求

```json
{
  "content": { "parts": [{ "text": "量子纠缠的定义" }] },
  "taskType": "RETRIEVAL_QUERY",
  "outputDimensionality": 768
}
```

`POST /v1beta/models/gemini-embedding-001:embedContent`。

### 单条嵌入响应

```json
{
  "embedding": {
    "values": [0.013, -0.027, 0.044, "...共 768 维"]
  }
}
```

### 批量请求

```json
{
  "requests": [
    {
      "model": "models/gemini-embedding-001",
      "content": { "parts": [{ "text": "苹果是什么？" }] },
      "taskType": "RETRIEVAL_QUERY"
    },
    {
      "model": "models/gemini-embedding-001",
      "content": { "parts": [{ "text": "苹果是一种蔷薇科植物的果实。" }] },
      "taskType": "RETRIEVAL_DOCUMENT",
      "title": "苹果百科"
    }
  ]
}
```

## 错误

参见 [errors.md](./errors.md)。注意：传入 `taskType` 与所选模型不兼容时返回 `INVALID_ARGUMENT`。

## 参考

- 端点文档：<https://ai.google.dev/api/embeddings?hl=zh-CN>
- 嵌入模型对比：<https://ai.google.dev/gemini-api/docs/embeddings>
