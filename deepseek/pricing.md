---
source: https://api-docs.deepseek.com/zh-cn/quick_start/pricing
fetched_at: 2026-05-26
api_version: N/A
---

# DeepSeek · 定价

> 全部价格以 **人民币元（¥）** 计，计价单位为 **每 1M tokens**。所有数字源自官方 `https://api-docs.deepseek.com/zh-cn/quick_start/pricing`，本仓库不做汇率换算与推算。
>
> 旧模型 ID `deepseek-chat` / `deepseek-reasoner` 计划下线（参见 `models.md`），分别等价于 `deepseek-v4-flash` 的非思考 / 思考模式，定价随主模型。

## 1. 模型定价

### `deepseek-v4-flash`

| 项 | 价格 |
| --- | --- |
| 上下文窗口 | 1M tokens |
| 最大输出 | 384K tokens |
| Input（缓存命中） | ¥0.02 / 1M tokens |
| Input（缓存未命中） | ¥1 / 1M tokens |
| Output | ¥2 / 1M tokens |
| 思考模式 | 支持，价格同非思考 |

### `deepseek-v4-pro`

| 项 | 标准价 | 折扣价（截至 2026-05-31） |
| --- | --- | --- |
| 上下文窗口 | 1M tokens | — |
| 最大输出 | 384K tokens | — |
| Input（缓存命中） | ¥0.10 / 1M tokens | ¥0.025 / 1M tokens |
| Input（缓存未命中） | ¥12 / 1M tokens | ¥3 / 1M tokens |
| Output | ¥24 / 1M tokens | ¥6 / 1M tokens |
| 思考模式 | 支持 | 同左 |

> **促销活动**：v4-pro 当前享 75% 折扣，截止 **2026-05-31**；之后调整为原价的 25%（即上述折扣价继续生效，但官方说明已不再标注为"促销"）。每次同步定价需重抓上述官方价格页核对活动是否仍生效。

## 2. 计费要点

- **缓存计费字段**：响应 `usage.prompt_cache_hit_tokens` / `usage.prompt_cache_miss_tokens` 分别对应两档输入价。
- **思考模式输出**：`completion_tokens_details.reasoning_tokens` 计入 `completion_tokens` 总数，按 Output 价计费。
- **赠送余额优先**：消耗时优先扣减赠送余额，再扣减充值余额。
- **限速**：DeepSeek 不公开 TPM / RPM 阈值（动态限流），不要补造数字。
- **无 Batch API、无独立向量端点、无微调**：定价表上不列对应项目。

## 参考

- 模型与价格：https://api-docs.deepseek.com/zh-cn/quick_start/pricing
- 上下文缓存：https://api-docs.deepseek.com/zh-cn/guides/kv_cache
- 更新日志：https://api-docs.deepseek.com/zh-cn/updates
