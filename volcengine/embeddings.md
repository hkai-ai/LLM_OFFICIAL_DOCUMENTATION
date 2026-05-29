---
source: https://www.volcengine.com/docs/82379/1523520
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 向量化（多模态 Embeddings）· POST /api/v3/embeddings/multimodal

> 将文本、图片、视频转化为向量，用于以图搜图、语义检索等任务。完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/embeddings/multimodal`。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID 或 Endpoint ID，如 `doubao-embedding-vision-250615`。 |
| `input` | object[] | ✓ | — | 待向量化内容列表，元素支持文本 / 图片 / 视频。不同模型支持情况不同。见 [`input[]`](#input)。 |
| `encoding_format` | string \| null | ✗ | `float` | embedding 返回格式：`float` / `base64` / `null`。 |
| `dimensions` | integer | ✗ | `2048` | 输出向量维度：`1024` 或 `2048`。 |
| `instructions` | string | ✗ | — | 推理提示词；未传时按输入模态生成默认值。 |
| `sparse_embedding` | object | ✗ | — | 稀疏向量开关，**仅纯文本输入**支持。`type`：`disabled`（仅稠密向量）/ `enabled`（同时输出稀疏向量）。 |

### `input[]`

由 `type` 区分模态：

#### 文本（`type` = `text`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `text`。 |
| `text` | string | ✓ | 文本内容（UTF-8，长度不超过模型最大输入 token 数）。 |

#### 图片（`type` = `image_url`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `image_url`。 |
| `image_url` | object | ✓ | 含 `url`：图片 URL 或 Base64（格式 `data:image/{格式};base64,{编码}`）。 |

#### 视频（`type` = `video_url`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `video_url`。 |
| `video_url` | object | ✓ | 含 `url`：视频链接或 Base64。格式 `.mp4`/`.avi`/`.mov`（小写），单文件 ≤50 MB，暂不理解音频。 |

## 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 请求唯一标识。 |
| `model` | string | 实际使用的模型名称和版本。 |
| `created` | integer | 创建 Unix 时间戳（秒）。 |
| `object` | string | 固定 `list`。 |
| `data` | object | 算法输出，见下表。 |
| `usage` | object | token 用量，见下表。 |

### `data`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `embedding` | float[] | 稠密向量化结果。 |
| `sparse_embedding` | array | 稀疏向量，仅 `sparse_embedding.type="enabled"` 时返回；每个成员为 `{"index": 维度索引, "value": 非零值}`，仅含非零元素。 |
| `object` | string | 固定 `embedding`。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt_tokens` | integer | 输入内容 token 数。 |
| `total_tokens` | integer | 总 token 数。 |
| `prompt_tokens_details.text_tokens` | integer | 文本内容（含视频时间轴）token 数；传图片 / 视频时会生成少量预设文本 token。 |
| `prompt_tokens_details.image_tokens` | integer | 图片内容及视频抽帧图片 token 数。 |

## 示例

### 请求

```bash
curl https://ark.cn-beijing.volces.com/api/v3/embeddings/multimodal \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "doubao-embedding-vision-250615",
    "encoding_format": "float",
    "input": [
      {"type": "video_url", "video_url": {"url": "https://example.com/v.mp4"}},
      {"type": "image_url", "image_url": {"url": "https://example.com/tower.png"}},
      {"type": "text", "text": "视频和图片里有什么"}
    ]
  }'
```

### 响应

```json
{
  "created": 1743575029,
  "data": {
    "embedding": [-0.123046875, -0.35546875, "...", -0.255859375],
    "object": "embedding"
  },
  "id": "021743575029461acbe49a31755bec77b2f09448eb15fa9a88e47",
  "model": "doubao-embedding-vision-250615",
  "object": "list",
  "usage": {
    "prompt_tokens": 13987,
    "prompt_tokens_details": { "image_tokens": 13800, "text_tokens": 187 },
    "total_tokens": 13987
  }
}
```

## 参考

- 向量化 API：https://www.volcengine.com/docs/82379/1523520
- 模型列表：https://www.volcengine.com/docs/82379/1330310
