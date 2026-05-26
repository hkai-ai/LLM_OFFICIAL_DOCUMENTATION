---
source: https://developers.openai.com/api/reference/resources/audio
fetched_at: 2026-05-19
api_version: N/A
---

# Audio · /v1/audio/{speech,transcriptions,translations}

> 音频相关端点：TTS（文字转语音）、STT（语音转文字）、Translation（语音翻译成英文）。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | TTS 用 `application/json`；transcriptions/translations 用 `multipart/form-data`。 |

---

## POST /v1/audio/speech （TTS）

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `tts-1` / `tts-1-hd` / `gpt-4o-mini-tts`（或带日期版本）。 |
| `input` | string | ✓ | — | 待合成文本，最长 4096 字符。 |
| `voice` | string \| object | ✓ | — | 内置：`alloy` / `ash` / `ballad` / `coral` / `echo` / `sage` / `shimmer` / `verse` / `marin` / `cedar`；或自定义 voice 对象 `{ id }`（来自 `/v1/audio/voices`）。 |
| `response_format` | string | ✗ | `mp3` | `mp3` / `opus` / `aac` / `flac` / `wav` / `pcm`。 |
| `speed` | number | ✗ | `1.0` | 倍速，范围 `[0.25, 4.0]`。 |
| `instructions` | string | ✗ | — | 风格 / 情感指令，≤ 4096 字符。`tts-1` / `tts-1-hd` 不支持。 |
| `stream_format` | string | ✗ | — | `sse` / `audio`。`sse` 不适用于 `tts-1` / `tts-1-hd`。 |

### 响应

二进制音频流（`Content-Type` 取决于 `response_format`）。设置 `stream_format: sse` 时返回 SSE 事件流。

### 示例

```json
{
  "model": "gpt-4o-mini-tts",
  "input": "The quick brown fox jumped over the lazy dog.",
  "voice": "alloy",
  "response_format": "mp3"
}
```

---

## POST /v1/audio/transcriptions （STT）

请求体为 `multipart/form-data`。

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file` | file | ✓ | — | 音频文件。支持 `flac` / `mp3` / `mp4` / `mpeg` / `mpga` / `m4a` / `ogg` / `wav` / `webm`，单文件 ≤ 25 MB。 |
| `model` | string | ✓ | — | `whisper-1` / `gpt-4o-transcribe` / `gpt-4o-mini-transcribe` / `gpt-4o-transcribe-diarize`（含带日期版本）。 |
| `language` | string | ✗ | — | ISO-639-1 输入语言代码，如 `en` / `zh`。提升精度与延迟。 |
| `prompt` | string | ✗ | — | 风格 / 上下文提示；可用于纠正专有名词。 |
| `response_format` | string | ✗ | `json` | `json` / `text` / `srt` / `verbose_json` / `vtt` / `diarized_json`。 |
| `temperature` | number | ✗ | `0` | `[0, 1]`。 |
| `timestamp_granularities` | array&lt;string&gt; | ✗ | — | `word` / `segment`，需配合 `response_format: verbose_json`。 |
| `include` | array&lt;string&gt; | ✗ | — | 附加返回，目前支持 `logprobs`。 |
| `stream` | boolean | ✗ | `false` | SSE 流式（`gpt-4o-(mini-)transcribe` 支持，`whisper-1` 不支持）。 |
| `chunking_strategy` | string \| object | ✗ | — | `"auto"` 或 VAD 配置 `{ type: "server_vad", prefix_padding_ms, silence_duration_ms, threshold }`。`gpt-4o-transcribe-diarize` 处理 >30s 音频时必填。 |
| `known_speaker_names` | array&lt;string&gt; | ✗ | — | 仅 diarize：≤ 4 个说话人标识。 |
| `known_speaker_references` | array&lt;string&gt; | ✗ | — | 仅 diarize：每人 2–10 秒 data URL 参考音频。 |

### 响应

**`json`**

```json
{
  "text": "...",
  "logprobs": [ { "token": "...", "logprob": -0.5, "bytes": [104, 105] } ],
  "usage": {
    "type": "tokens",
    "input_tokens": 14,
    "input_token_details": { "audio_tokens": 14, "text_tokens": 0 },
    "output_tokens": 45,
    "total_tokens": 59
  }
}
```

**`verbose_json`**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `task` | string | 固定 `transcribe`。 |
| `language` | string | 识别语言。 |
| `duration` | number | 音频时长（秒）。 |
| `text` | string | 全文。 |
| `segments[]` | array | `{ id, seek, start, end, text, tokens[], temperature, avg_logprob, compression_ratio, no_speech_prob }`。 |
| `words[]` | array | `{ word, start, end }`，仅 `timestamp_granularities` 含 `word` 时返回。 |
| `usage` | object | `{ type: "duration", seconds }`。 |

**`diarized_json`**

```json
{
  "task": "transcribe",
  "duration": 27.4,
  "text": "...",
  "segments": [
    {
      "type": "transcript.text.segment",
      "id": "seg_001",
      "start": 0.0,
      "end": 4.7,
      "text": "...",
      "speaker": "agent"
    }
  ],
  "usage": { "type": "duration", "seconds": 27 }
}
```

**`text` / `srt` / `vtt`** 直接返回对应纯文本。

---

## POST /v1/audio/translations

请求体为 `multipart/form-data`。把任意支持语言的音频翻译为英文文本。

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file` | file | ✓ | — | 同 transcriptions：`flac` / `mp3` / `mp4` / `mpeg` / `mpga` / `m4a` / `ogg` / `wav` / `webm`，≤ 25 MB。 |
| `model` | string | ✓ | — | 文档目前列出 `whisper-1` / `gpt-4o-transcribe` / `gpt-4o-mini-transcribe` / `gpt-4o-transcribe-diarize`。生产中以 `whisper-1` 为稳态。 |
| `prompt` | string | ✗ | — | 风格 / 上下文提示（应为英文）。 |
| `response_format` | string | ✗ | `json` | `json` / `text` / `srt` / `verbose_json` / `vtt`。 |
| `temperature` | number | ✗ | `0` | `[0, 1]`。 |

### 响应

`json` / `text` 返回 `{ "text": "..." }` 或纯文本。`verbose_json` 字段同 transcriptions（`language` 始终为 `english`，不返回 `words[]`）。

---

## 示例（transcriptions，curl）

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F file="@audio.mp3" \
  -F model="gpt-4o-transcribe" \
  -F response_format="verbose_json" \
  -F timestamp_granularities[]="word"
```

## 错误码

| HTTP | error.type | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 文件格式不支持、超过 25 MB、参数错误。 |
| `401` | `authentication_error` | API key 无效。 |
| `429` | `rate_limit_error` | 限流或配额耗尽。 |

## 参考

- TTS：<https://developers.openai.com/api/reference/resources/audio/subresources/speech/methods/create>
- STT：<https://developers.openai.com/api/reference/resources/audio/subresources/transcriptions/methods/create>
- Translation：<https://developers.openai.com/api/reference/resources/audio/subresources/translations/methods/create>
- 上级目录：<https://developers.openai.com/api/reference/resources/audio>
