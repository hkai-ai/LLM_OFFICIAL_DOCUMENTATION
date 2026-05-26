---
source: https://docs.bigmodel.cn/api-reference/模型-api/文本嵌入
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM Embeddings · POST /paas/v4/embeddings

> 文本向量化服务。OpenAI 兼容 schema。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json` |

完整 URL：`POST https://open.bigmodel.cn/api/paas/v4/embeddings`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `embedding-3` / `embedding-2` |
| `input` | string \| array | ✓ | — | 单条字符串或字符串数组 |
| `dimensions` | integer | ✗ | — | 输出维度。`embedding-3` 取 `256` / `512` / `1024` / `2048`（默认 `2048`）；`embedding-2` 固定 `1024`，传值无效 |

### Token / 数组限制

| 模型 | 单条最大 tokens | 数组限制 |
| --- | --- | --- |
| `embedding-2` | 512 tokens | 数组总 tokens ≤ 8192 |
| `embedding-3` | 3072 tokens | 最多 64 条 |

## 响应

```json
{
  "model": "embedding-3",
  "object": "list",
  "data": [
    {
      "index": 0,
      "object": "embedding",
      "embedding": [0.123, -0.456, ...]
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 0,
    "total_tokens": 10
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `data[].index` | integer | 与 input 数组顺序一致 |
| `data[].embedding` | array<number> | 浮点向量，长度 = `dimensions` |
| `usage.prompt_tokens` | integer | 总输入 token 数（不区分单条 / 数组） |

## 示例

### 最小请求

```json
{
  "model": "embedding-3",
  "input": "你好，今天天气怎么样"
}
```

### 自定义维度 + 批量

```json
{
  "model": "embedding-3",
  "input": ["第一条", "第二条", "第三条"],
  "dimensions": 1024
}
```

## 参考

- 文本嵌入：https://docs.bigmodel.cn/api-reference/模型-api/文本嵌入
- Embedding-2 模型页：https://docs.bigmodel.cn/cn/guide/models/embedding/embedding-2
- Embedding-3 模型页：https://docs.bigmodel.cn/cn/guide/models/embedding/embedding-3
