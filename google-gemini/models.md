---
source: https://ai.google.dev/api/models?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 模型 · /v1beta/models

> 列出与获取 Gemini Developer API 提供的模型与其能力元数据。

## 端点一：models.get

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/{name=models/*}` |
| Path 参数 | `name`（必填，格式 `models/{model}`） |
| 响应 | 单个 `Model` 对象 |

## 端点二：models.list

| 维度 | 值 |
| --- | --- |
| 方法 | GET |
| 路径 | `/v1beta/models` |

Query 参数：

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `pageSize` | integer | ✗ | `50` | 最大 `1000`。 |
| `pageToken` | string | ✗ | — | 翻页 token。 |

响应：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `models` | array&lt;Model&gt; | 当前页模型列表。 |
| `nextPageToken` | string | 下一页 token；为空表示终页。 |

## Model 对象字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | 资源名，格式 `models/{model}`，如 `models/gemini-2.5-pro`。 |
| `baseModelId` | string | 基础模型 ID，供发起请求使用（如 `gemini-2.5-pro`）。 |
| `version` | string | 大版本号，如 `001`、`002`。 |
| `displayName` | string | 展示名，最多 128 字符。 |
| `description` | string | 简介。 |
| `inputTokenLimit` | integer | 最大输入 token 数。 |
| `outputTokenLimit` | integer | 最大输出 token 数。 |
| `supportedGenerationMethods` | array&lt;string&gt; | 支持的方法名（如 `generateContent`、`streamGenerateContent`、`countTokens`、`embedContent`、`batchEmbedContents`、`bidiGenerateContent`）。 |
| `temperature` | number | 默认采样温度。 |
| `maxTemperature` | number | 允许的最大温度。 |
| `topP` | number | 默认 top-p。 |
| `topK` | integer | 默认 top-k；为 `0` / 不存在表示模型不支持 top-k。 |

## 当前主推模型 ID（按官方实时清单为准）

| 模型 ID | 类别 | 关键能力 |
| --- | --- | --- |
| `gemini-2.5-pro` | 文本/多模态（含思考） | 最强综合模型，长上下文，思考默认开启。 |
| `gemini-2.5-flash` | 文本/多模态（含思考） | 性价比最高的通用模型。 |
| `gemini-2.5-flash-lite` | 文本/多模态 | 2.5 系列中最快、最便宜。 |
| `gemini-2.0-flash` | 文本/多模态 | 上一代主力，支持 Live API。 |
| `gemini-2.0-flash-lite` | 文本/多模态 | 2.0 系列轻量版。 |
| `gemini-embedding-001` | Embedding | 通用文本嵌入，默认 3072 维，可截断。 |
| `text-embedding-004` | Embedding | 前代文本嵌入，768 维。 |
| `embedding-001` | Embedding | 老版嵌入模型，仅向后兼容。 |
| `imagen-3.0-*` / `nano-banana-*` | 图像生成 | 见 Imagen / Nano Banana 专栏。 |
| `veo-*` | 视频生成 | Veo 系列。 |
| `lyria-*` | 音乐生成 | Lyria 系列。 |
| `gemini-*-flash-tts` | TTS | 文本转语音。 |
| `gemini-*-flash-live` | Live API | 实时双向流式。 |

> 上下文窗口、是否支持思考、是否支持 `responseModalities=AUDIO/IMAGE` 等以 `models.get` 实时返回为准；上方表格中的能力描述仅作分类索引。

## 示例

### 列模型

`GET /v1beta/models?pageSize=2`：

```json
{
  "models": [
    {
      "name": "models/gemini-2.5-pro",
      "baseModelId": "gemini-2.5-pro",
      "version": "002",
      "displayName": "Gemini 2.5 Pro",
      "description": "Most capable Gemini 2.5 model ...",
      "inputTokenLimit": 1048576,
      "outputTokenLimit": 65536,
      "supportedGenerationMethods": ["generateContent", "streamGenerateContent", "countTokens"],
      "temperature": 1.0,
      "maxTemperature": 2.0,
      "topP": 0.95,
      "topK": 64
    }
  ],
  "nextPageToken": "..."
}
```

### 取单个模型

`GET /v1beta/models/gemini-2.5-flash` 直接返回上方结构的单个 `Model`。

## 错误

参见 [errors.md](./errors.md)。

## 参考

- 端点文档：<https://ai.google.dev/api/models?hl=zh-CN>
- 模型对比页：<https://ai.google.dev/gemini-api/docs/models>
