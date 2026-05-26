---
source: https://platform.kimi.com/docs/pricing/limits
fetched_at: 2026-05-20
api_version: N/A
---

# 充值与限速 Tier

Moonshot 按账户累计**充值金额**划分 6 个 Tier（含 Tier0 免费档），各档对应不同的并发 / RPM / TPM / TPD 上限。代金券不计入累计充值额。

## Tier 阶梯

| Tier | 累计充值门槛 | 并发 | RPM | TPM | TPD |
| --- | --- | --- | --- | --- | --- |
| Tier0 | ¥0 | 1 | 3 | 500,000 | 1,500,000 |
| Tier1 | ¥50 | 50 | 200 | 2,000,000 | 无限制 |
| Tier2 | ¥100 | 100 | 500 | 3,000,000 | 无限制 |
| Tier3 | ¥500 | 200 | 5,000 | 3,000,000 | 无限制 |
| Tier4 | ¥5,000 | 400 | 5,000 | 4,000,000 | 无限制 |
| Tier5 | ¥20,000 | 1,000 | 10,000 | 5,000,000 | 无限制 |

## 字段定义

| 指标 | 含义 |
| --- | --- |
| 并发 | 同一时刻 API Key 在用的最大请求数。 |
| RPM | Requests Per Minute，每分钟最大请求数。 |
| TPM | Tokens Per Minute，每分钟最大输入 + 输出 token 总量。 |
| TPD | Tokens Per Day，每日最大输入 + 输出 token 总量；非 Tier0 默认无限制。 |

## 触发后的表现

- 触发任一上限返回 `429`，`error.code` 为 `rate_limit_reached_error`。
- 服务端集群满载时可能短时压低限速，需指数退避重试。
- 超额需求可在控制台提交「提升速率」表单，逐 case 评估。

## 与计费关系

- Tier 仅决定限速上限，价格不随 Tier 变化；具体单价见 [models.md](./models.md) 与官方 [pricing/chat](https://platform.kimi.com/docs/pricing/chat) 系列页面。
- 输入价区分缓存命中 / 未命中（仅 K2.5 / K2.6 系列）。

## 保活机制

- 非流式请求等待期间，服务端可能在 HTTP body 中插入若干空行作为保活，客户端需容忍。
- 流式请求等待期间发送 `: keep-alive` SSE 注释行。

## 参考

- 限速页面：https://platform.kimi.com/docs/pricing/limits
- 定价 FAQ：https://platform.kimi.com/docs/pricing/faq
