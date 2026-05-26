---
source: https://platform.minimaxi.com/docs/api-reference/voice-cloning-clone
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 音色复刻 / 设计 / 管理

本文覆盖 5 个音色相关端点：

1. 上传复刻音频（走通用文件上传） — `POST /v1/files/upload`（`purpose=voice_clone`）
2. 音色快速复刻 — `POST /v1/voice_clone`
3. 音色设计 — `POST /v1/voice_design`
4. 查询可用音色 — `POST /v1/get_voice`
5. 删除音色 — `POST /v1/delete_voice`

> **声音复刻需要实名认证**（个人 / 企业）。未认证账户调用会返回 `2038`。

## 鉴权

所有端点统一：`Authorization: Bearer <API_KEY>` + `Content-Type: application/json`（除上传走 multipart）。Base URL `https://api.minimaxi.com`。

## 1. 上传复刻音频 — `POST /v1/files/upload`

### multipart/form-data 字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `purpose` | string | ✓ | 固定 `voice_clone` |
| `file` | binary | ✓ | 音频文件 |

### 文件要求

- 格式：MP3 / M4A / WAV
- 时长：10 秒 ～ 5 分钟
- 大小：≤ 20 MB

### 响应

```json
{
  "file": {
    "file_id": 123456789,
    "bytes": 5896337,
    "created_at": 1700469398,
    "filename": "audio.mp3",
    "purpose": "voice_clone"
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

> 若用作 prompt 样本而非复刻样本，`purpose` 改为 `prompt_audio`（详见 [files.md](./files.md)）。

## 2. 音色快速复刻 — `POST /v1/voice_clone`

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `file_id` | integer | ✓ | — | 待复刻音频的 file_id（来自上一步上传）。 |
| `voice_id` | string | ✓ | — | 自定义音色 ID。长度 8–256，首字符英文字母，可含 `[A-Za-z0-9_-]`，末位不能为 `-` / `_`。 |
| `clone_prompt` | object | ✗ | — | 示例音频对象，含 `prompt_audio`（file_id）与 `prompt_text`。 |
| `text` | string | ✗ | — | 试听文本，≤ 1000 字符；支持 `(laughs)` / `(breath)` 等标签。 |
| `model` | string | ✗ | — | 试听合成模型，speech-2.8-hd / -turbo / 2.6-hd / -turbo 等。 |
| `language_boost` | string | ✗ | — | 同 [speech.md](./speech.md)。 |
| `need_noise_reduction` | boolean | ✗ | `false` | 是否降噪。 |
| `need_volume_normalization` | boolean | ✗ | `false` | 是否音量归一化。 |
| `aigc_watermark` | boolean | ✗ | `false` | 添加音频标识。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_sensitive` | object | 风控检测结果（`type` 同敏感词分类）。 |
| `demo_audio` | string | 试听音频 URL（提供 `text` + `model` 时返回）。 |
| `extra_info` | object | 试听音频元数据（`audio_length` / `audio_sample_rate` / `word_count` / `usage_characters` 等）。 |
| `base_resp` | object | 通用响应封装。 |

### 最小请求

```json
{ "file_id": 123456789, "voice_id": "MyVoice001" }
```

## 3. 音色设计 — `POST /v1/voice_design`

由文本描述生成新音色。试听产生的字符按 ¥0.02 / 万字符计费。

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `prompt` | string | ✓ | — | 音色特征描述。 |
| `preview_text` | string | ✓ | — | 试听文本，≤ 500 字符。 |
| `voice_id` | string | ✗ | 自动生成 | 自定义音色 ID。 |
| `aigc_watermark` | boolean | ✗ | `false` | 在试听输出中添加音频标识。 |

> 性别 / 年龄 / 口音 / 音调等特征都通过 `prompt` 描述传达，**没有单独的离散字段**。

### 响应

```json
{
  "voice_id": "ttv-voice-2025060717322425-xxxxxxxx",
  "trial_audio": "<hex 编码音频>",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 4. 查询可用音色 — `POST /v1/get_voice`

### 请求体

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `voice_type` | string | ✓ | `system` / `voice_cloning` / `voice_generation` / `all` |

> `voice_cloning` 与 `voice_generation` 仅在该音色**首次合成成功后**可查询到。

### 响应

```json
{
  "system_voice": [
    {
      "voice_id": "Chinese (Mandarin)_Reliable_Executive",
      "voice_name": "沉稳高管",
      "description": ["Steady, trustworthy middle-aged male executive voice"],
      "created_time": "1970-01-01"
    }
  ],
  "voice_cloning": [],
  "voice_generation": [],
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 5. 删除音色 — `POST /v1/delete_voice`

### 请求体

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `voice_type` | string | ✓ | `voice_cloning` / `voice_generation` |
| `voice_id` | string | ✓ | 待删除的 ID |

仅支持删除通过克隆或生成 API 创建的音色，**系统音色不可删除**。**删除后该 `voice_id` 不可再使用**。

### 响应

```json
{
  "voice_id": "yanshang11123",
  "created_time": "1728962464",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 错误码

详见 [errors.md](./errors.md)。常见：`2037` 时长不符 / `2038` 未开通复刻 / `2039` voice_id 重复 / `2042` 无权访问 / `2048` prompt 音频过长。

## 参考

- 上传复刻音频：https://platform.minimaxi.com/docs/api-reference/voice-cloning-uploadcloneaudio
- 上传示例音频：https://platform.minimaxi.com/docs/api-reference/voice-cloning-uploadprompt
- 快速复刻：https://platform.minimaxi.com/docs/api-reference/voice-cloning-clone
- 音色设计：https://platform.minimaxi.com/docs/api-reference/voice-design-design
- 查询音色：https://platform.minimaxi.com/docs/api-reference/voice-management-get
- 删除音色：https://platform.minimaxi.com/docs/api-reference/voice-management-delete
- 系统音色 ID 表：https://platform.minimaxi.com/docs/faq/system-voice-id
