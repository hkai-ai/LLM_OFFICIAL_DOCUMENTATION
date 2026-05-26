---
source: https://openrouter.ai/docs/api/reference/limits
fetched_at: 2026-05-19
api_version: N/A
---

# 限流

> OpenRouter 在全局层面统一管理速率与额度。开新账户或新增 API key 不会绕过限流。具体每分钟请求数（RPM）/每日请求数（RPD）依模型与账户层级动态变化。

## 限流维度

| 维度 | 说明 |
| --- | --- |
| 全局 | 同一账户的所有 key 共享全局速率上限。 |
| 免费模型 | 免费模型变体（model ID 以 `:free` 结尾）受单独的 RPM 与 RPD 限制。 |
| 付费模型 | 由你已购买的 credits 与账户层级动态决定。 |

## 免费层规则

- 免费模型：固定 RPM 上限（官方文档以变量 `FREE_MODEL_RATE_LIMIT_RPM` 表示，文档站内动态渲染具体数值）。
- 日请求上限按账户已购 credits 分两档：
  - 未购买 credits：`FREE_MODEL_NO_CREDITS_RPD`（较低）。
  - 已购买达到门槛：`FREE_MODEL_HAS_CREDITS_RPD`（较高）。
- 账户余额负数时返回 `402`，需充值后恢复。

## 查询当前限额

调用 `GET /api/v1/key` 可查询当前 key 的额度信息（见 [api-keys.md](./api-keys.md)）。注意：响应中的 `rate_limit` 字段是 legacy，固定返回 `-1`，不再代表真实速率上限。

## 响应 Header

| Header | 说明 |
| --- | --- |
| `Retry-After` | 429 或 503 响应携带，单位为秒，提示客户端等待时间。 |

> 文档没有承诺暴露 `X-RateLimit-Limit` / `X-RateLimit-Remaining` 等 OpenAI 风格的限流 header。当前权威的限流查询方式是 `GET /api/v1/key`。文档未明确这些 header 的存在与否。

## DDoS 与滥用保护

OpenRouter 部署了 DDoS 保护层，异常突发流量会被前置拦截。

## 处理建议

- 429：读取 `Retry-After` 后退避重试；可同时配合 `provider.order` 或 `route: "fallback"` 切换 provider/模型。
- 402：用 management key 调 `GET /api/v1/credits` 检查总余额；用 `GET /api/v1/key` 检查单 key 限额。

## 参考

- 限流文档：<https://openrouter.ai/docs/api/reference/limits>
- API Key 信息端点：<https://openrouter.ai/docs/api/api-reference/api-keys/get-current-key>
