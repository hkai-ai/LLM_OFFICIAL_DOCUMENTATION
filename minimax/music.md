---
source: https://platform.minimaxi.com/docs/api-reference/music-generation
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 音乐生成（Music Generation）

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

完整 URL：`POST https://api.minimaxi.com/v1/music_generation`

## 模型

| 模型 ID | 备注 |
| --- | --- |
| `music-2.6` | 文生音乐主力，**仅付费用户** |
| `music-cover` | 基于参考音频生成翻唱，**仅付费用户** |
| `music-2.6-free` | 免费版文生音乐 |
| `music-cover-free` | 免费版翻唱 |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 见上表 |
| `prompt` | string | ✗ | — | 音乐描述，范围 `[1, 2000]` 字符 |
| `lyrics` | string | ✗ | — | 歌词，支持结构标签（[verse]、[chorus] 等），范围 `[1, 3500]` 字符 |
| `is_instrumental` | boolean | ✗ | `false` | 是否纯音乐（无人声） |
| `lyrics_optimizer` | boolean | ✗ | `false` | 自动生成歌词 |
| `stream` | boolean | ✗ | `false` | 流式传输 |
| `output_format` | string | ✗ | `hex` | `url` / `hex` |
| `aigc_watermark` | boolean | ✗ | `false` | 添加水印 |
| `audio_url` | string | ✗ | — | **`music-cover` 专用**：参考音频 URL |
| `audio_base64` | string | ✗ | — | **`music-cover` 专用**：base64 编码音频 |
| `cover_feature_id` | string | ✗ | — | **`music-cover` 专用**：翻唱前处理特征 ID（由 `/v1/music/cover/preprocess` 返回） |
| `audio_setting` | object | ✗ | — | 见下 |

### `audio_setting`

| 字段 | 类型 | 取值 |
| --- | --- | --- |
| `sample_rate` | integer | `16000` / `24000` / `32000` / `44100` |
| `bitrate` | integer | `32000` / `64000` / `128000` / `256000` |
| `format` | string | `mp3` / `wav` / `pcm` |

## 响应

```json
{
  "data": {
    "status": 2,
    "audio": "<hex 编码音频>"
  },
  "trace_id": "...",
  "extra_info": {
    "music_duration": 30000,
    "music_sample_rate": 44100,
    "music_channel": 2,
    "bitrate": 128000,
    "music_size": 480000
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

`data.status`：`1` 处理中（流式 chunk），`2` 完成。

## 最小请求

```json
{ "model": "music-2.6", "prompt": "流行音乐,快乐,积极向上" }
```

## 关联端点

| 端点 | 用途 |
| --- | --- |
| `POST /v1/music/cover/preprocess` | 翻唱前处理，返回 `cover_feature_id` 供 `music-cover` 使用 |
| `POST /v1/lyrics/generation` | 单独的歌词生成端点 |

## 错误码与计费

- 常见错误：`1008` 余额不足 / `1026` prompt 涉敏。
- 价格：见 [pricing.md §音乐生成](./pricing.md)。

## 参考

- 音乐生成：https://platform.minimaxi.com/docs/api-reference/music-generation
- 翻唱前处理：https://platform.minimaxi.com/docs/api-reference/music-cover-preprocess
- 歌词生成：https://platform.minimaxi.com/docs/api-reference/lyrics-generation
- Guide：https://platform.minimaxi.com/docs/guides/music-generation
