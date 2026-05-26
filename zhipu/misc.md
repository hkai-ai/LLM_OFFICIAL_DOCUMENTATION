---
source: https://docs.bigmodel.cn/api-reference/模型-api
fetched_at: 2026-05-26
api_version: v4
---

# 模型 API 杂项 · rerank / tokenizer / layout_parsing

汇总智谱模型 API 中未在专门 md 文件展开的三个端点：文本重排序、文本分词器、文档解析（同步 OCR）。其余「工具 API」类（web_search / web_reader / file parser / OCR 通用接口 / 内容安全）见 [tools.md](./tools.md)。

| 端点 | METHOD | PATH |
| --- | --- | --- |
| 文本重排序 | POST | `/paas/v4/rerank` |
| 文本分词器 | POST | `/paas/v4/tokenizer` |
| 文档解析（同步） | POST | `/paas/v4/layout_parsing` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ZHIPU_API_KEY`。 |
| `Content-Type` | ✓ | `application/json`。 |

## 1. 文本重排序 · POST /paas/v4/rerank

### 请求 body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 目前只有 `rerank` 一个模型 ID。 |
| `query` | string | ✓ | — | 查询文本，长度 ≤4096。 |
| `documents` | `array<string>` | ✓ | — | 候选文本，≤128 条，每条 ≤4096 字符。 |
| `top_n` | integer | ✗ | `0` | 返回前 N 条；`0` 表示全部。 |
| `return_documents` | boolean | ✗ | `false` | 是否回填 `document` 原文。 |
| `return_raw_scores` | boolean | ✗ | `false` | 是否返回 logit 原始分数（未做归一化）。 |
| `request_id` | string | ✗ | — | 6–64 字符，建议 UUID。 |
| `user_id` | string | ✗ | — | 终端用户 ID。 |

### 响应

```json
{
  "id": "...",
  "request_id": "...",
  "created": 1716700000,
  "results": [
    { "index": 2, "relevance_score": 0.92, "document": "..." },
    { "index": 0, "relevance_score": 0.81 }
  ],
  "usage": { "prompt_tokens": 142, "total_tokens": 142 }
}
```

## 2. 文本分词器 · POST /paas/v4/tokenizer

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 与 chat 模型同名：`glm-4.6` / `glm-4.6v` / `glm-4.5` / `glm-4.5-air` / `glm-4-0520` / `glm-4-long` / `glm-4-air` / `glm-4-flash` 等。 |
| `messages` | array | ✓ | 与 Chat Completions 同结构；`role`：`system` / `user` / `assistant`。 |
| `tools` | array | ✗ | 同 Chat Completions，最多 128 个 function。 |
| `request_id` | string | ✗ | — |
| `user_id` | string | ✗ | — |

### 响应

```json
{
  "id": "...",
  "request_id": "...",
  "created": 1716700000,
  "usage": {
    "prompt_tokens": 412,
    "image_tokens": 0,
    "video_tokens": 0,
    "total_tokens": 412
  }
}
```

> 该接口不真正发起一次模型推理，仅按目标模型 tokenizer 计数。多模态模型上可同时返回 `image_tokens` / `video_tokens`。

## 3. 文档解析 · POST /paas/v4/layout_parsing

### 请求 body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 固定 `glm-ocr`。 |
| `file` | string | ✓ | — | 图像或 PDF；支持 URL 或 `data:...;base64,...`。 |
| `return_crop_images` | boolean | ✗ | `false` | 是否返回各布局元素的截图 URL。 |
| `need_layout_visualization` | boolean | ✗ | `false` | 是否返回带框图的可视化页面 URL。 |
| `start_page_id` | integer | ✗ | `1` | PDF 起始页码（含）。 |
| `end_page_id` | integer | ✗ | — | PDF 结束页码（含），最大值 = 总页数。 |
| `request_id` | string | ✗ | — | 6–64 字符。 |
| `user_id` | string | ✗ | — | 6–128 字符。 |

### 响应

```json
{
  "id": "...",
  "request_id": "...",
  "created": 1716700000,
  "model": "glm-ocr",
  "md_results": "# 标题\n正文...",
  "layout_details": [
    {
      "index": 0,
      "label": "title" ,
      "bbox_2d": [12, 30, 800, 90],
      "content": "..."
    }
  ],
  "layout_visualization": ["https://..."],
  "data_info": { "pages": 5, "page_size": [1024, 1448] },
  "usage": { "prompt_tokens": 0, "total_tokens": 1234 }
}
```

| 字段 | 说明 |
| --- | --- |
| `md_results` | 全文 Markdown，最直接的下游可消费形式。 |
| `layout_details[].label` | 元素类型：`title` / `paragraph` / `table` / `figure` / `header` / `footer` / `formula` 等。 |
| `layout_details[].bbox_2d` | `[x1, y1, x2, y2]`，与 `data_info.page_size` 同比例。 |
| `layout_visualization[]` | 仅 `need_layout_visualization: true` 返回。 |

> 异步 / 大文件场景请改用 [tools.md](./tools.md) §「文件解析（异步）」端点。

## 错误码

| HTTP | code | 触发 |
| --- | --- | --- |
| `400` | `1010` | 字段越界 / file 既非 URL 也非 base64 |
| `413` | `1016` | 文件过大 |
| `429` | `1113` | 限流 |
| `500` | `2001` | 上游异常 |

完整错误码见 [errors.md](./errors.md)。

## 参考

- 文本重排序：<https://docs.bigmodel.cn/api-reference/模型-api/文本重排序>
- 文本分词器：<https://docs.bigmodel.cn/api-reference/模型-api/文本分词器>
- 文档解析：<https://docs.bigmodel.cn/api-reference/模型-api/文档解析>
- 工具 API（异步文件解析 / OCR / Web Search）：[tools.md](./tools.md)
