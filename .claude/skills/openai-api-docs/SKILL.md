---
name: openai-api-docs
description: 需要查阅、更新或扩充 OpenAI API 官方文档（包括 Responses / Chat Completions / Embeddings / Audio / Images / Files / Batches / Models / Moderations / Realtime / Fine-tuning / Vector Stores / Webhooks、**定价（pricing.md）** 等）时使用。覆盖 developers.openai.com 与 platform.openai.com 两个站点的抓取策略、字段差异、新模型滚动更新流程，以及 Standard / Batch / Flex / Priority 多档定价的同步规则。触发示例："更新 OpenAI 文档"、"刷 OpenAI 定价"、"gpt-5.5 价格"、"sora-2 价格"、"OpenAI flex priority 档位"。
---

# OpenAI API 文档维护 Skill

本 skill 帮助维护 `openai/` 目录下的中文整理文档，与 `developers.openai.com/api/reference` 官方页面对齐。

## 一、文档站全貌

OpenAI 同时存在两个文档站点：

| 站点 | 用途 | 可抓性 |
| --- | --- | --- |
| `https://developers.openai.com/api/reference` | 新版 API Reference（HTML SSR + 部分 hydrate） | **可** 用 WebFetch 直接抓取，多数子页返回完整 markdown。本仓库以此为主源。 |
| `https://platform.openai.com/docs/api-reference` | 老控制台文档（CSR + 反爬） | **403** —— WebFetch 几乎总是失败。仅用于核对截图或交叉对照。 |
| `https://developers.openai.com/api/docs/...` | guide 类页面（含 models、error-codes、authentication、deprecations） | 可抓。 |

### 一级分类（developers 站）

```
Responses API
  ├── Responses（create/retrieve/delete/list-input-items/count-tokens/cancel/compact）
  ├── Conversations
  ├── Items
  └── Streaming events
Webhooks
  └── Events
Platform APIs
  ├── Audio（transcriptions/translations/speech/voices/voice_consents）
  ├── Videos（OpenAI Video 生成；非本仓库覆盖范围）
  ├── Images（generate/edit/create_variation/streaming events）
  ├── Embeddings
  ├── Evals
  ├── Fine Tuning
  ├── Batches
  ├── Files
  ├── Uploads
  ├── Models
  └── Moderations
Vector Stores
ChatKit
Containers
Skills（OpenAI Skills，非 Claude skills）
Realtime
Administration（admin API key 限定）
Chat Completions
Legacy（Completions、Assistants v2、Realtime Beta）
```

## 二、抓取要点

### URL 规律

`developers.openai.com` 的 endpoint 页 URL 形如：

```
/api/reference/resources/<resource>/methods/<method>
/api/reference/resources/<resource>/subresources/<sub>/methods/<method>
```

- `<method>` 命名**不统一**：images 用 `generate` / `edit` / `create_variation`；audio 用 `create`；chat completions 用 `create`；responses 用 `create`。**别套模板猜，先 WebSearch 找具体 URL。**
- 子资源结构有时是 `subresources/...`（chat → completions、audio → speech/transcriptions/translations），有时是直接 methods（images）。

### 实战流程

1. 先抓总览 `https://developers.openai.com/api/reference/overview` 获取导航树。
2. 抓 `https://developers.openai.com/api/reference/resources/<resource>` 取该资源的方法列表。
3. 若 404，立即 WebSearch `"developers.openai.com api reference <resource> create"` 找真实 URL。
4. 单个 endpoint 页用 WebFetch + 精准 prompt，让小模型按表格输出 `字段 / 类型 / 必填 / 默认 / 说明 / 枚举值`。
5. WebFetch 单页输出可能被截断，把 prompt 拆细：第一次问参数，第二次问响应，第三次问枚举。

### 抓取 prompt 范式

```
输出 POST /v1/<path> 完整 API 文档。
- 列出所有请求参数：每个参数的 name / type / required / default / description / 可选枚举值。
- 列出响应对象的所有顶层字段与嵌套结构。
- 保留字段名英文原文，不翻译。
- 用 markdown 表格输出。
```

## 三、更新流程

### 触发场景

- 新模型发布（如 `gpt-5.x` / `gpt-image-2` / 新 realtime 模型）。
- 新参数（最近一两年高频新增的有 `prompt_cache_key`、`safety_identifier`、`verbosity`、`reasoning_effort`、`service_tier: priority`、`context_management`、`background`、`max_tool_calls`）。
- 新内置工具（Responses `tools[]` 类型频繁扩张：`web_search` GA、`mcp`、`image_generation`、`local_shell`、`custom`）。
- 新端点（如 Conversations、Containers、ChatKit、Skills、Videos）。

### 检查清单

1. 抓 `https://developers.openai.com/api/reference/overview` 比对一级分类是否新增。
2. 抓 `https://developers.openai.com/api/docs/models` 更新 `models.md`。
3. 抓 `https://developers.openai.com/api/docs/deprecations` 注意要标记 deprecated 的字段（如 `max_tokens`、`generate_summary`、`user`）。
4. 抓 `https://developers.openai.com/api/docs/guides/error-codes` 更新 `errors.md`。
5. 关键 endpoint 至少季度复查一次：`chat/completions`、`responses`、`embeddings`、`audio/*`、`images/*`、`files`、`batches`。

### 同步步骤

1. WebFetch 抓最新 URL → 与本仓库对应 `.md` 文件做 diff。
2. 字段新增 / 删除 / 默认值变化 / 枚举变化都要更新。
3. 在每个文件 `fetched_at` 字段更新为当天日期（北京时间）。
4. `README.md` 的端点索引表是入口，新端点先加索引再补单独 md。

## 三-A、定价文档维护（`openai/pricing.md`）

- **唯一权威源**：`https://developers.openai.com/api/docs/pricing`。**不要**用 `platform.openai.com/docs/pricing`（CSR + 反爬）。
- **同步触发**：新增模型 / 新增档位（Standard / Batch / Flex / Priority）/ 工具价格调整（web search、containers、file search、ChatKit）/ 新增视频或图像分辨率档 / fine-tuning 价格调整 / 区域处理（data residency）+10% 规则变化。
- **抓取要点**：
  - 单页页面非常长，按章节拆分 prompt：先抓 "Flagship / Specialized"，再抓 "Realtime / Audio / Image / Video"，最后抓 "Tools / Fine-tuning"。
  - WebFetch 输出有时被截断，可针对单模型补抓："列出 `gpt-5.5` 的 Standard / Batch / Flex / Priority 四档完整数字"。
  - **不要漏 Cached Input 列**：旗舰文本模型的 cached input 价是 standard input 的 1/10，与 prompt caching 配合使用，写表时务必保留。
  - **长上下文版本独立成行**：gpt-5.4 / gpt-5.4-pro 有"长上下文版本"单独一档，与普通版价格不同，不要合并。
- **同步步骤**：
  1. 抓 pricing 页 → diff `openai/pricing.md` 各表格。
  2. 修改 `openai/pricing.md` frontmatter `fetched_at`。
  3. 同步 `openai/README.md` 的 `last_updated`。
  4. 模型新增 / 弃用时同步 `openai/README.md` 的模型清单与 `models.md`。
- **坑点提醒**：
  - 区域处理 +10% **不是**所有模型都适用，仅作用于 `gpt-5.5` / `5.5-pro` / `5.4` / `5.4-mini` / `5.4-nano` / `5.4-pro`；记录时把这条规则放在文件首部说明。
  - `gpt-realtime-translate` 是按分钟（$0.034 / 分钟）计费而非按 token，单独成节。
  - `sora-2-pro` 视频按分辨率三档（720p / 1024p / 1080p）；不要只记一档。
  - Containers 按"会话内存档 + 20 分钟会话"两个维度计价，写表时两轴都要保留。
  - Fine-tuning 当前 pricing 页只列了 `o4-mini-2025-04-16` 一个 SKU，但旧模型（gpt-4o-2024-08-06 等）的 fine-tuning 仍可用且价格不同 —— 文档未列的不要凭印象补。

## 四、坑点清单

1. **`platform.openai.com` 反爬**：WebFetch 几乎都是 403，**直接放弃**，转 `developers.openai.com`。
2. **URL method 段命名不一致**：images 是 `generate/edit/create_variation`，audio 是 `create`，responses 是 `create`，models 是 `list/retrieve/delete`。一旦 404 立刻 WebSearch，不要硬试。
3. **WebFetch socket 偶发断开**：换 URL 或重试。可以先抓 `resources/<resource>` 总览再深入子页。
4. **Responses 与 Chat Completions 参数名差异**：
   - max token：`max_completion_tokens`（chat）vs `max_output_tokens`（responses）。
   - 推理：`reasoning_effort` + `verbosity` 顶层字段（chat）vs `reasoning.effort` + `text.verbosity`（responses）。
   - 输入：`messages[]`（chat）vs `input[]`（responses，item 类型多得多）。
   - 历史：调用方维护 `messages`（chat）vs `previous_response_id` / `conversation` 服务端维护（responses）。
   - 内置工具：chat 只有 `web_search_options`；responses 的 `tools[].type` 覆盖 function / file_search / web_search(_preview) / computer_use_preview / code_interpreter / image_generation / mcp / local_shell / custom。
5. **`max_tokens` 在 chat 上对 reasoning 模型完全失效**（被推理 token 占满会直接 `length` 截断），新代码必须 `max_completion_tokens`。
6. **`generate_summary` 已弃用**，等价于 `reasoning.summary != null`；旧示例代码中常见，文档里要标 deprecated。
7. **`user` 字段已弃用**，新版叫 `safety_identifier`。两者在过渡期同时存在。
8. **Audio Translations 文档历史上只标注 `whisper-1`**，但 API 实际接受 `gpt-4o-transcribe*` 系列；以最新页面为准。
9. **Image variations 只支持 `dall-e-2`**：`gpt-image-*` 不出现在 variations 端点，需通过 edits 实现类似效果。
10. **Batches `endpoint` 枚举**会随时间扩张（已新增 `/v1/responses`、`/v1/videos`）；更新时核对最新清单。
11. **SDK 命名 ≠ HTTP 字段**：例如 `client.chat.completions.create({ stream_options: { include_usage: True } })` 中 snake_case 与 camelCase 在不同 SDK 不一致，文档以 HTTP JSON 为准（**全 snake_case**）。
12. **Model 滚动 alias**：`gpt-5` 这种 ID 指向最新快照；要复现行为需引用 `gpt-5-YYYY-MM-DD` 日期版本。`chatgpt-*-latest` 跟随 ChatGPT 产品节奏，不保证 API 兼容。
13. **WebFetch 抓出的"主推模型"列表会有时差**：developers 站会把某些尚未公开 GA 的模型列出。撰写 `models.md` 时以"账户可见"为更可靠的标尺，或同时抓 GET `/v1/models` 验证。
14. **错误 `error.message` 文本经常微调**，做精确匹配会脆；判错优先看 HTTP code + `error.type` + `error.code`。
15. **流式错误**：Chat Completions 在 stream 中出错以 `data: {"error":...}` chunk 下发，不会改 HTTP 状态码；Responses 用 `response.failed` 事件。两者处理方式完全不同。

## 五、关键链接表

| 主题 | 官方 URL |
| --- | --- |
| API 总览 | <https://developers.openai.com/api/reference/overview> |
| Pricing（旗舰 / 工具 / 微调） | <https://developers.openai.com/api/docs/pricing> → 对应 `openai/pricing.md` |
| 模型清单 | <https://developers.openai.com/api/docs/models> |
| 弃用计划 | <https://developers.openai.com/api/docs/deprecations> |
| 错误码 | <https://developers.openai.com/api/docs/guides/error-codes> |
| 生产实践 | <https://developers.openai.com/api/docs/guides/production-best-practices> |
| Chat Completions create | <https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create> |
| Responses create | <https://developers.openai.com/api/reference/resources/responses/methods/create> |
| Responses streaming | <https://developers.openai.com/api/reference/resources/responses/streaming> |
| Conversations | <https://developers.openai.com/api/reference/resources/conversations> |
| Embeddings create | <https://developers.openai.com/api/reference/resources/embeddings/methods/create> |
| Audio Speech | <https://developers.openai.com/api/reference/resources/audio/subresources/speech/methods/create> |
| Audio Transcriptions | <https://developers.openai.com/api/reference/resources/audio/subresources/transcriptions/methods/create> |
| Audio Translations | <https://developers.openai.com/api/reference/resources/audio/subresources/translations/methods/create> |
| Images generate | <https://developers.openai.com/api/reference/resources/images/methods/generate> |
| Images edit | <https://developers.openai.com/api/reference/resources/images/methods/edit> |
| Images variation | <https://developers.openai.com/api/reference/resources/images/methods/create_variation> |
| Files | <https://developers.openai.com/api/reference/resources/files> |
| Batches | <https://developers.openai.com/api/reference/resources/batches> |
| Batches create | <https://developers.openai.com/api/reference/resources/batches/methods/create> |
| Models | <https://developers.openai.com/api/reference/resources/models> |
| Moderations | <https://developers.openai.com/api/reference/resources/moderations> |
| Realtime | <https://developers.openai.com/api/reference/resources/realtime> |
| Fine-tuning | <https://developers.openai.com/api/reference/resources/fine_tuning> |
| Vector Stores | <https://developers.openai.com/api/reference/resources/vector_stores> |
| Webhooks events | <https://developers.openai.com/api/reference/resources/webhooks> |
| Uploads | <https://developers.openai.com/api/reference/resources/uploads> |
| Containers | <https://developers.openai.com/api/reference/resources/containers> |
| Administration | <https://developers.openai.com/api/reference/resources/administration> |
