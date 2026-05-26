---
source: https://platform.minimaxi.com/docs/api-reference/video-generation-t2v
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 视频生成（Video Generation）

所有视频生成请求统一打到 `POST https://api.minimaxi.com/v1/video_generation`；通过 `model` 与不同参数区分 文生 / 图生 / 首尾帧 / 主体参考 四种模式。

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

## 端点速查

| 用途 | 模型 / 参数特征 | 文档 |
| --- | --- | --- |
| 文生视频（t2v） | `MiniMax-Hailuo-2.3` / `T2V-01-Director` / `T2V-01` 等，必填 `prompt` | https://platform.minimaxi.com/docs/api-reference/video-generation-t2v |
| 图生视频（i2v） | `I2V-01-*` / `MiniMax-Hailuo-*`，必填 `first_frame_image` | https://platform.minimaxi.com/docs/api-reference/video-generation-i2v |
| 首尾帧生成（fl2v） | `MiniMax-Hailuo-02` 等，必填 `first_frame_image` + `last_frame_image` | https://platform.minimaxi.com/docs/api-reference/video-generation-fl2v |
| 主体参考（s2v） | `S2V-01` 等，必填 `subject_reference` | https://platform.minimaxi.com/docs/api-reference/video-generation-s2v |
| 任务查询 | `GET /v1/query/video_generation?task_id=` | https://platform.minimaxi.com/docs/api-reference/video-generation-query |
| 视频下载 | `GET /v1/files/retrieve?file_id=` | https://platform.minimaxi.com/docs/api-reference/video-generation-download |
| 视频 Agent（模板） | `POST /v1/video-agent/create` + `GET /v1/video-agent/query` | https://platform.minimaxi.com/docs/api-reference/video-agent-create |

## 1. 创建生成任务 — `POST /v1/video_generation`

### 公共请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 见 [models.md §视频生成](./models.md)。 |
| `prompt` | string | 视模式 | — | 视频描述，≤ 2000 字符。支持运镜指令如 `[左移]` / `[推进]` / `[固定]` 等 15 种。 |
| `duration` | integer | ✗ | `6` | 秒数，受 `model` × `resolution` 限制。 |
| `resolution` | string | ✗ | 依模型默认 | `512P` / `720P` / `768P` / `1080P`。 |
| `prompt_optimizer` | boolean | ✗ | `true` | 是否自动优化 prompt。 |
| `fast_pretreatment` | boolean | ✗ | `false` | 缩短优化耗时。 |
| `callback_url` | string | ✗ | — | 异步状态回调地址。 |
| `aigc_watermark` | boolean | ✗ | `false` | 添加水印。 |

### 模式专属参数

| 模式 | 必填参数 | 说明 |
| --- | --- | --- |
| t2v | `prompt` | 仅文本。 |
| i2v | `first_frame_image` | URL 或 base64 data URI；JPG/PNG/WebP；≤ 20MB；宽高比 `[2:5, 5:2]`；短边 ≥ 300px。 |
| fl2v | `first_frame_image` + `last_frame_image` | 两张图片同 i2v 要求。 |
| s2v | `subject_reference: [{ type: "character", image: ["<url>"] }]` | 人物图像数组，要求同 i2v。 |

### 模型 × 时长 × 分辨率支持矩阵

| 模型 | 6s | 10s | 可选分辨率 |
| --- | --- | --- | --- |
| `MiniMax-Hailuo-2.3` | ✓ | ✓（仅 768P） | 768P（默认） / 1080P |
| `MiniMax-Hailuo-2.3-Fast` | ✓ | ✓（仅 768P） | 768P / 1080P |
| `MiniMax-Hailuo-02` | ✓ | ✓（仅 512P / 768P） | 512P / 768P / 1080P |
| `T2V-01-Director` | ✓ | ✗ | 720P |
| `T2V-01` | ✓ | ✗ | 720P |
| `I2V-01-Director` | ✓ | ✗ | 720P |
| `I2V-01-live` | ✓ | ✗ | 720P |
| `I2V-01` | ✓ | ✗ | 720P |
| `S2V-01` | ✓ | ✗ | 720P |

### 响应

```json
{
  "task_id": "106916112212032",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

### 最小请求

```json
{ "model": "MiniMax-Hailuo-2.3", "prompt": "A man picks up a book, then reads it." }
```

```json
{
  "model": "MiniMax-Hailuo-02",
  "first_frame_image": "https://example.com/start.jpg",
  "last_frame_image":  "https://example.com/end.jpg",
  "prompt": "A girl grows up",
  "duration": 6,
  "resolution": "1080P"
}
```

```json
{
  "model": "S2V-01",
  "prompt": "人物向镜头跑来并微笑眨眼",
  "subject_reference": [{ "type": "character", "image": ["https://example.com/person.jpg"] }]
}
```

## 2. 任务状态查询 — `GET /v1/query/video_generation?task_id=<id>`

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `task_id` | string | 同请求。 |
| `status` | string | `Preparing` / `Queueing` / `Processing` / `Success` / `Fail`。 |
| `file_id` | string | 成功时返回的视频 file_id。 |
| `video_width` | integer | 成功时视频宽度（像素）。 |
| `video_height` | integer | 成功时视频高度（像素）。 |
| `base_resp` | object | 通用响应封装。 |

### 示例

```json
{
  "task_id": "176843862716480",
  "status": "Success",
  "file_id": "176844028768320",
  "video_width": 1920,
  "video_height": 1080,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 3. 视频下载 — `GET /v1/files/retrieve?file_id=<id>`

> 与文件管理共用 retrieve 端点；返回的 `file.download_url` 有效期 **1 小时**。

```json
{
  "file": {
    "file_id": 12345,
    "bytes": 5242880,
    "created_at": 1700469398,
    "filename": "output_aigc.mp4",
    "purpose": "video_generation",
    "download_url": "https://..."
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 4. 视频 Agent（模板）

| 端点 | 用途 |
| --- | --- |
| `POST /v1/video-agent/create` | 通过模板创建复合视频任务 |
| `GET /v1/video-agent/query` | 查询模板任务状态 |

模板列表见 https://platform.minimaxi.com/docs/faq/video-agent-templates。

## 错误码与计费

- 错误码：见 [errors.md](./errors.md)。常见 `1008` 余额不足 / `1026` prompt 涉敏。
- 价格：见 [pricing.md §视频生成](./pricing.md)。

## 参考

- t2v：https://platform.minimaxi.com/docs/api-reference/video-generation-t2v
- i2v：https://platform.minimaxi.com/docs/api-reference/video-generation-i2v
- fl2v：https://platform.minimaxi.com/docs/api-reference/video-generation-fl2v
- s2v：https://platform.minimaxi.com/docs/api-reference/video-generation-s2v
- 任务查询：https://platform.minimaxi.com/docs/api-reference/video-generation-query
- 下载：https://platform.minimaxi.com/docs/api-reference/video-generation-download
- 视频 Agent：https://platform.minimaxi.com/docs/api-reference/video-agent-create
