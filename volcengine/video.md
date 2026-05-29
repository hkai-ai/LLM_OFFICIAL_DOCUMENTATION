---
source: https://www.volcengine.com/docs/82379/1520757
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 视频生成（Video Generation）· /api/v3/contents/generations/tasks

> 调用 Seedance 系列模型（`doubao-seedance-2-0-260128` / `2-0-fast` / `1.5-pro` / `1.0-pro` / `1.0-pro-fast`）进行文生视频、图生视频（首帧 / 首尾帧）、多模态参考生视频。创建为**异步任务**，需轮询查询任务状态获取视频 URL。

| 操作 | 方法 | 路径 |
| --- | --- | --- |
| 创建视频生成任务 | POST | `/api/v3/contents/generations/tasks` |
| 查询视频生成任务 | GET | `/api/v3/contents/generations/tasks/{id}` |
| 查询任务列表 | GET | `/api/v3/contents/generations/tasks` |
| 取消 / 删除任务 | DELETE | `/api/v3/contents/generations/tasks/{id}` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

> 开通 Seedance 2.0 / 2.0 fast 需账户余额 ≥200 元或已购资源包。视频 URL 有效期 24 小时，请及时下载或转存。任务记录仅保留 7 天。

## 创建任务 · 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID 或 Endpoint ID。 |
| `content` | object[] | ✓ | — | 输入信息，支持文本 / 图片 / 视频 / 音频 / 样片任务 ID。见 [`content[]`](#content)。 |
| `callback_url` | string | ✗ | — | 任务状态变化回调地址（POST 推送，内容同查询任务返回体）。状态 `queued`/`running`/`succeeded`/`failed`/`expired`；发送失败重试 3 次。 |
| `return_last_frame` | boolean | ✗ | `false` | `true` 返回视频尾帧图（png，无水印，同视频宽高），可经查询接口获取，用于连续视频生成。 |
| `service_tier` | string | ✗ | `default` | `default`（在线推理）/ `flex`（离线推理，价格 50%，TPD 配额更高）。Seedance 2.0 系列仅支持在线推理，不可配置。 |
| `execution_expires_after` | integer | ✗ | `172800` | 任务超时（秒），从 `created_at` 起算，范围 `[3600, 259200]`。超时标记 `expired`。 |
| `generate_audio` | boolean | ✗ | `true` | 视频是否含同步声音。仅 Seedance 2.0 系列、1.5 pro 支持。有声视频为单声道。 |
| `draft` | boolean | ✗ | `false` | 样片模式（480p 预览，token 更少）。仅 Seedance 1.5 pro 支持，不支持尾帧 / 离线推理。 |
| `tools` | object[] | ✗ | — | `tools.type=web_search`（联网搜索），仅 Seedance 2.0 系列支持。次数见查询返回 `usage.tool_usage.web_search`。 |
| `safety_identifier` | string | ✗ | — | 终端用户唯一标识（英文，≤64 字符），建议哈希处理。 |
| `priority` | integer | ✗ | `0` | 执行优先级 `0~9`，越大越优先（同 Endpoint 内插队）。仅 Seedance 2.0 系列支持；离线推理不支持。 |
| `resolution` | string | ✗ | 因模型而异 | `480p` / `720p` / `1080p`（Seedance 2.0 fast 不支持 1080p）。2.0 系列 / 1.5 pro 默认 `720p`；1.0 pro / fast 默认 `1080p`。 |
| `ratio` | string | ✗ | 因模型而异 | `16:9` / `4:3` / `1:1` / `3:4` / `9:16` / `21:9` / `adaptive`。2.0 系列 / 1.5 pro 默认 `adaptive`。 |
| `duration` | integer | ✗ | `5` | 时长（整数秒）。1.0 pro/fast `[2,12]`；1.5 pro `[4,12]` 或 `-1`；2.0 系列 `[4,15]` 或 `-1`（`-1` 由模型自选）。与 `frames` 二选一。 |
| `frames` | integer | ✗ | — | 帧数 = 时长 × 24，范围 `[29, 289]` 满足 `25+4n`。优先级高于 `duration`。Seedance 2.0 系列、1.5 pro 暂不支持。 |
| `seed` | integer | ✗ | `-1` | 随机种子，`[-1, 2^32-1]`。相同请求 + 相同 seed 生成类似结果（不保证完全一致）。 |
| `camera_fixed` | boolean | ✗ | `false` | 是否固定摄像头（追加提示词，效果不保证）。参考图场景、Seedance 2.0 系列暂不支持。 |
| `watermark` | boolean | ✗ | `false` | `true` 在右下角加「AI生成」水印。 |

> `resolution` / `ratio` / `duration` / `frames` / `seed` / `camera_fixed` / `watermark` 支持两种传入方式：推荐在 request body 直接传（强校验）；旧方式在文本提示词后追加 `--rs 720p --rt 16:9 --dur 5 --seed 11 --cf false --wm true`（弱校验）。

### `content[]`

由 `type` 区分。支持组合：纯文本、文本+图片、文本+视频、文本+图片+音频、文本+图片+视频、文本+视频+音频、文本+图片+视频+音频。

#### 文本（`type` = `text`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `text`。 |
| `text` | string | ✓ | 提示词。所有模型支持中英文；2.0 / 2.0 fast 额外支持日 / 印尼 / 西 / 葡语。建议中文 ≤500 字、英文 ≤1000 词。 |

#### 图片（`type` = `image_url`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `image_url`。 |
| `image_url.url` | string | ✓ | 图片 URL / Base64（`data:image/<格式小写>;base64,<编码>`）/ 素材 ID（`asset://<ASSET_ID>`）。 |
| `role` | string | 条件必填 | `first_frame`（首帧，或不填）/ `last_frame`（尾帧）/ `reference_image`（参考图，2.0 系列 1~9 张）。三种场景互斥。 |

图片要求：格式 jpeg/png/webp/bmp/tiff/gif（1.5 pro、2.0 系列另支持 heic/heif）；宽高比 `(0.4, 2.5)`；宽高 `(300, 6000)` px；≤30 MB；请求体 ≤64 MB。

#### 视频（`type` = `video_url`，仅 Seedance 2.0 系列）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `video_url`。 |
| `video_url.url` | string | ✓ | 视频 URL 或素材 ID。 |
| `role` | string | 条件必填 | 当前仅 `reference_video`。 |

视频要求：mp4/mov（H.264/H.265，音频 AAC/MP3）；480p/720p/1080p；单个 `[2,15]`s，最多 3 个、总时长 ≤15s；宽高比 `[0.4, 2.5]`、总像素 `[409600, 2086876]`；≤50 MB；帧率 `[24, 60]`。

#### 音频（`type` = `audio_url`，仅 Seedance 2.0 系列）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `audio_url`。 |
| `audio_url.url` | string | ✓ | 音频 URL / Base64 / 素材 ID。 |
| `role` | string | 条件必填 | 当前仅 `reference_audio`。不可单独输入音频，须至少含 1 个参考视频或图片。 |

音频要求：wav/mp3；单个 `[2,15]`s，最多 3 段、总时长 ≤15s；≤15 MB。

#### 样片（`type` = `draft_task`，仅 Seedance 1.5 pro）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `draft_task`。 |
| `draft_task.id` | string | ✓ | 样片任务 ID。平台复用 Draft 的输入（model/text/image_url/generate_audio/seed/ratio/duration/camera_fixed）生成正式视频。 |

### 创建任务响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 视频生成任务 ID（保存 7 天）。异步接口，需轮询查询任务状态。 |

## 查询任务 · GET /api/v3/contents/generations/tasks/{id}

路径参数 `id`（必填）。返回 task 对象：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 任务 ID。 |
| `model` | string | 模型名称-版本。 |
| `status` | string | `queued` / `running` / `cancelled` / `succeeded` / `failed` / `expired`。 |
| `error` | object \| null | 失败时返回 `code` / `message`；成功为 `null`。 |
| `created_at` / `updated_at` | integer | 创建 / 更新 Unix 时间戳（秒）。 |
| `content` | object | `video_url`（mp4，24h 有效）、`last_frame_url`（`return_last_frame=true` 时返回）。 |
| `seed` | integer | 使用的种子值。 |
| `resolution` / `ratio` | string | 分辨率 / 宽高比。 |
| `duration` \| `frames` | integer | 时长（秒）或帧数（二者只返回其一，取决于创建时传哪个）。 |
| `framespersecond` | integer | 帧率。 |
| `generate_audio` | boolean | 是否含同步声音（2.0 系列、1.5 pro 返回）。 |
| `tools` | object[] | 实际使用的工具（如 `web_search`），未使用不返回。 |
| `safety_identifier` | string | 原样返回创建时所传。 |
| `priority` | integer | 执行优先级。 |
| `draft` / `draft_task_id` | boolean / string | 是否 Draft 视频 / 基于的 Draft 任务 ID（1.5 pro）。 |
| `service_tier` | string | 实际服务等级。 |
| `execution_expires_after` | integer | 超时阈值（秒）。 |
| `usage` | object | `completion_tokens`（生成视频 token，计费依据；2.0 系列有最低用量限制）、`total_tokens`（= completion_tokens，不计输入）、`tool_usage.web_search`。 |

## 查询任务列表 · GET /api/v3/contents/generations/tasks

查询参数（Query String）：

| 参数 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `page_num` | integer \| null | `1` | 页码，`[1, 500]`。 |
| `page_size` | integer \| null | `20` | 每页数量，`[1, 500]`。 |
| `filter.status` | string \| null | — | 按状态过滤：`queued`/`running`/`cancelled`/`succeeded`/`failed`。 |
| `filter.task_ids` | string[] \| null | — | 按任务 ID 精确搜索，重复参数名传多个：`filter.task_ids=id1&filter.task_ids=id2`。 |
| `filter.model` | string \| null | — | 按推理接入点 ID（Endpoint ID）精确搜索。 |
| `filter.service_tier` | string \| null | `default` | `default` / `flex`。 |

响应：`items` 为 task 对象数组（字段同查询任务）。

## 取消 / 删除任务 · DELETE /api/v3/contents/generations/tasks/{id}

路径参数 `id`（必填）。本接口无返回参数。按当前状态执行不同操作：

| 当前状态 | 是否支持 DELETE | 含义 |
| --- | --- | --- |
| `queued` | 是 | 取消排队，状态变为 `cancelled`。 |
| `running` | 否 | — |
| `succeeded` | 是 | 删除任务记录（后续不可查询）。 |
| `failed` | 是 | 删除任务记录。 |
| `cancelled` | 否 | — |
| `expired` | 是 | 删除任务记录。 |

## 示例

### 创建任务（文生视频，curl）

```bash
curl -X POST "https://ark.cn-beijing.volces.com/api/v3/contents/generations/tasks" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "doubao-seedance-2-0-260128",
    "content": [{"type": "text", "text": "小猫对着镜头打哈欠"}],
    "resolution": "720p",
    "ratio": "16:9",
    "duration": 5,
    "watermark": true
  }'
```

### 查询任务响应

```json
{
  "id": "cgt-2026****-****",
  "model": "doubao-seedance-2-0-260128",
  "status": "succeeded",
  "content": { "video_url": "https://.../xxx.mp4" },
  "usage": { "completion_tokens": 108900, "total_tokens": 108900 },
  "created_at": 1779348818,
  "updated_at": 1779348874,
  "seed": 78674,
  "resolution": "720p",
  "ratio": "16:9",
  "duration": 5,
  "framespersecond": 24,
  "service_tier": "default",
  "execution_expires_after": 172800,
  "generate_audio": true,
  "draft": false,
  "priority": 0
}
```

## 参考

- 创建视频生成任务：https://www.volcengine.com/docs/82379/1520757
- 查询视频生成任务：https://www.volcengine.com/docs/82379/1521309
- 查询任务列表：https://www.volcengine.com/docs/82379/1521675
- 取消 / 删除任务：https://www.volcengine.com/docs/82379/1521720
- Seedance 提示词指南：https://www.volcengine.com/docs/82379/2168087
