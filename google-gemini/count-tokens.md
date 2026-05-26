---
source: https://ai.google.dev/api/tokens?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 统计 token · POST /v1beta/{model=models/*}:countTokens

> 在不真正发起生成的情况下，估算给定输入会被模型计入多少 token。`contents` 与 `generateContentRequest` 二选一。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` | ✓（或 query `key`） | API Key。 |
| `Content-Type` | ✓ | `application/json`。 |

## Path 参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型资源名，格式 `models/{model}`。 |

## 请求 Body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `contents` | array&lt;Content&gt; | ✗ | — | 要统计的内容；若同时设置 `generateContentRequest` 则本字段被忽略。结构同 [generate-content.md](./generate-content.md#contents)。 |
| `generateContentRequest` | GenerateContentRequest | ✗ | — | 完整的生成请求（含 `contents`、`systemInstruction`、`tools`、`toolConfig`、`generationConfig`、`cachedContent` 等），用于得到「连同系统指令、工具声明与缓存一起计费」的准确数字。 |

> 二者必须至少传其一；推荐使用 `generateContentRequest`，因为只数 `contents` 会漏掉 systemInstruction、tools、cachedContent 引入的 token。

## 响应（CountTokensResponse）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `totalTokens` | integer | 总 token 数（非负）。 |
| `cachedContentTokenCount` | integer | 来自 `cachedContent` 的 token 数。 |
| `promptTokensDetails` | array&lt;ModalityTokenCount&gt; | 输入按模态拆分。 |
| `cacheTokensDetails` | array&lt;ModalityTokenCount&gt; | 缓存按模态拆分。 |

`ModalityTokenCount`：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `modality` | string | `MODALITY_UNSPECIFIED` / `TEXT` / `IMAGE` / `VIDEO` / `AUDIO` / `DOCUMENT`。 |
| `tokenCount` | integer | 对应模态 token 数。 |

## 示例

### 最小请求

```json
{
  "contents": [
    { "role": "user", "parts": [{ "text": "用一句话介绍量子纠缠。" }] }
  ]
}
```

`POST /v1beta/models/gemini-2.5-flash:countTokens`。

### 最小响应

```json
{
  "totalTokens": 12,
  "promptTokensDetails": [
    { "modality": "TEXT", "tokenCount": 12 }
  ]
}
```

### 携带工具与系统指令

```json
{
  "generateContentRequest": {
    "model": "models/gemini-2.5-pro",
    "systemInstruction": { "parts": [{ "text": "你是一个严谨的助手。" }] },
    "contents": [
      { "role": "user", "parts": [{ "text": "查询明天北京天气。" }] }
    ],
    "tools": [
      {
        "functionDeclarations": [
          {
            "name": "get_weather",
            "parameters": {
              "type": "OBJECT",
              "properties": { "city": { "type": "STRING" } },
              "required": ["city"]
            }
          }
        ]
      }
    ]
  }
}
```

## 错误

参见 [errors.md](./errors.md)。

## 参考

- 端点文档：<https://ai.google.dev/api/tokens?hl=zh-CN>
