---
source: https://www.volcengine.com/docs/82379/1330310
fetched_at: 2026-05-28
api_version: N/A
---

# 火山方舟模型列表

火山方舟（豆包 Doubao 系列）模型清单。调用时 `model` 字段填 **Model ID** 或自建的 **Endpoint ID**（Access Key 鉴权下必须用 Endpoint ID）。限流（RPM / TPM）为非刚性保障，受平台负载与调用方式影响。长度单位为 token，`k` = 1000。

> 火山方舟没有独立的 List Models API；模型清单以本文及[控制台模型广场](https://console.volcengine.com/ark/region:ark+cn-beijing/openManagement)为准。OpenAI 兼容的 `GET /api/v3/models` 行为见 [openai-compat.md](./openai-compat.md)。

## 深度思考 / 文本生成 / 多模态（Doubao Seed 2.0，推荐）

| Model ID | 能力 | 上下文窗口 | 最大输入 | 最大回答(默认4k) | 最大思维链 | RPM / TPM |
| --- | --- | --- | --- | --- | --- | --- |
| `doubao-seed-2-0-lite-260428` | 深度思考 / 文本 / 多模态理解 / 工具调用 | 256k | 224k | 128k | 128k | 30000 / 5000000 |
| `doubao-seed-2-0-mini-260428` | 深度思考 / 文本 / 多模态理解 / 工具调用 | 256k | 224k | 128k | 128k | 30000 / 5000000 |
| `doubao-seed-2-0-pro-260215` | 深度思考 / 文本 / 多模态理解 / 工具调用 / 视觉定位 | 256k | 224k | 128k | 128k | 30000 / 5000000 |
| `doubao-seed-2-0-lite-260215` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 / 结构化输出 | 256k | 224k | 128k | 128k | 30000 / 5000000 |
| `doubao-seed-2-0-mini-260215` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 / 结构化输出 | 256k | 224k | 128k | 128k | 30000 / 5000000 |
| `doubao-seed-2-0-code-preview-260215` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 | 256k | 224k | 128k | 128k | 30000 / 5000000 |

## 往期主力模型

| Model ID | 能力 | 上下文窗口 | 最大输入 | 最大回答 | 最大思维链 | RPM / TPM |
| --- | --- | --- | --- | --- | --- | --- |
| `doubao-seed-1-8-251228` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 / 结构化输出 | 256k | 224k | 64k | 32k | 30000 / 5000000 |
| `doubao-seed-code-preview-251028` | 深度思考 / 编程增强 / 文本 / 多模态理解 / 工具调用 | 256k | 224k | 32k | 32k | 5000 / 1200000 |
| `doubao-seed-1-6-251015` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 / 结构化输出 | 256k | 224k | 32k | 32k | 30000 / 5000000 |
| `doubao-seed-1-6-250615` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / 工具调用 / 结构化输出 | 256k | 224k | 32k | 32k | 30000 / 5000000 |
| `doubao-seed-1-6-flash-250828` | 深度思考 / 文本 / 视觉定位 / 多模态理解 / 工具调用 / 结构化输出 | 256k | 224k | 32k | 32k | 30000 / 5000000 |
| `doubao-seed-1-6-flash-250615` | 深度思考 / 文本 / 视觉定位 / 多模态理解 / 工具调用 / 结构化输出 | 256k | 224k | 32k | 32k | 30000 / 5000000 |
| `doubao-seed-1-6-vision-250815` | 深度思考 / 文本 / 多模态理解 / 视觉定位 / GUI 任务 / 工具调用 / 结构化输出 | 256k | 224k | 32k | 32k | 30000 / 5000000 |
| `doubao-seed-character-251128` | 文本生成 / 工具调用（角色扮演增强） | 128k | 96k | 32k | — | 30000 / 5000000 |
| `doubao-seed-translation-250915` | 文本生成 / 翻译增强 | 4k | 1k | 3k | — | 5000 / 500000 |
| `doubao-1-5-pro-32k-250115` | 文本生成 / 工具调用 | 128k | — | 16k | — | 30000 / 5000000 |
| `doubao-1-5-pro-32k-character-250715` | 文本生成 / 角色扮演增强 | 32k | — | 12k | — | — |
| `doubao-1-5-lite-32k-250115` | 文本生成 / 工具调用 | 32k | — | 12k | — | 30000 / 5000000 |
| `doubao-1-5-vision-pro-32k-250115` | 图片理解 / 工具调用（仅 Chat API） | 32k | — | 12k | — | 30000 / 5000000 |

## 平台托管的第三方模型

| Model ID | 能力 | 上下文窗口 | 最大输入 | 最大回答 | 最大思维链 | RPM / TPM |
| --- | --- | --- | --- | --- | --- | --- |
| `glm-4-7-251222` | 深度思考 / 文本 / 工具调用 | 200k | 200k | 128k | 128k | 15000 / 1500000 |
| `deepseek-v4-pro-260425` | 深度思考 / 文本 / 工具调用 | 1024k | 1024k | 384k | 384k | 15000 / 1500000 |
| `deepseek-v4-flash-260425` | 深度思考 / 文本 / 工具调用 | 1024k | 1024k | 384k | 384k | 15000 / 1500000 |
| `deepseek-v3-2-251201` | 深度思考 / 文本 / 工具调用 | 128k | 128k | 32k | 32k | 15000 / 1500000 |

## 视频生成模型（Doubao Seedance，Video Generation API）

| Model ID | 能力 | 输出分辨率 | 帧率 | 时长 | 格式 |
| --- | --- | --- | --- | --- | --- |
| `doubao-seedance-2-0-260128` | 文/图/多模态生视频、首帧/首尾帧、延长、编辑、音画同生 | 480p / 720p / 1080p | 24 fps | 4~15 秒 | mp4 |
| `doubao-seedance-2-0-fast-260128` | 同上（不支持 1080p） | 480p / 720p | 24 fps | 4~15 秒 | mp4 |

> 此外定价页列出 `doubao-seedance-1.5-pro`、`doubao-seedance-1.0-pro`、`doubao-seedance-1.0-pro-fast` 等往期视频模型。

## 图片生成模型（Doubao Seedream，Image Generation API）

| Model ID | 说明 |
| --- | --- |
| `doubao-seedream-5-0-260128` | Seedream 5.0，最强生图模型，搭载联网检索（快速入门示例所用）。 |
| `doubao-seedream-5.0-lite` | Seedream 5.0 lite（定价页列出）。 |
| `doubao-seedream-4.5` | Seedream 4.5（定价页列出）。 |
| `doubao-seedream-4.0` | Seedream 4.0（定价页列出）。 |

## 向量模型（Embeddings API）

| Model ID | 说明 |
| --- | --- |
| `doubao-embedding-vision` | 文本 + 图片向量化模型。 |

## 能力维度说明

- **深度思考**：支持 `thinking` / `reasoning_effort` 参数，输出 `reasoning_content`，见 [chat-completions.md](./chat-completions.md)。
- **结构化输出（beta）**：支持 `response_format` 的 `json_object` / `json_schema`。
- **上下文缓存**：隐式缓存（Responses / Chat / Batch API）与显式缓存（前缀缓存 / Session 缓存），见 [context-cache.md](./context-cache.md)。
- **工具调用**：Function Calling（Responses / Chat API），知识库 / MCP / 联网内容插件 / 图像处理 / 豆包助手（多为 Responses API）。

## 参考

- 模型列表：https://www.volcengine.com/docs/82379/1330310
- 查询 Model ID（控制台开通管理）：https://console.volcengine.com/ark/region:ark+cn-beijing/openManagement
- 获取 Endpoint ID：https://www.volcengine.com/docs/82379/1099522
