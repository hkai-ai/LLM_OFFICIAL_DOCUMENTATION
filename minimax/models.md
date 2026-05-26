---
source: https://platform.minimaxi.com/docs/api-reference/api-overview
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 模型清单与 List Models

## 模型清单

> 价格与限速以官方[按量付费定价](https://platform.minimaxi.com/docs/guides/pricing-paygo)与 [pricing.md](./pricing.md) 为准。

### 语言模型（chat）

| 模型 ID | 上下文 | 思考 | 用途 |
| --- | --- | --- | --- |
| `MiniMax-M2.7` | 204,800 | 支持 | 旗舰，含 highspeed 子档 |
| `MiniMax-M2.7-highspeed` | 204,800 | 支持 | 高吞吐低延迟版（价格 2×） |
| `MiniMax-M2.5` | 204,800 | 支持 | 上一代旗舰 |
| `MiniMax-M2.5-highspeed` | 204,800 | 支持 | 上一代高速版 |
| `MiniMax-M2.1` | 204,800 | 支持 | 经济档 |
| `MiniMax-M2` | 204,800 | 支持 | 历史版本 |
| `M2-her` | 204,800 | 支持 | 角色对话特化 |

### 语音合成（speech）

| 模型 ID | 档位 | 备注 |
| --- | --- | --- |
| `speech-2.8-hd` | HD（高保真） | 当前最新 |
| `speech-2.8-turbo` | Turbo（低延迟） | 当前最新 |
| `speech-2.6-hd` | HD | — |
| `speech-2.6-turbo` | Turbo | — |
| `speech-02-hd` | HD | v2 系列 |
| `speech-02-turbo` | Turbo | v2 系列 |
| `speech-01-hd` | HD | v1 系列 |
| `speech-01-turbo` | Turbo | v1 系列 |

### 视频生成（video）

| 模型 ID | 支持时长 | 支持分辨率 |
| --- | --- | --- |
| `MiniMax-Hailuo-2.3` | 6s / 10s | 768P / 1080P（10s 仅 768P） |
| `MiniMax-Hailuo-2.3-Fast` | 6s / 10s | 768P / 1080P（10s 仅 768P） |
| `MiniMax-Hailuo-02` | 6s / 10s | 512P / 768P / 1080P（10s 仅 512P / 768P） |
| `T2V-01-Director` | 6s | 720P |
| `T2V-01` | 6s | 720P |
| `I2V-01-Director` | 6s | 720P |
| `I2V-01-live` | 6s | 720P |
| `I2V-01` | 6s | 720P |
| `S2V-01` | 6s | 720P |

### 图像生成（image）

| 模型 ID | 备注 |
| --- | --- |
| `image-01` | 文生图 / 图生图主力 |
| `image-01-live` | 支持 `style.style_type`（漫画 / 元气 / 中世纪 / 水彩） |

### 音乐生成（music）

| 模型 ID | 备注 |
| --- | --- |
| `music-2.6` | 文生音乐（付费） |
| `music-2.6-free` | 免费版文生音乐 |
| `music-cover` | 翻唱（付费） |
| `music-cover-free` | 免费版翻唱 |

## List Models 端点

### OpenAI 规范

| 项 | 值 |
| --- | --- |
| URL | `GET https://api.minimaxi.com/v1/models` |
| 鉴权 | `Authorization: Bearer <API_KEY>` |
| 响应 | `{ object: "list", data: [{ id, object, created, owned_by }] }` |

### Anthropic 规范

| 项 | 值 |
| --- | --- |
| URL | `GET https://api.minimaxi.com/anthropic/v1/models` |
| 鉴权 | `X-Api-Key: <API_KEY>` |
| 查询参数 | `limit`（integer）/ `after_id`（string）/ `before_id`（string） |
| 响应 | `{ data: [{ id, created_at, display_name, type: "model" }], first_id, last_id, has_more }` |

#### Anthropic 响应示例

```json
{
  "data": [
    {
      "id": "MiniMax-M2.7",
      "created_at": "2026-03-18T02:00:00Z",
      "display_name": "MiniMax-M2.7",
      "type": "model"
    }
  ],
  "first_id": "MiniMax-M2.7",
  "last_id": "MiniMax-M2.7",
  "has_more": false
}
```

> 当前 Anthropic 规范 List Models 仅返回 chat 类语言模型，不包含 speech / video / image / music 模型；这些只能通过 OpenAI 规范 `/v1/models` 或文档清单查询。

## 参考

- API 总览：https://platform.minimaxi.com/docs/api-reference/api-overview
- OpenAI List Models：https://platform.minimaxi.com/docs/api-reference/models/openai/list-models
- Anthropic List Models：https://platform.minimaxi.com/docs/api-reference/models/anthropic/list-models
- 模型体系：https://platform.minimaxi.com/docs/guides/models-intro
