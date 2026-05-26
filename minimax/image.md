---
source: https://platform.minimaxi.com/docs/api-reference/image-generation-t2i
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 图像生成（Image Generation）

文生图与图生图共用端点 `POST /v1/image_generation`；图生图通过 `subject_reference` 字段传入参考图。

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

完整 URL：`POST https://api.minimaxi.com/v1/image_generation`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `image-01` / `image-01-live` |
| `prompt` | string | ✓ | — | 图像描述，≤ 1500 字符 |
| `subject_reference` | array | ✗ | — | 图生图人物参考；结构见下 |
| `style` | object | ✗ | — | 风格设置；**仅 `image-01-live` 有效** |
| `aspect_ratio` | string | ✗ | `1:1` | `1:1` / `16:9` / `4:3` / `3:2` / `2:3` / `3:4` / `9:16` / `21:9` |
| `width` | integer | ✗ | — | 宽（像素），范围 `[512, 2048]`，8 的倍数。**仅 `image-01` 有效** |
| `height` | integer | ✗ | — | 高（像素），同上 |
| `response_format` | string | ✗ | `url` | `url` / `base64`；URL 有效期 24 小时 |
| `n` | integer | ✗ | `1` | 生成数量，范围 `[1, 9]` |
| `seed` | integer | ✗ | — | 复现种子 |
| `prompt_optimizer` | boolean | ✗ | `false` | 自动优化 prompt |
| `aigc_watermark` | boolean | ✗ | `false` | 添加水印 |

### `subject_reference[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `character`（人物肖像参考） |
| `image_file` | string \| array | ✓ | 公网 URL 或 base64 data URL；JPG / PNG；≤ 10MB |

### `style`（仅 `image-01-live`）

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `style_type` | string | — | `漫画` / `元气` / `中世纪` / `水彩` |
| `style_weight` | number | `0.8` | 范围 `(0, 1]` |

## 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 生成任务 ID |
| `data.image_urls` | array | `response_format=url` 时返回的图片 URL 数组 |
| `data.image_base64` | array | `response_format=base64` 时返回的 base64 字符串数组 |
| `metadata.success_count` | integer | 成功生成的图片数 |
| `metadata.failed_count` | integer | 因安全检查失败的图片数 |
| `base_resp` | object | 通用响应封装 |

## 示例

### 最小请求（文生图）

```json
{ "model": "image-01", "prompt": "A serene landscape with mountains" }
```

### 最小请求（图生图，使用人物参考）

```json
{
  "model": "image-01",
  "prompt": "A girl looking into the distance from a library window",
  "subject_reference": [
    { "type": "character", "image_file": "https://example.com/person.jpg" }
  ]
}
```

### 最小响应

```json
{
  "id": "03ff3cd0820949eb8a410056b5f21d38",
  "data": { "image_urls": ["https://..."] },
  "metadata": { "success_count": 1, "failed_count": 0 },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

## 错误码与计费

- 常见错误：`1008` 余额不足 / `1026` 涉敏 / `2013` 参数异常。
- 价格：¥0.025 / 张，详见 [pricing.md §图像生成](./pricing.md)。

## 参考

- 文生图：https://platform.minimaxi.com/docs/api-reference/image-generation-t2i
- 图生图：https://platform.minimaxi.com/docs/api-reference/image-generation-i2i
- Guide：https://platform.minimaxi.com/docs/guides/image-generation
