---
source: https://www.volcengine.com/docs/82379/1541523
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 图片生成（Images）· POST /api/v3/images/generations

> 调用 Seedream 系列模型（`doubao-seedream-5.0-lite` / `4.5` / `4.0` 等）进行文生图、图生图、多图融合、组图生成。完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/images/generations`。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY`（本接口仅支持 API Key 鉴权）。 |
| `Content-Type` | ✓ | `application/json` |

## 能力概览

- **组图**（`sequential_image_generation=auto`）：文生组图（≤15 张）、单图生组图（≤14 张）、多图生组图（参考图 2–14 张，参考图 + 生成图 ≤15 张）。
- **单图**（`sequential_image_generation=disabled`）：文生图、单图生图、多图生图。

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID 或 Endpoint ID，如 `doubao-seedream-5-0-260128`。 |
| `prompt` | string | ✓ | — | 生成提示词，支持中英文。建议 ≤300 汉字 / 600 英文单词。 |
| `image` | string \| array | ✗ | — | 参考图，URL 或 Base64（`data:image/<格式小写>;base64,<编码>`）。`doubao-seedream-5.0-lite/4.5/4.0` 支持单图 / 多图（最多 14 张）。 |
| `size` | string | ✗ | `2048x2048` | 见 [size 取值](#size-取值)。 |
| `sequential_image_generation` | string | ✗ | `disabled` | `auto`（模型自主判断是否返回组图及数量）/ `disabled`（仅生成一张）。仅 5.0-lite/4.5/4.0 支持。 |
| `sequential_image_generation_options` | object | ✗ | — | 组图配置，仅 `sequential_image_generation=auto` 生效。`max_images`（默认 15，范围 `[1,15]`；受参考图数量约束，参考图+生成图 ≤15）。 |
| `tools` | array | ✗ | — | 模型可调用工具。`tools.type=web_search`（联网搜索，仅 5.0-lite 支持）；开启后模型自主判断是否联网，结果次数见 `usage.tool_usage.web_search`。 |
| `stream` | boolean | ✗ | `false` | 流式输出（即时返回每张图）。仅 5.0-lite/4.5/4.0 支持，单图 / 组图均生效。 |
| `guidance_scale` | float | ✗ | — | 与 prompt 的一致程度，范围 `[1, 10]`，越大自由度越小。`doubao-seedream-5.0-lite/4.5/4.0` 不支持。 |
| `output_format` | string | ✗ | `jpeg` | `png` / `jpeg`。仅 5.0-lite 支持自定义；4.5/4.0 固定 `jpeg`。 |
| `response_format` | string | ✗ | `url` | `url`（24 小时内有效）/ `b64_json`（Base64）。 |
| `watermark` | boolean | ✗ | `true` | `true` 在右下角添加「AI生成」水印。 |
| `optimize_prompt_options` | object | ✗ | — | 提示词优化配置，仅 5.0-lite/4.5/4.0 支持。`mode`：`standard`（默认，质量高耗时长）/ `fast`（耗时短，5.0-lite/4.5 暂不支持）。 |

### size 取值

两种方式，不可混用：

- **方式 1（分辨率档位）**：`2K` / `3K` / `4K`，配合 prompt 中自然语言描述宽高比，由模型判断最终大小。
- **方式 2（精确像素）**：`<宽>x<高>`，默认 `2048x2048`。需同时满足：总像素 `[2560×1440=3686400, 4096×4096=16777216]`，宽高比 `[1/16, 16]`（总像素是宽高乘积限制）。如 `3750x1250` 有效，`1500x1500` 无效（总像素不足）。

参考图要求：格式 `jpeg`/`png`（5.0-lite/4.5/4.0 另支持 `webp`/`bmp`/`tiff`/`gif`/`heic`/`heif`）；宽高比 `[1/16, 16]`；宽高 >14 px；大小 ≤30 MB；总像素 ≤6000×6000。

## 响应（非流式）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `model` | string | 使用的模型 ID。 |
| `created` | integer | 创建 Unix 时间戳（秒）。 |
| `data` | array | 输出图像信息，见下表。 |
| `tools` | array | 本次配置的工具（`tools.type`，如 `web_search`）。 |
| `usage` | object | 用量，见下表。 |
| `error` | object | 整请求错误（`error.code` / `error.message`），见 [errors.md](./errors.md)。 |

### `data[]`

成功元素（图片信息）与失败元素（错误信息）可混合出现：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | string | 图片下载链接，`response_format=url` 时返回（24 小时内有效）。 |
| `b64_json` | string | 图片 Base64，`response_format=b64_json` 时返回。 |
| `size` | string | 图像宽高像素值（如 `2048x2048`），仅 5.0-lite/4.5/4.0 返回。 |
| `error` | object | 某张图生成失败的 `error.code` / `error.message`。 |

> 组图场景下某张图失败：审核不通过时继续生成后续图；内部服务异常（500）时中止后续生成。

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `generated_images` | integer | 成功生成图片张数（不含失败），按此计费。 |
| `output_tokens` | integer | 图片花费 token，计算 `sum(图片长×图片宽)/256` 取整。 |
| `total_tokens` | integer | 总 token（当前不计输入 token，与 `output_tokens` 一致）。 |
| `tool_usage.web_search` | integer | 联网搜索调用次数，仅开启时返回。 |

## 流式响应

`stream=true` 时即时返回每张图片的输出结果，流式响应参数详见 [图片生成流式响应](https://www.volcengine.com/docs/82379/1824137)。

## 示例

### 请求

```bash
curl https://ark.cn-beijing.volces.com/api/v3/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "doubao-seedream-5-0-260128",
    "prompt": "充满活力的特写编辑肖像，工作室灯光效果强烈。",
    "size": "2K",
    "output_format": "png",
    "watermark": false
  }'
```

### 响应

```json
{
  "model": "doubao-seedream-5-0-260128",
  "created": 1757321139,
  "data": [
    { "url": "https://...", "size": "3104x1312" }
  ],
  "usage": { "generated_images": 1, "output_tokens": 0, "total_tokens": 0 }
}
```

## 参考

- 图片生成 API：https://www.volcengine.com/docs/82379/1541523
- 图片生成流式响应：https://www.volcengine.com/docs/82379/1824137
- Seedream 提示词指南：https://www.volcengine.com/docs/82379/1829186
