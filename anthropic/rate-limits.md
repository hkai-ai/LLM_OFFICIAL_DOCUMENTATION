---
source: https://platform.claude.com/docs/en/api/rate-limits
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01
---

# Rate Limits（限流）

Anthropic API 在组织级 (organization) 上限制：

- **Spend limit**：每月消费上限。
- **Rate limit**：单位时间内的请求数（RPM）、输入 token 数（ITPM）、输出 token 数（OTPM）。

算法采用 **token bucket**：容量持续按速率补充而非固定周期重置；短时突发可能瞬时超限并触发 `429`。

## Usage tier 与 Spend limit

满足相应**累计充值 (Credit Purchase)** 后自动升档，并解锁更高的月度消费上限：

| Tier | 升档累计充值 (USD) | 单笔最大充值 | 月度消费上限 |
| --- | --- | --- | --- |
| Tier 1 | `$5` | `$500` | `$500` |
| Tier 2 | `$40` | `$500` | `$500` |
| Tier 3 | `$200` | `$1,000` | `$1,000` |
| Tier 4 | `$400` | `$200,000` | `$200,000` |
| Monthly Invoicing | — | — | 无上限 |

> 客户也可在 Console 自行设定一个**低于 tier ceiling** 的上限。

## Messages API · Rate limit

按模型族单独计算；下表为标准 Tier 默认值，Custom Tier 需联系销售。

### Tier 1

| 模型族 | RPM | ITPM | OTPM |
| --- | --- | --- | --- |
| Claude Sonnet 4.x※ | `50` | `30,000` | `8,000` |
| Claude Haiku 4.5 | `50` | `50,000` | `10,000` |
| Claude Haiku 3.5† | `50` | `50,000` | `10,000` |
| Claude Opus 4.x※ | `50` | `500,000` | `80,000` |

### Tier 2

| 模型族 | RPM | ITPM | OTPM |
| --- | --- | --- | --- |
| Claude Sonnet 4.x※ | `1,000` | `450,000` | `90,000` |
| Claude Haiku 4.5 | `1,000` | `450,000` | `90,000` |
| Claude Haiku 3.5† | `1,000` | `100,000` | `20,000` |
| Claude Opus 4.x※ | `1,000` | `2,000,000` | `200,000` |

### Tier 3

| 模型族 | RPM | ITPM | OTPM |
| --- | --- | --- | --- |
| Claude Sonnet 4.x※ | `2,000` | `800,000` | `160,000` |
| Claude Haiku 4.5 | `2,000` | `1,000,000` | `200,000` |
| Claude Haiku 3.5† | `2,000` | `200,000` | `40,000` |
| Claude Opus 4.x※ | `2,000` | `5,000,000` | `400,000` |

### Tier 4

| 模型族 | RPM | ITPM | OTPM |
| --- | --- | --- | --- |
| Claude Sonnet 4.x※ | `4,000` | `2,000,000` | `400,000` |
| Claude Haiku 4.5 | `4,000` | `4,000,000` | `800,000` |
| Claude Haiku 3.5† | `4,000` | `400,000` | `80,000` |
| Claude Opus 4.x※ | `4,000` | `10,000,000` | `800,000` |

> `*` Opus 4.x 的 RPM 在 Opus 4、4.1、4.5、4.6、4.7 之间共享。
> `**` Sonnet 4.x 的 RPM 在 Sonnet 4、4.5、4.6 之间共享。
> `†` Haiku 3.5 将 `cache_read_input_tokens` 计入 ITPM；**其余模型 `cache_read_input_tokens` 不计 ITPM**，缓存读取也以 10% 输入单价计费。

### Cache-aware ITPM 计算

只有缓存外的输入 token 计入 ITPM：

| 字段 | 是否计入 ITPM |
| --- | --- |
| `input_tokens`（最后一个 cache breakpoint 之后的 token） | ✓ |
| `cache_creation_input_tokens` | ✓ |
| `cache_read_input_tokens` | ✗（除 Haiku 3.5 外） |

故总输入 token：

```text
total_input_tokens = cache_read_input_tokens + cache_creation_input_tokens + input_tokens
```

`max_tokens` 不会预占 OTPM；OTPM 仅按**实际产出的 token** 实时计数。

## Message Batches API · 独立限流

Batches 与 Messages 共用 RPM tier 数字，但额外限制「队列内未处理请求总数」与「单 batch 请求数」：

| Tier | RPM | 队列中最大请求数 | 单 batch 最大请求数 |
| --- | --- | --- | --- |
| Tier 1 | `50` | `100,000` | `100,000` |
| Tier 2 | `1,000` | `200,000` | `100,000` |
| Tier 3 | `2,000` | `300,000` | `100,000` |
| Tier 4 | `4,000` | `500,000` | `100,000` |

## Managed Agents · 独立限流

| 操作 | 限制 |
| --- | --- |
| Create 类（agents / sessions / environments） | `300 RPM` |
| Read 类（retrieve / list / stream） | `600 RPM` |

## Fast mode · 独立限流

在 Opus 4.6 / Opus 4.7 + `speed: "fast"` 时，触发独立的 Fast mode 池；响应中以 `anthropic-fast-*` 系列 header 暴露。

## 响应 Header

所有 Messages 响应都会返回当前最严格生效维度的限流状态：

| Header | 说明 |
| --- | --- |
| `retry-after` | 触发 429 时，需等待的秒数。 |
| `anthropic-ratelimit-requests-limit` | RPM 上限。 |
| `anthropic-ratelimit-requests-remaining` | 剩余 RPM。 |
| `anthropic-ratelimit-requests-reset` | RFC 3339；RPM 完全补满时刻。 |
| `anthropic-ratelimit-tokens-limit` | 合并 token 上限。 |
| `anthropic-ratelimit-tokens-remaining` | 合并 token 剩余（约整千）。 |
| `anthropic-ratelimit-tokens-reset` | RFC 3339。 |
| `anthropic-ratelimit-input-tokens-limit` / `-remaining` / `-reset` | ITPM 三件套。 |
| `anthropic-ratelimit-output-tokens-limit` / `-remaining` / `-reset` | OTPM 三件套。 |
| `anthropic-priority-input-tokens-*` / `anthropic-priority-output-tokens-*` | Priority Tier 专属，含 `limit` / `remaining` / `reset` 三档。 |

> `anthropic-ratelimit-tokens-*` 显示**当前最严苛维度**的数值；可能是 Workspace 限制或 Organization 限制。

## 429 错误结构

```json
{
  "type": "error",
  "error": {
    "type": "rate_limit_error",
    "message": "Rate limit reached for ..."
  }
}
```

收到 `429` 时建议：

1. 读取 `retry-after`，至少等待对应秒数后重试（指数退避更稳）。
2. 检查 `anthropic-ratelimit-*` 头判断是 RPM / ITPM / OTPM 哪个维度被打满。
3. 大量稳定流量场景考虑：
   - 充分利用 prompt caching（命中后不占 ITPM 且只收 10% 输入费）。
   - 评估 Batch 档（折扣 + 独立限流池）。
   - 联系销售升 Priority Tier 或 Custom Tier。

> 突发性流量也可能触发**加速限流 (acceleration limit)** ；逐步爬升流量并保持稳定模式可规避。

## 编程读取限流配置

`GET https://api.anthropic.com/v1/organizations/rate-limits`（[Rate Limits API](https://platform.claude.com/docs/en/manage-claude/rate-limits-api)）可读取当前组织 / Workspace 的实时 RPM / ITPM / OTPM。

## Workspace 子限制

可在 Console 给某个 Workspace 设置低于组织上限的 RPM / ITPM / OTPM。规则：

- 不能给 **default workspace** 设上限。
- 未设置则等于组织上限。
- 组织上限始终生效，即便所有 Workspace 上限之和大于组织上限。

## 参考

- 限流主页：<https://platform.claude.com/docs/en/api/rate-limits>
- Service Tiers：<https://platform.claude.com/docs/en/api/service-tiers>
- Rate Limits API：<https://platform.claude.com/docs/en/manage-claude/rate-limits-api>
- Errors：[errors.md](./errors.md)
