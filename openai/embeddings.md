---
source: https://developers.openai.com/api/reference/resources/embeddings/methods/create
fetched_at: 2026-05-19
api_version: N/A
---

# Embeddings · POST /v1/embeddings

> 将输入文本（或 token 数组）转换为高维向量，常用于检索、聚类、相似度计算。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | `application/json` |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `text-embedding-3-small` / `text-embedding-3-large` / `text-embedding-ada-002`（后者维护态）。 |
| `input` | string \| array&lt;string&gt; \| array&lt;integer&gt; \| array&lt;array&lt;integer&gt;&gt; | ✓ | — | 单条文本、文本数组、token 数组或 token 二维数组。单条 ≤ 8192 tokens，所有 input 合计 ≤ 300000 tokens。不可为空字符串。 |
| `encoding_format` | string | ✗ | `float` | `float` 或 `base64`。`base64` 把向量编码为 base64 字符串，传输更紧凑。 |
| `dimensions` | integer | ✗ | — | 仅 `text-embedding-3-*`：把向量截断到指定维度（最小 1）。`text-embedding-3-small` 默认 1536，`text-embedding-3-large` 默认 3072。 |
| `user` | string | ✗ | — | 终端用户标识，用于 abuse 监控。 |

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `list`。 |
| `data` | array | 向量项数组。 |
| `model` | string | 实际模型 ID。 |
| `usage.prompt_tokens` | integer | 输入 token 数。 |
| `usage.total_tokens` | integer | 总 token 数（与 prompt_tokens 相同）。 |

### `data[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `embedding`。 |
| `index` | integer | 在 input 中的位置索引。 |
| `embedding` | array&lt;number&gt; \| string | `encoding_format: float` 时为浮点数组；`base64` 时为 base64 字符串。 |

## 示例

### 最小请求

```json
{
  "model": "text-embedding-3-small",
  "input": "The quick brown fox jumped over the lazy dog."
}
```

### 最小响应

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [0.0023064255, -0.009327292, 0.015797347, "..."]
    }
  ],
  "model": "text-embedding-3-small",
  "usage": { "prompt_tokens": 10, "total_tokens": 10 }
}
```

### 多输入 + 维度裁剪

```json
{
  "model": "text-embedding-3-large",
  "input": ["foo", "bar"],
  "dimensions": 512,
  "encoding_format": "base64"
}
```

## 错误码

| HTTP | error.type | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 输入为空、超 token、`dimensions` 对不支持的模型使用。 |
| `401` | `authentication_error` | API key 无效。 |
| `429` | `rate_limit_error` | 触发限流或配额耗尽。 |

## 参考

- 端点文档：<https://developers.openai.com/api/reference/resources/embeddings/methods/create>
- 使用指南：<https://developers.openai.com/api/docs/guides/embeddings>
