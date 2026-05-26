---
source: https://docs.bigmodel.cn/cn/api/introduction
fetched_at: 2026-05-26
api_version: paas/v4
last_updated: 2026-05-26
---

> 2026-05-26 更新：补齐 Agent API（agents.md）、知识库 API（knowledge-base.md）、实时 API（realtime.md）、模型 API 杂项（misc.md，rerank / tokenizer / 文档解析）。

# 智谱 BigModel（GLM）API 概览

智谱 BigModel 平台（GLM 系列）覆盖语言、视觉、图像、视频、音频、嵌入、批处理、文件、知识库、智能体、工具等能力。除自家 RESTful API（`/paas/v4`）外，还提供 **OpenAI 兼容** 与 **Anthropic / Claude 兼容** 两套 SDK 接入路径。

## 基本信息

| 项 | 值 |
| --- | --- |
| 主 Base URL | `https://open.bigmodel.cn/api/paas/v4` |
| Coding 专用 Base URL | `https://open.bigmodel.cn/api/coding/paas/v4` |
| 鉴权 | `Authorization: Bearer <API_KEY>` |
| 协议风格 | RESTful + JSON；POST 接口 `Content-Type` 固定 `application/json`，上传走 `multipart/form-data` |
| 流式响应 | SSE（`stream: true` 启用） |
| OpenAPI 规范 | https://docs.bigmodel.cn/openapi/openapi.json |
| AsyncAPI 规范 | https://docs.bigmodel.cn/asyncapi/asyncapi.json |

## 协议兼容

| 协议 | 说明 | 文档 |
| --- | --- | --- |
| 原生 HTTP（paas/v4） | 完整字段（含 `thinking` / `tool_stream` / `do_sample` 等智谱特有字段） | 本仓库 |
| OpenAI 兼容 | 直接用 `openai` SDK，替换 `base_url` 与 key 即可 | https://docs.bigmodel.cn/cn/guide/develop/openai/introduction |
| Anthropic / Claude 兼容 | 直接用 `@anthropic-ai/sdk`，替换 `base_url` | https://docs.bigmodel.cn/cn/guide/develop/claude/introduction |
| LangChain | 官方集成 | https://docs.bigmodel.cn/cn/guide/develop/langchain/introduction |

## 厂商目录结构（本仓库）

- `README.md` — 本文件
- `chat-completions.md` — 对话补全（含异步、思考模式、工具调用、视觉、音频）
- `embeddings.md` — 文本嵌入（embedding-2 / embedding-3）
- `images.md` — 图像生成（GLM-Image / CogView）
- `videos.md` — 视频生成（CogVideoX / Vidu）
- `audio.md` — 语音转文本（GLM-ASR）/ 文本转语音（GLM-TTS）/ 音色复刻（GLM-TTS-Clone）
- `tools.md` — 工具 API（Web Search / 文件解析 / OCR / 内容安全 / 网页阅读）
- `batch.md` — 批处理任务
- `files.md` — 文件管理
- `models.md` — 模型清单
- `misc.md` — 文本重排序 / 文本分词器 / 文档解析（同步）
- `agents.md` — Agent API（智能体对话 / 异步结果 / 对话历史）
- `knowledge-base.md` — 知识库 API（CRUD + 检索 + 文档管理 + 问答 Agent）
- `realtime.md` — GLM-Realtime WSS 音视频通话
- `errors.md` — 错误码
- `pricing.md` — 计费要点（详细单价见官方）

## 端点速查

### 模型 API（核心）

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /paas/v4/chat/completions` | 对话补全（同步 + 流式） | [chat-completions.md](./chat-completions.md) |
| `POST /paas/v4/async/chat/completions` | 异步对话补全 | [chat-completions.md §异步](./chat-completions.md) |
| `POST /paas/v4/async-result/{id}` | 查询异步结果 | [chat-completions.md §异步](./chat-completions.md) |
| `POST /paas/v4/embeddings` | 文本嵌入 | [embeddings.md](./embeddings.md) |
| `POST /paas/v4/images/generations` | 图像生成 | [images.md](./images.md) |
| `POST /paas/v4/videos/generations` | 视频生成（异步） | [videos.md](./videos.md) |
| `POST /paas/v4/audio/transcriptions` | 语音转文本（GLM-ASR） | [audio.md §ASR](./audio.md) |
| `POST /paas/v4/audio/speech` | 文本转语音（GLM-TTS） | [audio.md §TTS](./audio.md) |
| `POST /paas/v4/voice/clone` | 音色复刻 | [audio.md §音色复刻](./audio.md) |
| `POST /paas/v4/rerank` | 文本重排序（Rerank） | [misc.md](./misc.md) |
| `POST /paas/v4/tokenizer` | 文本分词器 | [misc.md](./misc.md) |
| `POST /paas/v4/layout_parsing` | 文档解析（同步 OCR） | [misc.md](./misc.md) |

### 工具 API

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /paas/v4/web_search` | 网络搜索 | [tools.md §Web Search](./tools.md) |
| `POST /paas/v4/web_reader` | 网页阅读 | [tools.md](./tools.md) |
| `POST /paas/v4/files/parser/create` | 文件解析（异步） | [tools.md §文件解析](./tools.md) |
| `POST /paas/v4/files/parser/sync` | 文件解析（同步） | [tools.md](./tools.md) |
| `POST /paas/v4/ocr` | OCR | [tools.md](./tools.md) |
| `POST /paas/v4/content_safety` | 内容安全 | [tools.md](./tools.md) |

### 批处理 / 文件

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /paas/v4/batches` | 创建批处理任务 | [batch.md](./batch.md) |
| `GET /paas/v4/batches` | 列出批处理任务 | [batch.md](./batch.md) |
| `GET /paas/v4/batches/{id}` | 检索批处理任务 | [batch.md](./batch.md) |
| `POST /paas/v4/batches/{id}/cancel` | 取消批处理任务 | [batch.md](./batch.md) |
| `POST /paas/v4/files` | 上传文件 | [files.md](./files.md) |
| `GET /paas/v4/files` | 文件列表 | [files.md](./files.md) |
| `GET /paas/v4/files/{id}/content` | 文件内容（Batch 输出） | [files.md](./files.md) |
| `DELETE /paas/v4/files/{id}` | 删除文件 | [files.md](./files.md) |

### Agent / 知识库 / 实时 API

| 类别 | 文档 | 说明 |
| --- | --- | --- |
| Agent API | [agents.md](./agents.md) | `POST /api/v1/agents` 智能体对话 / 异步结果 / 对话历史 |
| 知识库 API | [knowledge-base.md](./knowledge-base.md) | `/llm-application/open/knowledge`、`/zrag/retrieval` 等 |
| 实时 API（WSS） | [realtime.md](./realtime.md) | `wss://open.bigmodel.cn/api/paas/v4/realtime` GLM-Realtime |
| 助理 API（已弃用） | — | 见官方 https://docs.bigmodel.cn/api-reference/助理-api/助手对话 |

## 模型清单速查

> 完整清单与上下文窗口见 [models.md](./models.md)；价格见 [pricing.md](./pricing.md) 与官方[计费说明](https://www.bigmodel.cn/pricing)。

| 系列 | 旗舰 ID | 备注 |
| --- | --- | --- |
| 文本 | `glm-5.1` / `glm-5` / `glm-5-turbo` / `glm-4.7` / `glm-4.6` | 上下文最高 128K，含 `-flash` / `-flashx` / `-air` / `-airx` 子档 |
| 视觉 | `glm-5v-turbo` / `glm-4.6v` / `glm-4.1v-thinking` | 多模态（图 / 视频 / 文件 / 音频） |
| 图像生成 | `glm-image` / `cogview-4` / `cogview-3-flash` | — |
| 视频生成 | `cogvideox-3` / `viduq1-*` / `vidu2-*` | 含文本 / 图像 / 首尾帧 / 主体参考多变体 |
| 音视频 | `glm-4-voice` / `glm-asr-2512` / `glm-tts` / `glm-tts-clone` / `glm-realtime` | ASR / TTS / 实时对话 / 复刻 |
| 嵌入 | `embedding-3` / `embedding-2` | — |
| 免费 | `glm-4.7-flash` / `glm-4-flash-250414` / `glm-4v-flash` / `cogview-3-flash` / `cogvideox-flash` 等 | 限速较严 |

## 通用约定

- **思考模式**：通过顶层 `thinking: {type: enabled|disabled}` 控制；返回字段为 `choices[].message.reasoning_content`。流式时 `delta.reasoning_content` 与 `delta.content` 分别下发。GLM-5.1 / 5 / 4.7 / 4.5v 强制思考，其他模型自动判断。
- **上下文缓存（隐式）**：自动开启，命中体现在 `usage.prompt_tokens_details.cached_tokens`，缓存命中价约为标准 input 价的 50%。无显式 `cache_control` 字段。
- **工具调用**：`tool_choice` **仅支持 `"auto"`**；`tool_stream: true` 启用流式 function call（仅 GLM-5.1 / 5 / 5-Turbo / 4.7 / 4.6）。
- **`do_sample: false`** 等价于贪心解码，`temperature` / `top_p` 会被忽略。
- **`request_id` 长度限制 6–64**，`user_id` 长度 6–128（敏感词风控关联终端用户）。

## 计费要点

- 按 token 计费，区分 input / output / cached_input / cached_output。
- 思考模式产生的 `reasoning_content` token 计入 output。
- 视频 / 图像 / TTS / ASR 各自单价见官方[计费说明](https://www.bigmodel.cn/pricing) 与 [pricing.md](./pricing.md)。
- 知识库按字数另计费；详见 https://docs.bigmodel.cn/cn/guide/tools/knowledge/price。

## 参考

- API 概述：https://docs.bigmodel.cn/cn/api/introduction
- 速率限制：https://docs.bigmodel.cn/cn/api/rate-limit
- 错误码：https://docs.bigmodel.cn/cn/api/api-code
- 文档总索引：https://docs.bigmodel.cn/llms.txt
- 平台首页：https://open.bigmodel.cn/
- 计费说明：https://www.bigmodel.cn/pricing
- OpenAPI：https://docs.bigmodel.cn/openapi/openapi.json
