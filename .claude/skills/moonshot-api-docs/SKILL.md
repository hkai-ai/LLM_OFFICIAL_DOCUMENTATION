---
name: moonshot-api-docs
description: 当需要查阅、更新或扩充 Moonshot Kimi 官方 API 文档（/v1/chat/completions、partial mode、思考模式 thinking、prompt_cache_key、estimate-token-count、files、user/balance、batch、tool-use 与 $web_search 内置工具、JSON Mode、Vision、错误码、Tier 限速、入门指南（guides.md）、**定价（pricing.md）** 等）在本仓库 `moonshot/` 目录下的中文整理版本时使用。触发场景包括："Moonshot 怎么用"、"Kimi API"、"kimi-k2.6 思考模式"、"Partial Mode 怎么用"、"Moonshot 缓存命中"、"Moonshot 限速 Tier"、"Moonshot 错误码 429"、"刷新 Kimi 文档"、"Kimi 新版本 K2.6 / K2.5"、"Kimi 价格"、"Kimi 批量推理 Batch 60%"、"K2 模型定价"、"Kimi 工具调用"、"Kimi web search"、"Kimi 视觉模型"、"Kimi 入门指南"、"刷新 Kimi guide"。
---

# Moonshot Kimi API 文档撰写与维护

## 1. 文档站全貌

- 入口：`https://platform.kimi.com/docs/`
- 文档采用 SSR 静态渲染，每页都有对应 `.md` 镜像（在 URL 末尾追加 `.md` 即得纯文本版本，对 WebFetch 极友好）。
- 顶级板块：
  - **概览 / Quickstart**：`/docs/overview.md`、`/docs/introduction.md`、`/docs/models.md`、`/docs/api/quickstart.md`
  - **API 参考**：`/docs/api/<name>.md`（包括 `overview` / `chat` / `tool-use` / `partial` / `estimate` / `errors` / `list-models` / `models-overview` / `balance`）
  - **文件**：`/docs/api/files-*.md`（upload / list / retrieve / content / delete）
  - **Batch**：`/docs/api/batch-*.md`（create / list / retrieve / cancel）
  - **使用指南**：`/docs/guide/<slug>.md`（涵盖思考模式、视觉、JSON Mode、Partial、多轮、Web Search、Tools、Playground、Kimi CLI、MoonPalace、OpenClaw 等）
  - **定价**：`/docs/pricing/<slug>.md`（chat / chat-k26 / chat-k25 / chat-k2 / chat-v1 / batch / tools / limits / faq）
  - **OpenAPI 原文**：`/docs/openapi.json`（权威字段源）
  - **协议**：`/docs/agreement/*.md`
- 无需登录、无 paywall；部分 Playground 是 CSR 渲染，但参数定义全部在 SSR `.md` 中。
- **文档总索引**：`https://platform.kimi.com/docs/llms.txt`，列出全部 `.md` 路径与主题，强烈建议每次更新先抓这个。

## 2. 抓取要点

- **优先 `.md` 镜像**：所有页面在路径末尾加 `.md`，WebFetch 直接返回 markdown，省去 HTML 解析失真。
- **OpenAPI 是权威**：`/docs/openapi.json` 是字段类型 / 必填 / 默认值的原始来源，但内容庞大，宜按 endpoint 局部读取或先从 `.md` 拿字段表再去 OpenAPI 验证。
- **定价分两层**：`/docs/pricing/chat.md` 是索引，单价在 `chat-k26.md` / `chat-k25.md` / `chat-k2.md` / `chat-v1.md` 子页里。
- **`/docs/api/models-overview.md`**：写每个模型系列的参数差异（temperature 默认值不同、thinking 是否可关），与 `/models.md`（人工概念页）和 `/api/list-models.md`（OpenAPI 端点页）共三处都要核对。
- **失败回退**：极少数页面 SSR 渲染缺失时，可改抓 `https://platform.moonshot.cn/docs/...` 旧域名（早期使用），但 platform.kimi.com 已是当前主域。
- **不要抓** GitHub README / 第三方教程 / 知乎软文，避免引入未经核实的字段。

## 3. 更新流程

1. **抓 `llms.txt`**：比较与本仓库已记录的页面集合差异，发现新增 / 移除主题。
2. **抓 `/docs/pricing/limits.md`**：核对 Tier 阶梯（充值门槛 / 并发 / RPM / TPM / TPD）是否变动，同步 `moonshot/rate-limits.md`。
3. **抓 `/docs/pricing/chat-k26.md`、`chat-k25.md`、`chat-k2.md`、`chat-v1.md`**：核对各系列单价（含缓存命中 / 未命中），同步 `moonshot/README.md` 与 `moonshot/models.md`。
4. **抓 `/docs/api/chat.md`**：diff 字段表，更新 `moonshot/chat-completions.md`；注意 thinking / partial / prompt_cache_key / response_format 等特有字段。
5. **抓 `/docs/api/list-models.md`**：核对 List Models 响应字段（`supports_image_in` / `supports_video_in` / `supports_reasoning` 等扩展位）。
6. **抓 `/docs/api/errors.md`**：核对错误码 type / code 列表，同步 `moonshot/errors.md`。
7. **抓 `/docs/api/partial.md` / `/docs/api/estimate.md` / `/docs/api/balance.md`**：维护辅助端点文档。
8. **更新 `fetched_at`**：所有改动文件统一改为新日期（北京时间）。

## 3.05 入门指南 / 能力文档维护（`moonshot/guides.md` + 各能力 md）

本仓库为 Kimi 的"能力 / 入门指南"建立了独立的 md 文件：

| 能力 / 指南 | 仓库文件 | 对应 `/guide/` 来源 |
| --- | --- | --- |
| 入门指南索引 | `moonshot/guides.md` | 汇总 `/guide/*` 所有页面 |
| Batch API（API + Guide 合并） | `moonshot/batch.md` | `/api/batch-*` + `/guide/use-batch-api` + `/guide/use-batch-inference` |
| Files API | `moonshot/files.md` | `/api/files*` + `/guide/use-kimi-api-for-file-based-qa` |
| 工具调用 | `moonshot/tool-use.md` | `/api/tool-use` + `/guide/use-kimi-api-to-complete-tool-calls` + `/guide/use-official-tools` |
| JSON Mode | `moonshot/json-mode.md` | `/guide/use-json-mode-feature-of-kimi-api` |
| 视觉 / 视频多模态 | `moonshot/vision.md` | `/guide/use-kimi-vision-model` |
| 思考模式 | `moonshot/thinking.md` | `/guide/use-kimi-k2-thinking-model` + `/guide/kimi-k2-6-quickstart` |
| 联网搜索 `$web_search` | `moonshot/web-search.md` | `/guide/use-web-search` |
| Partial Mode | `moonshot/partial-mode.md` | `/api/partial` + `/guide/use-partial-mode-feature-of-kimi-api` |

### 同步策略

1. **首选 `.md` 镜像**：所有 `/guide/<slug>` 与 `/api/<slug>` 都有同名 `.md` 镜像（URL 末尾加 `.md`），WebFetch 直接返回纯 markdown。
2. **每次刷新先抓 `llms.txt`**：对照 `moonshot/guides.md` 的索引，看是否有新增 guide（K2.7 / 新工具 / 新 SDK 集成等）。
3. **能力 md 与 chat-completions.md 互相引用**：能力 md 主要覆盖参数语义、限制、组合规则；具体字段表（如 `response_format` / `thinking` / `tools`）以 `chat-completions.md` 为权威。
4. **guides.md 是入门索引**：当某个 guide 内容简单（只是"如何接入 IDE / CLI"），保留在 `guides.md` 的链接表里即可，不必为每个 guide 都展开成独立 md。
5. 更新任一 md 后，同步 `moonshot/README.md` 的 `last_updated`。

### 与 chat-completions 字段表的边界

- 字段定义（参数表 / 响应表 / 流式事件）：写在 `chat-completions.md`。
- 字段的使用模式、组合规则、限制、示例：可以同时出现在能力 md 中。
- 当 API 字段变动时（如 `thinking` 新增 `keep` 选项），先改 `chat-completions.md`，再同步到对应能力 md。

## 3.1 定价文档维护（`moonshot/pricing.md`）

- **权威源（分散在多个子页）**：所有 pricing 子页都有 `.md` 镜像，强烈优先抓 `.md` 版本。
  - 总索引：`https://platform.kimi.com/docs/pricing/chat.md`
  - K2.6：`https://platform.kimi.com/docs/pricing/chat-k26.md`
  - K2.5：`https://platform.kimi.com/docs/pricing/chat-k25.md`
  - Moonshot V1：`https://platform.kimi.com/docs/pricing/chat-v1.md`
  - Batch：`https://platform.kimi.com/docs/pricing/batch.md`
  - 工具（Web Search）：`https://platform.kimi.com/docs/pricing/tools.md`
  - 限速 / Tier：`https://platform.kimi.com/docs/pricing/limits.md`
  - FAQ：`https://platform.kimi.com/docs/pricing/faq.md`
- **同步触发**：模型新增（如 K2.7）/ 子型号下线（如 `kimi-k2-*-preview` 计划 2026-05-25 下线）/ 批量推理折扣比例调整（当前 60%）/ Web Search 单价调整 / Tier 阶梯变动。
- **抓取要点**：
  - `chat.md` 是导航页，**不包含实际价格**——必须深入每个子页抓数字。
  - K2.5 / K2.6 区分 **缓存命中价 / 未命中价**；`moonshot-v1` 系列**没有**缓存档位，仅一档输入价。
  - Vision-preview 版本（`moonshot-v1-Nk-vision-preview`）与同上下文文本版同价。
  - Batch 价 = 标准价 × 60%（不是 50%），文档须显式标注。
- **同步步骤**：
  1. 先抓 `llms.txt` 比对 pricing 子页清单是否新增（如未来出现 `chat-k27.md`）。
  2. 逐子页 WebFetch `.md` 镜像 → 更新 `moonshot/pricing.md` 对应小节。
  3. 改 `pricing.md` 的 `fetched_at`，同步 `moonshot/README.md` 的 `last_updated`。
  4. 模型 ID 变动同步 `moonshot/README.md` 与 `moonshot/models.md`。
- **坑点提醒**：
  - `kimi-k2-*-preview` 下线后定价子页可能仍存在但标记为"已下线"，撰写时把这些模型移到"已下线"小节，不删 URL 引用。
  - Batch 价目前只列出 K2.6 / K2.5；其他模型按 60% 推导写在 pricing.md 时要注明"按公式推导，官方未列"。
  - 缓存创建价（cache write）在 K2.5 / K2.6 子页**未公开**；不要凭印象补造数字。
  - Web Search ¥0.03/次是"工具调用费"，搜索返回内容的 token 另按输入价计；写文档要把两部分分开列。

## 4. 坑点清单

- **`platform.kimi.com` ≠ `kimi.ai`**：前者是开发者平台（中国大陆），后者是消费版（海外），API Key 不互通；用户报「鉴权失败」时先确认走的是哪边。
- **思考模式与采样参数互斥**：`thinking.type=enabled` 时，`temperature` / `top_p` / `presence_penalty` / `frequency_penalty` / `top_k` 不报错但被静默忽略；`reasoning_content` 不保证一定返回。
- **`partial` 的位置**：写在 `messages[].assistant` 内（不是顶层参数），且必须是 messages 数组**最后一条** assistant 消息；不可与 `response_format.type=json_object` 同用。
- **`thinking.keep` 是 K2.6 专属**：用于多轮中保留历史 `reasoning_content`，K2.5 没有这个字段。
- **`max_tokens` 已弃用**：现在叫 `max_completion_tokens`，默认 1024；老代码不报错但语义需关注。
- **moonshot-v1 vs K2 系列的 temperature 默认值不同**：v1 默认 `0.0`，K2 系列默认 `0.6`；迁移时若不显式传值行为会改变。
- **缓存命中字段**：K2.5 / K2.6 才有 `cached_tokens` 与缓存命中 / 未命中两档价；moonshot-v1 系列只有单一输入价。
- **`prompt_cache_key`**：值需在同一前缀场景内保持一致以提升命中率；与 OpenAI 的同名参数语义一致。
- **`ms://<file_id>` 协议**：多模态 `image_url.url` / `video_url.url` 支持百炼平台文件引用；上传后通过 `/v1/files` 拿到的 `file_id` 即可拼接。
- **Tier0 限速极严**：3 RPM、500K TPM、1.5M TPD；接入测试若用免费额度容易触发限流被误判为 bug。
- **K2 预览系列下线日期**：`kimi-k2-*-preview` 全部计划 2026-05-25 下线，定期核对 News 与 updates 调整 README 表格。
- **错误结构是 `{error: {...}}` 形式**：与 OpenAI 一致，三字段 `message` / `type` / `code`；其中 `type` 决定 HTTP 状态，`code` 是细分。
- **保活机制**：非流式响应等待期间插入空行，流式发送 `: keep-alive` SSE 注释，客户端需容忍。
- **`stream_options.include_usage`** 默认 `false`，流式不带 usage；需要计费统计时必须显式开。

## 5. 关键链接表

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| 入口 / 概览 | https://platform.kimi.com/docs/api/overview | `moonshot/README.md` |
| 文档总索引 | https://platform.kimi.com/docs/llms.txt | （查找用） |
| OpenAPI | https://platform.kimi.com/docs/openapi.json | （字段校验权威） |
| 入门指南索引 | https://platform.kimi.com/docs/guide/start-using-kimi-api | `moonshot/guides.md` |
| Chat Completions | https://platform.kimi.com/docs/api/chat | `moonshot/chat-completions.md` |
| 工具调用 API | https://platform.kimi.com/docs/api/tool-use | `moonshot/tool-use.md` |
| 工具调用 Guide | https://platform.kimi.com/docs/guide/use-kimi-api-to-complete-tool-calls | `moonshot/tool-use.md` |
| 官方工具集成 Guide | https://platform.kimi.com/docs/guide/use-official-tools | `moonshot/tool-use.md` |
| JSON Mode Guide | https://platform.kimi.com/docs/guide/use-json-mode-feature-of-kimi-api | `moonshot/json-mode.md` |
| 视觉模型 Guide | https://platform.kimi.com/docs/guide/use-kimi-vision-model | `moonshot/vision.md` |
| 思考模式 Guide | https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model | `moonshot/thinking.md` |
| K2.6 快速开始 | https://platform.kimi.com/docs/guide/kimi-k2-6-quickstart | `moonshot/thinking.md` |
| 联网搜索 Guide | https://platform.kimi.com/docs/guide/use-web-search | `moonshot/web-search.md` |
| Partial Mode API | https://platform.kimi.com/docs/api/partial | `moonshot/partial-mode.md` |
| Partial Mode Guide | https://platform.kimi.com/docs/guide/use-partial-mode-feature-of-kimi-api | `moonshot/partial-mode.md` |
| 多轮对话 Guide | https://platform.kimi.com/docs/guide/engage-in-multi-turn-conversations-using-kimi-api | `moonshot/chat-completions.md` |
| 流式输出 Guide | https://platform.kimi.com/docs/guide/utilize-the-streaming-output-feature-of-kimi-api | `moonshot/chat-completions.md` §流式 |
| 自动断线重连 Guide | https://platform.kimi.com/docs/guide/auto-reconnect | `moonshot/chat-completions.md` |
| 文件问答 Guide | https://platform.kimi.com/docs/guide/use-kimi-api-for-file-based-qa | `moonshot/files.md` |
| Batch API（创建） | https://platform.kimi.com/docs/api/batch-create | `moonshot/batch.md` |
| Batch API（列表 / 检索 / 取消） | https://platform.kimi.com/docs/api/batch-list, /batch-retrieve, /batch-cancel | `moonshot/batch.md` |
| Batch Guide | https://platform.kimi.com/docs/guide/use-batch-api | `moonshot/batch.md` |
| Batch 控制台 Guide | https://platform.kimi.com/docs/guide/use-batch-inference | `moonshot/batch.md` |
| Files API（5 端点） | https://platform.kimi.com/docs/api/files, /files-upload, /files-list, /files-retrieve, /files-content, /files-delete | `moonshot/files.md` |
| Estimate Tokens | https://platform.kimi.com/docs/api/estimate | `moonshot/estimate-tokens.md` |
| List Models | https://platform.kimi.com/docs/api/list-models | `moonshot/models.md` |
| 模型参数总览 | https://platform.kimi.com/docs/api/models-overview | `moonshot/models.md` |
| User Balance | https://platform.kimi.com/docs/api/balance | `moonshot/user-balance.md` |
| 错误码 | https://platform.kimi.com/docs/api/errors | `moonshot/errors.md` |
| 限速 / Tier | https://platform.kimi.com/docs/pricing/limits | `moonshot/rate-limits.md` |
| Prompt 最佳实践 | https://platform.kimi.com/docs/guide/prompt-best-practice | （未整理） |
| Benchmark 最佳实践 | https://platform.kimi.com/docs/guide/benchmark-best-practice | （未整理） |
| 组织认证最佳实践 | https://platform.kimi.com/docs/guide/org-best-practice | （未整理） |
| Kimi CLI | https://platform.kimi.com/docs/guide/kimi-cli-support | `moonshot/guides.md` |
| MoonPalace 调试 | https://platform.kimi.com/docs/guide/use-moonpalace | `moonshot/guides.md` |
| Playground | https://platform.kimi.com/docs/guide/use-playground-to-debug-the-model | `moonshot/guides.md` |
| OpenClaw 集成 | https://platform.kimi.com/docs/guide/use-kimi-in-openclaw | `moonshot/guides.md` |
| ModelScope MCP | https://platform.kimi.com/docs/guide/configure-the-modelscope-mcp-server | `moonshot/guides.md` |
| 编程工具集成 | https://platform.kimi.com/docs/guide/agent-support | `moonshot/guides.md` |
| 定价索引 | https://platform.kimi.com/docs/pricing/chat | `moonshot/pricing.md`、`moonshot/README.md`、`moonshot/models.md` |
| K2.6 定价 | https://platform.kimi.com/docs/pricing/chat-k26 | `moonshot/pricing.md` |
| K2.5 定价 | https://platform.kimi.com/docs/pricing/chat-k25 | `moonshot/pricing.md` |
| K2 定价（预览系列） | https://platform.kimi.com/docs/pricing/chat-k2 | `moonshot/pricing.md`（已下线小节） |
| V1 定价 | https://platform.kimi.com/docs/pricing/chat-v1 | `moonshot/pricing.md` |
| Batch 价（60% off） | https://platform.kimi.com/docs/pricing/batch | `moonshot/pricing.md` §3 |
| Web Search 工具价 | https://platform.kimi.com/docs/pricing/tools | `moonshot/pricing.md` §4 |
| Pricing FAQ | https://platform.kimi.com/docs/pricing/faq | （核对用） |
| 思考模型指南 | https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model | `moonshot/chat-completions.md` §thinking |
| 从 OpenAI 迁移 | https://platform.kimi.com/docs/guide/migrating-from-openai-to-kimi | `moonshot/README.md` |
| FAQ | https://platform.kimi.com/docs/guide/faq | `moonshot/errors.md` 排查段 |
