---
source: https://platform.minimaxi.com/docs/api-reference/lyrics-generation
fetched_at: 2026-05-26
api_version: v1
---

# 歌词生成 + 翻唱前处理

把 [music.md](./music.md) 之外的两条音乐辅助接口合并到本页：

| 端点 | METHOD | PATH |
| --- | --- | --- |
| 歌词生成 | POST | `/v1/lyrics_generation` |
| 翻唱前处理 | POST | `/v1/music_cover_preprocess` |

> Base URL：`https://api.minimaxi.com`。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $MINIMAX_API_KEY`。 |
| `Content-Type` | ✓ | `application/json`。 |

## 1. 歌词生成 · POST /v1/lyrics_generation

### 请求 body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `mode` | enum | ✓ | — | `write_full_song`（完整歌曲）或 `edit`（基于已有歌词编辑 / 续写）。 |
| `prompt` | string | ✗ | — | 主题 / 风格 / 编辑方向描述，≤2000 字符。 |
| `lyrics` | string | edit 时 ✓ | — | 现有歌词文本，≤3500 字符。 |
| `title` | string | ✗ | — | 指定歌曲标题；提供则原样回写。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `song_title` | string | 生成的歌曲名（若请求未传 `title`）。 |
| `style_tags` | string | 逗号分隔的风格标签，例如 `"Pop, Upbeat"`。 |
| `lyrics` | string | 完整歌词，包含结构标签。 |
| `base_resp.status_code` | integer | `0` 表示成功。 |
| `base_resp.status_msg` | string | — |

#### 支持的结构标签

`[Intro]` / `[Verse]` / `[Pre-Chorus]` / `[Chorus]` / `[Hook]` / `[Drop]` / `[Bridge]` / `[Solo]` / `[Build-up]` / `[Instrumental]` / `[Breakdown]` / `[Break]` / `[Interlude]` / `[Outro]`。

### 最小请求

```bash
curl https://api.minimaxi.com/v1/lyrics_generation \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "write_full_song",
    "prompt": "一首关于雨夜咖啡馆的爵士流行",
    "title": "Midnight Brew"
  }'
```

## 2. 翻唱前处理 · POST /v1/music_cover_preprocess

两步翻唱流程的第一步：上传参考音频，平台抽取乐曲结构 + ASR 歌词；返回 `cover_feature_id` 作为后续 [music.md](./music.md) 的翻唱输入。

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 固定 `music-cover`。 |
| `audio_url` | string | 二选一 | 公网音频 URL；与 `audio_base64` 二选一。 |
| `audio_base64` | string | 二选一 | base64 字节流（不含 `data:` 前缀）。 |

参考音频限制：

- 时长 **6 秒 – 6 分钟**。
- 大小 ≤ **50 MB**。
- 格式：`mp3` / `wav` / `flac`。

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `cover_feature_id` | string | 唯一音频特征 ID，**24 小时内有效**；过期后需重新预处理。 |
| `formatted_lyrics` | string | ASR 抽取并加好段落标签（`[Verse]` / `[Chorus]` / `[Bridge]` 等）的歌词；可在客户端修改后再传给翻唱接口。 |
| `structure_result` | object | 结构分析；每段含 `type`（`intro` / `verse` / `chorus` / `bridge` / `outro` / `inst` / `silence`）+ `start_time` + `end_time`。 |
| `audio_duration` | number | 参考音频时长（秒）。 |
| `trace_id` | string | 链路 ID。 |
| `base_resp.status_code` | integer | `0` 成功。 |

### 接入翻唱

把 `cover_feature_id` 传入 [music.md](./music.md) 的音乐生成接口 `cover_feature_id` 参数，并把 `formatted_lyrics`（可手工调整）传 `lyrics`，即可生成一首结构 / 节奏对齐源曲的翻唱版本。

### 最小请求

```bash
curl https://api.minimaxi.com/v1/music_cover_preprocess \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-cover",
    "audio_url": "https://example.com/reference.mp3"
  }'
```

## 错误码

| status_code | 含义 | 触发 |
| --- | --- | --- |
| `1002` | InvalidParam | `mode` 不合法 / 参考音频时长越界 |
| `1004` | AuthenticationError | API Key 错误 |
| `1008` | InsufficientBalance | 余额不足 |
| `2013` | InvalidAudio | 参考音频格式不支持 / 损坏 |
| `2049` | ContentRiskDetected | 歌词或参考音频被风控拦截 |

完整错误码见 [errors.md](./errors.md)。

## 参考

- 歌词生成：<https://platform.minimaxi.com/docs/api-reference/lyrics-generation>
- 翻唱前处理：<https://platform.minimaxi.com/docs/api-reference/music-cover-preprocess>
- 音乐生成（含 `cover_feature_id` 入参）：[music.md](./music.md)
