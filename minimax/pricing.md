---
source: https://platform.minimaxi.com/docs/guides/pricing-paygo
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 定价（按量付费 / Pay-As-You-Go）

> 全部价格以 **人民币元（¥）** 计；token 类计价单位为 **每 1M tokens**，其他模态按其各自单位（每万字符 / 每秒 / 每张图 / 每首 / 每音色 / 每次调用）。所有数字源自官方 `https://platform.minimaxi.com/docs/guides/pricing-paygo`，本仓库不做汇率换算与推算。

## 1. 语言模型（每 1M tokens, ¥）

| 模型 ID | Input | Output | Cache Read | Cache Write |
| --- | --- | --- | --- | --- |
| `MiniMax-M2.7` | ¥2.1 | ¥8.4 | ¥0.42 | ¥2.625 |
| `MiniMax-M2.7-highspeed` | ¥4.2 | ¥16.8 | ¥0.42 | ¥2.625 |
| `MiniMax-M2.5` | ¥2.1 | ¥8.4 | ¥0.21 | ¥2.625 |
| `MiniMax-M2.5-highspeed` | ¥4.2 | ¥16.8 | ¥0.21 | ¥2.625 |
| `M2-her` | ¥2.1 | ¥8.4 | — | — |

> `-highspeed` 后缀为高优先级 / 高吞吐档，价格 2 倍但延迟更低；标准档与高速档共享缓存价。

## 2. 语音合成（T2A, ¥）

> 计价单位为 **每万字符**。同步 / 异步同价。

| 模型 ID | 类型 | 价格 |
| --- | --- | --- |
| `speech-2.8-hd` | HD（高音质） | ¥3.5 / 万字符 |
| `speech-2.6-hd` | HD | ¥3.5 / 万字符 |
| `speech-02-hd` | HD | ¥3.5 / 万字符 |
| `speech-2.8-turbo` | Turbo（快速） | ¥2 / 万字符 |
| `speech-2.6-turbo` | Turbo | ¥2 / 万字符 |

## 3. 音色设计 / 声音克隆（¥）

| 项 | 价格 | 计费时机 |
| --- | --- | --- |
| Voice Design | ¥9.9 / 音色 | 首次合成使用时扣费 |
| Voice Cloning | ¥9.9 / 音色 | 首次合成使用时扣费 |

## 4. 视频生成（¥）

> 计价单位为 **每条视频**，价格区间 **¥0.60–¥4.00**，取决于分辨率与时长。MiniMax-Hailuo / Hailuo-02 系列按官方原表细分；具体单价请重抓 https://platform.minimaxi.com/docs/guides/pricing-paygo 视频小节。

## 5. 音乐生成（¥）

| 模型 ID | 价格 |
| --- | --- |
| `music-2.0` | ¥0.25 / 首 |
| `music-2.5` | ¥1.0 / 首 |
| `music-2.5+` | ¥1.0 / 首 |
| `music-2.6` | ¥1.0 / 首 |
| Lyrics（生成歌词） | ¥0.05 / 次 |

## 6. 图像生成（¥）

| 模型 ID | 价格 |
| --- | --- |
| `image-01` | ¥0.025 / 张 |
| `image-01-live` | ¥0.025 / 张 |

## 7. MCP / 其他（¥）

| 项 | 价格 |
| --- | --- |
| `API-vlm`（VLM 视觉接口，MCP 内调用） | ¥0.4 / 次 |

## 计费要点

- **缓存计费字段**：响应 `usage.cached_tokens` / `cache_creation` 字段对应缓存档位。
- **`-highspeed` 档**：仅影响推理延迟与限速，缓存读取价不变。
- **任务失败**：异步视频 / 长音频任务失败不计费。
- **充值与额度**：见 https://platform.minimaxi.com/docs/account/recharge （官方）。

## 参考

- 按量付费定价：https://platform.minimaxi.com/docs/guides/pricing-paygo
- API 总览：https://platform.minimaxi.com/docs/api-reference/api-overview
