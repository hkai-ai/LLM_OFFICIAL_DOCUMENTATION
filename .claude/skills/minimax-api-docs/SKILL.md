---
name: minimax-api-docs
description: 需要查阅、更新或扩充本仓库 `minimax/` 目录下 MiniMax 平台 API 中文文档时使用。涵盖语言模型（M2 系列，Anthropic / OpenAI 双 SDK 兼容）、语音合成（T2A，sync / WebSocket / async）、声音克隆、音色设计、视频生成（Hailuo 系列）、图像生成（image-01）、音乐生成（music-2.x）、文件管理、按量付费定价（pricing.md）等。触发场景包括："MiniMax 怎么用"、"M2.7 模型"、"Hailuo 视频生成"、"speech-2.8-hd 价格"、"MiniMax 定价"、"Anthropic 兼容 MiniMax"、"刷新 MiniMax 文档"、"MiniMax 声音克隆"。
---

# MiniMax API 文档撰写与维护

## 1. 文档站全貌

- 入口：`https://platform.minimaxi.com/docs/`
- API 总览（推荐起点）：`https://platform.minimaxi.com/docs/api-reference/api-overview`
- 按量付费定价：`https://platform.minimaxi.com/docs/guides/pricing-paygo`
- 语言模型采用 **Anthropic SDK（推荐）** + **OpenAI SDK** 双兼容，HTTP 原生路径也保留。
- 文档分区（侧边栏）大致：
  - **API Reference**：chat（M2 系列）、speech（T2A）、voice-cloning、voice-design、video-generation、image-generation、music-generation、file。
  - **Guides**：pricing-paygo / token-plan、quickstart、SDK 接入示例、限速、错误码、模型对照。
- 公开访问，无登录可读 API 参考；个别能力（如声音克隆）需要在控制台完成实名认证后才能调用。

## 2. 抓取要点

- **优先 `.md` 镜像**：官方文档每个页面都有 `<url>.md` 后缀的纯文本版本（如 `https://platform.minimaxi.com/docs/api-reference/speech-t2a-http.md`），对 WebFetch 极友好。**不要**用没有 `.md` 的 URL（部分子页直接 404，需要带 `.md` 才能命中）。
- **`platform.minimaxi.com` 偶发"无法验证域名安全"**：与 DeepSeek 类似，直接重试同一 URL；持续失败时切到 `https://platform.minimaxi.com/docs/llms.txt` 找镜像链接清单。
- **API 双协议兼容**：
  - OpenAI 兼容路径在 `/v1`（鉴权 `Authorization: Bearer`）。
  - **Anthropic 兼容路径在 `/anthropic/v1`**（不是 `/v1`），鉴权用 `X-Api-Key`，**不**需要 `anthropic-version` header。
- **统一响应封装**：所有端点返回顶层 `base_resp.status_code` / `status_msg`；HTTP 通常 200，业务错误从 base_resp 读。撰写时务必把这个字段写进每一份 endpoint md。
- **视频端点共用 URL**：t2v / i2v / fl2v / s2v 全部打到 `POST /v1/video_generation`，靠 `model` 与 `first_frame_image` / `last_frame_image` / `subject_reference` 字段区分。
- **异步语音 `audio_setting` 字段名差异**：异步端点用 `audio_sample_rate`，同步 HTTP / WebSocket 用 `sample_rate`。这是真实差异，不是文档错。
- **定价子页较散**：当前 `pricing-paygo` 单页包含语言模型 / 语音 / 视频 / 图像 / 音乐多模态全部价格，但视频段（Hailuo 各分辨率 × 时长）的细化矩阵在 paygo 页中以下拉表呈现，WebFetch 偶尔抓不全 → 需要单独 prompt 要求"列出 Hailuo 视频生成的完整价格矩阵：每个模型 × 分辨率 × 时长组合的元数"。

## 3. 更新流程

1. **先抓 `llms.txt`**：`https://platform.minimaxi.com/docs/llms.txt`。diff 与本仓库已记录的端点清单（详见 §6 关键链接表）；若有新增主题，在 README 端点速查表和本 SKILL 的链接表同步补行。
2. **抓 `api-overview`**：`https://platform.minimaxi.com/docs/api-reference/api-overview.md`，核对 Base URL、鉴权 header 是否变动。
3. **抓 `pricing-paygo.md`** → 同步 `minimax/pricing.md`：
   - 语言模型按 `MiniMax-M2.7` / `-highspeed` / `M2.5` / `-highspeed` / `M2-her` 全量列。
   - 语音按模型 ID 与 hd/turbo 档位列。
   - 视频按完整 (模型, 分辨率, 时长) 矩阵列。
   - 图像、音乐、MCP API-vlm 等小模块完整列。
4. **抓 `errorcode.md`** → 同步 `minimax/errors.md`：核对 1000–2056 段错误码是否新增；MiniMax 错误码列表会随能力新增而扩张（例如新增视频 / 音乐场景的专用码）。
5. **chat / messages 同步**：抓 `text-chat-openai.md` 与 `text-chat-anthropic.md` → 对照 `chat-completions.md` 与 `messages.md`；同时核对 `text-prompt-caching.md` 与 `anthropic-api-compatible-cache.md` 的缓存规则。
6. **多模态 endpoint 同步**：逐个抓 §6 列出的 speech / voice / video / image / music / file md 镜像，diff 字段。
7. **修改对应 md 的 `fetched_at`**，同步 `minimax/README.md` 的 `last_updated`。
8. **若新增模型 ID**：同步 `minimax/models.md` 与 `minimax/README.md` 的"代表模型"段。

## 4. 定价文档维护（`minimax/pricing.md`）

- **唯一权威源**：`https://platform.minimaxi.com/docs/guides/pricing-paygo`。
- **同步触发**：新模型上线（如未来 M2.8）、Hailuo 视频价表调整、`speech-3.x` 系列推出、声音克隆 / 音色设计单价变动、`-highspeed` 档位价格调整、MCP API-vlm 单次调用价变动。
- **抓取要点**：
  - 全部 ¥ 计价，**不要换算成 USD**。
  - 语言模型缓存价分 `cache_read` 与 `cache_write` 两档；`-highspeed` 档与标准档共享缓存价。
  - 语音同步 / 异步同价，按"每万字符"计；声音克隆与音色设计按"首次合成时一次性扣 ¥9.9"模型，文档要标计费时机。
  - 视频 Hailuo 系列价格区间为 ¥0.60–¥4.00，**官方按 (model, resolution, duration) 矩阵给出**；本仓库当前 pricing.md 仅写区间，后续应补全完整矩阵（需要重抓时显式 prompt）。
- **同步步骤**：
  1. WebFetch pricing-paygo 页 → 比对 `minimax/pricing.md` 数字。
  2. 改 `pricing.md` 的 `fetched_at`，同步 `minimax/README.md` 的 `last_updated`。
- **坑点提醒**：
  - `M2-her` 当前未公开缓存价档位；不要凭印象套用 M2.5 的缓存价。
  - `music-2.5+` 是独立模型 ID（带加号），写表时不要把它视为 `music-2.5` 的别名。
  - 声音克隆需要实名认证，撰写时要在 `pricing.md` 或 `README.md` 提醒。

## 5. 坑点清单

- **平台域名混淆**：MiniMax 国内开发者平台是 `platform.minimaxi.com`（注意末尾 `i`），消费 / 开放平台另有 `api.minimax.io` / `minimax.io` 等域名；API Key 与文档版本不互通。
- **Anthropic SDK 接入**：官方推荐通过 Anthropic SDK 替换 `base_url` 接入 MiniMax 语言模型，撰写 chat 文档时建议先写 Anthropic 路径再写 OpenAI 路径。
- **长上下文**：M2 系列默认上下文为 204,800 tokens；不要假设与 Qwen / Claude 同窗口。
- **任务失败不计费**：异步视频 / 长音频任务失败不扣款；这条规则写入 `pricing.md` 计费要点。
- **文件接口**：单文件 ≤ 512MB，账户总容量 100GB；超出会拒绝上传，不是分片上传场景。

## 6. 关键链接表

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| API 总览 | https://platform.minimaxi.com/docs/api-reference/api-overview | `minimax/README.md` |
| 文档总索引 | https://platform.minimaxi.com/docs/llms.txt | （查找用） |
| 按量付费定价 | https://platform.minimaxi.com/docs/guides/pricing-paygo | `minimax/pricing.md` |
| 错误码 | https://platform.minimaxi.com/docs/api-reference/errorcode | `minimax/errors.md` |
| Chat - OpenAI 兼容 HTTP | https://platform.minimaxi.com/docs/api-reference/text-chat-openai | `minimax/chat-completions.md` |
| Chat - Anthropic 兼容 HTTP | https://platform.minimaxi.com/docs/api-reference/text-chat-anthropic | `minimax/messages.md` |
| Chat - OpenAI SDK 接入 | https://platform.minimaxi.com/docs/api-reference/text-openai-api | `minimax/chat-completions.md` |
| Chat - Anthropic SDK 接入 | https://platform.minimaxi.com/docs/api-reference/text-anthropic-api | `minimax/messages.md` |
| Prompt 缓存（被动） | https://platform.minimaxi.com/docs/api-reference/text-prompt-caching | `minimax/chat-completions.md` §上下文缓存 |
| Anthropic 主动缓存（cache_control） | https://platform.minimaxi.com/docs/api-reference/anthropic-api-compatible-cache | `minimax/messages.md` §显式 Prompt Caching |
| Models - OpenAI List | https://platform.minimaxi.com/docs/api-reference/models/openai/list-models | `minimax/models.md` |
| Models - Anthropic List | https://platform.minimaxi.com/docs/api-reference/models/anthropic/list-models | `minimax/models.md` |
| 语音合成 - 同步 HTTP | https://platform.minimaxi.com/docs/api-reference/speech-t2a-http | `minimax/speech.md` §1 |
| 语音合成 - WebSocket | https://platform.minimaxi.com/docs/api-reference/speech-t2a-websocket | `minimax/speech.md` §2 |
| 语音合成 - 异步创建 | https://platform.minimaxi.com/docs/api-reference/speech-t2a-async-create | `minimax/speech.md` §3 |
| 语音合成 - 异步查询 | https://platform.minimaxi.com/docs/api-reference/speech-t2a-async-query | `minimax/speech.md` §4 |
| 上传复刻音频 | https://platform.minimaxi.com/docs/api-reference/voice-cloning-uploadcloneaudio | `minimax/voice.md` §1 |
| 上传 prompt 音频 | https://platform.minimaxi.com/docs/api-reference/voice-cloning-uploadprompt | `minimax/voice.md` §1 |
| 音色快速复刻 | https://platform.minimaxi.com/docs/api-reference/voice-cloning-clone | `minimax/voice.md` §2 |
| 音色设计 | https://platform.minimaxi.com/docs/api-reference/voice-design-design | `minimax/voice.md` §3 |
| 查询音色 | https://platform.minimaxi.com/docs/api-reference/voice-management-get | `minimax/voice.md` §4 |
| 删除音色 | https://platform.minimaxi.com/docs/api-reference/voice-management-delete | `minimax/voice.md` §5 |
| 视频 - 文生 | https://platform.minimaxi.com/docs/api-reference/video-generation-t2v | `minimax/video.md` §1 |
| 视频 - 图生 | https://platform.minimaxi.com/docs/api-reference/video-generation-i2v | `minimax/video.md` §1 |
| 视频 - 首尾帧 | https://platform.minimaxi.com/docs/api-reference/video-generation-fl2v | `minimax/video.md` §1 |
| 视频 - 主体参考 | https://platform.minimaxi.com/docs/api-reference/video-generation-s2v | `minimax/video.md` §1 |
| 视频 - 任务查询 | https://platform.minimaxi.com/docs/api-reference/video-generation-query | `minimax/video.md` §2 |
| 视频 - 下载 | https://platform.minimaxi.com/docs/api-reference/video-generation-download | `minimax/video.md` §3 |
| 视频 Agent - 创建 | https://platform.minimaxi.com/docs/api-reference/video-agent-create | `minimax/video.md` §4 |
| 视频 Agent - 查询 | https://platform.minimaxi.com/docs/api-reference/video-agent-query | `minimax/video.md` §4 |
| 图像 - 文生 | https://platform.minimaxi.com/docs/api-reference/image-generation-t2i | `minimax/image.md` |
| 图像 - 图生 | https://platform.minimaxi.com/docs/api-reference/image-generation-i2i | `minimax/image.md` |
| 音乐 - 生成 | https://platform.minimaxi.com/docs/api-reference/music-generation | `minimax/music.md` |
| 音乐 - 翻唱前处理 | https://platform.minimaxi.com/docs/api-reference/music-cover-preprocess | `minimax/music.md` §关联端点 |
| 音乐 - 歌词 | https://platform.minimaxi.com/docs/api-reference/lyrics-generation | `minimax/music.md` §关联端点 |
| 文件 - 上传 | https://platform.minimaxi.com/docs/api-reference/file-management-upload | `minimax/files.md` §1 |
| 文件 - 列表 | https://platform.minimaxi.com/docs/api-reference/file-management-list | `minimax/files.md` §2 |
| 文件 - 检索 | https://platform.minimaxi.com/docs/api-reference/file-management-retrieve | `minimax/files.md` §3 |
| 文件 - 视频下载 | https://platform.minimaxi.com/docs/api-reference/file-management-retrieve-content | `minimax/files.md` §3 |
| 文件 - 删除 | https://platform.minimaxi.com/docs/api-reference/file-management-delete | `minimax/files.md` §4 |
