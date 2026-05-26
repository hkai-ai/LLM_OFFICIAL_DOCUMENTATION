---
source: https://platform.minimaxi.com/docs/api-reference/file-management-upload
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 文件管理（Files）

| 操作 | 端点 |
| --- | --- |
| 上传 | `POST /v1/files/upload`（multipart/form-data） |
| 列出 | `GET /v1/files/list?purpose=<...>` |
| 检索（含下载链接） | `GET /v1/files/retrieve?file_id=<...>` |
| 删除 | `POST /v1/files/delete` |

## 鉴权

`Authorization: Bearer <API_KEY>`。Base `https://api.minimaxi.com`。

## 文件 `purpose` 枚举

| 值 | 用途 | 支持格式 / 限制 |
| --- | --- | --- |
| `voice_clone` | 音色快速复刻样本 | MP3 / M4A / WAV，时长 10s–5min，≤ 20MB |
| `prompt_audio` | 复刻示例音频（搭配 `text_validation`） | MP3 / M4A / WAV，prompt 音频 < 8 秒 |
| `t2a_async_input` | 异步语音合成的文本文件 | txt / zip |
| `t2a_async` | 异步语音合成生成的音频 | 系统返回 |
| `video_generation` | 视频生成产出 | 系统返回 |

> 当前官方仅明确文档化以上 `purpose`。其他 chat / image / retrieval / fine-tune 等场景目前并未通过 Files API 暴露。

## 1. 上传 — `POST /v1/files/upload`

### 请求（multipart/form-data）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `purpose` | string | ✓ | 见 §文件 purpose 枚举 |
| `file` | binary | ✓ | 文件二进制 |

### 响应

```json
{
  "file": {
    "file_id": 123456789,
    "bytes": 5896337,
    "created_at": 1700469398,
    "filename": "example.docx",
    "purpose": "t2a_async_input"
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 2. 列出 — `GET /v1/files/list`

### 查询参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `purpose` | string | ✓ | 必须传入；按文件用途分类返回 |

### 响应

```json
{
  "files": [
    {
      "file_id": 123456,
      "bytes": 5896337,
      "created_at": 1699964873,
      "filename": "example.tar",
      "purpose": "t2a_async_input"
    }
  ],
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 3. 检索 — `GET /v1/files/retrieve?file_id=<id>`

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `file.file_id` | integer | 同请求 |
| `file.bytes` | integer | 文件大小（字节） |
| `file.created_at` | integer | Unix 时间戳 |
| `file.filename` | string | 文件名 |
| `file.purpose` | string | purpose |
| `file.download_url` | string | 下载 URL，**有效期 1 小时** |
| `base_resp` | object | 通用响应封装 |

```json
{
  "file": {
    "file_id": 12345678,
    "bytes": 5896337,
    "created_at": 1700469398,
    "filename": "output_aigc.mp4",
    "purpose": "video_generation",
    "download_url": "https://..."
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 4. 删除 — `POST /v1/files/delete`

### 请求体

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file_id` | integer | ✓ | 待删除 file_id |
| `purpose` | string | ✓ | 必传；与文件实际 purpose 一致：`voice_clone` / `prompt_audio` / `t2a_async` / `t2a_async_input` / `video_generation` |

### 响应

```json
{
  "file_id": 12345,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 单文件与账户限制

- 单文件大小：voice 类 ≤ 20MB；其他类目无单独官方上限。
- 账户总容量：100 GB。

## 错误码

详见 [errors.md](./errors.md)。

## 参考

- 上传：https://platform.minimaxi.com/docs/api-reference/file-management-upload
- 列表：https://platform.minimaxi.com/docs/api-reference/file-management-list
- 检索：https://platform.minimaxi.com/docs/api-reference/file-management-retrieve
- 文件内容下载（视频专用）：https://platform.minimaxi.com/docs/api-reference/file-management-retrieve-content
- 删除：https://platform.minimaxi.com/docs/api-reference/file-management-delete
