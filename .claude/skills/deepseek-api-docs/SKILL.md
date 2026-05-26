---
name: deepseek-api-docs
description: 当需要查阅、更新或扩充 DeepSeek API 官方文档（chat/completions、FIM、models、user/balance、上下文缓存、思考模式、prefix 续写、错误码、限流、**定价（pricing.md）** 等）在本仓库 `deepseek/` 目录下的中文整理版本时使用。触发场景包括："DeepSeek API 怎么用"、"deepseek-reasoner 的 reasoning_content"、"DeepSeek 上下文缓存命中"、"DeepSeek 错误码 402"、"DeepSeek 限流"、"DeepSeek prefix 续写"、"刷新 DeepSeek 文档"、"DeepSeek 新版模型 v4-pro / v4-flash"、"DeepSeek 价格"、"v4-pro 75% off"、"DeepSeek 优惠到期"。
---

# DeepSeek API 文档撰写与维护

## 1. 文档站全貌

- 入口：`https://api-docs.deepseek.com/zh-cn/`
- 中文页路径前缀：`/zh-cn/`；英文站路径前缀：`/`。中英文同结构同 slug，互为镜像。
- 左侧目录组织：
  - **快速开始**（quick_start）
    - `/zh-cn/`（首次调用 API）
    - `/zh-cn/quick_start/pricing`（模型 & 价格）
    - `/zh-cn/quick_start/token_usage`
    - `/zh-cn/quick_start/rate_limit`
    - `/zh-cn/quick_start/error_codes`
    - `/zh-cn/quick_start/agent_integrations/...`
  - **API 指南**（guides）
    - `/zh-cn/guides/thinking_mode`
    - `/zh-cn/guides/multi_round_chat`
    - `/zh-cn/guides/chat_prefix_completion`（Beta）
    - `/zh-cn/guides/fim_completion`（Beta）
    - `/zh-cn/guides/json_mode`
    - `/zh-cn/guides/tool_calls`
    - `/zh-cn/guides/kv_cache`
    - `/zh-cn/guides/anthropic_api`
  - **API 文档**（api）
    - `/zh-cn/api/deepseek-api`（总览）
    - `/zh-cn/api/create-chat-completion`
    - `/zh-cn/api/create-completion`（FIM）
    - `/zh-cn/api/list-models`
    - `/zh-cn/api/get-user-balance`
  - **新闻**（news）：`/zh-cn/news/<slug>`
  - **常见问题**：`/zh-cn/faq`
  - **更新日志**：`/zh-cn/updates`
- 公开访问，无登录、无 paywall。
- 部分内嵌 API Playground 由 CSR 渲染，WebFetch 抓不到，但参数定义本身是 SSR / 静态文本，可正常抓取。

## 2. 抓取要点

- **优先 WebFetch**：上述 `/zh-cn/...` 页面均可直接抓。
- **OpenAPI 原文**：`/zh-cn/api/...` 这一族页面背后有 OpenAPI 1.0.0 定义，WebFetch 返回的是 OpenAPI 渲染后的字段表（类型、必填、默认值、约束）。撰写时以此为权威源。
- **指南补充字段语义**：参数语义（如 `prefix` 限制、`thinking` 与温度互斥）经常只在 `/zh-cn/guides/...` 写明，需要交叉抓取。
- **失败回退**：如果中文页 WebFetch 返回内容稀疏，切到英文路径 `https://api-docs.deepseek.com/<同 slug>` 再抓；两者字段定义一致。
- **不要抓** GitHub README / 知乎软文 / 第三方教程，会引入未经官方核实的字段。

## 3. 更新流程

1. **检查更新日志**：抓 `/zh-cn/updates`，比对最新条目日期是否晚于本仓库各 md 文件的 `fetched_at`。
2. **检查模型清单**：抓 `/zh-cn/quick_start/pricing`，确认 `model` ID、上下文窗口、最大输出、价格是否变动；同步 `deepseek/README.md` 与 `deepseek/models.md`。
3. **检查 News**：抓 `/zh-cn/news/...` 最新一两条，看是否有新端点、新参数（例如 `thinking.reasoning_effort`、`stream_options.include_usage`）。
4. **逐端点重抓**：对每个 `/zh-cn/api/<endpoint>` 重新 WebFetch，diff 字段表，更新对应 md 的参数表。
5. **同步示例**：若字段或返回结构变化，更新文档末尾的最小请求/响应示例。
6. **更新 `fetched_at`**：所有改动文件统一改为新日期（北京时间）。

## 3.1 定价文档维护（`deepseek/pricing.md`）

- **唯一权威源**：`https://api-docs.deepseek.com/zh-cn/quick_start/pricing`（中文页；英文页同结构 `/quick_start/pricing`）。
- **同步触发**：促销活动到期（**v4-pro 75% off 截至 2026-05-31**，必查）、新增 / 弃用模型、缓存命中价档位调整、上下文窗口或最大输出 token 变动、`deepseek-chat` / `deepseek-reasoner` 旧 ID 下线（计划 2026-07-24）。
- **抓取要点**：
  - 抓取偶发"无法验证域名安全"错误 → 直接重试同一 URL；其次回退到英文页面 `https://api-docs.deepseek.com/quick_start/pricing`。
  - 页面同时包含两档价格表：**标准价** 与 **折扣价**，撰写 `pricing.md` 时必须同时列出，写明截止日期。
  - 单价以 ¥ 计、单位每 1M tokens；不要换算成 USD（汇率会过期）。
- **同步步骤**：
  1. WebFetch pricing 页 → 对比 `deepseek/pricing.md` 数字。
  2. 改 `pricing.md` 的 `fetched_at`；活动到期日变动一并改正文。
  3. 同步 `deepseek/README.md` 的 `last_updated`。
  4. 若有新模型 ID，同步 `deepseek/README.md` 与 `deepseek/models.md`。
- **坑点提醒**：
  - v4-pro 折扣官方表述为"75% 折扣"（结算价 = 原价 25%）；不要把"75% off"理解成"75% 的钱"。
  - `deepseek-v4-flash` 当前没有"思考价"档位差，但 `reasoning_tokens` 仍计入 output。
  - 限速不公开，不要在 pricing.md 里补造 TPM / RPM 数字。

## 4. 坑点清单

- **OpenAI 协议兼容但有特有字段**：`reasoning_content`、`prefix`、`prompt_cache_hit_tokens`、`prompt_cache_miss_tokens`、`completion_tokens_details.reasoning_tokens` 这些不在 OpenAI 原 schema 中；写文档时务必单独列出，避免与 OpenAI 文档混淆。
- **Beta 端点必须切 base URL**：`prefix` 续写和 FIM `/beta/completions` 都必须把 base URL 改成 `https://api.deepseek.com/beta`，普通 base URL 上调用会 404 或忽略 `prefix`。
- **`prefix` 位置限制**：只能用于 messages 数组**最后一条**消息，且该消息 `role` 必须为 `assistant`，且 `prefix: true`。
- **`thinking` 与温度互斥**：思考模式开启时，`temperature` / `top_p` / `frequency_penalty` / `presence_penalty` 不生效，但不会报错。撰写文档时要在参数表里说明。
- **`reasoning_content` 多轮回传规则**：仅当上一轮发生 `tool_calls` 时必须在下一轮回传；否则可省略。
- **思考模式不一定返回 `reasoning_content`**：某些场景模型内部 think 但不暴露；不要把它当成「一定有」的字段。
- **`frequency_penalty` / `presence_penalty` 在 FIM 已弃用**：在 chat 端点仍兼容（非思考模式下生效），在 `/beta/completions` 上明确标注 deprecated。
- **`logprobs` 类型 chat vs FIM 不一致**：chat 端点是 `boolean`+`top_logprobs:integer`；FIM 端点 `logprobs:integer` 直接指 top-N。
- **旧模型 ID 即将下线**：`deepseek-chat` / `deepseek-reasoner` 计划 2026-07-24 下线，分别映射到 `deepseek-v4-flash` 的非思考 / 思考模式。文档要双写，但鼓励读者迁移到 v4 显式 ID。
- **价格页 75% 折扣是阶段性活动**：当前对 v4-pro 有 75% off，截止 2026-05-31。每次更新都要重抓 pricing 页确认折扣是否仍有效。
- **限速无静态值**：DeepSeek 不公开 TPM / RPM 阈值（动态限流），不要补造数字。
- **保活占位符**：非流式响应在等待时可能返回多个空行；流式响应在等待时发送 `: keep-alive`。需要在 chat-completions / rate-limits 两处说明。
- **`/v1` 别名**：`api.deepseek.com` 与 `api.deepseek.com/v1` 是同一套接口，仅为兼容 OpenAI SDK 默认路径。不是 API 版本号。

## 5. 关键链接表

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| 入口 / 首次调用 | https://api-docs.deepseek.com/zh-cn/ | `deepseek/README.md` |
| API 总览 | https://api-docs.deepseek.com/zh-cn/api/deepseek-api | `deepseek/README.md` |
| 模型 & 价格 | https://api-docs.deepseek.com/zh-cn/quick_start/pricing | `deepseek/README.md`、`deepseek/models.md`、`deepseek/pricing.md` |
| Chat Completions | https://api-docs.deepseek.com/zh-cn/api/create-chat-completion | `deepseek/chat-completions.md` |
| FIM Completion | https://api-docs.deepseek.com/zh-cn/api/create-completion | `deepseek/fim-completion.md` |
| List Models | https://api-docs.deepseek.com/zh-cn/api/list-models | `deepseek/models.md` |
| User Balance | https://api-docs.deepseek.com/zh-cn/api/get-user-balance | `deepseek/user-balance.md` |
| 思考模式 | https://api-docs.deepseek.com/zh-cn/guides/thinking_mode | `deepseek/chat-completions.md` |
| 多轮对话 | https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat | `deepseek/chat-completions.md` |
| Prefix 续写 | https://api-docs.deepseek.com/zh-cn/guides/chat_prefix_completion | `deepseek/chat-completions.md` |
| FIM 指南 | https://api-docs.deepseek.com/zh-cn/guides/fim_completion | `deepseek/fim-completion.md` |
| JSON Output | https://api-docs.deepseek.com/zh-cn/guides/json_mode | `deepseek/chat-completions.md` |
| Tool Calls | https://api-docs.deepseek.com/zh-cn/guides/tool_calls | `deepseek/chat-completions.md` |
| Context Cache | https://api-docs.deepseek.com/zh-cn/guides/kv_cache | `deepseek/caching.md` |
| Anthropic 兼容 | https://api-docs.deepseek.com/zh-cn/guides/anthropic_api | （未整理） |
| 错误码 | https://api-docs.deepseek.com/zh-cn/quick_start/error_codes | `deepseek/errors.md` |
| 限速 | https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit | `deepseek/rate-limits.md` |
| 更新日志 | https://api-docs.deepseek.com/zh-cn/updates | — |
