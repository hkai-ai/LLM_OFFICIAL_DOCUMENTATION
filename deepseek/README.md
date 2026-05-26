---
source: https://api-docs.deepseek.com/zh-cn/
fetched_at: 2026-05-19
api_version: N/A（OpenAPI 标注 1.0.0；OpenAI 兼容路径无版本前缀）
last_updated: 2026-05-26
---

# DeepSeek API 概览

DeepSeek 官方 API 与 OpenAI Chat Completions、Anthropic Messages 两套协议兼容，允许直接复用 OpenAI / Anthropic SDK 通过更换 `base_url` 接入。本仓库文档以 OpenAI 兼容形式为主。

## Base URL

| 用途 | URL |
| --- | --- |
| OpenAI 兼容（推荐） | `https://api.deepseek.com` |
| OpenAI 兼容（带 v1 前缀，与 OpenAI SDK 保持一致） | `https://api.deepseek.com/v1` |
| Beta 端点（FIM、`prefix` 续写等） | `https://api.deepseek.com/beta` |
| Anthropic 兼容 | `https://api.deepseek.com/anthropic` |

> `v1` 与无前缀两种写法均指向同一套接口；`v1` 并非真实版本号，只是为兼容默认带 `/v1` 的 OpenAI SDK 而保留的别名。

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DEEPSEEK_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |

API Key 在 https://platform.deepseek.com/api_keys 创建。

## 与 OpenAI SDK 的兼容性

- 请求与响应字段命名、SSE 流式格式与 OpenAI Chat Completions 一致。
- Python：`OpenAI(api_key=..., base_url="https://api.deepseek.com")`。
- Node.js：`new OpenAI({ apiKey, baseURL: "https://api.deepseek.com" })`。
- 特有字段（`reasoning_content`、`prefix`、`prompt_cache_hit_tokens` 等）作为额外字段返回，不影响 SDK 反序列化。
- FIM `/beta/completions` 与 `prefix` 续写需将 base URL 切到 `https://api.deepseek.com/beta`。

## 端点索引

| 端点 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 对话补全 | POST | `/chat/completions` | [chat-completions.md](./chat-completions.md) |
| FIM 补全（Beta） | POST | `/beta/completions` | [fim-completion.md](./fim-completion.md) |
| 模型列表 | GET | `/models` | [models.md](./models.md) |
| 账户余额 | GET | `/user/balance` | [user-balance.md](./user-balance.md) |

辅助主题：

| 主题 | 文档 |
| --- | --- |
| 上下文硬盘缓存 | [caching.md](./caching.md) |
| 错误码 | [errors.md](./errors.md) |
| 限速策略 | [rate-limits.md](./rate-limits.md) |

## 模型清单

下表汇总当前在售的模型 ID。上下文窗口、最大输出、价格以官方[模型与价格页](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)为准。

| 模型 ID | 上下文窗口 | 最大输出 | 思考模式 | 擅长场景 |
| --- | --- | --- | --- | --- |
| `deepseek-v4-pro` | 1M tokens | 384K tokens | 支持（可开关） | 通用对话、Agent、复杂推理；默认 `reasoning_effort=high`，复杂 Agent 任务自动升 `max`。 |
| `deepseek-v4-flash` | 1M tokens | 384K tokens | 支持（可开关） | 轻量、低成本、高吞吐场景。 |
| `deepseek-chat`（旧名） | 1M tokens | 384K tokens | 默认关闭 | 对应 `deepseek-v4-flash` 非思考模式，将于 2026-07-24 下线。 |
| `deepseek-reasoner`（旧名） | 1M tokens | 384K tokens | 默认开启 | 对应 `deepseek-v4-flash` 思考模式，将于 2026-07-24 下线。 |

> 思考模式开启时，模型在 `choices[].message.reasoning_content` 输出思维链，最终回答在 `content` 中。思考模式不支持 `temperature` / `top_p` / `presence_penalty` / `frequency_penalty`（传入不会报错，但会被忽略）。

## 计费要点

- 输入 token 区分缓存命中（`prompt_cache_hit_tokens`）与未命中（`prompt_cache_miss_tokens`）两档价格，命中价显著低于未命中价。
- 思考模式下，`completion_tokens_details.reasoning_tokens` 计入输出 token 总数。
- 具体单价以官方价格页为准。

## 参考

- 入口：https://api-docs.deepseek.com/zh-cn/
- API 总览：https://api-docs.deepseek.com/zh-cn/api/deepseek-api
- 首次调用：https://api-docs.deepseek.com/zh-cn/
- 模型与价格：https://api-docs.deepseek.com/zh-cn/quick_start/pricing
- 更新日志：https://api-docs.deepseek.com/zh-cn/updates
