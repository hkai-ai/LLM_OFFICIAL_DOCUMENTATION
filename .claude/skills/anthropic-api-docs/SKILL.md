---
name: anthropic-api-docs
description: 需要查阅、更新或扩充本仓库 anthropic/ 下 Anthropic Claude API 中文文档时使用。涵盖 Messages、Streaming、Count Tokens、Models、Errors、Versioning、**定价（pricing.md）** 等端点与主题，包括字段定义、模型清单、anthropic-version / anthropic-beta header、API 重定向（docs.anthropic.com → platform.claude.com）、prompt caching / Batch / Fast mode / 工具与 Managed Agents 定价等。触发条件示例："更新 Anthropic API 文档"、"Claude API 字段对一下"、"补全 Messages API 的 content block"、"刷新模型清单"、"同步 Claude 定价"、"Claude pricing"、"prompt caching 倍率"、"Opus 4.7 价格"。
---

# Anthropic Claude API 文档 skill

## 1. 文档站全貌

### 域名

- **现行域名**：`https://platform.claude.com/docs/en/api/`
- **旧域名**：`https://docs.anthropic.com/en/api/` — 返回 `301 Moved Permanently` 永久重定向到新域名。WebFetch 抓旧域名会返回 `REDIRECT DETECTED`，需用新 URL 重发请求。
- API 调用 base URL 不变：`https://api.anthropic.com`。
- 无需登录即可查看（部分 Console 链接如 `/settings/keys` 需要登录，但 API 文档本体公开）。

### 顶级目录（按侧边栏组织）

- **API reference**
  - `overview`
  - Messages 系列：`messages`、`messages/create`、`messages-streaming`、`messages-count-tokens`
  - Models 系列：`models`、`models-list`（List 与 Retrieve 同页）
  - Batch：`creating-message-batches` 等
  - Files：`files-create` 等
  - Skills：`skills/create-skill` 等
  - Managed Agents：`/docs/en/managed-agents/...`（与 API reference 平级）
- **Forming requests**：`client-sdks`、`authentication`、`workload-identity-federation`
- **Handling responses**：`errors`、`rate-limits`、`service-tiers`、`supported-regions`
- **Versioning**：`versioning`、`beta-headers`
- **About Claude / Models**：`about-claude/models/overview`、`models/model-ids-and-versions`、`model-deprecations`、`pricing`
- **Build with Claude / Capabilities**：`build-with-claude/streaming`、`extended-thinking`、`adaptive-thinking`、`structured-outputs`、`prompt-caching`、`vision`、`pdf-support`、`batch-processing`、`tool-use/*`、`web-search`、`web-fetch`、`code-execution`、`memory`、`citations` 等
- **Manage Claude**：`authentication`、`workload-identity-federation`、`spend-limits`、`primary-owner` 等

## 2. 抓取要点

### 哪些 URL 可被 WebFetch 直接抓取

`platform.claude.com` 在多数请求中会返回正常 Markdown 渲染（WebFetch 内部已将 HTML 转 Markdown）。**但偶发返回 "Unable to verify if domain platform.claude.com is safe to fetch."**（域名安全校验失败）。该错误是间歇性的，**直接重试同一 URL 通常即可**。不要切到 `docs.anthropic.com`，因为它只会再次给出 301 重定向到 `platform.claude.com`，徒增一次往返。

### 推荐流程

1. 先抓 `https://platform.claude.com/docs/en/api/overview` 获得侧边栏全貌。
2. 直接按已知 URL 抓子页（参见下方「关键链接表」）。
3. 如果不知道具体 URL，可用 WebSearch（限定 `site:platform.claude.com` 或 `site:docs.anthropic.com`）定位。
4. 抓 OpenAPI 风格的字段表（例如 `messages-list` 页面），适合用 WebFetch 的 `prompt` 参数明确点名要哪些字段，避免被 LLM 抽样省略。
5. 单次抓取被截断时（>50KB），结果会写到 persisted-output 文件，按提示用 Read 读出。

### 大页处理

`messages-streaming` 这种全量带多语言代码示例的页面，原文超过 50KB，会触发 WebFetch 的输出 persist。读取路径形如：
`C:\Users\<User>\.claude\projects\<project-id>\<session-id>\tool-results\<tool_use_id>.txt`

### 无内容页

`https://platform.claude.com/docs/en/api/models-get` 实际上不存在（返回 "Not Found" + Loading 占位 → WebFetch 拿到的是空 SPA 壳）。Retrieve 端点的真正文档位于 `…/api/models`（与 List 同页），或 `…/api/models-list` 也包含它。

## 3. 更新流程

### 检查变动的入口

1. **版本号 / changelog**：`https://platform.claude.com/docs/en/api/versioning` 记录 `anthropic-version` 历史；`anthropic-beta` 清单可从 `/v1/models` 的 OpenAPI（在 `models-list` 页面）反查全部已知 beta 名。
2. **模型清单**：`https://platform.claude.com/docs/en/about-claude/models/overview` 是单一权威表，比 `models-list` 端点的实时返回更易读。
3. **弃用计划**：`https://platform.claude.com/docs/en/about-claude/model-deprecations`。
4. **Messages 字段**：`https://platform.claude.com/docs/en/api/messages` 与子页 `messages/create`。注意 `messages` 主页是「全景介绍」，`messages/create` 才是 OpenAPI 风格字段表。
5. **流式事件类型**：`https://platform.claude.com/docs/en/api/messages-streaming`。

### 同步本仓库

1. 抓上述页面 → 把字段表、枚举值、模型清单更新到 `anthropic/` 下对应 `.md`。
2. 全部使用绝对 URL 作为 `source`，更新 frontmatter 的 `fetched_at` 为当天日期（北京时间）。
3. 保持字段名、枚举值、HTTP header、URL、模型 ID 等英文原文不翻译。
4. 不臆测；遇到歧义写 "文档未明确"。

## 3.1 定价文档维护（`anthropic/pricing.md`）

- **唯一权威源**：`https://platform.claude.com/docs/en/about-claude/pricing`。
- **同步触发**：模型新增 / 弃用（pricing 表新增列或加 deprecated 标记）、prompt caching 倍率调整、Batch / Fast mode 折扣率变动、新增工具附加费、Managed Agents 价格变动、`inference_geo` 倍率或区域路由价格调整。
- **抓取要点**：
  - 单页文档结构清晰但很长，建议用 WebFetch 时显式 prompt 要求列出 §model pricing / §batch / §long context / §data residency / §fast mode / §tool use / §code execution / §managed agents 等小节的所有数字。
  - 若遇间歇 `Unable to verify if domain platform.claude.com is safe to fetch.`，直接重试同一 URL；不要切回 `docs.anthropic.com`（只会 301）。
  - Fast mode 当前为 beta（research preview），需在 pricing.md 文末维持 beta 标识；GA 后再去掉。
- **同步步骤**：
  1. WebFetch pricing 页 → 比对 `anthropic/pricing.md` 的 §1～§9 全部数字与表格。
  2. 修改 `pricing.md` frontmatter 的 `fetched_at`。
  3. 同步 `anthropic/README.md` frontmatter 中的 `last_updated`。
  4. 如新增 / 弃用模型，同步 `anthropic/README.md` 与 `anthropic/models.md` 的模型清单表。
- **坑点提醒**：
  - 缓存倍率（1.25× / 2× / 0.1×）是相对 base input 价的"乘数"，文档要保留乘数定义而非只列绝对值，因为 Opus 4.1 等高价模型乘数相同但绝对数字不同。
  - Long context（1M 窗口）当前不另立单价档；900k 与 9k 请求同价。注意官方表述未来可能改变。
  - 工具附加费分两类：**按次**（web search $10/1k）与 **按 token**（tool use 系统 prompt 开销）；写文档时要把两类分开。
  - Code Execution 在与 `web_search_*` / `web_fetch_*` 同用时**免费**，单独使用才计费。这条规则常被遗漏。

## 4. 坑点清单（基于本次撰写的实际经验）

1. **301 重定向**：`docs.anthropic.com/en/api/<page>` 全部 301 到 `platform.claude.com/docs/en/api/<page>`。WebFetch 命中 301 会返回 `REDIRECT DETECTED`，必须用新 URL 重试。
2. **域名安全校验间歇失败**：`platform.claude.com` 偶发 `Unable to verify if domain ... is safe to fetch.`。**直接重试同一 URL** 即可，不要切回旧域名（旧域名只会再次 301）。
3. **`models-get` 是 404 SPA 壳**：Retrieve 模型的真实文档在 `…/api/models`（与 List 同页）或 `…/api/models-list`。
4. **`messages` vs `messages/create`**：`messages` 主页是导览，正式字段表在 `messages/create` 子页。撰写参数文档时优先抓后者。
5. **`messages-streaming` 体积超 50KB**：会触发 WebFetch 的 persisted output，需要按提示 Read 持久化文件。
6. **`Anthropic` 与 `Claude` 双品牌**：新域名是 `platform.claude.com`，但响应里、SDK 包名里依然大量出现 `anthropic` 前缀（`anthropic-version`、`anthropic-beta`、`api.anthropic.com`、`anthropic-sdk-*`）。**不要随手把 `anthropic-*` header 改成 `claude-*`**。
7. **Opus 4.7 用新 tokenizer**：1M token 不再大致对应 750k 词，而是 ~555k 词。涉及上下文容量描述要分开列。
8. **Opus 4.7 / 4.6 / Sonnet 4.6 不支持 prefill**：易踩。文档明确返回 400。
9. **`message_delta.usage` 是累积值**：流式累加时容易重复计。
10. **`input_json_delta` 的 `partial_json` 是字符串**：最终 `tool_use.input` 才是 object；累计到 `content_block_stop` 后再解析。
11. **`alias` 不是 evergreen**：4.6 及之后 dateless ID 本身是 pinned snapshot。`claude-opus-4-7` ≠ "永远指向最新 Opus 4 系列"。
12. **`betas` 字段 vs `anthropic-beta` header**：两种方式等价，但 SDK 多用前者；OpenAPI 文档里 header 取值是 `array of AnthropicBeta`，多个用逗号分隔写在单条 header 内。
13. **Managed Agents 路由**：在 `/docs/en/managed-agents/...` 下，不在 `/docs/en/api/` 里，撰写端点索引时容易漏。
14. **request_id 字段位**：非流式错误响应顶层有 `request_id`；流式 `event: error` 的 data 里没有，要靠 HTTP `request-id` header。

## 5. 关键链接表

| 文档 | 官方 URL |
| --- | --- |
| API overview | `https://platform.claude.com/docs/en/api/overview` |
| Messages 主页 | `https://platform.claude.com/docs/en/api/messages` |
| Messages create（OpenAPI 字段） | `https://platform.claude.com/docs/en/api/messages/create` |
| Messages streaming | `https://platform.claude.com/docs/en/api/messages-streaming` |
| Count tokens | `https://platform.claude.com/docs/en/api/messages-count-tokens` |
| Models（List + Retrieve） | `https://platform.claude.com/docs/en/api/models` |
| Models List 单页 | `https://platform.claude.com/docs/en/api/models-list` |
| Errors | `https://platform.claude.com/docs/en/api/errors` |
| Versioning | `https://platform.claude.com/docs/en/api/versioning` |
| Beta headers | `https://platform.claude.com/docs/en/api/beta-headers` |
| Client SDKs | `https://platform.claude.com/docs/en/api/client-sdks` |
| Rate limits | `https://platform.claude.com/docs/en/api/rate-limits` |
| Service tiers | `https://platform.claude.com/docs/en/api/service-tiers` |
| Supported regions | `https://platform.claude.com/docs/en/api/supported-regions` |
| Authentication | `https://platform.claude.com/docs/en/manage-claude/authentication` |
| Message Batches create | `https://platform.claude.com/docs/en/api/creating-message-batches` |
| Files create | `https://platform.claude.com/docs/en/api/files-create` |
| Skills create | `https://platform.claude.com/docs/en/api/skills/create-skill` |
| Managed Agents：Sessions | `https://platform.claude.com/docs/en/managed-agents/sessions` |
| Managed Agents：Environments | `https://platform.claude.com/docs/en/managed-agents/environments` |
| Managed Agents：Agent setup | `https://platform.claude.com/docs/en/managed-agents/agent-setup` |
| Models overview（清单/对比） | `https://platform.claude.com/docs/en/about-claude/models/overview` |
| Model IDs & versions | `https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions` |
| Model deprecations | `https://platform.claude.com/docs/en/about-claude/model-deprecations` |
| Pricing | `https://platform.claude.com/docs/en/about-claude/pricing` → 对应 `anthropic/pricing.md` |
| Streaming（能力页） | `https://platform.claude.com/docs/en/build-with-claude/streaming` |
| Extended thinking | `https://platform.claude.com/docs/en/build-with-claude/extended-thinking` |
| Adaptive thinking | `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking` |
| Structured outputs | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs` |
| Prompt caching | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching` |
| Batch processing | `https://platform.claude.com/docs/en/build-with-claude/batch-processing` |
| Tool use（总） | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview` |
| Fine-grained tool streaming | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/fine-grained-tool-streaming` |

## 6. 本仓库文件映射

| 端点 / 主题 | 本仓库文件 |
| --- | --- |
| 厂商概览 + 端点索引 | `anthropic/README.md` |
| `POST /v1/messages` | `anthropic/messages.md` |
| `POST /v1/messages`（流式） | `anthropic/messages-streaming.md` |
| `POST /v1/messages/count_tokens` | `anthropic/count-tokens.md` |
| `GET /v1/models` / `GET /v1/models/{id}` | `anthropic/models.md` |
| 错误响应 | `anthropic/errors.md` |
| `anthropic-version` / `anthropic-beta` | `anthropic/versioning.md` |
| 定价（模型 / 缓存 / batch / fast mode / 工具 / Managed Agents） | `anthropic/pricing.md` |
