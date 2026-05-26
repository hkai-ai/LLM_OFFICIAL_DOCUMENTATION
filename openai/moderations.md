---
source: https://developers.openai.com/api/reference/resources/moderations/methods/create
fetched_at: 2026-05-26
api_version: v1
---

# Moderations · POST /v1/moderations

判断输入文本 / 图像是否触发内容安全分类。**免费**，常用于上线前做用户输入过滤、模型输出复核。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | ✓ | `application/json`。 |

## 请求 body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `input` | `string \| array<string> \| array<object>` | ✓ | — | 单字符串、字符串数组或多模态 part 数组。 |
| `model` | string | ✗ | `omni-moderation-latest` | 见模型清单。 |

### input 三种形式

1. **单条字符串**
   ```json
   { "input": "I want to kill them." }
   ```
2. **字符串数组**（同时分类多条）
   ```json
   { "input": ["text1", "text2"] }
   ```
3. **多模态 part 数组**（`omni-moderation-*` 起支持）
   ```json
   {
     "input": [
       { "type": "text", "text": "..." },
       { "type": "image_url", "image_url": { "url": "https://..." } }
     ]
   }
   ```

## 模型清单

| 模型 ID | 输入 | 类目数量 | 备注 |
| --- | --- | --- | --- |
| `omni-moderation-latest` | 文本 + 图像 | 13 类 | 推荐新项目使用。 |
| `omni-moderation-2024-09-26` | 同上 | 同上 | 固定版本。 |
| `text-moderation-latest` | 仅文本 | 较少 | 老版。 |
| `text-moderation-stable` | 仅文本 | 同上 | 固定版本。 |

## 响应

### 顶层

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 请求 ID（`modr-...`）。 |
| `model` | string | 实际使用的模型 ID。 |
| `results` | `array<ModerationResult>` | 与 `input` 顺序对应。 |

### results[]

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `flagged` | boolean | 任一类目触发即 `true`。 |
| `categories` | object | 各类目命中布尔；见下方枚举。 |
| `category_scores` | object | 各类目 0–1 分数。 |
| `category_applied_input_types` | object | 每个类目实际评估的输入类型（`text` / `image`）。 |

### categories 枚举

仅文本类目：

- `harassment`
- `harassment/threatening`
- `hate`
- `hate/threatening`
- `illicit`
- `illicit/violent`
- `sexual/minors`

文本 + 图像类目：

- `self-harm`
- `self-harm/instructions`
- `self-harm/intent`
- `sexual`
- `violence`
- `violence/graphic`

> 老模型 `text-moderation-*` 仅包含一部分类目（无 `illicit*`、`harassment*`、`category_applied_input_types`）。

## 最小请求

```bash
curl https://api.openai.com/v1/moderations \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "omni-moderation-latest",
    "input": "I want to kill them."
  }'
```

## 最小响应

```json
{
  "id": "modr-abc123",
  "model": "omni-moderation-latest",
  "results": [
    {
      "flagged": true,
      "categories": {
        "violence": true,
        "hate": false,
        "self-harm": false,
        "sexual": false,
        "sexual/minors": false,
        "hate/threatening": false,
        "violence/graphic": false,
        "self-harm/intent": false,
        "self-harm/instructions": false,
        "harassment": false,
        "harassment/threatening": false,
        "illicit": false,
        "illicit/violent": false
      },
      "category_scores": {
        "violence": 0.95,
        "hate": 0.001
      },
      "category_applied_input_types": {
        "violence": ["text"],
        "self-harm": ["text"],
        "sexual": ["text"]
      }
    }
  ]
}
```

## 计费

Moderations 接口本身免费；只针对正常运营使用，不可作为大规模数据清洗的纯白嫖工具。

## 参考

- API：<https://developers.openai.com/api/reference/resources/moderations/methods/create>
- 指南：<https://developers.openai.com/api/docs/guides/moderation>
- omni-moderation-latest 模型：<https://developers.openai.com/api/docs/models/omni-moderation-latest>
- Cookbook 示例：<https://developers.openai.com/cookbook/examples/how_to_use_moderation>
