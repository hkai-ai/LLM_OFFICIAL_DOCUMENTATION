---
source: https://docs.bigmodel.cn/cn/guide/start/model-overview
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 GLM · 模型清单 + Tokenizer + Rerank

> 完整模型说明详见官方[模型概览](https://docs.bigmodel.cn/cn/guide/start/model-overview)；价格见 [pricing.md](./pricing.md)。

## 文本模型（Chat）

| 模型 ID | 上下文 | 思考 | 视觉 | 工具流式 | 备注 |
| --- | --- | --- | --- | --- | --- |
| `glm-5.1` | 128K | 强制 | ✗ | ✓ | 最新旗舰 |
| `glm-5` | 128K | 强制 | ✗ | ✓ | — |
| `glm-5-turbo` | 128K | auto | ✗ | ✓ | 性价比 |
| `glm-4.7` | 128K | 强制 | ✗ | ✓ | — |
| `glm-4.7-flash` | — | — | ✗ | ✗ | 免费档 |
| `glm-4.7-flashx` | — | — | ✗ | ✗ | — |
| `glm-4.6` | 128K | auto | ✗ | ✓ | — |
| `glm-4.5-air` | — | auto | ✗ | ✗ | — |
| `glm-4.5-airx` | — | auto | ✗ | ✗ | — |
| `glm-4.5-flash` | — | auto | ✗ | ✗ | — |
| `glm-4-flash-250414` | — | ✗ | ✗ | ✗ | 免费档 |
| `glm-4-flashx-250414` | — | ✗ | ✗ | ✗ | — |

## 视觉模型（VLM）

| 模型 ID | 上下文 | 备注 |
| --- | --- | --- |
| `glm-5v-turbo` | 128K | 最多 50 张图，128K 输出 |
| `glm-4.6v` | — | 支持 `tools` / `tool_choice` |
| `glm-4.6v-flash` | — | 免费档 |
| `glm-4.6v-flashx` | — | — |
| `glm-4.1v-thinking-flashx` | — | 强制思考 |
| `glm-4.1v-thinking-flash` | — | 免费档，强制思考 |
| `glm-4v-flash` | — | 免费档 |
| `autoglm-phone` | — | 电话交互特化 |

### Vision 输入限制

- 图片：单张 ≤ 5MB，分辨率 ≤ 6000×6000
- 视频：≤ 200MB（旧 GLM-4V-Plus ≤ 20MB、≤ 30 秒）
- 音频：≤ 10 分钟（仅 `glm-4-voice` 支持）

## 图像生成

| 模型 ID | 默认尺寸 | 备注 |
| --- | --- | --- |
| `glm-image` | 1280×1280 | 支持 `quality: "hd"` |
| `cogview-4-250304` | 1024×1024 | — |
| `cogview-4` | 1024×1024 | — |
| `cogview-3-flash` | 1024×1024 | 免费档 |

详见 [images.md](./images.md)。

## 视频生成

| 模型 ID | 支持模式 | 备注 |
| --- | --- | --- |
| `cogvideox-3` | t2v / i2v | 最新 |
| `cogvideox-2` | t2v / i2v | — |
| `cogvideox-flash` | t2v / i2v | 免费档 |
| `viduq1-text` | t2v | — |
| `viduq1-image` | i2v | — |
| `viduq1-start-end` | 首尾帧 | — |
| `vidu2-image` | i2v | — |
| `vidu2-start-end` | 首尾帧 | — |
| `vidu2-reference` | 参考图 | — |

详见 [videos.md](./videos.md)。

## 音频 / 语音

| 模型 ID | 用途 | 端点 |
| --- | --- | --- |
| `glm-4-voice` | 实时语音对话（Chat Completions 端点的 `audio` 模式） | `/paas/v4/chat/completions` |
| `glm-asr-2512` | 语音转文本 | `/paas/v4/audio/transcriptions` |
| `glm-tts` | 文本转语音 | `/paas/v4/audio/speech` |
| `glm-tts-clone` | 音色复刻 + TTS | `/paas/v4/voice/clone` |
| `glm-realtime` | 实时音视频 | — |

详见 [audio.md](./audio.md)。

## 角色 / 心理咨询模型

| 模型 ID | 用途 |
| --- | --- |
| `charglm-4` | 角色扮演 |
| `emohaa` | 情感陪伴；支持顶层 `meta` 字段 |

## Embedding 模型

| 模型 ID | 默认维度 | 可选维度 | 单条 / 数组限制 |
| --- | --- | --- | --- |
| `embedding-3` | `2048` | `256` / `512` / `1024` / `2048` | 单条 ≤ 3072 tokens；数组最多 64 条 |
| `embedding-2` | `1024`（固定） | — | 单条 ≤ 512 tokens；数组总长 ≤ 8K |

详见 [embeddings.md](./embeddings.md)。

## 免费模型一览

`glm-4-flash-250414` / `glm-4.7-flash` / `glm-4v-flash` / `glm-4.6v-flash` / `glm-4.1v-thinking-flash` / `cogview-3-flash` / `cogvideox-flash`。

> 免费档与付费档共享 API key 与端点，但限速更严格，详见 https://docs.bigmodel.cn/cn/api/rate-limit。

## Tokenizer · POST /paas/v4/tokenizer

按模型计算文本 token 数。常用于估算请求成本。

```json
{
  "model": "glm-5.1",
  "messages": [{"role": "user", "content": "你好"}]
}
```

响应：

```json
{
  "model": "glm-5.1",
  "usage": {"prompt_tokens": 2, "completion_tokens": 0, "total_tokens": 2}
}
```

详见 https://docs.bigmodel.cn/api-reference/模型-api/文本分词器。

## Rerank · POST /paas/v4/rerank

文本相关性重排。常用于检索 + 重排两阶段管线。

```json
{
  "model": "rerank-2",
  "query": "什么是大语言模型？",
  "documents": ["LLM 是...", "大模型指...", "..."]
}
```

详见 https://docs.bigmodel.cn/api-reference/模型-api/文本重排序。

## 参考

- 模型总览：https://docs.bigmodel.cn/cn/guide/start/model-overview
- GLM-4.5 / 4.6 / 4.7 / 5 / 5.1 各自细节页：https://docs.bigmodel.cn/cn/guide/models/text/
- 视觉模型：https://docs.bigmodel.cn/cn/guide/models/vlm/
- 图像 / 视频 / 音视频：https://docs.bigmodel.cn/cn/guide/models/image-generation/、`.../video-generation/`、`.../sound-and-video/`
- 嵌入：https://docs.bigmodel.cn/cn/guide/models/embedding/
- 免费：https://docs.bigmodel.cn/cn/guide/models/free/
- 迁移到 GLM-5.1：https://docs.bigmodel.cn/cn/guide/start/migrate-to-glm-new
