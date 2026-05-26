---
source: https://developers.openai.com/api/reference/resources/images
fetched_at: 2026-05-19
api_version: N/A
---

# Images · /v1/images/{generations,edits,variations}

> 文生图、图像编辑、图像变体三个端点。新一代 `gpt-image-1` / `gpt-image-1.5` / `gpt-image-1-mini` 与遗留 `dall-e-3` / `dall-e-2` 并存。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |
| `Content-Type` | ✓ | generations 用 `application/json`；edits / variations 用 `multipart/form-data`。 |

---

## POST /v1/images/generations

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `prompt` | string | ✓ | — | 文字描述。`gpt-image-*` 最长 32000 字符；`dall-e-2` 1000；`dall-e-3` 4000。 |
| `model` | string | ✗ | `dall-e-2` | `gpt-image-1.5` / `gpt-image-1` / `gpt-image-1-mini` / `dall-e-3` / `dall-e-2`。 |
| `n` | integer | ✗ | `1` | 1–10。`dall-e-3` 仅支持 1。 |
| `size` | string | ✗ | 模型相关 | `gpt-image-*`：`auto` / `1024x1024` / `1536x1024` / `1024x1536`；`dall-e-3`：`1024x1024` / `1024x1792` / `1792x1024`；`dall-e-2`：`256x256` / `512x512` / `1024x1024`。 |
| `quality` | string | ✗ | `auto` | `gpt-image-*`：`low` / `medium` / `high` / `auto`；`dall-e-3`：`standard` / `hd`。 |
| `style` | string | ✗ | — | 仅 `dall-e-3`：`vivid` / `natural`。 |
| `response_format` | string | ✗ | `url` | `url` / `b64_json`（仅 dall-e-*；`gpt-image-*` 始终返回 base64）。 |
| `output_format` | string | ✗ | `png` | 仅 `gpt-image-*`：`png` / `jpeg` / `webp`。 |
| `output_compression` | integer | ✗ | `100` | 仅 `gpt-image-*` + `jpeg`/`webp`：0–100。 |
| `background` | string | ✗ | `auto` | 仅 `gpt-image-*`：`transparent` / `opaque` / `auto`。 |
| `moderation` | string | ✗ | `auto` | 仅 `gpt-image-*`：`low` / `auto`。 |
| `partial_images` | integer | ✗ | — | 仅 `gpt-image-*`：流式时返回的中间帧数（0–3）。 |
| `stream` | boolean | ✗ | `false` | 仅 `gpt-image-*`：SSE 流式生成。 |
| `user` | string | ✗ | — | 终端用户标识。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `created` | integer | Unix 时间戳。 |
| `data[]` | array | 生成结果。 |
| `data[].url` | string | URL（dall-e-* + `response_format: url`，60 分钟有效）。 |
| `data[].b64_json` | string | base64 图像数据。 |
| `data[].revised_prompt` | string | 仅 `dall-e-3`：模型改写后的 prompt。 |
| `output_format` | string | 实际格式。 |
| `quality` | string | 实际质量。 |
| `size` | string | 实际尺寸。 |
| `background` | string | 实际背景。 |
| `usage` | object | 仅 `gpt-image-*`：token 使用统计。 |

---

## POST /v1/images/edits

请求体为 `multipart/form-data`。

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `image` / `image[]` | file \| array&lt;file&gt; | ✓ | — | `dall-e-2`：单张正方形 PNG ≤ 4 MB；`gpt-image-*`：最多 16 张参考图，可传 `file_id` 或 `image_url`。 |
| `prompt` | string | ✓ | — | 编辑描述，1–32000 字符（dall-e-2 ≤ 1000）。 |
| `mask` | file \| object | ✗ | — | 遮罩 PNG（与 `image` 同尺寸，透明区域为编辑区）；或 `{ file_id, image_url }`。 |
| `model` | string | ✗ | `dall-e-2` | `gpt-image-1.5` / `gpt-image-1` / `gpt-image-1-mini` / `chatgpt-image-latest` / `dall-e-2`。 |
| `n` | integer | ✗ | `1` | 1–10。 |
| `size` | string | ✗ | `auto` | 同 generations 各模型范围。 |
| `response_format` | string | ✗ | `url` | `url` / `b64_json`（仅 `dall-e-2`）。 |
| `output_format` | string | ✗ | — | 仅 `gpt-image-*`：`png` / `jpeg` / `webp`。 |
| `output_compression` | integer | ✗ | — | 仅 `gpt-image-*` + `jpeg`/`webp`：0–100。 |
| `quality` | string | ✗ | `auto` | `low` / `medium` / `high` / `auto`。 |
| `background` | string | ✗ | — | 仅 `gpt-image-*`：`transparent` / `opaque` / `auto`。 |
| `input_fidelity` | string | ✗ | — | 仅 `gpt-image-*`：`high` / `low`，控制对原图的保真度。 |
| `moderation` | string | ✗ | — | 仅 `gpt-image-*`：`low` / `auto`。 |
| `partial_images` | integer | ✗ | — | 仅 `gpt-image-*`：0–3。 |
| `stream` | boolean | ✗ | `false` | 仅 `gpt-image-*`：流式。 |
| `user` | string | ✗ | — | 终端用户标识。 |

### 响应

结构同 generations 的 `ImagesResponse`：`{ created, data[], output_format, quality, size, background, usage }`。

---

## POST /v1/images/variations

请求体为 `multipart/form-data`。**仅 `dall-e-2` 支持**。

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `image` | file | ✓ | — | 正方形 PNG ≤ 4 MB。 |
| `model` | string | ✗ | `dall-e-2` | 当前仅 `dall-e-2`。 |
| `n` | integer | ✗ | `1` | 1–10。 |
| `response_format` | string | ✗ | `url` | `url` / `b64_json`。 |
| `size` | string | ✗ | `1024x1024` | `256x256` / `512x512` / `1024x1024`。 |
| `user` | string | ✗ | — | 终端用户标识。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `created` | integer | Unix 时间戳。 |
| `data[].url` | string | URL（60 分钟有效）。 |
| `data[].b64_json` | string | base64 图像（`response_format: b64_json`）。 |

---

## 示例

### generations 最小请求

```json
{
  "model": "gpt-image-1",
  "prompt": "A studio photograph of a cat wearing tiny round sunglasses",
  "size": "1024x1024",
  "quality": "high"
}
```

### edits（curl）

```bash
curl https://api.openai.com/v1/images/edits \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F model="gpt-image-1" \
  -F image[]="@input.png" \
  -F mask="@mask.png" \
  -F prompt="Replace the sky with a sunset"
```

## 错误码

| HTTP | error.type | 触发原因 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 不支持的尺寸 / 模型组合，prompt 超长，文件非 PNG 等。 |
| `400` | `content_policy_violation` | prompt 或图像违反 usage policy。 |
| `401` | `authentication_error` | API key 无效。 |
| `429` | `rate_limit_error` | 限流。 |

## 参考

- generations：<https://developers.openai.com/api/reference/resources/images/methods/generate>
- edits：<https://developers.openai.com/api/reference/resources/images/methods/edit>
- variations：<https://developers.openai.com/api/reference/resources/images/methods/create_variation>
- 上级目录：<https://developers.openai.com/api/reference/resources/images>
