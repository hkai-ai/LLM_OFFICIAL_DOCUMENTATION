---
source: https://developers.openai.com/api/docs/pricing
fetched_at: 2026-05-26
api_version: N/A
---

# OpenAI API · 定价

> 全部价格以 **USD** 计；token 类计价单位为 **每 1M tokens**，其他单位（每秒 / 每分钟 / 每张图 / 每次调用 / 每小时）在表中显式标注。所有数字源自官方 `https://developers.openai.com/api/docs/pricing`，本仓库不做汇率换算与推算。
>
> 区域处理（data residency）端点对 `gpt-5.5` / `gpt-5.5-pro` / `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.4-nano` / `gpt-5.4-pro` 全档位额外 +10%。

## 1. 旗舰文本模型（每 1M tokens, USD）

### `gpt-5.5`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $5.00 | $0.50 | $30.00 |
| Batch | $2.50 | $0.25 | $15.00 |
| Flex | $2.50 | $0.25 | $15.00 |
| Priority | $12.50 | $1.25 | $75.00 |

### `gpt-5.5-pro`

| 档位 | Input | Output |
| --- | --- | --- |
| Standard | $30.00 | $180.00 |
| Batch | $15.00 | $90.00 |
| Flex | $15.00 | $90.00 |

### `gpt-5.4`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $2.50 | $0.25 | $15.00 |
| Batch | $1.25 | $0.13 | $7.50 |
| Flex | $1.25 | $0.13 | $7.50 |
| Priority | $5.00 | $0.50 | $30.00 |
| Standard（长上下文版本） | $5.00 | $0.50 | $22.50 |

### `gpt-5.4-mini`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $0.75 | $0.075 | $4.50 |
| Batch | $0.375 | $0.0375 | $2.25 |
| Flex | $0.375 | $0.0375 | $2.25 |
| Priority | $1.50 | $0.15 | $9.00 |

### `gpt-5.4-nano`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $0.20 | $0.02 | $1.25 |
| Batch | $0.10 | $0.01 | $0.625 |
| Flex | $0.10 | $0.01 | $0.625 |

### `gpt-5.4-pro`

| 档位 | Input | Output |
| --- | --- | --- |
| Standard | $30.00 | $180.00 |
| Batch | $15.00 | $90.00 |
| Flex | $15.00 | $90.00 |
| Standard（长上下文版本） | $60.00 | $270.00 |
| Batch（长上下文版本） | $30.00 | $135.00 |

## 2. 专门模型（每 1M tokens, USD）

### `chat-latest`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $5.00 | $0.50 | $30.00 |
| Batch | $2.50 | $0.25 | $15.00 |

### `gpt-5.3-codex`

| 档位 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Standard | $1.75 | $0.175 | $14.00 |
| Batch | $0.875 | $0.0875 | $7.00 |
| Priority | $3.50 | $0.35 | $28.00 |

### `o3-deep-research`

| 档位 | Input | Output |
| --- | --- | --- |
| Batch | $5.00 | $20.00 |

### `o4-mini-deep-research`

| 档位 | Input | Output |
| --- | --- | --- |
| Batch | $1.00 | $4.00 |

### `computer-use-preview`

| 档位 | Input | Output |
| --- | --- | --- |
| Batch | $1.50 | $6.00 |

## 3. 实时与音频生成（每 1M tokens / 每分钟, USD）

### `gpt-realtime-2`

| 模态 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Audio | $32.00 / MTok | $0.40 / MTok | $64.00 / MTok |
| Text | $4.00 / MTok | $0.40 / MTok | $24.00 / MTok |
| Image | $5.00 / MTok | $0.50 / MTok | — |

### `gpt-realtime-1.5`

| 模态 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Audio | $32.00 / MTok | $0.40 / MTok | $64.00 / MTok |
| Text | $4.00 / MTok | $0.40 / MTok | $16.00 / MTok |
| Image | $5.00 / MTok | $0.50 / MTok | — |

### `gpt-realtime-mini`

| 模态 | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Audio | $10.00 / MTok | $0.30 / MTok | $20.00 / MTok |
| Text | $0.60 / MTok | $0.06 / MTok | $2.40 / MTok |
| Image | $0.80 / MTok | $0.08 / MTok | — |

### `gpt-realtime-translate`

| 项 | 价格 |
| --- | --- |
| Output | $0.034 / 分钟 |

## 4. 图像生成（每 1M tokens, USD）

### `gpt-image-2`

| 档位 | Image Input | Image Cached | Image Output | Text Input | Text Cached |
| --- | --- | --- | --- | --- | --- |
| Standard | $8.00 | $2.00 | $30.00 | $5.00 | $1.25 |
| Batch | $4.00 | $1.00 | $15.00 | $2.50 | $0.625 |

### `gpt-image-1.5`

| 档位 | Image Input | Image Cached | Image Output | Text Input | Text Cached | Text Output |
| --- | --- | --- | --- | --- | --- | --- |
| Standard | $8.00 | $2.00 | $32.00 | $5.00 | $1.25 | $10.00 |
| Batch | $4.00 | $1.00 | $16.00 | $2.50 | $0.63 | $5.00 |

### `gpt-image-1-mini`

| 档位 | Image Input | Image Cached | Image Output | Text Input | Text Cached |
| --- | --- | --- | --- | --- | --- |
| Standard | $2.50 | $0.25 | $8.00 | $2.00 | $0.20 |
| Batch | $1.25 | $0.13 | $4.00 | $1.00 | $0.10 |

## 5. 视频生成（每秒, USD）

### `sora-2`

| 档位 | 720p (Portrait 720×1280 / Landscape 1280×720) |
| --- | --- |
| Standard | $0.10 / 秒 |
| Batch | $0.05 / 秒 |

### `sora-2-pro`

| 档位 | 720p | 1024p | 1080p |
| --- | --- | --- | --- |
| Standard | $0.30 / 秒 | $0.50 / 秒 | $0.70 / 秒 |
| Batch | $0.15 / 秒 | $0.25 / 秒 | $0.35 / 秒 |

## 6. 转录（每 1M tokens / 估算每分钟, USD）

| 模型 | Input | Output | 估算成本 |
| --- | --- | --- | --- |
| `gpt-4o-transcribe` | $2.50 | $10.00 | ≈ $0.006 / 分钟 |
| `gpt-4o-mini-transcribe` | $1.25 | $5.00 | ≈ $0.003 / 分钟 |

## 7. 工具定价（USD）

| 工具 | 定价 |
| --- | --- |
| Web search（全模型） | $10.00 / 1k calls + 搜索内容 tokens 按模型费率 |
| Web search preview（reasoning models） | $10.00 / 1k calls + 搜索内容 tokens 按模型费率 |
| Web search preview（非 reasoning models） | $25.00 / 1k calls + 搜索内容 tokens 免费 |
| Containers（Hosted Shell / Code Interpreter） | 1GB $0.03 / 4GB $0.12 / 16GB $0.48 / 64GB $1.92 — 每 20 分钟 session |
| File search 存储 | $0.10 / GB·day（每账户 1GB 免费） |
| File search 工具调用 | $2.50 / 1k calls |
| ChatKit 文件 / 图像上传存储 | $0.10 / GB·day（每账户每月 1GB 免费） |

## 8. 微调（Fine-tuning, 每 1M tokens / 每小时, USD）

### `o4-mini-2025-04-16`

| 档位 | 训练 | Input | Cached Input | Output |
| --- | --- | --- | --- | --- |
| Standard | $100.00 / 小时 | $4.00 | $1.00 | $16.00 |
| Standard（with data sharing） | $100.00 / 小时 | $2.00 | $0.50 | $8.00 |
| Batch | — | $2.00 | $0.50 | $8.00 |
| Batch（with data sharing） | — | $1.00 | $0.25 | $4.00 |

> Tokens 用于模型 grading 的强化微调按对应模型 per-token 费率计费。

## 参考

- Pricing 主页：https://developers.openai.com/api/docs/pricing
- 模型清单：见 `openai/README.md` §模型清单
