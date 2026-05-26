---
source: https://platform.minimaxi.com/docs/api-reference/api-overview
fetched_at: 2026-05-26
api_version: N/A
last_updated: 2026-05-26
---

> 2026-05-26 更新：补齐 Prompt 缓存（caching.md，含 OpenAI 兼容被动缓存 + Anthropic 主动缓存）、歌词生成 + 翻唱前处理（lyrics.md）。

# MiniMax 平台 API 概览

MiniMax 平台覆盖语言模型、语音合成、声音克隆、图像 / 视频 / 音乐生成、文件管理等能力。对语言模型同时提供 **Anthropic 兼容** 与 **OpenAI 兼容** 两套 HTTP API；其他模态走 MiniMax 自家协议（POST JSON + Bearer Token）。

## 基本信息

| 项 | 值 |
| --- | --- |
| Base URL（国内） | `https://api.minimaxi.com` |
| Base URL（国际） | `https://api.minimax.io` |
| 备用主域 | `https://api-bj.minimaxi.com`（部分语音端点） |
| 鉴权（OpenAI 兼容 / 多模态） | `Authorization: Bearer <API_KEY>` |
| 鉴权（Anthropic 兼容） | `X-Api-Key: <API_KEY>`（**不需要 `anthropic-version`**） |
| 上下文窗口（M2 系列） | 204,800 tokens |
| 通用响应封装 | 顶层 `base_resp.status_code` / `status_msg`（HTTP 通常 200，业务错误从 base_resp 读） |

## 厂商目录结构（本仓库）

- `README.md` — 本文件（厂商概览 + 端点索引）
- `chat-completions.md` — OpenAI 兼容 chat 端点
- `messages.md` — Anthropic 兼容 messages 端点（含显式 cache_control）
- `models.md` — 全部模型清单 + List Models（OpenAI / Anthropic 两套）
- `speech.md` — 语音合成 4 个端点（同步 HTTP / WebSocket / 异步创建 / 异步查询）
- `voice.md` — 音色复刻 / 设计 / 管理 5 个端点
- `video.md` — 视频生成（t2v / i2v / fl2v / s2v）+ 任务查询 + 下载 + Agent
- `image.md` — 文生图 + 图生图（共用 `/v1/image_generation`）
- `music.md` — 音乐生成
- `lyrics.md` — 歌词生成 + 翻唱前处理（与 music 共用）
- `caching.md` — Prompt 缓存（被动 + Anthropic 主动 cache_control）
- `files.md` — 文件上传 / 列表 / 检索 / 删除
- `errors.md` — 错误码与排查
- `pricing.md` — 全模型定价表（CNY）

## 端点速查

### 语言模型（LLM）

| 协议 | 端点 | 文档 |
| --- | --- | --- |
| OpenAI 兼容 | `POST /v1/chat/completions` | [chat-completions.md](./chat-completions.md) |
| Anthropic 兼容 | `POST /anthropic/v1/messages` | [messages.md](./messages.md) |

代表模型：`MiniMax-M2.7` / `MiniMax-M2.7-highspeed` / `MiniMax-M2.5` / `MiniMax-M2.5-highspeed` / `MiniMax-M2.1` / `MiniMax-M2` / `M2-her`。详见 [models.md](./models.md)。

### 语音合成（T2A, Text-to-Audio）

| 端点 | 形态 | 文档 |
| --- | --- | --- |
| `POST /v1/t2a_v2` | 同步 HTTP | [speech.md §1](./speech.md) |
| `wss://api.minimaxi.com/ws/v1/t2a_v2` | 同步 WebSocket | [speech.md §2](./speech.md) |
| `POST /v1/t2a_async_v2` | 异步任务创建 | [speech.md §3](./speech.md) |
| `GET /v1/query/t2a_async_query_v2` | 异步任务查询 | [speech.md §4](./speech.md) |

代表模型：`speech-2.8-hd` / `speech-2.8-turbo` / `speech-2.6-hd` / `speech-2.6-turbo` / `speech-02-*` / `speech-01-*`。

### 声音克隆 / 音色设计 / 音色管理

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /v1/files/upload`（`purpose=voice_clone`） | 上传复刻样本（需实名认证） | [voice.md §1](./voice.md) |
| `POST /v1/voice_clone` | 执行音色复刻 | [voice.md §2](./voice.md) |
| `POST /v1/voice_design` | 由文本生成新音色 | [voice.md §3](./voice.md) |
| `POST /v1/get_voice` | 查询可用音色 ID | [voice.md §4](./voice.md) |
| `POST /v1/delete_voice` | 删除音色 | [voice.md §5](./voice.md) |

### 视频生成

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /v1/video_generation` | 文生 / 图生 / 首尾帧 / 主体参考视频（按 `model` 与字段区分） | [video.md §1](./video.md) |
| `GET /v1/query/video_generation?task_id=` | 任务状态查询 | [video.md §2](./video.md) |
| `GET /v1/files/retrieve?file_id=` | 视频下载（与文件管理共用） | [video.md §3](./video.md) |
| `POST /v1/video-agent/create` + `GET /v1/video-agent/query` | 视频 Agent（模板） | [video.md §4](./video.md) |

代表模型：`MiniMax-Hailuo-2.3` / `MiniMax-Hailuo-2.3-Fast` / `MiniMax-Hailuo-02` / `T2V-01-*` / `I2V-01-*` / `S2V-01`。

### 图像生成

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /v1/image_generation` | 文生图 / 图生图（共用，按 `subject_reference` 区分） | [image.md](./image.md) |

代表模型：`image-01` / `image-01-live`。

### 音乐生成

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /v1/music_generation` | 文生音乐 / 翻唱 | [music.md](./music.md) |
| `POST /v1/music/cover/preprocess` | 翻唱前处理 | [music.md §关联端点](./music.md) |
| `POST /v1/lyrics/generation` | 歌词生成 | [music.md §关联端点](./music.md) |

代表模型：`music-2.6` / `music-cover` / 同名 `-free` 版。

### 文件管理

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `POST /v1/files/upload` | 上传（multipart） | [files.md §1](./files.md) |
| `GET /v1/files/list?purpose=` | 列出 | [files.md §2](./files.md) |
| `GET /v1/files/retrieve?file_id=` | 检索（含 download_url） | [files.md §3](./files.md) |
| `POST /v1/files/delete` | 删除 | [files.md §4](./files.md) |

支持的 `purpose`：`voice_clone` / `prompt_audio` / `t2a_async_input` / `t2a_async` / `video_generation`。

### 模型管理

| 端点 | 用途 | 文档 |
| --- | --- | --- |
| `GET /v1/models` | OpenAI 规范 List Models | [models.md](./models.md) |
| `GET /anthropic/v1/models` | Anthropic 规范 List Models（含 `display_name` / 分页） | [models.md](./models.md) |

## 计费要点

- 全部按量付费（pay-as-you-go），具体单价见 [pricing.md](./pricing.md)。
- 语言模型按 token 计费，区分 input / output / cache read / cache write；Prompt caching 最小触发阈值 input ≥ 512 tokens。
- 音色复刻 / 设计采用"首次合成使用时一次性扣 ¥9.9"模型。
- 异步端点（语音 / 视频 / 翻唱）失败不计费。
- 无状态接口（不留存对话上下文，需客户端自行拼接 messages）。

## 已知坑点

- **平台域名易混淆**：MiniMax 国内开发者平台是 `platform.minimaxi.com`（末尾 `i`），与海外 `minimax.io` / 老 `api.minimax.io` 各自独立鉴权。
- **Anthropic 兼容 base path**：是 `/anthropic/v1`，**不是** `/v1`；鉴权用 `X-Api-Key`，不需要 `anthropic-version`。
- **业务错误码 ≠ HTTP code**：HTTP 通常 200，业务成功与否看 `base_resp.status_code == 0`。
- **异步语音 `audio_setting` 字段名差异**：异步端点用 `audio_sample_rate`，同步 HTTP / WebSocket 用 `sample_rate`。
- **视频任务下载 URL 仅 1 小时**；异步语音 file_id 有效期 9 小时。
- **`-highspeed` 档**：仅影响吞吐与延迟，缓存价不变，价格为标准档 2×。
- **声音复刻需实名认证**：未认证账户调用返回 `2038`。

## 参考

- API Overview：https://platform.minimaxi.com/docs/api-reference/api-overview
- 文档总索引：https://platform.minimaxi.com/docs/llms.txt
- 按量付费定价：https://platform.minimaxi.com/docs/guides/pricing-paygo
- 速率限制：https://platform.minimaxi.com/docs/guides/rate-limits
- 错误码原文：https://platform.minimaxi.com/docs/api-reference/errorcode
