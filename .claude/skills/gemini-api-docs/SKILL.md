---
name: gemini-api-docs
description: 需要查阅、更新或扩充 Google Gemini Developer API（generativelanguage.googleapis.com）的官方文档时使用。触发场景包括：撰写或更新 google-gemini/ 目录下任一端点文档（generateContent、streamGenerateContent、countTokens、embedContent、files、caching、models、errors、**pricing** 等）；核对字段、枚举、新增能力（thinking、Live API、URL Context、Structured Output、Search Grounding）；同步官方 model 列表与上下文窗口；同步定价（含免费层 / 付费层、≤200k/>200k 分档、batch 50% off、Grounding 配额）；区分 Developer API 与 Vertex AI Gemini API 的字段差异。仅覆盖 Developer API，不写 Vertex 字段。触发示例："Gemini 价格"、"gemini-2.5-pro 长上下文定价"、"Imagen 定价"、"Veo 价格"、"Search Grounding 配额"。
---

# gemini-api-docs

本 skill 指导维护 `google-gemini/` 目录下的中文整理文档。

## 1. 官方文档站全貌

入口：`https://ai.google.dev/api?hl=zh-CN`（中文，可去掉 `?hl=zh-CN` 看英文，两者结构一致）。

一级分类（左侧导航）：

| 分类 | 子页 URL 模板 |
| --- | --- |
| Generating content | `/api/generate-content`、`/api/generate-content#method:-models.streamgeneratecontent` |
| Models | `/api/models` |
| Tokens | `/api/tokens` |
| Embeddings | `/api/embeddings` |
| Files | `/api/files` |
| Caching | `/api/caching` |
| Batch API | `/api/batch-api` |
| Live API | `/api/live`（WebSocket `BidiGenerateContent`） |
| File Search | `/api/file-search/file-search-stores`、`/api/file-search/documents` |
| Tuning | `/api/tuning` |
| Interactions API | `/api/interactions-api` |
| Live Music | `/api/live_music` |

每个端点页都自带 SDK Tab（Python / JS / Go / Java / REST / cURL），抓取时关注 REST/cURL 段以获取真正的 HTTP 字段。

**两个 API 版本**：

- `/v1beta`：主版本，所有新字段（`thinkingConfig`、`urlContext`、Live API、Batches、File Search、Tuning）只在这里。
- `/v1`：稳定版，字段是 v1beta 的子集；写文档默认以 v1beta 为准。

## 2. 抓取要点

- WebFetch `https://ai.google.dev/api/<topic>` 在大多数情况下可行。**少数页**（`/api/embeddings`、`/api/tokens`、`/api/caching`）首次抓取偶发 socket 关闭，**去掉 `?hl=zh-CN`** 用英文页可显著提高成功率。
- 想列出全部子页时，先抓入口 `/api?hl=zh-CN` 让 LLM 提取导航；它常常只给出大致分类，需要再逐一访问具体端点。
- 文档站是部分 CSR 渲染：纯 WebFetch 拿到的是预渲染后的内容，已经包含全部字段表格，**不需要**额外 JS 执行；但页面内的"枚举值展开块"必须明确 prompt 才能拿全（特别是 `HarmCategory`、`FinishReason`、`HarmBlockThreshold`）。
- **OpenAPI / Discovery 文档**：Discovery 文档为 `https://generativelanguage.googleapis.com/$discovery/rest?version=v1beta`（与 `?version=v1`），返回完整 JSON，是字段层面的最权威来源；当人工页面与 Discovery 冲突时以 Discovery 为准。
- SDK 仓库 README（`googleapis/python-genai` 等）有时比文档先一步列出新字段，可作为对照。

## 3. 更新流程

1. 抓 `https://ai.google.dev/api?hl=zh-CN`，对照本仓库 `google-gemini/README.md` 的端点索引检查是否有新分类。
2. 对每个已有 `.md`，重新抓对应 `/api/<topic>` 页，重点比对：
   - 顶层字段表（新增/移除/重命名）。
   - 枚举值（特别是 `HarmCategory`、`HarmBlockThreshold`、`FinishReason`、`taskType`、`File.state`）。
   - `GenerationConfig` 子结构（每个版本几乎都会新增 1-2 个字段）。
3. 抓 `https://ai.google.dev/gemini-api/docs/models` 同步 `models.md` 中的主推模型列表与上下文窗口；模型代次大改（如 3.x 系上线）时也要在 `README.md` 端点索引中校对方法支持矩阵。
4. 抓 `https://ai.google.dev/gemini-api/docs/troubleshooting` 校对 `errors.md` 的 `error.details[].@type` 与典型 `reason` 列表。
5. 修改每个 `.md` 的 `fetched_at` 为修改当天日期；如官方页面给出明确的版本/changelog 日期，可加在文档顶部注释。

## 3.1 定价文档维护（`google-gemini/pricing.md`）

- **唯一权威源**：`https://ai.google.dev/gemini-api/docs/pricing?hl=zh-cn`（中文，可去掉 `?hl=zh-cn` 取英文）。
- **同步触发**：新模型代次上线（如 Gemini 3.x 系列已上线）、`gemini-2.0-flash` 弃用日（**2026-06-01**）、Imagen / Veo / Lyria 子型号增减、Grounding 免费配额变动（当前每月 5,000 prompt）、Batch 折扣比例调整、`gemini-2.5-pro` ≤200k vs >200k 阈值变化、上下文缓存价（按 token·小时存储 + 按 token 读取）调整。
- **抓取要点**：
  - 单页内容非常多，**分小节抓**：第一次 prompt 要文本模型，第二次要图像 / 视频，第三次要 TTS / Live API / Embedding / Tools。
  - 免费层（Free tier）所有数字标 $0，但需在 pricing.md 文首声明"免费层内容会用于改进产品；付费层不会"。
  - **`gemini-2.5-pro` 的双档**（≤200k vs >200k 输入 token）非常容易漏；表头一定要显式列两列。
  - Veo 3.1 / Veo 3 / Veo 2 价格表按分辨率（720p / 1080p / 4K）与档位（Standard / Fast / Lite）矩阵化；不要折叠。
  - Imagen 4 三档（Fast / 标准 / Ultra）每张图统一定价。
- **同步步骤**：
  1. 抓 pricing 页 → 对比 `google-gemini/pricing.md` 各章节。
  2. 改 `pricing.md` 的 `fetched_at`。
  3. 同步 `google-gemini/README.md` 的 `last_updated`。
  4. 模型新增 / 弃用同步 `google-gemini/models.md`。
- **坑点提醒**：
  - "Search Grounding 5000 free / 月" 仅对 Gemini 3 系列；2.x / 旧模型 grounding 计费方式不同（通常无免费配额或单价不同），写文档不要把规则套到全部模型。
  - 缓存价区分"存储"（按 token·小时）与"读取"（按 token），与 Anthropic 的"乘数模型"不一致，写表时要保留两个数字。
  - `gemini-3-pro` 在主清单中出现但定价表可能未列单价（文档刚上线时常见）；遇到此情形写"文档未列出"，不要从 3.5 Flash 推导。
  - Batch 50% 折扣对部分模型并未覆盖所有模态价（如 audio input 可能没有 batch 价）；以页面表格为准。

## 4. 坑点清单

| 坑 | 说明 |
| --- | --- |
| **Developer API ≠ Vertex AI Gemini API** | 二者域名、鉴权、资源命名都不同。不要把 Vertex 才有的 `projects/{p}/locations/{l}/publishers/google/models/{m}`、OAuth、`groundingConfig.dynamicRetrievalConfig` 高级字段塞进本目录。 |
| **v1 vs v1beta** | `/v1` 缺 `thinkingConfig`、`urlContext`、`responseJsonSchema`、`speechConfig` 的较新选项等。文档主体以 v1beta 为准，差异处显式注明 `v1beta only`。 |
| **camelCase vs snake_case** | REST/JSON 强制 camelCase；不要被 proto/SDK 文档里出现的 `max_output_tokens`、`stop_sequences` 误导。 |
| **HarmCategory 枚举混杂** | 共 12 个值，其中 6 个（`HARM_CATEGORY_DEROGATORY` 到 `HARM_CATEGORY_DANGEROUS`）是 PaLM 旧分类，对 Gemini 模型无效；实际可用的是 `HARASSMENT`、`HATE_SPEECH`、`SEXUALLY_EXPLICIT`、`DANGEROUS_CONTENT`，部分模型多了 `CIVIC_INTEGRITY`。 |
| **`OFF` 阈值需要资格** | `HarmBlockThreshold = OFF` 完全关闭过滤，普通 API Key 通常会被驳回；这是文档没有明说的限制，写文档时要标"需账户具备相应权限"。 |
| **`thinkingConfig` 受模型限制** | 仅 `gemini-2.5-*` / `3.x` 等思考模型支持；对其他模型设置无效或报错。`thinkingBudget=0` 在某些模型上也不能完全禁用思考。 |
| **`cachedContent` 最低 token 门槛** | 1.5 与 2.5 系一般要求最少 4096 token 才能创建缓存，低于门槛报 `INVALID_ARGUMENT`。 |
| **File 默认 48h 过期** | 别假设 file URI 永久可用；视频上传后还要轮询 `state=ACTIVE`。 |
| **流式响应没有显式结束事件** | SSE 流靠连接关闭判定结束；JSON 数组流靠数组闭合。`finishReason` 出现的那一块就是末块，要立刻 flush 调用方 UI。 |
| **Streaming 与 `?alt=sse`** | 默认 `alt=json`（chunked JSON 数组），需要标准 SSE 必须显式 `?alt=sse`。 |
| **`responseSchema` 用 OpenAPI 子集** | 类型大写：`OBJECT` / `STRING` / `INTEGER` / `NUMBER` / `BOOLEAN` / `ARRAY`，与标准 JSON Schema 不同；想用标准 JSON Schema 要走 `responseJsonSchema`。 |
| **Search Grounding 工具二选一** | `googleSearch`（2.x 推荐）与 `googleSearchRetrieval`（1.5 旧）互斥；新模型不再接受 `googleSearchRetrieval`。 |
| **Discovery 与人工页面字段差** | 部分实验性字段（如 `routingConfig`、`functionCallingConfig.mode=VALIDATED`）只出现在 Discovery JSON 里，人工页面延迟更新。两者冲突以 Discovery 为准。 |
| **错误体不是统一 `code` 字段** | 顶层是 `error.code`（HTTP int）、`error.status`（canonical string），别混淆；机器可读的稳定错误代码在 `error.details[?(@type=ErrorInfo)].reason`。 |

## 5. 关键链接表

| 主题 | URL |
| --- | --- |
| API 总入口 | <https://ai.google.dev/api?hl=zh-CN> |
| generateContent / streamGenerateContent | <https://ai.google.dev/api/generate-content?hl=zh-CN> |
| Models | <https://ai.google.dev/api/models?hl=zh-CN> |
| Tokens | <https://ai.google.dev/api/tokens?hl=zh-CN> |
| Embeddings | <https://ai.google.dev/api/embeddings?hl=zh-CN> |
| Files | <https://ai.google.dev/api/files?hl=zh-CN> |
| Caching | <https://ai.google.dev/api/caching?hl=zh-CN> |
| Batch API | <https://ai.google.dev/api/batch-api?hl=zh-CN> |
| Live API | <https://ai.google.dev/api/live?hl=zh-CN> |
| File Search | <https://ai.google.dev/api/file-search/file-search-stores?hl=zh-CN> |
| Tuning | <https://ai.google.dev/api/tuning?hl=zh-CN> |
| 错误故障排查 | <https://ai.google.dev/gemini-api/docs/troubleshooting?hl=zh-CN> |
| 模型对比 | <https://ai.google.dev/gemini-api/docs/models> |
| Pricing（中文） | <https://ai.google.dev/gemini-api/docs/pricing?hl=zh-cn> → 对应 `google-gemini/pricing.md` |
| Pricing（英文） | <https://ai.google.dev/gemini-api/docs/pricing> |
| Thinking | <https://ai.google.dev/gemini-api/docs/thinking> |
| 结构化输出 | <https://ai.google.dev/gemini-api/docs/structured-output> |
| 安全设置 | <https://ai.google.dev/gemini-api/docs/safety-settings> |
| Discovery (v1beta) | <https://generativelanguage.googleapis.com/$discovery/rest?version=v1beta> |
| Discovery (v1) | <https://generativelanguage.googleapis.com/$discovery/rest?version=v1> |
| Google API 错误模型 | <https://cloud.google.com/apis/design/errors> |

## 6. 本仓库映射

| 端点 | 文件 |
| --- | --- |
| 概览 | `google-gemini/README.md` |
| generateContent | `google-gemini/generate-content.md` |
| streamGenerateContent | `google-gemini/stream-generate-content.md` |
| countTokens | `google-gemini/count-tokens.md` |
| embedContent / batchEmbedContents | `google-gemini/embed-content.md` |
| Files API | `google-gemini/files.md` |
| Cached Content | `google-gemini/caching.md` |
| Models | `google-gemini/models.md` |
| Errors | `google-gemini/errors.md` |
