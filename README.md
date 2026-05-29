# LLM_OFFICIAL_DOCUMENTATION

收集主流 LLM 厂商官方 API 文档的中文整理版本，重点覆盖**调用参数、字段格式、响应结构与各家特有约定**。所有正文使用中文，字段名 / 枚举值 / header / URL / 模型 ID 等技术标识符保留英文原文。

> 使用说明（怎么查、怎么和 Claude Code 配合、怎么贡献）见 [USAGE.md](./USAGE.md)。
> 撰写规范见 [CONVENTIONS.md](./CONVENTIONS.md)，单端点模板见 [_template.md](./_template.md)。

## 安装到本地

仓库公开托管在 GitHub：<https://github.com/hkai-ai/LLM_OFFICIAL_DOCUMENTATION>，无需鉴权即可克隆。

### 完整克隆

```bash
git clone https://github.com/hkai-ai/LLM_OFFICIAL_DOCUMENTATION.git
cd LLM_OFFICIAL_DOCUMENTATION
```

### 只取单个厂商（sparse checkout）

仅关注某一厂商时可省去其余厂商目录，节省磁盘与拉取时间：

```bash
git clone --filter=blob:none --no-checkout https://github.com/hkai-ai/LLM_OFFICIAL_DOCUMENTATION.git
cd LLM_OFFICIAL_DOCUMENTATION
git sparse-checkout init --cone
git sparse-checkout set anthropic .claude/skills/anthropic-api-docs
git checkout main
```

把 `anthropic` 与 `.claude/skills/anthropic-api-docs` 替换为目标厂商目录与对应 skill 即可（厂商目录与 skill 名一一对应，见上方「厂商索引」表）。

### 配合 Claude Code 使用

在克隆后的仓库根目录直接启动 Claude Code，`.claude/skills/*` 会自动注册为可用 skill，询问类似「Claude Batch 字段」「qwen3.6-max 价格」「DeepSeek 缓存命中字段」等问题时，Claude 会通过对应 skill 在本地文档中检索，无需联网抓取。

若希望在**其他项目**中复用这份资料，把仓库的绝对路径传给 Claude 即可，例如：

```
请参考 D:\path\to\LLM_OFFICIAL_DOCUMENTATION\anthropic\messages.md
为这段代码补全 Anthropic 的 tool_use 字段
```

### 保持更新

```bash
git pull origin main
```

各厂商目录的 `README.md` frontmatter 中 `last_updated` 字段记录该厂商资料最后同步日期（北京时间），可据此判断是否已过期。

## 厂商索引

| 厂商 | 类型 | 概览 | 入口 | Skill |
| --- | --- | --- | --- | --- |
| Anthropic Claude | 直连 | [anthropic/README.md](./anthropic/README.md) | https://platform.claude.com/docs/en/api/overview | [anthropic-api-docs](./.claude/skills/anthropic-api-docs/SKILL.md) |
| OpenAI | 直连 | [openai/README.md](./openai/README.md) | https://developers.openai.com/api/reference/overview | [openai-api-docs](./.claude/skills/openai-api-docs/SKILL.md) |
| Google Gemini | 直连 | [google-gemini/README.md](./google-gemini/README.md) | https://ai.google.dev/api | [gemini-api-docs](./.claude/skills/gemini-api-docs/SKILL.md) |
| DeepSeek | 直连 | [deepseek/README.md](./deepseek/README.md) | https://api-docs.deepseek.com/zh-cn/ | [deepseek-api-docs](./.claude/skills/deepseek-api-docs/SKILL.md) |
| Moonshot Kimi | 直连 | [moonshot/README.md](./moonshot/README.md) | https://platform.kimi.com/docs/api/overview | [moonshot-api-docs](./.claude/skills/moonshot-api-docs/SKILL.md) |
| 阿里百炼 Model Studio | 直连 + 聚合 | [alibaba-bailian/README.md](./alibaba-bailian/README.md) | https://help.aliyun.com/zh/model-studio/model-api-reference/ | [alibaba-bailian-api-docs](./.claude/skills/alibaba-bailian-api-docs/SKILL.md) |
| MiniMax | 直连 | [minimax/README.md](./minimax/README.md) | https://platform.minimaxi.com/docs/api-reference/api-overview | [minimax-api-docs](./.claude/skills/minimax-api-docs/SKILL.md) |
| 智谱 BigModel（GLM） | 直连 | [zhipu/README.md](./zhipu/README.md) | https://docs.bigmodel.cn/cn/api/introduction | [zhipu-api-docs](./.claude/skills/zhipu-api-docs/SKILL.md) |
| OpenRouter | 聚合网关 | [openrouter/README.md](./openrouter/README.md) | https://openrouter.ai/docs/api/reference/overview | [openrouter-api-docs](./.claude/skills/openrouter-api-docs/SKILL.md) |
| 火山方舟（豆包 Doubao） | 直连 | [volcengine/README.md](./volcengine/README.md) | https://www.volcengine.com/docs/82379/1494384 | [volcengine-api-docs](./.claude/skills/volcengine-api-docs/SKILL.md) |

## 目录结构

```
LLM_OFFICIAL_DOCUMENTATION/
├── README.md                 # 本文件
├── CONVENTIONS.md            # 撰写规范
├── _template.md              # 单端点文档模板
├── anthropic/                # Claude Messages API
├── openai/                   # Chat Completions & Responses
├── google-gemini/            # generateContent (Developer API)
├── deepseek/                 # OpenAI 兼容 + Beta 扩展
├── moonshot/                 # Kimi K2.6 / K2.5 / Moonshot-v1
├── alibaba-bailian/          # 通义千问 + DashScope + 4 套协议
├── minimax/                  # MiniMax M2 / Hailuo / 语音 / 图像
├── zhipu/                    # 智谱 GLM-5 / CogVideoX / GLM-Image / GLM-ASR
├── openrouter/               # 聚合网关，多 provider 路由
├── volcengine/               # 火山方舟 豆包 Doubao / Seedance / Seedream / Responses
└── .claude/skills/           # 各厂商 API 文档维护 skill
```

每个厂商目录的典型文件：
- `README.md` — 厂商概览、鉴权、端点索引、模型清单；frontmatter 含 `last_updated` 字段记录该厂商目录上次更新日期（北京时间）
- `<endpoint>.md` — 单个端点的请求参数与响应字段
- `errors.md` — 错误响应结构
- `models.md` — 模型清单与 List Models 端点
- `pricing.md` — 厂商当前在售模型 / 工具 / 套餐的定价表，含货币与计价单位

## 端点速查

### 文本生成（核心端点）

| 厂商 | 端点 | 文档 |
| --- | --- | --- |
| Anthropic | `POST /v1/messages` | [messages.md](./anthropic/messages.md) |
| Anthropic | Messages 流式 SSE | [messages-streaming.md](./anthropic/messages-streaming.md) |
| OpenAI | `POST /v1/chat/completions` | [chat-completions.md](./openai/chat-completions.md) |
| OpenAI | `POST /v1/responses` | [responses.md](./openai/responses.md) |
| Google Gemini | `POST /v1beta/{model}:generateContent` | [generate-content.md](./google-gemini/generate-content.md) |
| Google Gemini | `POST /v1beta/{model}:streamGenerateContent` | [stream-generate-content.md](./google-gemini/stream-generate-content.md) |
| DeepSeek | `POST /chat/completions` | [chat-completions.md](./deepseek/chat-completions.md) |
| DeepSeek | `POST /beta/completions` (FIM) | [fim-completion.md](./deepseek/fim-completion.md) |
| Moonshot Kimi | `POST /v1/chat/completions` | [chat-completions.md](./moonshot/chat-completions.md) |
| 阿里百炼 | `POST /compatible-mode/v1/chat/completions` | [chat-completions.md](./alibaba-bailian/chat-completions.md) |
| 阿里百炼 | `POST /compatible-mode/v1/responses` | [responses.md](./alibaba-bailian/responses.md) |
| 阿里百炼 | `POST /api/v1/services/aigc/text-generation/generation` (DashScope) | [dashscope-generation.md](./alibaba-bailian/dashscope-generation.md) |
| OpenRouter | `POST /api/v1/chat/completions` | [chat-completions.md](./openrouter/chat-completions.md) |
| OpenRouter | `POST /api/v1/completions` | [completions.md](./openrouter/completions.md) |
| 火山方舟 | `POST /api/v3/chat/completions` | [chat-completions.md](./volcengine/chat-completions.md) |
| 火山方舟 | `POST /api/v3/responses` | [responses.md](./volcengine/responses.md) |

### Token 计数

| 厂商 | 端点 | 文档 |
| --- | --- | --- |
| Anthropic | `POST /v1/messages/count_tokens` | [count-tokens.md](./anthropic/count-tokens.md) |
| Google Gemini | `POST /v1beta/{model}:countTokens` | [count-tokens.md](./google-gemini/count-tokens.md) |
| Moonshot Kimi | `POST /v1/tokenizers/estimate-token-count` | [estimate-tokens.md](./moonshot/estimate-tokens.md) |
| 火山方舟 | `POST /api/v3/tokenization` | [tokenization.md](./volcengine/tokenization.md) |
| OpenAI | 无独立端点（使用 tokenizer 库） | — |
| DeepSeek | 无独立端点 | — |
| 阿里百炼 | 无独立端点 | — |

### 嵌入向量

| 厂商 | 端点 | 文档 |
| --- | --- | --- |
| OpenAI | `POST /v1/embeddings` | [embeddings.md](./openai/embeddings.md) |
| Google Gemini | `POST /v1beta/{model}:embedContent` | [embed-content.md](./google-gemini/embed-content.md) |
| 阿里百炼 | `POST /compatible-mode/v1/embeddings` + DashScope `/api/v1/services/embeddings/text-embedding/text-embedding` | [embeddings.md](./alibaba-bailian/embeddings.md) |
| 智谱 | `POST /paas/v4/embeddings` | [embeddings.md](./zhipu/embeddings.md) |
| 火山方舟 | `POST /api/v3/embeddings/multimodal`（多模态，方舟 SDK） | [embeddings.md](./volcengine/embeddings.md) |
| Anthropic | 无（推荐第三方） | — |
| DeepSeek | 无 | — |

### 文件 / 缓存 / 批处理

| 厂商 | 文件 | 缓存 | 批处理 |
| --- | --- | --- | --- |
| Anthropic | `/v1/files`（Beta） · [files.md](./anthropic/files.md) | `cache_control` 内联，无独立端点 | `/v1/messages/batches` GA · [messages-batches.md](./anthropic/messages-batches.md) |
| OpenAI | `/v1/files` | `prompt_cache_key` 内联 | `/v1/batches`，详见 [files-and-batches.md](./openai/files-and-batches.md) |
| Google Gemini | `/v1beta/files` | `/v1beta/cachedContents`（[caching.md](./google-gemini/caching.md)） | `batchGenerateContent` · [batches.md](./google-gemini/batches.md) |
| DeepSeek | 无 | 自动硬盘缓存，详见 [caching.md](./deepseek/caching.md) | 无 |
| 阿里百炼 | OpenAI 兼容 `/v1/files` + OSS 路径 | 显式 `cache_control` 内联 | `/compatible-mode/v1/batches` · [batch.md](./alibaba-bailian/batch.md) |
| 火山方舟 | `/api/v3/files`（[files.md](./volcengine/files.md)） | 显式 `/api/v3/context/*`（前缀 / Session 缓存，[context-cache.md](./volcengine/context-cache.md)） | `/api/v3/batch/chat/completions` · [batch.md](./volcengine/batch.md) |

### 音频 / 图像 / 视频

| 厂商 | 端点 | 文档 |
| --- | --- | --- |
| OpenAI | `/v1/audio/{speech,transcriptions,translations}` | [audio.md](./openai/audio.md) |
| OpenAI | `/v1/images/{generations,edits,variations}` | [images.md](./openai/images.md) |
| 火山方舟 | `POST /api/v3/images/generations`（Seedream） | [images.md](./volcengine/images.md) |
| 火山方舟 | `POST /api/v3/contents/generations/tasks`（Seedance 视频，异步） | [video.md](./volcengine/video.md) |

### 定价表

| 厂商 | 货币 | 文档 |
| --- | --- | --- |
| Anthropic Claude | USD | [anthropic/pricing.md](./anthropic/pricing.md) |
| OpenAI | USD | [openai/pricing.md](./openai/pricing.md) |
| Google Gemini | USD | [google-gemini/pricing.md](./google-gemini/pricing.md) |
| DeepSeek | 人民币 ¥ | [deepseek/pricing.md](./deepseek/pricing.md) |
| Moonshot Kimi | 人民币 ¥ | [moonshot/pricing.md](./moonshot/pricing.md) |
| MiniMax | 人民币 ¥ | [minimax/pricing.md](./minimax/pricing.md) |
| 智谱 BigModel（GLM） | 人民币 ¥（具体单价见官方计费页） | [zhipu/pricing.md](./zhipu/pricing.md) |
| 火山方舟（豆包 Doubao） | 人民币 ¥ | [volcengine/pricing.md](./volcengine/pricing.md) |

> 阿里百炼 / OpenRouter 价格随上游 / 厂商页变动频繁，目前未单独维护 `pricing.md`；前者查官方[模型广场](https://bailian.console.aliyun.com/?tab=model#/model-market)，后者通过 `GET /api/v1/models` 实时返回 `pricing` 字段。

## 能力横向对照

| 维度 | Anthropic | OpenAI | Google Gemini | DeepSeek | Moonshot Kimi | 阿里百炼 | OpenRouter | 火山方舟 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Base URL | `api.anthropic.com` | `api.openai.com/v1` | `generativelanguage.googleapis.com/v1beta` | `api.deepseek.com` | `api.moonshot.cn/v1` | `dashscope.aliyuncs.com/compatible-mode/v1` 或 `/api/v1` | `openrouter.ai/api/v1` | `ark.cn-beijing.volces.com/api/v3` |
| 鉴权 header | `x-api-key` + `anthropic-version` | `Authorization: Bearer` | `x-goog-api-key` 或 `?key=` | `Authorization: Bearer` | `Authorization: Bearer` | `Authorization: Bearer` (+ DashScope SSE 需 `X-DashScope-SSE`) | `Authorization: Bearer` (+ `HTTP-Referer` / `X-Title` 可选) | `Authorization: Bearer`（API Key）或 AK/SK 签名 |
| 协议风格 | 私有（Messages） | 私有（Chat/Responses） | 私有（generateContent） | OpenAI 兼容 | OpenAI 兼容 | 4 套：OpenAI 兼容 Chat/Responses + Anthropic 兼容 + DashScope 原生 | OpenAI 兼容（路由聚合） | OpenAI 兼容 Chat + 私有 Responses |
| 推理 / 思考 | `thinking` 块 + `thinking.budget_tokens` | `reasoning_effort` / `reasoning.effort` | `thinkingConfig.thinkingBudget` | `thinking.type` / `reasoning_content` | `thinking.type` + `thinking.keep`（K2.6） | `enable_thinking` + `thinking_budget` / `reasoning.effort`（Responses） | `reasoning`（透传上游字段） | `thinking.type`（enabled/disabled/auto）+ `reasoning_effort` |
| 视觉输入 | `image` content block（base64 / URL） | `image_url` part | `inlineData` / `fileData` | 多模态模型支持 | `image_url`（含 `ms://<file_id>`） | OpenAI 兼容：`image_url`；DashScope：`{"image":"..."}` | OpenAI 风格 `image_url`（依模型而定） | Chat：`image_url`；Responses：`input_image`（URL / Base64 / file_id） |
| 音频输入 | 暂不支持 | `input_audio` + `audio` 输出 | `inlineData` / `fileData`（audio mime） | 暂不支持 | 暂不支持 | OpenAI 兼容：`input_audio`；DashScope：`{"audio":"..."}`（Omni） | 依上游模型 | Chat：`input_audio`；Responses：`input_audio` |
| 函数调用 | `tools` + `tool_choice` | `tools` + `tool_choice` + `parallel_tool_calls` | `tools.functionDeclarations` + `toolConfig` | OpenAI 兼容 `tools` | OpenAI 兼容 `tools`（最多 128 个） | OpenAI 兼容 `tools` + `parallel_tool_calls` | OpenAI 兼容 `tools` | OpenAI 兼容 `tools` + `tool_choice` + `parallel_tool_calls` |
| 内置工具 | code_execution / bash / text_editor / memory / web_search / web_fetch | file_search / web_search / computer_use / code_interpreter / mcp | googleSearch / codeExecution / urlContext | 暂无 | `enable_search`（K2.6 联网搜索） | `web_search` / `code_interpreter` / `web_extract` / `image_search`（Responses） | `web_search_options`（统一封装） | web_search / image_process / knowledge_search / mcp / doubao_app（Responses） |
| JSON 输出 | `output_config`（JSON Schema） | `response_format`（text / json_object / json_schema） | `responseMimeType` + `responseSchema` | `response_format`（text / json_object） | `response_format`（text / json_object / json_schema） | `response_format`（text / json_object）；与 `enable_thinking` 互斥 | `response_format`（透传） | `response_format` / `text.format`（text / json_object / json_schema，beta） |
| 流式 | SSE，事件型 delta | SSE，`delta` 字段 | JSON 分块 或 `?alt=sse` | SSE，OpenAI 风格 | SSE，OpenAI 风格 | OpenAI 兼容：`stream:true`；DashScope：`X-DashScope-SSE: enable` | SSE，OpenAI 风格 | SSE：Chat OpenAI 风格 / Responses 事件型 |
| 上下文缓存 | `cache_control: ephemeral`（5m / 1h TTL） | 自动 + `prompt_cache_key` 路由 | `cachedContents` 资源 | 自动硬盘缓存 | 自动 + `prompt_cache_key` 路由（K2.5 / K2.6） | 显式 `cache_control` + Responses 的 `x-dashscope-session-cache` header | 透传上游策略 | 显式 Context API（前缀 / Session 缓存）+ 隐式缓存 |
| 缓存计费字段 | `cache_creation_input_tokens` / `cache_read_input_tokens` | `prompt_tokens_details.cached_tokens` | `cachedContentTokenCount` | `prompt_cache_hit_tokens` / `prompt_cache_miss_tokens` | `cached_tokens`（usage 顶层） | `prompt_tokens_details.cached_tokens` + `cache_creation.ephemeral_5m_input_tokens` | `usage.prompt_tokens_details.cached_tokens`（需 `usage.include: true`） | `prompt_tokens_details.cached_tokens` |
| 中断续写 | 末尾 `assistant` 消息 | Responses `previous_response_id` | 多轮 `contents` | `prefix: true`（仅 `/beta`） | `partial: true`（assistant 消息内） | Responses `previous_response_id` | 透传上游能力 | `partial: true`（assistant 消息）/ Responses `previous_response_id` |
| 路由 / 回退 | — | — | — | — | — | 仅同账号内模型切换；不做跨厂商路由 | `models[]` 候选 + `route: fallback` + `provider.order` / `allow_fallbacks` / `sort` | — |
| 旗舰模型 | claude-opus-4-7 / sonnet-4-6 / haiku-4-5 | gpt-5 / o4 系列 / gpt-4.1 | gemini-2.5-pro / 2.5-flash | deepseek-v4-pro / v4-flash | kimi-k2.6 / k2.5 / moonshot-v1 | qwen3.6-max-preview / qwen3.6-plus / qwen3.6-flash / qwen3.5-omni-plus / qvq / qwq | `openrouter/auto` + 数百款上游 | doubao-seed-2.0-pro / lite / mini + seedance-2.0 + seedream-5.0 |

> 上表仅作快速对比，**字段名、默认值、约束条件以各厂商目录下的详细文档为准**。

## Skill 系统

`.claude/skills/` 下为每个厂商维护了一份 skill，记录：

1. **文档站全貌** — 一级目录结构、域名说明、是否需登录
2. **抓取要点** — 哪些 URL 可被 WebFetch 抓取、需要换源的子页、SPA 壳页面
3. **更新流程** — 如何检查版本变动、对照官方 changelog
4. **坑点清单** — 域名重定向、403、CSR 渲染、字段命名差异、SDK 与 HTTP 不一致等
5. **关键链接表** — endpoint → 官方 URL

当需要刷新某厂商文档时，让 Claude 调用对应 skill 可快速回忆该厂商的文档结构与已知坑点。

## 维护说明

- 每个 endpoint 文档顶部 frontmatter 标注 `source` / `fetched_at` / `api_version`，便于追踪是否过期。
- 每个厂商的 `README.md` 在 frontmatter 额外追加 `last_updated` 字段，记录该厂商目录下任一文件最近一次有意义更新的日期（北京时间）；查看仓库时直接看该字段即可大致判断该厂商资料是否过期。
- 模型 ID 与 API 版本变化较快，需定期对照官方 changelog 同步。
- 新增厂商时，复制现有厂商结构 + 写一份新的 skill，并在本文件「厂商索引」与「能力横向对照」补行。

## 当前状态

| 厂商 | 文档完成度 | 抓取日期 |
| --- | --- | --- |
| Anthropic | Messages 系列 + count tokens + Message Batches + Files + Skills + Managed Agents + Rate Limits + Service Tiers + models + errors + versioning + pricing | 2026-05-26 |
| OpenAI | Responses / Chat / Embeddings / Audio / Images / Videos / Files & Batches / Uploads / Moderations / Fine-tuning / Vector Stores / Containers / Conversations / Evals / Skills / Realtime / Webhooks / ChatKit / Admin / Legacy / Models / Errors / pricing | 2026-05-26 |
| Google Gemini | generateContent / streamGenerate / countTokens / embedContent / files / caching / batches / live-api / file-search / tuning / models / errors / pricing | 2026-05-26 |
| DeepSeek | Chat / FIM / Anthropic 兼容 / guides（思考 / 多轮 / JSON / 工具 / 前缀续写 / token 用量）/ models / balance / caching / errors / rate-limits / pricing | 2026-05-26 |
| Moonshot Kimi | Chat / partial-mode / estimate-tokens / models / user-balance / errors / rate-limits / pricing / batch / files / tool-use / json-mode / vision / web-search / thinking / guides | 2026-05-26 |
| 阿里百炼 | Chat（OpenAI 兼容）/ Responses / DashScope generation / embeddings（双协议）/ batch / models / errors | 2026-05-26 |
| MiniMax | chat（OpenAI / Anthropic 双兼容）/ messages / models / speech / voice / video / image / music / lyrics / caching / files / errors / pricing | 2026-05-26 |
| 智谱 BigModel | chat（同步/异步/思考/工具/视觉/音频）/ embeddings / images / videos / audio / tools / misc（rerank/tokenizer/OCR）/ agents / knowledge-base / realtime / batch / files / models / errors / pricing | 2026-05-26 |
| OpenRouter | Chat / Completions / generation / models / credits / api-keys / auth / provider-routing / model-routing / transforms / errors / rate-limits | 2026-05-19 |
| 火山方舟（豆包 Doubao） | Chat / Responses（创建 / 查询 / 上下文 / 删除）/ embeddings（多模态）/ images（Seedream）/ video（Seedance 任务）/ context-cache / batch / files / tokenization / bot / openai-compat / models / errors / pricing | 2026-05-28 |

> 2026-05-26 第三轮补齐：OpenAI 新增 14 篇（moderations / fine-tuning / vector-stores / realtime / containers / conversations / evals / uploads / webhooks / skills / videos / chatkit / admin / legacy）；DeepSeek 新增 anthropic-api / guides；智谱新增 agents / knowledge-base / realtime / misc；MiniMax 新增 caching / lyrics。
> 2026-05-28 新增火山方舟（豆包 Doubao）厂商，覆盖 Chat / Responses / 向量化 / 图片（Seedream）/ 视频（Seedance）/ 上下文缓存 / 批量 / 文件 / 分词 / bot / 错误码 / 定价共 14 篇。

待补：xAI Grok、Mistral 等。
