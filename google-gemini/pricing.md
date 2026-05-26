---
source: https://ai.google.dev/gemini-api/docs/pricing?hl=zh-cn
fetched_at: 2026-05-26
api_version: v1beta
---

# Google Gemini Developer API · 定价

> 全部价格以 **USD** 计；token 类计价单位为 **每 1M tokens**，图像 / 视频 / 音乐 / 搜索按其各自单位（每张图 / 每秒 / 每首 / 每 1k 查询）。所有数字源自官方 `https://ai.google.dev/gemini-api/docs/pricing`，本仓库不做汇率换算与推算。
>
> 仅覆盖 **Gemini Developer API**（`generativelanguage.googleapis.com`）；Vertex AI Gemini API 价格不同，见 GCP 价格页。
>
> 免费层（Free tier）输入 / 输出 token 计费为 $0，但官方会用免费层内容用于产品改进；付费层（Paid tier）数据不用于产品改进。

## 1. 文本模型

### `gemini-3-flash-preview`

> 与 `gemini-3.5-flash` 定位描述一致（"most intelligent model built for speed..."），大概率是 3.5-flash 转正前的 preview 版。preview 期价格约为稳定版的 1/3。

| 项 | Standard | Batch | Flex | Priority |
| --- | --- | --- | --- | --- |
| Input（text / image / video） | $0.50 / 1M | $0.25 / 1M | $0.25 / 1M | $0.90 / 1M |
| Input（audio） | $1.00 / 1M | $0.50 / 1M | $0.50 / 1M | $1.80 / 1M |
| Output（含 thinking） | $3.00 / 1M | $1.50 / 1M | $1.50 / 1M | $5.40 / 1M |
| Context caching（text / image / video） | $0.05 / 1M | $0.05 / 1M（页面注「Batch 价未实现，同 Standard」） | $0.05 / 1M | $0.09 / 1M |
| Context caching（audio） | $0.10 / 1M | $0.10 / 1M | $0.10 / 1M | $0.18 / 1M |
| Context caching storage | $1.00 / 1M tokens · 小时 | $1.00 | $1.00 | $1.80 |

免费层：Standard / Priority 的 input / output / cache 标 Free of charge；Batch / Flex 在免费层不可用。Search Grounding / Maps Grounding：5,000 prompts/月免费（Gemini 3 系列共享），之后 $14 / 1,000 次查询。

### `gemini-3-pro`

> 文档抓取时该 ID 在主清单中存在但定价细节未列于 pricing 页主表，需重抓 https://ai.google.dev/gemini-api/docs/pricing 核对。**注**：2026-05-26 重新核对时，pricing 页主表已不再列出 `gemini-3-pro` 独立条目，文本/思考模型仅有 `gemini-3.1-pro-preview`（区分 ≤200k / >200k 输入档）；该 ID 可能已合并入 3.1 Pro Preview 或仅在 models 清单中存在。

### `gemini-3.5-flash`

| 项 | 免费层 | 付费标准 | Batch（50% off） |
| --- | --- | --- | --- |
| Input | $0 | $1.50 / 1M tokens | $0.75 / 1M tokens |
| Output | $0 | $9.00 / 1M tokens | $4.50 / 1M tokens |
| 上下文缓存读取 | — | $0.15 / 1M tokens | — |
| 上下文缓存存储 | — | $1.00 / 1M tokens · 小时 | — |

### `gemini-3.1-flash-lite`

| 项 | 免费层 | 付费标准 | Batch（50% off） |
| --- | --- | --- | --- |
| Input（text / image / video） | $0 | $0.25 / 1M tokens | $0.125 / 1M tokens |
| Input（audio） | $0 | $0.50 / 1M tokens | — |
| Output | $0 | $1.50 / 1M tokens | $0.75 / 1M tokens |

### `gemini-2.5-pro`

> 区分 prompt 长度档位。

| 项 | 付费标准（≤200k 输入） | 付费标准（>200k 输入） | Batch（50% off） |
| --- | --- | --- | --- |
| Input | $1.25 / 1M tokens | $2.50 / 1M tokens | 减半 |
| Output | $10.00 / 1M tokens | $15.00 / 1M tokens | 减半 |

### `gemini-2.5-flash`

| 项 | 免费层 | 付费标准 | Batch |
| --- | --- | --- | --- |
| Input（text / image / video） | $0 | $0.30 / 1M tokens | $0.15 / 1M tokens |
| Input（audio） | $0 | $1.00 / 1M tokens | — |
| Output | $0 | $2.50 / 1M tokens | $1.25 / 1M tokens |

### `gemini-2.5-flash-lite`

| 项 | 免费层 | 付费标准 | Batch |
| --- | --- | --- | --- |
| Input（text / image / video） | $0 | $0.10 / 1M tokens | $0.05 / 1M tokens |
| Input（audio） | $0 | $0.30 / 1M tokens | — |
| Output | $0 | $0.40 / 1M tokens | $0.20 / 1M tokens |

### `gemini-2.0-flash`

> **已弃用**，**2026-06-01** 停止服务。

| 项 | 付费标准 |
| --- | --- |
| Input（text / image / video） | $0.10 / 1M tokens |
| Input（audio） | $0.70 / 1M tokens |
| Output | $0.40 / 1M tokens |

## 2. 图像生成

### `gemini-3.1-flash-image-preview`（昵称"nano-banana"）

| 档位 | Input | Output | 估算每图 |
| --- | --- | --- | --- |
| Standard | $0.50 / 1M tokens | $60.00 / 1M tokens | ≈ $0.045–0.151 / 图（按分辨率） |
| Batch | $0.25 / 1M tokens | $30.00 / 1M tokens | 减半 |

### `gemini-2.5-flash-image`

| 档位 | Input | Output |
| --- | --- | --- |
| Standard | $0.30 / 1M tokens | ≈ $0.039 / 图（1024×1024） |
| Batch | $0.15 / 1M tokens | ≈ $0.0195 / 图 |

### Imagen 4 系列

| 模型 | 价格 |
| --- | --- |
| `imagen-4.0-fast-generate-001` | $0.02 / 图 |
| `imagen-4.0-generate-001` | $0.04 / 图 |
| `imagen-4.0-ultra-generate-001` | $0.06 / 图 |

## 3. 视频生成

### Veo 3.1

| 档位 / 模型 | 价格 |
| --- | --- |
| `veo-3.1-generate-preview` Standard（720p/1080p） | $0.40 / 秒 |
| `veo-3.1-generate-preview` Standard（4K） | $0.60 / 秒 |
| Veo 3.1 Fast 720p | $0.10 / 秒 |
| Veo 3.1 Fast 1080p | $0.12 / 秒 |
| Veo 3.1 Fast 4K | $0.30 / 秒 |
| Veo 3.1 Lite 720p | $0.05 / 秒 |
| Veo 3.1 Lite 1080p | $0.08 / 秒 |

### Veo 3

| 档位 / 模型 | 价格 |
| --- | --- |
| `veo-3.0-generate-001` Standard | $0.40 / 秒 |
| Veo 3 Fast 720p | $0.10 / 秒 |
| Veo 3 Fast 1080p | $0.12 / 秒 |
| Veo 3 Fast 4K | $0.30 / 秒 |

### Veo 2

| 模型 | 价格 |
| --- | --- |
| `veo-2.0-generate-001` | $0.35 / 秒（统一定价） |

## 4. 音乐与语音

### Lyria 3

| 模型 | 价格 |
| --- | --- |
| Lyria 3 Clip（30 秒） | $0.04 / 首 |
| Lyria 3 Pro（完整歌曲） | $0.08 / 首 |

### `gemini-3.1-flash-tts-preview`（TTS）

| 项 | Standard | Batch |
| --- | --- | --- |
| Text input | $1.00 / 1M tokens | $0.50 / 1M tokens |
| Audio output | $20.00 / 1M tokens | $10.00 / 1M tokens |

### `gemini-3.1-flash-live-preview`（Live API）

| 项 | 价格 |
| --- | --- |
| Text input | $0.75 / 1M tokens |
| Text output | $4.50 / 1M tokens |
| Audio input | $3.00 / 1M tokens（≈ $0.005 / 分钟） |
| Audio output | $12.00 / 1M tokens（≈ $0.018 / 分钟） |

## 5. 嵌入

### `gemini-embedding-2`

| 输入类型 | 价格 |
| --- | --- |
| Text | $0.20 / 1M tokens |
| Image | $0.45 / 1M tokens（≈ $0.00012 / 张） |
| Audio | $6.50 / 1M tokens（≈ $0.00016 / 秒） |
| Video | $12.00 / 1M tokens（≈ $0.00079 / 帧） |

### `gemini-embedding-001`

| 档位 | 价格 |
| --- | --- |
| Standard | $0.15 / 1M tokens |
| Batch | $0.075 / 1M tokens |

## 6. 工具与接地

| 工具 | 价格 |
| --- | --- |
| Google Search Grounding（Gemini 3 系列） | 每月前 5,000 个 prompt 免费；之后 $14 / 1,000 次查询 |
| Google Maps Grounding | 每月前 5,000 个 prompt 免费；之后 $14 / 1,000 次查询 |
| Code Execution | 按对应模型 token 价计费，无附加费 |
| URL Context | 按对应模型 Input token 价计费 |
| File Search 嵌入 | $0.15 / 1M tokens；检索文档按对应模型标准 token 价计费 |

## 7. 计费要点

- **免费 vs 付费数据使用差异**：免费层内容可能被用于改进产品；付费层不用。
- **上下文缓存**：仅 2.5/3.x 部分模型支持，区分"读取"（与 input 同 token 计量）与"存储"（按 token·小时）。
- **Batch / Flex 50% 折扣**：批处理统一减半，但不与上下文缓存自动叠加，需以官方页 confirm。
- **代理（Agents）预览**：环境计算（CPU / 内存 / 沙盒）预览期不收费；基础模型 token 与工具调用按各自费率计费。
- **`gemini-2.0-flash` 弃用**：自 2026-06-01 停止服务；新接入请使用 2.5 / 3.x 系列。

## 参考

- Pricing 主页（zh）：https://ai.google.dev/gemini-api/docs/pricing?hl=zh-cn
- Pricing 主页（en）：https://ai.google.dev/gemini-api/docs/pricing
- 模型清单：https://ai.google.dev/gemini-api/docs/models?hl=zh-cn
- 限速：https://ai.google.dev/gemini-api/docs/rate-limits?hl=zh-cn
