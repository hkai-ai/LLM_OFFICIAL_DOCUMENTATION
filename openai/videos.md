---
source: https://developers.openai.com/api/docs/guides/video-generation
fetched_at: 2026-05-26
api_version: v1
---

# Videos · /v1/videos

Sora 2 视频生成 API。文字 / 图片输入生成最长 20 秒带音频的视频；可在生成完成的视频上做扩展（extend）、编辑（edit）、混剪（remix），也可创建可复用的「角色」（character）。

| 资源 | 路径前缀 |
| --- | --- |
| Videos | `/v1/videos` |
| Characters | `/v1/videos/characters` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | 默认 `application/json`；带 `input_reference` 图像或上传视频时为 `multipart/form-data`。 |

## 1. Create video · POST /v1/videos

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | enum | ✓ | `sora-2` / `sora-2-pro`。 |
| `prompt` | string | ✓ | 自然语言描述。 |
| `size` | enum | ✗ | `1280x720` / `720x1280` / `1024x1792` / `1792x1024` / `1920x1080` / `1080x1920`。 |
| `seconds` | enum | ✗ | `4` / `8` / `12` / `16` / `20`。 |
| `seed` | integer | ✗ | 可复现采样。 |
| `input_reference` | file | ✗ | JPEG / PNG / WebP，分辨率必须与 `size` 一致；作为首帧参考。 |
| `characters` | `array<{id}>` | ✗ | 引用已上传 character。 |

> 创建后立即返回 `status: queued`；客户端轮询 retrieve 直至 `completed` 再下载内容。

### 最小请求

```bash
curl https://api.openai.com/v1/videos \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora-2",
    "prompt": "A cat playing piano in the rain, cinematic.",
    "size": "1280x720",
    "seconds": 8
  }'
```

## 2. Video 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `video_...`。 |
| `object` | string | 固定 `video`。 |
| `status` | enum | `queued` / `in_progress` / `completed` / `failed`。 |
| `progress` | integer | `0`–`100`，进度百分比。 |
| `model` | string | 实际使用的模型。 |
| `size` | string | — |
| `seconds` | `string \| integer` | — |
| `prompt` | string | — |
| `error` | `object \| null` | `{ "message": "..." }`。 |
| `created_at` | integer | Unix 秒。 |
| `completed_at` | integer | Unix 秒，终态填写。 |
| `remixed_from_video_id` | string | 仅 remix 产物有。 |

## 3. Retrieve / List / Delete

| 动作 | METHOD | PATH | 说明 |
| --- | --- | --- | --- |
| Retrieve | GET | `/v1/videos/{video_id}` | 返回 Video 对象。 |
| List | GET | `/v1/videos` | query：`after` / `before` / `limit` / `order`。 |
| Delete | DELETE | `/v1/videos/{video_id}` | 永久删除。 |

## 4. Content · GET /v1/videos/{video_id}/content

| Query | 说明 |
| --- | --- |
| `variant` | `video`（默认 MP4 字节流） / `thumbnail` / `spritesheet`。 |

返回对应字节流，`Content-Type` 与文件类型一致。

## 5. Extend · POST /v1/videos/{video_id}/extend

接龙生成；最多 **6 次扩展，总长不超过 120 秒**。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `prompt` | string | ✓ | 续写描述。 |
| `seconds` | enum | ✗ | `4`–`20`。 |
| `seed` | integer | ✗ | — |

> 旧路径 `POST /v1/videos/extensions`（body 含 `video.id`）也接受同语义请求。

## 6. Edit · POST /v1/videos/{video_id}/edits

按 prompt 修改指定时段或风格。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `prompt` | string | ✓ | 修改描述。 |
| `start_seconds` | number | ✗ | 编辑起点。 |
| `end_seconds` | number | ✗ | 终点。 |
| `model` | enum | 上传新视频时 ✓ | `sora-2` / `sora-2-pro`。 |

也可走 `POST /v1/videos/edits` body 形式 `{ "video": {"id":"video_..."} \| <multipart file> }`。

## 7. Remix · POST /v1/videos/{video_id}/remix

基于原视频换风格 / 角度生成新视频；返回的新 Video 对象 `remixed_from_video_id` 指向源。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `prompt` | string | ✓ | remix 描述。 |
| `seed` | integer | ✗ | — |

## 8. Characters

可复用角色：上传一段 2–4 秒 / 720p–1080p 的 MP4，得到 `character_id`，后续 create / extend / remix 通过 `characters: [{"id":"char_..."}]` 引用。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/videos/characters` |
| Retrieve | GET | `/v1/videos/characters/{character_id}` |

### Create body（multipart）

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `video` | ✓ | MP4 字节流，2–4 秒。 |
| `name` | ✓ | 角色名。 |

### Character 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `char_...`。 |
| `object` | string | 固定 `video.character`。 |
| `name` | string | — |
| `created_at` | integer | — |

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | `size` 与 `input_reference` 分辨率不一致 / `seconds` 不在枚举内 / 超出扩展上限 |
| `404` | `not_found_error` | video / character 不存在 |
| `429` | `rate_limit_error` | 视频生成独立配额 |

## 计费

按模型 + 时长 + 分辨率计费；详见 [pricing.md](./pricing.md) §Videos。

## 参考

- 指南：<https://developers.openai.com/api/docs/guides/video-generation>
- API：<https://developers.openai.com/api/reference/resources/videos>
- Sora 2 模型：<https://developers.openai.com/api/docs/models/sora-2>
- Sora 2 Pro：<https://developers.openai.com/api/docs/models/sora-2-pro>
- Prompting 指南：<https://developers.openai.com/cookbook/examples/sora/sora2_prompting_guide>
