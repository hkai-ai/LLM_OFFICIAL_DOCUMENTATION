---
source: https://platform.claude.com/docs/en/api/service-tiers
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01
---

# Service Tiers（服务档位）

Anthropic 提供三档服务，平衡可用性、性能与成本：

| 档位 | 定位 | 入口字段 |
| --- | --- | --- |
| **Standard** | 默认档；按尽力而为分配容量 | 不传或 `service_tier: "standard_only"` |
| **Priority** | 高可用 / 承诺消费 / 独立 token 池 | `service_tier: "auto"` + 已采购 Priority 容量 |
| **Batch** | 异步批量；折扣价、独立限流 | 通过 [Message Batches API](./messages-batches.md) 调用 |

## Standard Tier

所有未声明 `service_tier` 的请求默认进入 Standard。容量按尽力而为分配，与全网请求公平排队。

## Priority Tier

请求被优先调度，可显著降低 `overloaded_error`。需采购承诺：

| 维度 | 说明 |
| --- | --- |
| 承诺类型 | 每分钟输入 token 数 + 每分钟输出 token 数 |
| 承诺周期 | `1` / `3` / `6` / `12` 个月 |
| 绑定模型 | 必须指定具体模型版本（如 `claude-opus-4-7`） |
| SLA | 目标 99.5% uptime |
| 不支持模型 | Claude Mythos Preview（其余包括 Opus 4.7 在内全系支持） |

> 采购入口：联系销售 <https://claude.com/contact-sales/priority-tier>。

### 请求字段

```json
{
  "model": "claude-opus-4-7",
  "max_tokens": 1024,
  "messages": [{"role": "user", "content": "Hello, Claude!"}],
  "service_tier": "auto"
}
```

| 取值 | 含义 |
| --- | --- |
| `"auto"`（默认） | 优先用 Priority 池；不足则回落到 Standard。 |
| `"standard_only"` | 强制 Standard，不消耗 Priority 容量。 |

### 命中规则

被指派到 Priority 的条件是「Priority Tier 容量中**输入 token / 输出 token 同时充足**」，否则回落 Standard。

> Priority 命中的请求**同时**扣减 Priority 池与对应模型常规 RPM/ITPM/OTPM；若常规限流超额仍会 `429`。

### Priority 容量扣减权重

每 1 token 实际消耗的 Priority 容量并非恒等于 1：

| 类型 | 扣减率（每 1 个 token） |
| --- | --- |
| `cache_read_input_tokens` | `0.1` |
| `cache_creation_input_tokens`（5m TTL） | `1.25` |
| `cache_creation_input_tokens`（1h TTL） | `2.00` |
| `inference_geo: "us"` 输入（Opus 4.6 / Sonnet 4.6 及以后） | `1.1` |
| `inference_geo: "us"` 输出（同上） | `1.1` |
| 其余 input / output | `1.0` |

> 该比例与各类型实际定价比相对应。

## Batch Tier

调用 [Message Batches API](./messages-batches.md) 即视为 Batch 档；价格按 batch 折扣（详见 [pricing.md](./pricing.md)），限流参数独立（详见 [rate-limits.md](./rate-limits.md) §Message Batches API）。

## 响应 usage.service_tier

每条响应的 `usage` 包含实际生效档位：

```json
{
  "usage": {
    "input_tokens": 410,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0,
    "output_tokens": 585,
    "service_tier": "priority"
  }
}
```

`service_tier` 枚举：`"standard"` / `"priority"` / `"batch"`。

## 响应 Header

Priority 容量可用时，响应附带以下头（不论该请求是否最终命中 Priority，只要 `service_tier: "auto"` 且模型已采购 Priority）：

| Header | 说明 |
| --- | --- |
| `anthropic-priority-input-tokens-limit` | Priority 池输入 token 上限。 |
| `anthropic-priority-input-tokens-remaining` | 剩余输入 token。 |
| `anthropic-priority-input-tokens-reset` | RFC 3339 重置时间。 |
| `anthropic-priority-output-tokens-limit` | Priority 池输出 token 上限。 |
| `anthropic-priority-output-tokens-remaining` | 剩余输出 token。 |
| `anthropic-priority-output-tokens-reset` | RFC 3339 重置时间。 |

> 这些 header 的存在与否即可用于判断「该请求是否被认定为 Priority eligible」。

## 与 Fast mode 的关系

Fast mode（`speed: "fast"`，Opus 4.6 / Opus 4.7）是独立于服务档位的 beta 功能，有自己的限流池（`anthropic-fast-*` headers）。Fast mode 与 Priority Tier 可同时使用。

## 参考

- 主页：<https://platform.claude.com/docs/en/api/service-tiers>
- Rate Limits：[rate-limits.md](./rate-limits.md)
- Message Batches：[messages-batches.md](./messages-batches.md)
- Pricing：[pricing.md](./pricing.md)
