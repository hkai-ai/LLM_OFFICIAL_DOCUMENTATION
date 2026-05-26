---
source: https://docs.bigmodel.cn/api-reference/模型-api/视频生成异步
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM Videos · POST /paas/v4/videos/generations（异步）

> 文本 / 图像 / 首尾帧 / 主体参考生视频，按 `model` 与字段切换模式。统一异步任务，立即返回 task_id，由查询接口拉取结果。

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

完整 URL：`POST https://open.bigmodel.cn/api/paas/v4/videos/generations`

## 支持模型

| 模型 ID | 模式 |
| --- | --- |
| `cogvideox-3` | 文本 / 图像 → 视频 |
| `cogvideox-2` | 文本 / 图像 → 视频 |
| `cogvideox-flash` | 快速档（免费） |
| `viduq1-text` | 文生视频 |
| `viduq1-image` | 图生视频 |
| `viduq1-start-end` | 首尾帧 |
| `vidu2-image` | 图生视频 |
| `vidu2-start-end` | 首尾帧 |
| `vidu2-reference` | 参考图 |

## 请求参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 见上表 |
| `prompt` | string | 视模式 | ≤ 512 字符。文生模式必填 |
| `image_url` | string \| array | 视模式 | 图像 URL 或 base64；图生 / 首尾帧 / 参考模式必填 |
| `size` | string | ✗ | 例如 `1920x1080` |
| `fps` | integer | ✗ | `30` / `60` |
| `duration` | integer | ✗ | `5` / `10`（秒） |
| `quality` | string | ✗ | `quality`（质量优先）/ `speed`（速度优先） |
| `with_audio` | boolean | ✗ | 是否生成 AI 音效 |
| `request_id` | string | ✗ | 唯一请求标识 |
| `user_id` | string | ✗ | 终端用户 ID，6–128 字符 |

> `prompt` 与 `image_url` 至少提供一个。具体每个模型支持的 `duration` / `fps` / `size` 组合差异较大，详见对应模型页（CogVideoX-3 / Vidu Q1 / Vidu 2）。

## 响应

```json
{
  "model": "cogvideox-3",
  "id": "task_uuid_string",
  "request_id": "req-1234567890",
  "task_status": "PROCESSING"
}
```

`task_status`：`PROCESSING` / `SUCCESS` / `FAIL`。

## 查询异步结果

使用 `id` 调用 `GET https://open.bigmodel.cn/api/paas/v4/async-result/{id}` 拉取最终视频（详见 https://docs.bigmodel.cn/api-reference/模型-api/查询异步结果）。

## 示例

### 文生视频

```json
{
  "model": "cogvideox-3",
  "prompt": "A cat is playing with a ball.",
  "quality": "quality",
  "with_audio": true,
  "size": "1920x1080",
  "fps": 30
}
```

### 图生视频

```json
{
  "model": "cogvideox-3",
  "image_url": "https://example.com/image.jpg",
  "prompt": "让画面动起来",
  "quality": "quality",
  "size": "1920x1080"
}
```

## 参考

- 视频生成异步：https://docs.bigmodel.cn/api-reference/模型-api/视频生成异步
- 查询异步结果：https://docs.bigmodel.cn/api-reference/模型-api/查询异步结果
- CogVideoX-3 模型页：https://docs.bigmodel.cn/cn/guide/models/video-generation/cogvideox-3
- Vidu 2 模型页：https://docs.bigmodel.cn/cn/guide/models/video-generation/vidu2
- Vidu Q1 模型页：https://docs.bigmodel.cn/cn/guide/models/video-generation/viduq1
- 视频 prompt 工程：https://docs.bigmodel.cn/cn/best-practice/prompt/video-prompt
