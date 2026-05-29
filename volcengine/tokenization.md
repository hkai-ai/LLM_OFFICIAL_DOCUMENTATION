---
source: https://www.volcengine.com/docs/82379/1528728
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 分词（Tokenization）· POST /api/v3/tokenization

> 将文本切分为 token 并返回 token id 与偏移量，可用于估算 token 数、配合 Chat API 的 `logit_bias`（按 token id 调整概率）。当前仅支持文本。

完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/tokenization`

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

## 请求参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | Model ID 或 Endpoint ID。 |
| `text` | string \| string[] | ✓ | 待分词内容，支持字符串或字符串列表。 |

## 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 请求唯一标识。 |
| `model` | string | 实际使用的模型 ID（模型名称-版本）。 |
| `created` | integer | 创建 Unix 时间戳（秒）。 |
| `object` | string | 固定 `list`。 |
| `data` | object[] | 分词结果，见下表。 |

### `data[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 在 `data` 列表的索引。 |
| `object` | string | 固定 `tokenization`。 |
| `total_tokens` | integer | 对应内容的总 token 数。 |
| `token_ids` | integer[] | 分词后词语在词表中的 id 列表。 |
| `offset_mapping` | array<array<integer>> | 每个 token 在原文中的 `[起始索引, 结束索引)`（起始含、结束不含，从 0 开始）。 |

## 示例

### 请求

```bash
curl https://ark.cn-beijing.volces.com/api/v3/tokenization \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "doubao-pro-32k-241215",
    "text": ["天空为什么这么蓝", "花儿为什么这么香"]
  }'
```

### 响应

```json
{
  "object": "list",
  "id": "0217441868631536d992e302715172e42d8bf34590a60cadc221f",
  "model": "doubao-pro-32k-241215",
  "data": [
    {
      "object": "tokenization",
      "index": 0,
      "total_tokens": 4,
      "token_ids": [14539, 4752, 5189, 5399],
      "offset_mapping": [[0, 2], [2, 5], [5, 7], [7, 8]]
    },
    {
      "object": "tokenization",
      "index": 1,
      "total_tokens": 4,
      "token_ids": [45018, 4752, 5189, 2920],
      "offset_mapping": [[0, 2], [2, 5], [5, 7], [7, 8]]
    }
  ],
  "created": 1744186863
}
```

## 参考

- 分词 API：https://www.volcengine.com/docs/82379/1528728
