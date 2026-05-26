---
name: zhipu-api-docs
description: 需要查阅、更新或扩充本仓库 `zhipu/` 目录下 智谱 BigModel（GLM 系列）API 中文文档时使用。涵盖 Chat Completions（含异步 / 思考模式 / 工具调用 / 视觉 / 音频）、Embeddings、图像（GLM-Image / CogView）、视频（CogVideoX / Vidu）、ASR / TTS / 音色复刻、Web Search / 文件解析 / OCR / 内容安全 / 网页阅读、Batch、Files、模型清单、错误码、定价等。触发场景示例："智谱怎么用"、"GLM-5.1 接入"、"glm-4.7 思考模式"、"CogVideoX 视频生成"、"BigModel batch"、"GLM Web Search"、"glm-asr"、"刷新智谱文档"、"GLM 价格"、"open.bigmodel.cn 接口"。
---

# 智谱 BigModel（GLM）API 文档撰写与维护

## 1. 文档站全貌

- 主入口：`https://docs.bigmodel.cn/cn/api/introduction`
- **文档总索引**：`https://docs.bigmodel.cn/llms.txt`（不在 `/cn/` 下！）—— 撰写或更新时**强烈建议先抓这个**，每个页面都有 `.md` 镜像形式。
- 一级分类（侧边栏）：
  - **快速开始 / 平台介绍**：`/cn/guide/start/...`
  - **API 参考**：`/api-reference/<分类-api>/<slug>`，分类含 `模型-api` / `工具-api` / `批处理-api` / `文件-api` / `知识库-api` / `助理-api` / `agent-api`。URL 含**中文路径**（如 `/对话补全.md`），需 URL 编码或直接复制粘贴。
  - **开发指南**：`/cn/guide/develop/<sdk>/...`（http / python / java / openai 兼容 / claude 兼容 / langchain）
  - **能力详解**：`/cn/guide/capabilities/<feature>.md`（streaming / function-calling / stream-tool / struct-output / cache / thinking / thinking-mode）
  - **工具与服务**：`/cn/guide/tools/<tool>.md`（batch / file-parser / web-search / zhipu-ocr / fine-tuning / model-deploy / evaluation / knowledge）
  - **模型说明**：`/cn/guide/models/<text|vlm|image-generation|video-generation|sound-and-video|embedding|free>/<slug>.md`
  - **最佳实践 / FAQ / 服务条款**：略
  - **OpenAPI / AsyncAPI 规范**：`https://docs.bigmodel.cn/openapi/openapi.json` 与 `asyncapi.json` —— 字段层面的最权威源
- 公开访问；登录后可在 `https://open.bigmodel.cn/` 控制台拿 API Key 与查看计费。

## 2. 抓取要点

- **优先 `.md` 镜像**：在 URL 末尾追加 `.md`，WebFetch 直接返回纯 markdown，省去 CSR 解析问题。
- **URL 含中文路径**：`api-reference` 下分类与 slug 都是中文（`/模型-api/对话补全.md` 等）。直接复制 llms.txt 的链接即可；不要手动转拼音。
- **OpenAPI 是权威**：当人工页面与 OpenAPI 规范冲突时，以 OpenAPI 为准。可抓 `https://docs.bigmodel.cn/openapi/openapi.json` 局部读取。
- **定价不在文档站**：单价见独立页 `https://www.bigmodel.cn/pricing`，不在 `docs.bigmodel.cn/llms.txt` 列表内。需要单独抓。
- **错误码页路径有点别扭**：`https://docs.bigmodel.cn/cn/api/api-code.md`（在 `/cn/api/` 下，不在 `/cn/api-reference/`）。
- **避免抓** GitHub README / 第三方教程 / 知乎软文。

## 3. 更新流程

1. **抓 `llms.txt`** 比对与本仓库的页面集合差异，发现新增 / 移除主题。
2. **抓 `api-overview` / `introduction`** 核对 Base URL、鉴权 header、协议兼容入口（OpenAI / Claude / LangChain）是否变动。
3. **抓 `api-code.md`** 同步 `zhipu/errors.md`：智谱错误码会随能力新增而扩张（10xx / 12xx / 13xx / 17xx 段）。
4. **同步 chat 核心**：抓 `对话补全.md` 与 `对话补全异步.md` → 对照 `chat-completions.md`；同时核对 `function-calling.md` / `thinking.md` / `cache.md` 的描述。
5. **多模态 endpoint 同步**：逐个抓 `文本嵌入` / `图像生成` / `视频生成异步` / `语音转文本` / `文本转语音` / `音色复刻`，diff 字段表。
6. **工具 API 同步**：`网络搜索` / `网页阅读` / `文件解析` / `文件解析同步` / `ocr-服务` / `内容安全`。
7. **批处理 / 文件 同步**：`批处理-api/*` / `文件-api/*`。
8. **模型清单同步**：抓 `https://docs.bigmodel.cn/cn/guide/start/model-overview.md` → 更新 `models.md`；逐个新增模型时还要看对应 `models/<category>/<slug>.md` 的能力差异。
9. **修改对应 md 的 `fetched_at`**，同步 `zhipu/README.md` 的 `last_updated`。

## 3.1 定价文档维护（`zhipu/pricing.md`）

- **权威源**：`https://www.bigmodel.cn/pricing`（独立站，不在文档子域）。
- **同步触发**：新模型上线（GLM-5.x / CogVideoX 新版 / 新 Vidu 模型）、订阅档位变动（GLM Coding Plan / Token Plan）、批量折扣率调整、Web Search 单价调整、免费模型清单变化。
- **抓取要点**：
  - 智谱定价表是 SPA / 动态加载，可能首次抓不到完整内容；如果 WebFetch 返回稀疏，可在 prompt 中明确按模型分类提问（"列出 GLM-5.1 / 5 / 4.7 / 4.6 各自的 input / output / cached_input 单价（¥ / 1M tokens）"）。
  - 同时抓 `https://docs.bigmodel.cn/cn/guide/tools/knowledge/price.md`，知识库价格在文档站独立维护。
  - 单位务必显式写明：token 类按"元 / 百万 tokens"，图像按"元 / 张"，视频按"元 / 秒"，ASR 按"元 / 分钟"，TTS 按"元 / 千字符"，Web Search 按"元 / 次"。
- **同步步骤**：
  1. WebFetch pricing 页 → 按 §计费维度更新 `zhipu/pricing.md`。
  2. 改 `pricing.md` 的 `fetched_at`，同步 `zhipu/README.md` 的 `last_updated`。
  3. 模型新增 / 弃用同步 `zhipu/models.md`。

## 4. 坑点清单

- **`tool_choice` 仅支持 `"auto"`**：传 `none` / `required` / 指定 function 名都会被忽略或返回参数错误。
- **`do_sample: false` 等价贪心解码**：`temperature` / `top_p` 被忽略；写文档要标注，避免被理解为"无效字段"。
- **`thinking` 强制模型差异**：GLM-5.1 / 5 / 4.7 / 4.5v 强制思考，不能通过 `thinking.type: disabled` 关闭；其他支持的模型才是真正可关。
- **隐式缓存，没有 `cache_control`**：与 Anthropic / OpenAI / 阿里百炼不同，智谱不暴露显式缓存断点；只能通过保持前缀稳定来提升命中率。最小命中阈值未公开。
- **`reasoning_content` 也计入 output tokens**：思考模式开启时计费要把 `reasoning_content` 计入输出。
- **URL 含中文 path**：`api-reference/模型-api/对话补全.md`。WebFetch 一般能直接吃，但 curl / 浏览器需要 URL 编码。
- **图像 `n` / `response_format` 字段未公开**：与 OpenAI Images 不一致；不要凭印象补造。如果用户需要批量生图，建议多次调用。
- **视频生成统一一个端点**：`POST /paas/v4/videos/generations`，靠 `model` + `image_url` / `prompt` 字段区分 t2v / i2v / 首尾帧 / 参考图。每个模型对 `duration` / `fps` / `size` 的支持矩阵不同，详见各模型说明页。
- **批量 API endpoint 当前仅 `/v4/chat/completions`**：不要把 `images.generations` / `audio.speech` 等塞进 batch，会被拒。
- **`completion_window` 字段不暴露**：与 OpenAI Batches 不同；服务端用默认完成时限。
- **`request_id` 强校验长度 6–64**：少于 6 字符会被拒；不传由服务端生成。
- **`user_id` 是风控关联键**：传入会与终端用户敏感词风控关联，导致单个 `user_id` 被锁定时该 ID 下所有请求都受影响。
- **`finish_reason: sensitive`** 与错误码 `1301` 同义；要把它当作错误处理而不是正常结束。
- **音频上传 ≤ 25MB ≤ 30 秒**：ASR 单次只能处理短音频；长音频要切片或走异步 / Realtime 模型。
- **TTS 默认 `response_format: pcm`**：直接保存需要加 wav 头；常忘记。
- **Coding 专用 base URL**：`https://open.bigmodel.cn/api/coding/paas/v4` —— GLM Coding Plan 套餐用户必须切到此路径，否则触发 `1309`。

## 5. 关键链接表

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| API 概述 | https://docs.bigmodel.cn/cn/api/introduction | `zhipu/README.md` |
| 速率限制 | https://docs.bigmodel.cn/cn/api/rate-limit | `zhipu/README.md` §通用约定 |
| 错误码 | https://docs.bigmodel.cn/cn/api/api-code | `zhipu/errors.md` |
| 文档总索引 | https://docs.bigmodel.cn/llms.txt | （查找用） |
| OpenAPI | https://docs.bigmodel.cn/openapi/openapi.json | （字段权威源） |
| Chat 同步 | https://docs.bigmodel.cn/api-reference/模型-api/对话补全 | `zhipu/chat-completions.md` |
| Chat 异步 | https://docs.bigmodel.cn/api-reference/模型-api/对话补全异步 | `zhipu/chat-completions.md` §异步 |
| 查询异步结果 | https://docs.bigmodel.cn/api-reference/模型-api/查询异步结果 | `zhipu/chat-completions.md` §异步 |
| Embeddings | https://docs.bigmodel.cn/api-reference/模型-api/文本嵌入 | `zhipu/embeddings.md` |
| 图像生成 | https://docs.bigmodel.cn/api-reference/模型-api/图像生成 | `zhipu/images.md` |
| 图像生成异步 | https://docs.bigmodel.cn/api-reference/模型-api/图像生成异步 | `zhipu/images.md` |
| 视频生成 | https://docs.bigmodel.cn/api-reference/模型-api/视频生成异步 | `zhipu/videos.md` |
| ASR | https://docs.bigmodel.cn/api-reference/模型-api/语音转文本 | `zhipu/audio.md` §ASR |
| TTS | https://docs.bigmodel.cn/api-reference/模型-api/文本转语音 | `zhipu/audio.md` §TTS |
| 音色复刻 | https://docs.bigmodel.cn/api-reference/模型-api/音色复刻 | `zhipu/audio.md` §音色复刻 |
| Tokenizer | https://docs.bigmodel.cn/api-reference/模型-api/文本分词器 | `zhipu/models.md` |
| Rerank | https://docs.bigmodel.cn/api-reference/模型-api/文本重排序 | `zhipu/models.md` |
| Web Search | https://docs.bigmodel.cn/api-reference/工具-api/网络搜索 | `zhipu/tools.md` §Web Search |
| 网页阅读 | https://docs.bigmodel.cn/api-reference/工具-api/网页阅读 | `zhipu/tools.md` |
| 文件解析（异步） | https://docs.bigmodel.cn/api-reference/工具-api/文件解析 | `zhipu/tools.md` §文件解析 |
| 文件解析（同步） | https://docs.bigmodel.cn/api-reference/工具-api/文件解析同步 | `zhipu/tools.md` |
| OCR | https://docs.bigmodel.cn/api-reference/工具-api/ocr-服务 | `zhipu/tools.md` |
| 内容安全 | https://docs.bigmodel.cn/api-reference/工具-api/内容安全 | `zhipu/tools.md` |
| Batch 创建 | https://docs.bigmodel.cn/api-reference/批处理-api/创建批处理任务 | `zhipu/batch.md` |
| Batch 列表 | https://docs.bigmodel.cn/api-reference/批处理-api/列出批处理任务 | `zhipu/batch.md` |
| Batch 检索 | https://docs.bigmodel.cn/api-reference/批处理-api/检索批处理任务 | `zhipu/batch.md` |
| Batch 取消 | https://docs.bigmodel.cn/api-reference/批处理-api/取消批处理任务 | `zhipu/batch.md` |
| Files 上传 | https://docs.bigmodel.cn/api-reference/文件-api/上传文件 | `zhipu/files.md` |
| Files 列表 | https://docs.bigmodel.cn/api-reference/文件-api/文件列表 | `zhipu/files.md` |
| Files 内容 | https://docs.bigmodel.cn/api-reference/文件-api/文件内容 | `zhipu/files.md` |
| Files 删除 | https://docs.bigmodel.cn/api-reference/文件-api/删除文件 | `zhipu/files.md` |
| 模型概览 | https://docs.bigmodel.cn/cn/guide/start/model-overview | `zhipu/models.md` |
| 工具调用 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/function-calling | `zhipu/chat-completions.md` |
| 思考模式 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/thinking | `zhipu/chat-completions.md` §思考 |
| 上下文缓存 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/cache | `zhipu/chat-completions.md` §上下文缓存 |
| 结构化输出 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/struct-output | `zhipu/chat-completions.md` |
| 流式 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/streaming | `zhipu/chat-completions.md` |
| 工具流式 Guide | https://docs.bigmodel.cn/cn/guide/capabilities/stream-tool | `zhipu/chat-completions.md` |
| OpenAI 兼容 | https://docs.bigmodel.cn/cn/guide/develop/openai/introduction | `zhipu/README.md` §协议兼容 |
| Claude 兼容 | https://docs.bigmodel.cn/cn/guide/develop/claude/introduction | `zhipu/README.md` §协议兼容 |
| LangChain 集成 | https://docs.bigmodel.cn/cn/guide/develop/langchain/introduction | `zhipu/README.md` §协议兼容 |
| 计费总览 | https://www.bigmodel.cn/pricing | `zhipu/pricing.md` |
| 知识库价格 | https://docs.bigmodel.cn/cn/guide/tools/knowledge/price | `zhipu/pricing.md` |

## 6. 本仓库文件映射

| 主题 | 仓库文件 |
| --- | --- |
| 厂商概览 + 端点索引 | `zhipu/README.md` |
| 对话补全（含异步 / 思考 / 工具 / 视觉 / 音频） | `zhipu/chat-completions.md` |
| 嵌入 | `zhipu/embeddings.md` |
| 图像 | `zhipu/images.md` |
| 视频 | `zhipu/videos.md` |
| 语音（ASR / TTS / 音色复刻） | `zhipu/audio.md` |
| 工具 API（Web Search / 文件解析 / OCR / 内容安全 / 网页阅读） | `zhipu/tools.md` |
| 批处理 | `zhipu/batch.md` |
| 文件 | `zhipu/files.md` |
| 模型清单 + Tokenizer + Rerank | `zhipu/models.md` |
| 错误码 | `zhipu/errors.md` |
| 定价要点 | `zhipu/pricing.md` |
