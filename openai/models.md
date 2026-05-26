---
source: https://developers.openai.com/api/reference/resources/models
fetched_at: 2026-05-19
api_version: N/A
---

# Models · /v1/models

> 列出 / 检索 / 删除模型，以及当前主推模型清单。具体模型生命周期以 `https://developers.openai.com/api/docs/models` 为准。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <OPENAI_API_KEY>` |

---

## GET /v1/models

列出当前账号可访问的所有模型。

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `list`。 |
| `data[]` | array | model object 数组。 |

### model object

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 模型 ID（API 调用使用）。 |
| `object` | string | 固定 `model`。 |
| `created` | integer | Unix 时间戳。 |
| `owned_by` | string | 拥有方，如 `openai` / `openai-internal` / `system` / `<your-org>`。 |

---

## GET /v1/models/{model}

返回单个 model object。

---

## DELETE /v1/models/{model}

删除自有 fine-tuned 模型（不能删除 OpenAI 基础模型）。

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 被删模型 ID。 |
| `object` | string | 固定 `model.deleted`。 |
| `deleted` | boolean | `true`。 |

---

## 当前主推模型清单

> 文档站快速变动，以官方页面为准。下表为 fetched_at 时点呈现。

### 旗舰文本 / 多模态

| Model ID | 说明 |
| --- | --- |
| `gpt-5` | 旗舰主力，长上下文 + 推理 + 多模态。 |
| `gpt-5-mini` | 平衡性价比版本。 |
| `gpt-5-nano` | 低延迟轻量版本。 |
| `gpt-5-codex` | 代码增强变体。 |
| `gpt-4.1` | 上一代旗舰，仍稳定提供。 |
| `gpt-4.1-mini` | — |
| `gpt-4.1-nano` | — |
| `gpt-4o` | 多模态 GA，文本 + 视觉 + 音频。 |
| `gpt-4o-mini` | — |
| `chatgpt-4o-latest` | 跟随 ChatGPT 产品更新的滚动 alias。 |

### 推理（o 系列）

| Model ID | 说明 |
| --- | --- |
| `o4-mini` | — |
| `o3` | — |
| `o3-mini` | — |
| `o1` | 早期推理模型，逐步退役。 |
| `o1-mini` | — |

### 实时 / 语音对话

| Model ID | 说明 |
| --- | --- |
| `gpt-realtime` | 当前 GA 的 Realtime 模型。 |
| `gpt-4o-realtime-preview` | 旧版预览。 |
| `gpt-4o-audio-preview` | Chat Completions 中的音频输入输出。 |
| `gpt-4o-mini-audio-preview` | — |

### Audio 专用

| Model ID | 说明 |
| --- | --- |
| `gpt-4o-mini-tts` | 文字转语音，支持 `instructions`。 |
| `tts-1` | 经典 TTS。 |
| `tts-1-hd` | 高保真 TTS。 |
| `gpt-4o-transcribe` | 新一代 STT。 |
| `gpt-4o-mini-transcribe` | 轻量 STT。 |
| `gpt-4o-transcribe-diarize` | 带说话人分离。 |
| `whisper-1` | 经典 STT（Whisper V2）。 |

### Embeddings

| Model ID | 默认维度 | 说明 |
| --- | --- | --- |
| `text-embedding-3-small` | 1536 | 主推小模型，支持 `dimensions` 裁剪。 |
| `text-embedding-3-large` | 3072 | 高质量，支持 `dimensions` 裁剪。 |
| `text-embedding-ada-002` | 1536 | 维护态遗留。 |

### Image

| Model ID | 说明 |
| --- | --- |
| `gpt-image-1.5` | 最新文生图。 |
| `gpt-image-1` | 主力文生图 / 编辑。 |
| `gpt-image-1-mini` | 轻量。 |
| `chatgpt-image-latest` | 跟随 ChatGPT。 |
| `dall-e-3` | 遗留。 |
| `dall-e-2` | 遗留，仅 variations 端点支持。 |

### Moderation

| Model ID | 说明 |
| --- | --- |
| `omni-moderation-latest` | 多模态 moderation 主推。 |
| `omni-moderation-2024-09-26` | 固定日期版。 |
| `text-moderation-latest` | 旧版文本 moderation。 |
| `text-moderation-stable` | — |

### Legacy completions

| Model ID | 说明 |
| --- | --- |
| `gpt-3.5-turbo-instruct` | 用于 `/v1/completions`。 |
| `babbage-002` / `davinci-002` | 仅 fine-tuning 基底。 |

---

## 命名约定

- 带日期后缀（如 `gpt-5-2026-01-15`）为快照版本，行为固定。
- 不带日期的 ID（如 `gpt-5`）是滚动 alias，可能在新版本上线后切换底层快照。
- `*-preview` 表示功能预览，不保证向后兼容。
- `chatgpt-*-latest` 跟随 ChatGPT 产品而非 API 节奏，更新更激进。

## 参考

- 端点目录：<https://developers.openai.com/api/reference/resources/models>
- 全部模型清单：<https://developers.openai.com/api/docs/models>
- Deprecations：<https://developers.openai.com/api/docs/deprecations>
