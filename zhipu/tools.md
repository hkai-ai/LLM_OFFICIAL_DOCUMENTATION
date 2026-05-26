---
source: https://docs.bigmodel.cn/api-reference/工具-api/网络搜索
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 BigModel · 工具 API

汇总 5 个独立工具端点（不通过 chat completions 的 `tools` 字段触发，而是直接独立 POST）：

| 工具 | 端点 |
| --- | --- |
| 网络搜索 | `POST /paas/v4/web_search` |
| 网页阅读 | `POST /paas/v4/web_reader` |
| 文件解析（异步） | `POST /paas/v4/files/parser/create` |
| 文件解析（同步） | `POST /paas/v4/files/parser/sync` |
| OCR | `POST /paas/v4/ocr` |
| 内容安全 | `POST /paas/v4/content_safety` |

鉴权统一：`Authorization: Bearer <API_KEY>`，Base `https://open.bigmodel.cn/api/paas/v4`。

## 1. Web Search · POST /paas/v4/web_search

### 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `search_query` | string | ✓ | — | 搜索内容，建议 ≤ 70 字符 |
| `search_engine` | string | ✓ | — | 见枚举 |
| `search_intent` | boolean | ✓ | `false` | 是否执行搜索意图识别 |
| `count` | integer | ✗ | `10` | 返回结果数，范围 `[1, 50]` |
| `search_domain_filter` | string | ✗ | — | 域名白名单 |
| `search_recency_filter` | string | ✗ | `noLimit` | 时间范围过滤 |
| `content_size` | string | ✗ | — | `medium` / `high` |
| `request_id` | string | ✗ | 自动 | 6–64 字符 |
| `user_id` | string | ✗ | — | 6–128 字符 |

### `search_engine` 枚举

| 值 | 说明 |
| --- | --- |
| `search_std` | 智谱基础版搜索 |
| `search_pro` | 智谱高阶版搜索 |
| `search_pro_sogou` | 搜狗 |
| `search_pro_quark` | 夸克 |

### 响应

```json
{
  "id": "...",
  "created": 1704067200,
  "request_id": "...",
  "search_intent": [/* 意图识别结果 */],
  "search_result": [
    {
      "title": "网页标题",
      "content": "内容摘要",
      "link": "https://...",
      "media": "网站名",
      "icon": "网站图标",
      "refer": "1",
      "publish_date": "2026-05-20"
    }
  ]
}
```

### 错误码

`1701` 并发超限 / `1702` 搜索引擎不可用 / `1703` 无有效数据返回。

### 最小请求

```json
{
  "search_query": "Python教程",
  "search_engine": "search_std",
  "search_intent": false
}
```

## 2. 网页阅读 · POST /paas/v4/web_reader

读取并解析指定 URL 的网页内容。字段详见 https://docs.bigmodel.cn/api-reference/工具-api/网页阅读 。

## 3. 文件解析（异步）· POST /paas/v4/files/parser/create

### 请求（multipart/form-data）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `file` | binary | ✓ | 待解析文件 |
| `tool_type` | string | ✓ | `lite` / `expert` / `prime`，能力依次增强 |
| `file_type` | string | ✗ | 文件格式提示 |

### 各 `tool_type` 支持的格式

| tool_type | 支持格式 |
| --- | --- |
| `lite` | pdf / docx / doc / xls / xlsx / ppt / pptx / png / jpg / jpeg / csv / txt / md |
| `expert` | 仅 pdf |
| `prime` | lite 的全集 + html / bmp / gif / webp / heic / eps / icns / im / pcx / ppm / tiff / xbm / heif / jp2 |

### 响应

```json
{
  "success": true,
  "message": "ok",
  "task_id": "..."
}
```

### 查询结果

`POST /paas/v4/files/parser/result`（详见 https://docs.bigmodel.cn/api-reference/工具-api/解析结果）。

## 4. 文件解析（同步）· POST /paas/v4/files/parser/sync

字段与异步一致，直接同步返回解析后的文本结构。详见 https://docs.bigmodel.cn/api-reference/工具-api/文件解析同步 。

## 5. OCR · POST /paas/v4/ocr

光学字符识别。字段详见 https://docs.bigmodel.cn/api-reference/工具-api/ocr-服务 。

## 6. 内容安全 · POST /paas/v4/content_safety

文本 / 图片 / 音视频内容检测。字段详见 https://docs.bigmodel.cn/api-reference/工具-api/内容安全 。

## 参考

- Web Search：https://docs.bigmodel.cn/api-reference/工具-api/网络搜索
- 网页阅读：https://docs.bigmodel.cn/api-reference/工具-api/网页阅读
- 文件解析（异步）：https://docs.bigmodel.cn/api-reference/工具-api/文件解析
- 文件解析（同步）：https://docs.bigmodel.cn/api-reference/工具-api/文件解析同步
- 解析结果：https://docs.bigmodel.cn/api-reference/工具-api/解析结果
- OCR：https://docs.bigmodel.cn/api-reference/工具-api/ocr-服务
- 内容安全：https://docs.bigmodel.cn/api-reference/工具-api/内容安全
- 联网搜索 Guide：https://docs.bigmodel.cn/cn/guide/tools/web-search
- 文件解析 Guide：https://docs.bigmodel.cn/cn/guide/tools/file-parser
