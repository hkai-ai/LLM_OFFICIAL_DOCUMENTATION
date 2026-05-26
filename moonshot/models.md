---
source: https://platform.kimi.com/docs/api/list-models
fetched_at: 2026-05-20
api_version: N/A
---

# 列出模型 · GET /v1/models

返回当前 API Key 可用的全部模型。

## 请求

无参数；仅需 `Authorization: Bearer ${MOONSHOT_API_KEY}` header。

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | `list`。 |
| `data` | `array<Model>` | 模型列表。 |

### Model 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 模型 ID，例如 `kimi-k2.6`。 |
| `object` | string | `model`。 |
| `created` | integer | 创建时间 Unix 秒。 |
| `owned_by` | string | `moonshot`。 |
| `context_length` | integer | 上下文窗口最大 token 数。 |
| `supports_image_in` | boolean | 是否接受图像输入。 |
| `supports_video_in` | boolean | 是否接受视频输入。 |
| `supports_reasoning` | boolean | 是否支持深度思考模式。 |

## 模型清单（人工整理）

下表汇总当前在售模型，价格单位 ¥ / 1M tokens。

### K2.6 系列

| 模型 ID | 上下文 | 输入（命中） | 输入（未命中） | 输出 | 视觉 | 视频 | 思考 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `kimi-k2.6` | 256K | 1.10 | 6.50 | 27.00 | ✓ | ✓ | ✓（默认开） |

### K2.5 系列

| 模型 ID | 上下文 | 输入（命中） | 输入（未命中） | 输出 | 视觉 | 视频 | 思考 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `kimi-k2.5` | 256K | 0.70 | 4.00 | 21.00 | ✓ | ✓ | ✓（默认开） |

### K2 预览系列（计划 2026-05-25 下线）

| 模型 ID | 上下文 | 思考 | 备注 |
| --- | --- | --- | --- |
| `kimi-k2-0905-preview` | 256K | ✗ | 通用对话。 |
| `kimi-k2-0711-preview` | 128K | ✗ | 早期版本。 |
| `kimi-k2-turbo-preview` | 256K | ✗ | 高吞吐变体。 |
| `kimi-k2-thinking` | 256K | ✓ | 思考模式版。 |
| `kimi-k2-thinking-turbo` | 256K | ✓ | 思考模式 + 高吞吐。 |

> K2 预览系列价格见 [chat-k2 定价页](https://platform.kimi.com/docs/pricing/chat-k2)。

### Moonshot V1 系列

| 模型 ID | 上下文 | 输入 | 输出 | 视觉 |
| --- | --- | --- | --- | --- |
| `moonshot-v1-8k` | 8,192 | 2.00 | 10.00 | ✗ |
| `moonshot-v1-32k` | 32,768 | 5.00 | 20.00 | ✗ |
| `moonshot-v1-128k` | 131,072 | 10.00 | 30.00 | ✗ |
| `moonshot-v1-8k-vision-preview` | 8,192 | 同 8k | 同 8k | ✓ |
| `moonshot-v1-32k-vision-preview` | 32,768 | 同 32k | 同 32k | ✓ |
| `moonshot-v1-128k-vision-preview` | 131,072 | 同 128k | 同 128k | ✓ |

> Moonshot V1 系列不区分缓存命中 / 未命中两档价格，全部按统一输入价计费。

## 响应示例

```json
{
  "object": "list",
  "data": [
    {
      "id": "kimi-k2.6",
      "object": "model",
      "created": 1740000000,
      "owned_by": "moonshot",
      "context_length": 262144,
      "supports_image_in": true,
      "supports_video_in": true,
      "supports_reasoning": true
    },
    {
      "id": "moonshot-v1-128k",
      "object": "model",
      "created": 1700000000,
      "owned_by": "moonshot",
      "context_length": 131072,
      "supports_image_in": false,
      "supports_video_in": false,
      "supports_reasoning": false
    }
  ]
}
```

## 参考

- 端点：https://platform.kimi.com/docs/api/list-models
- 模型总览：https://platform.kimi.com/docs/api/models-overview
- 定价首页：https://platform.kimi.com/docs/pricing/chat
- K2.6 定价：https://platform.kimi.com/docs/pricing/chat-k26
- K2.5 定价：https://platform.kimi.com/docs/pricing/chat-k25
- K2 定价：https://platform.kimi.com/docs/pricing/chat-k2
- V1 定价：https://platform.kimi.com/docs/pricing/chat-v1
