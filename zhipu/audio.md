---
source: https://docs.bigmodel.cn/api-reference/模型-api/语音转文本
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM · 语音 / 音色

涵盖 3 个端点：

- ASR（语音转文本）— `POST /paas/v4/audio/transcriptions`
- TTS（文本转语音）— `POST /paas/v4/audio/speech`
- 音色复刻 — `POST /paas/v4/voice/clone`

实时语音对话走 chat 端点（`glm-4-voice`），见 [chat-completions.md §`messages[].content[]`](./chat-completions.md)。

## 鉴权（统一）

`Authorization: Bearer <API_KEY>`，Base URL `https://open.bigmodel.cn/api/paas/v4`。

## 1. ASR · POST /paas/v4/audio/transcriptions

### 请求（multipart/form-data）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `glm-asr-2512` |
| `file` | binary | ✓（或 `file_base64`） | — | `.wav` / `.mp3`；≤ 25MB，≤ 30 秒 |
| `file_base64` | string | ✓（或 `file`） | — | base64 编码音频 |
| `prompt` | string | ✗ | — | 长文本场景中可填入"之前的转录结果"作为上下文，建议 < 8000 字 |
| `hotwords` | array | ✗ | — | 最多 100 个热词，提升识别率 |
| `stream` | boolean | ✗ | `false` | true 时通过 Event Stream 逐块返回 |
| `request_id` | string | ✗ | 自动 | UUID 格式 6–64 字符 |
| `user_id` | string | ✗ | — | 6–128 字符 |

### 响应（同步）

```json
{
  "id": "...",
  "created": 1704067200,
  "request_id": "...",
  "model": "glm-asr-2512",
  "text": "音频转录的完整内容"
}
```

### 流式响应

`stream: true` 时按事件下发：

| 事件 `type` | 说明 | 字段 |
| --- | --- | --- |
| `transcript.text.delta` | 转录增量 | `delta: string` |
| `transcript.text.done` | 完成 | — |

## 2. TTS · POST /paas/v4/audio/speech

### 请求

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | `glm-tts` | TTS 模型 |
| `input` | string | ✓ | — | 待合成文本 |
| `voice` | string | ✓ | `tongtong` | 音色名（见下） |
| `response_format` | string | ✗ | `pcm` | `wav` / `pcm` |
| `speed` | number | ✗ | `1.0` | 范围 `[0.5, 2]` |
| `volume` | number | ✗ | `1.0` | 范围 `(0, 10]` |
| `watermark_enabled` | boolean | ✗ | `true` | 添加水印 |
| `stream` | boolean | ✗ | `false` | 流式输出 |
| `encode_format` | string | ✗ | `base64` | 流式编码：`base64` / `hex` |

### 系统音色

`tongtong`（彤彤，默认）/ `chuichui`（锤锤）/ `xiaochen`（小陈）/ `jam` / `kazi` / `douji` / `luodo`（动物圈系列）。

### 响应

- 同步成功（HTTP 200）：直接返回 WAV / PCM 二进制流。
- 失败：JSON `{ "error": { "code": "...", "message": "..." } }`。

### 最小请求

```json
{
  "model": "glm-tts",
  "input": "你好，今天天气怎么样",
  "voice": "tongtong",
  "response_format": "wav"
}
```

## 3. 音色复刻 · POST /paas/v4/voice/clone

> 复刻一个新音色并合成指定文本。示例音频先通过 `POST /paas/v4/files`（`purpose=voice-clone-input`）上传，拿到 `file_id` 后调用本端点。

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | `glm-tts-clone` |
| `voice_name` | string | ✓ | 自定义音色唯一名 |
| `file_id` | string | ✓ | 示例音频的 file_id |
| `input` | string | ✓ | 目标合成文本 |
| `text` | string | ✗ | 示例音频对应文本（提高复刻效果） |
| `request_id` | string | ✗ | UUID 格式 6–64 字符 |

### 限制

- 音频文件 ≤ 10MB，推荐时长 3–30 秒。

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `voice` | string | 生成的音色标识 |
| `file_id` | string | 合成音频试听文件 ID |
| `file_purpose` | string | 固定 `voice-clone-output` |
| `request_id` | string | 请求 ID |

### 最小请求

```json
{
  "model": "glm-tts-clone",
  "voice_name": "my_voice_001",
  "file_id": "file_xyz123",
  "input": "生成此文本的语音"
}
```

## 参考

- ASR：https://docs.bigmodel.cn/api-reference/模型-api/语音转文本
- TTS：https://docs.bigmodel.cn/api-reference/模型-api/文本转语音
- 音色复刻：https://docs.bigmodel.cn/api-reference/模型-api/音色复刻
- GLM-ASR 模型：https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-asr-2512
- GLM-TTS 模型：https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-tts
- GLM-TTS-Clone：https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-tts-clone
- GLM-Realtime：https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-realtime
- GLM-4-Voice：https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-4-voice
