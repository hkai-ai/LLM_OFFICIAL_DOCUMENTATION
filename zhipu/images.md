---
source: https://docs.bigmodel.cn/api-reference/模型-api/图像生成
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM Images · POST /paas/v4/images/generations

> 文本生成图像。同步返回图片 URL（24 小时 / 30 天有效，按模型而异）。

## 鉴权

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

完整 URL：`POST https://open.bigmodel.cn/api/paas/v4/images/generations`

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | `glm-image` / `cogview-4-250304` / `cogview-4` / `cogview-3-flash` |
| `prompt` | string | ✓ | — | 图像描述 |
| `size` | string | ✗ | GLM-Image: `1280x1280`；其他: `1024x1024` | 输出尺寸 |
| `quality` | string | ✗ | `standard` | `hd`（20s，**仅 GLM-Image**）/ `standard`（5–10s） |
| `watermark_enabled` | boolean | ✗ | `true` | 关闭需签署免责声明 |
| `user_id` | string | ✗ | — | 终端用户唯一标识，6–128 字符 |

> 官方页未列出 `n` 与 `response_format`；如需多张请重抓页面确认。

## 响应

```json
{
  "created": 1704067200,
  "data": [
    { "url": "https://..." }
  ],
  "content_filter": [
    { "role": "assistant", "level": 0 }
  ]
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `created` | integer | Unix 时间戳 |
| `data[].url` | string | 图片 URL（30 天有效） |
| `content_filter` | array | 内容安全审核（`role` / `level`） |

## 异步图像生成

`POST /paas/v4/images/generations` 也提供异步形式（详见 https://docs.bigmodel.cn/api-reference/模型-api/图像生成异步），通过 `id` 查询异步结果。字段集合与同步一致。

## 示例

### 最小请求

```json
{
  "model": "glm-image",
  "prompt": "一只可爱的小猫咪，坐在阳光明媚的窗台上",
  "size": "1280x1280"
}
```

### HD 质量

```json
{
  "model": "glm-image",
  "prompt": "赛博朋克风格城市夜景",
  "quality": "hd"
}
```

## 参考

- 同步图像生成：https://docs.bigmodel.cn/api-reference/模型-api/图像生成
- 异步图像生成：https://docs.bigmodel.cn/api-reference/模型-api/图像生成异步
- GLM-Image 模型：https://docs.bigmodel.cn/cn/guide/models/image-generation/glm-image
- CogView-4 模型：https://docs.bigmodel.cn/cn/guide/models/image-generation/cogview-4
- 图像 prompt 工程：https://docs.bigmodel.cn/cn/best-practice/prompt/image-prompt
