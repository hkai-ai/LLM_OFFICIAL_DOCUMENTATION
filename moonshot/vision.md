---
source: https://platform.kimi.com/docs/guide/use-kimi-vision-model
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · 视觉模型（Vision）

通过 [chat-completions.md](./chat-completions.md) 的 `messages[].content` 数组传入图片 / 视频实现多模态对话。

## 支持模型

| 模型 ID | 图片 | 视频 | 备注 |
| --- | --- | --- | --- |
| `kimi-k2.6` | ✓ | ✓ | 最新，支持视频理解 |
| `kimi-k2.5` | ✓ | ✗ | 仅图片 |
| `moonshot-v1-8k-vision-preview` | ✓ | ✗ | 8K 上下文版 |
| `moonshot-v1-32k-vision-preview` | ✓ | ✗ | 32K 上下文版 |
| `moonshot-v1-128k-vision-preview` | ✓ | ✗ | 128K 上下文版 |

## `image_url` / `video_url` URL 格式

> Kimi **不支持公网 URL 直接传入**，只能 base64 或文件引用。

| 形式 | 说明 |
| --- | --- |
| `data:image/<mime>;base64,...` | base64 data URI |
| `ms://<file_id>` | 引用通过 [files.md](./files.md) `purpose=image` / `purpose=video` 上传后获得的 `file_id` |

## `messages[].content[]` 多模态块

```json
{
  "role": "user",
  "content": [
    {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,..."}},
    {"type": "video_url", "video_url": {"url": "ms://file-vid-123"}},
    {"type": "text", "text": "请描述图片和视频内容"}
  ]
}
```

## 格式与限制

| 项 | 范围 |
| --- | --- |
| 图片格式 | PNG / JPEG / WebP / GIF |
| 视频格式 | MP4 / MPEG / MOV / AVI / FLV / MPG / WebM / WMV / 3GPP |
| 图片推荐分辨率 | ≤ 4096×2160（4K） |
| 视频推荐分辨率 | ≤ 2048×1080（2K） |
| 单请求 body | ≤ 100MB |

## Token 计算

视觉模型按 **分辨率 + 视频关键帧数量** 动态计算 token。建议在大规模请求前用 [estimate-tokens.md](./estimate-tokens.md) 估算成本。

> 计费：`kimi-k2.6` / `k2.5` 走对应模型主价；`moonshot-v1-*-vision-preview` 与同上下文窗口的纯文本 `moonshot-v1-*` 同价。

## 采样参数注意

`kimi-k2.6` / `k2.5` 在视觉模式下：

- `temperature` 固定 `1.0`（思考模式开启）或 `0.6`（思考模式关闭）。
- `top_p` 固定 `0.95`。
- `max_tokens` 默认 `32,768`。

## 示例

### 单图问答

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "image_url", "image_url": {"url": "ms://file-img-001"}},
        {"type": "text", "text": "这是什么？"}
      ]
    }
  ]
}
```

### 视频理解（k2.6）

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "video_url", "video_url": {"url": "ms://file-vid-002"}},
        {"type": "text", "text": "用一句话总结视频内容"}
      ]
    }
  ]
}
```

## 参考

- Vision Guide：https://platform.kimi.com/docs/guide/use-kimi-vision-model
- 文件上传（image / video）：[files.md](./files.md) + https://platform.kimi.com/docs/api/files-upload
- Token 估算：[estimate-tokens.md](./estimate-tokens.md)
