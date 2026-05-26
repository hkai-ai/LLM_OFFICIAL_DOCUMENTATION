---
source: https://help.aliyun.com/zh/model-studio/embedding
fetched_at: 2026-05-26
api_version: N/A
---

# 向量化（Embeddings） · 双协议

阿里百炼提供两套语义对等的 embedding 接口：

- **OpenAI 兼容模式**：`POST /compatible-mode/v1/embeddings`，请求/响应字段与 OpenAI `embeddings` 接口一致。
- **DashScope 原生模式**：`POST /api/v1/services/embeddings/text-embedding/text-embedding`，结构沿用 DashScope 通用约定（`input` / `parameters` / `output`）。

两者公用同一组 `text-embedding-v*` 模型与同一份 API Key（按地域）。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DASHSCOPE_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |
| `X-DashScope-WorkSpace` | 多业务空间时 ✓ | 业务空间 ID |

> 单次请求最多 **10 条文本**；超出需自行分批。

## OpenAI 兼容模式

### 请求 Body 字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | embedding 模型 ID，例如 `text-embedding-v4`。 |
| `input` | `string \| array<string>` | ✓ | — | 待向量化文本或文本数组（≤10 条）。 |
| `dimensions` | integer | ✗ | 模型默认 | 输出向量维度；部分模型支持降维（如 `text-embedding-v4` 默认 `1024`，可设 `768` / `512` 等）。 |
| `encoding_format` | string | ✗ | `float` | `float` 或 `base64`。 |

### 响应

```json
{
  "object": "list",
  "data": [
    { "object": "embedding", "index": 0, "embedding": [0.0123, -0.045, ...] }
  ],
  "model": "text-embedding-v4",
  "usage": { "prompt_tokens": 12, "total_tokens": 12 }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `list`。 |
| `data` | `array<Embedding>` | 与 `input` 顺序一一对应。 |
| `data[].embedding` | `array<number>` 或 base64 string | 向量值，长度等于 `dimensions`。 |
| `data[].index` | integer | 对应 `input` 中的位置。 |
| `model` | string | 实际使用的模型 ID。 |
| `usage.prompt_tokens` | integer | 输入 token 数。 |
| `usage.total_tokens` | integer | 与 `prompt_tokens` 相等（embedding 无输出 token）。 |

### 最小请求示例

```bash
curl https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "text-embedding-v4",
    "input": ["阿里百炼的向量接口"],
    "dimensions": 1024,
    "encoding_format": "float"
  }'
```

## DashScope 原生模式

### 请求 Body 字段

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID。 |
| `input.texts` | `array<string>` | ✓ | — | 待向量化文本数组（≤10 条）。 |
| `parameters.dimension` | integer | ✗ | 模型默认 | 输出维度。 |
| `parameters.text_type` | string | ✗ | `document` | `query` / `document`。`query` 用于检索查询端，`document` 用于被检索语料；同模型下二者向量空间一致但有方向性优化。 |
| `parameters.output_type` | string | ✗ | `dense` | `dense` / `sparse` / `dense&sparse`，仅 `text-embedding-v4` 起支持稀疏向量。 |
| `parameters.instruct` | string | ✗ | — | 任务指令（英文短句），用于指令式嵌入；仅部分模型支持。 |

### 响应

```json
{
  "output": {
    "embeddings": [
      { "text_index": 0, "embedding": [0.0123, -0.045, ...] }
    ]
  },
  "usage": { "total_tokens": 12 },
  "request_id": "..."
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `output.embeddings[].embedding` | `array<number>` | 向量值。 |
| `output.embeddings[].sparse_embedding` | object | 稀疏向量（仅请求了 `sparse` 输出时）。 |
| `output.embeddings[].text_index` | integer | 与 `input.texts` 顺序一一对应。 |
| `usage.total_tokens` | integer | 总 token 数。 |
| `request_id` | string | DashScope 请求 ID，工单排查用。 |

### 最小请求示例

```bash
curl https://dashscope.aliyuncs.com/api/v1/services/embeddings/text-embedding/text-embedding \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "text-embedding-v4",
    "input": { "texts": ["阿里百炼的向量接口"] },
    "parameters": { "dimension": 1024, "text_type": "document" }
  }'
```

## 模型清单

| 模型 ID | 默认维度 | 可选维度 | 单条最大 token | 语种支持 | 备注 |
| --- | --- | --- | --- | --- | --- |
| `text-embedding-v4` | `1024` | `2048` / `1536` / `1024` / `768` / `512` / `256` / `128` / `64` | `8192` | 100+ | 支持 dense + sparse；推荐用于新项目。 |
| `text-embedding-v3` | `1024` | `1024` / `768` / `512` | `8192` | 50+ | — |
| `text-embedding-v2` | `1536` | 固定 | `2048` | 中英及部分小语种 | — |
| `text-embedding-v1` | `1536` | 固定 | `2048` | 主要中英 | 已老旧。 |
| `text-embedding-async-v2` / `-v1` | 同上 | 同上 | `2048` | 同上 | 通过 `/api/v1/services/embeddings/text-embedding/text-embedding/async` 异步提交。 |

> 单价、可降维范围以官方[模型广场](https://bailian.console.aliyun.com/?tab=model#/model-market) 为准。

## 限制

| 维度 | 限制 |
| --- | --- |
| 单次最多文本条数 | `10` |
| 单条最大 token | 见模型清单 |
| 请求体大小 | 默认 1 MB 级别（具体上限文档未明确，超过会触发 `Range.InvalidParameter` / `400`）。 |

## 错误码

错误响应结构与厂商通用结构一致（见 [errors.md](./errors.md)）。常见：

| HTTP | code（示例） | 触发场景 |
| --- | --- | --- |
| `400` | `InvalidParameter` | `input.texts` 为空、单条超 token 上限、`dimension` 不在允许集合 |
| `401` | `InvalidApiKey` | API Key 错误或地域不匹配 |
| `429` | `Throttling` | 触发 RPM / TPM 限流 |
| `500` | `InternalError` | 上游异常，重试即可 |

## 参考

- 文本向量（OpenAI 兼容）：<https://help.aliyun.com/zh/model-studio/embedding>
- 文本向量（DashScope 原生）：<https://help.aliyun.com/zh/model-studio/text-embedding-api-details>
- 模型广场：<https://bailian.console.aliyun.com/?tab=model#/model-market>
- 错误码：<https://help.aliyun.com/zh/model-studio/error-code>
