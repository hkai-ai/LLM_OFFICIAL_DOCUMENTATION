---
source: https://platform.kimi.com/docs/pricing/chat
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · 定价

> 全部价格以 **人民币元（¥）** 计；token 类计价单位为 **每 1M tokens（百万 tokens）**，其他工具按其各自单位（每次 / 每万字符）。所有数字源自官方 `https://platform.kimi.com/docs/pricing/...` 各子页，本仓库不做汇率换算与推算。

## 1. 多模态推理模型

### `kimi-k2.6`

| 项 | 价格 |
| --- | --- |
| 上下文窗口 | 262,144 tokens |
| Input（缓存命中） | ¥1.10 / 1M tokens |
| Input（缓存未命中） | ¥6.50 / 1M tokens |
| Output | ¥27.00 / 1M tokens |
| 思考模式 | 支持，默认开启（可关） |

来源：https://platform.kimi.com/docs/pricing/chat-k26

### `kimi-k2.5`

| 项 | 价格 |
| --- | --- |
| 上下文窗口 | 262,144 tokens |
| Input（缓存命中） | ¥0.70 / 1M tokens |
| Input（缓存未命中） | ¥4.00 / 1M tokens |
| Output | ¥21.00 / 1M tokens |
| 思考模式 | 支持，默认开启（可关） |

来源：https://platform.kimi.com/docs/pricing/chat-k25

## 2. 经典生成系列（`moonshot-v1`）

> 此系列**无缓存命中价档位**，输入按统一价计费。

| 模型 ID | 上下文 | Input | Output |
| --- | --- | --- | --- |
| `moonshot-v1-8k` | 8,192 | ¥2.00 / 1M tokens | ¥10.00 / 1M tokens |
| `moonshot-v1-32k` | 32,768 | ¥5.00 / 1M tokens | ¥20.00 / 1M tokens |
| `moonshot-v1-128k` | 131,072 | ¥10.00 / 1M tokens | ¥30.00 / 1M tokens |
| `moonshot-v1-8k-vision-preview` | 8,192 | ¥2.00 / 1M tokens | ¥10.00 / 1M tokens |
| `moonshot-v1-32k-vision-preview` | 32,768 | ¥5.00 / 1M tokens | ¥20.00 / 1M tokens |
| `moonshot-v1-128k-vision-preview` | 131,072 | ¥10.00 / 1M tokens | ¥30.00 / 1M tokens |

来源：https://platform.kimi.com/docs/pricing/chat-v1

> 旧版 `kimi-k2-0905-preview` / `kimi-k2-0711-preview` / `kimi-k2-turbo-preview` 计划于 2026-05-25 下线，定价已不在主表，详见各预览页面或迁移到 `kimi-k2.5` / `kimi-k2.6`。

## 3. 批量推理（Batch API，所有模型 60% 价）

| 模型 | Batch Input（缓存未命中） | Batch Output |
| --- | --- | --- |
| `kimi-k2.6` | ¥3.90 / 1M tokens | ¥16.20 / 1M tokens |
| `kimi-k2.5` | ¥2.40 / 1M tokens | ¥12.60 / 1M tokens |

来源：https://platform.kimi.com/docs/pricing/batch

> 批量推理整体为标准价的 60%。其他模型按各自标准价 ×60% 推导，官方文档以上表所列为准。

## 4. 工具定价

### Web Search

| 项 | 价格 |
| --- | --- |
| 触发一次 `$web_search` 工具调用 | ¥0.03 |

> 工具调用费用之外，搜索返回内容产生的 token 消耗按对应模型 Input 价单独计费。

来源：https://platform.kimi.com/docs/pricing/tools

## 5. 计费要点

- **缓存计费字段**：响应 `usage.cached_tokens` 顶层字段表示命中部分；K2.5 / K2.6 区分缓存档位，`moonshot-v1` 不区分。
- **思考模式输出**：思考产生的 token 计入 `completion_tokens` 总数，按 Output 价计费。
- **缓存创建价**：当前定价页未列出 K2.5 / K2.6 的 cache write 价；如需精确成本核算请重抓页面或参考 `usage` 中实际字段。
- **充值与限速**：见 https://platform.kimi.com/docs/pricing/limits（Tier 制限速，不在此页详列）。

## 参考

- Pricing 总索引：https://platform.kimi.com/docs/pricing/chat
- Kimi K2.6：https://platform.kimi.com/docs/pricing/chat-k26
- Kimi K2.5：https://platform.kimi.com/docs/pricing/chat-k25
- Moonshot V1：https://platform.kimi.com/docs/pricing/chat-v1
- Batch：https://platform.kimi.com/docs/pricing/batch
- Tools：https://platform.kimi.com/docs/pricing/tools
- FAQ：https://platform.kimi.com/docs/pricing/faq
- 充值与限速：https://platform.kimi.com/docs/pricing/limits
