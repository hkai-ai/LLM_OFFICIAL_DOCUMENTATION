---
source: https://platform.minimaxi.com/docs/api-reference/speech-t2a-http
fetched_at: 2026-05-26
api_version: t2a_v2
---

# MiniMax · 语音合成（Text-to-Audio）

本文覆盖 4 个语音合成端点：

1. 同步 HTTP — `POST /v1/t2a_v2`
2. 同步 WebSocket — `wss://api.minimaxi.com/ws/v1/t2a_v2`
3. 异步任务创建 — `POST /v1/t2a_async_v2`
4. 异步任务查询 — `GET /v1/query/t2a_async_query_v2`

## 鉴权与备用端点

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>` |
| `Content-Type` | ✓ | `application/json`（同步 HTTP / 异步） |

主端点：`https://api.minimaxi.com`，备用：`https://api-bj.minimaxi.com`。

## 共享 payload 子结构

### `voice_setting`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `voice_id` | string | ✓ | — | 系统 / 复刻 / 文生音色 ID。 |
| `speed` | number | ✗ | `1.0` | 范围 `[0.5, 2]`。 |
| `vol` | number | ✗ | `1.0` | 范围 `(0, 10]`。 |
| `pitch` | integer | ✗ | `0` | 范围 `[-12, 12]`。 |
| `emotion` | string | ✗ | — | `happy` / `sad` / `angry` / `fearful` / `disgusted` / `surprised` / `calm` / `fluent` / `whisper`。 |
| `text_normalization` | boolean | ✗ | `false` | 文本规范化（仅同步 HTTP）。 |
| `english_normalization` | boolean | ✗ | `false` | 英文规范化（仅 WebSocket / 异步）。 |
| `latex_read` | boolean | ✗ | `false` | 朗读 LaTeX 公式。 |

### `audio_setting`

| 字段 | 类型 | 默认 | 取值 |
| --- | --- | --- | --- |
| `sample_rate` | integer | `32000` | `8000` / `16000` / `22050` / `24000` / `32000` / `44100` |
| `bitrate` | integer | `128000` | `32000` / `64000` / `128000` / `256000`（仅 mp3） |
| `format` | string | `mp3` | `mp3` / `pcm` / `flac` / `wav` / `pcmu_raw` / `pcmu_wav` / `opus` |
| `channel` | integer | `1` | `1` 单声 / `2` 双声 |
| `force_cbr` | boolean | `false` | 强制 CBR 编码 |

### `pronunciation_dict`

```json
{ "tone": ["燕少飞/(yan4)(shao3)(fei1)", "omg/oh my god"] }
```

### `timbre_weights`（音色混合，最多 4 个）

```json
{ "timbre_weights": [
  { "voice_id": "female-chengshu", "weight": 30 },
  { "voice_id": "female-tianmei",  "weight": 70 }
]}
```

### `voice_modify`（声音效果器）

| 字段 | 范围 | 说明 |
| --- | --- | --- |
| `pitch` | `[-100, 100]` | 音高调整 |
| `intensity` | `[-100, 100]` | 强度 |
| `timbre` | `[-100, 100]` | 音色 |
| `sound_effects` | 枚举 | `spacious_echo` / `auditorium_echo` / `lofi_telephone` / `robotic` |

### 文本控制标记（所有同步端点）

- `<#x#>` — 停顿 x 秒，范围 `[0.01, 99.99]`，最多两位小数。
- 换行符 — 段落切换。
- 语气词标签（仅 speech-2.8 系列）：`(laughs)` / `(coughs)` / `(sighs)` 等。

## 1. 同步 HTTP — `POST /v1/t2a_v2`

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | speech-2.8-hd / speech-2.8-turbo / speech-2.6-hd / speech-2.6-turbo / speech-02-hd / speech-02-turbo / speech-01-hd / speech-01-turbo |
| `text` | string | ✓ | — | 合成文本，< 10,000 字符 |
| `stream` | boolean | ✗ | `false` | 流式输出 |
| `stream_options.exclude_aggregated_audio` | boolean | ✗ | `false` | true 时最后 chunk 不再返回拼接后的整段音频 |
| `voice_setting` | object | ✗ | — | 见上 |
| `audio_setting` | object | ✗ | — | 见上 |
| `pronunciation_dict` | object | ✗ | — | 见上 |
| `timbre_weights` | array | ✗ | — | 见上 |
| `voice_modify` | object | ✗ | — | 见上 |
| `language_boost` | string | ✗ | `null` | 小语种增强：Chinese / English / Arabic / Russian / Spanish / Japanese / auto 等 20+ |
| `subtitle_enable` | boolean | ✗ | `false` | 启用字幕 |
| `subtitle_type` | string | ✗ | `sentence` | `sentence` / `word` / `word_streaming` |
| `output_format` | string | ✗ | `hex` | `url` / `hex` |
| `aigc_watermark` | boolean | ✗ | `false` | 添加音频标识 |

### 响应

```json
{
  "data": {
    "audio": "<hex 编码音频或 https://... URL>",
    "status": 2
  },
  "extra_info": {
    "audio_length": 9900,
    "audio_sample_rate": 32000,
    "audio_size": 160323,
    "bitrate": 128000,
    "word_count": 52,
    "invisible_character_ratio": 0,
    "usage_characters": 26,
    "audio_format": "mp3",
    "audio_channel": 1
  },
  "trace_id": "01b8bf9bb7433cc75c18eee6cfa8fe21",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

`data.status`：`1` = 流式 chunk 合成中，`2` = 合成完成。

### 最小请求

```json
{ "model": "speech-2.8-hd", "text": "今天天气很好" }
```

## 2. 同步 WebSocket — `wss://api.minimaxi.com/ws/v1/t2a_v2`

按 `task_start → task_continue → task_finish` 三阶段消息驱动。服务端事件：`connected_success` / `task_started` / `task_continued` / `task_finished` / `task_failed`。

### `task_start`

```json
{
  "event": "task_start",
  "model": "speech-2.8-turbo",
  "voice_setting": { "voice_id": "male-qn-qingse", "speed": 1.0, "vol": 1.0, "pitch": 0, "emotion": "calm" },
  "audio_setting": { "sample_rate": 32000, "bitrate": 128000, "format": "mp3", "channel": 1 },
  "language_boost": "Chinese"
}
```

### `task_continue`

```json
{
  "event": "task_continue",
  "text": "你好，这是语音合成测试。<#1.5#>现在进行停顿演示。"
}
```

### `task_finish`

```json
{ "event": "task_finish" }
```

### 服务端事件 `task_continued`

```json
{
  "session_id": "...",
  "event": "task_continued",
  "trace_id": "...",
  "data": { "audio": "<hex>" },
  "is_final": true,
  "extra_info": { /* 同步 HTTP extra_info */ },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

> 连接超时：最后收到结果后 120 秒无新事件则自动断开。
>
> 单次 `task_continue` 文本最多 10,000 字符。

## 3. 异步任务创建 — `POST /v1/t2a_async_v2`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 同同步 HTTP |
| `text` | string | ✗ | 与 `text_file_id` 二选一；最多 50,000 字符 |
| `text_file_id` | integer | ✗ | 上传文件接口返回的 file_id（txt/zip 格式） |
| `voice_setting` | object | ✓ | 见上 |
| `audio_setting` | object | ✗ | **`audio_sample_rate` 字段名与同步 HTTP 的 `sample_rate` 不一致**；其他同上 |
| `pronunciation_dict` | object | ✗ | — |
| `language_boost` | string | ✗ | — |
| `voice_modify` | object | ✗ | — |
| `aigc_watermark` | boolean | ✗ | 默认 `false` |

### 响应

```json
{
  "task_id": "95157322514444",
  "task_token": "eyJhbGciOiJSUz...",
  "file_id": 95157322514444,
  "usage_characters": 101,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

`file_id` 有效期 9 小时，用于后续 `/v1/files/retrieve` 拉取音频。

## 4. 异步任务查询 — `GET /v1/query/t2a_async_query_v2?task_id=<id>`

### 响应

```json
{
  "task_id": 95157322514444,
  "status": "success",
  "file_id": 95157322514496,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

`status` 枚举：`processing` / `success` / `failed` / `expired`。**查询限速 10 次 / 秒**。

## 错误码

详见 [errors.md](./errors.md)。常见：`1001` 超时 / `1002` 限流 / `1004` 鉴权失败 / `1042` 非法字符占比超 10%。

## 参考

- 同步 HTTP：https://platform.minimaxi.com/docs/api-reference/speech-t2a-http
- WebSocket：https://platform.minimaxi.com/docs/api-reference/speech-t2a-websocket
- 异步创建：https://platform.minimaxi.com/docs/api-reference/speech-t2a-async-create
- 异步查询：https://platform.minimaxi.com/docs/api-reference/speech-t2a-async-query
- 系统音色 ID 表：https://platform.minimaxi.com/docs/faq/system-voice-id
