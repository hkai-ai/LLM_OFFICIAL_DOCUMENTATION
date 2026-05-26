---
source: https://help.aliyun.com/zh/model-studio/models
fetched_at: 2026-05-20
api_version: N/A
---

# 模型清单与定价

百炼托管 Qwen 系列与多家第三方模型；完整模型 ID、上下文窗口、单价以**模型广场**（https://bailian.console.aliyun.com/?tab=model#/model-market）为权威源。本文件只列归类与官方文档已明示的关键属性，**未明示的字段标注「文档未说明」**而非臆测。

## 列出模型 · GET /compatible-mode/v1/models

OpenAI 兼容模式提供同名端点；返回结构与 OpenAI 一致：

```json
{
  "object": "list",
  "data": [
    {"id": "qwen3.6-plus", "object": "model", "created": 1735000000, "owned_by": "system"}
  ]
}
```

字段：`id` / `object` / `created` / `owned_by`。注意此处不像 Moonshot 返回 `context_length` / `supports_*` 等扩展字段，需自行查模型广场。

## 模型分类

### Qwen 通义千问 · 文本生成

| 模型 ID | 系列定位 | 思考模式 | 仅流式 |
| --- | --- | --- | --- |
| `qwen3.6-max-preview` | 旗舰版 | 可开 / 关 | 部分变体 |
| `qwen3.6-plus` | 主力商业版 | 可开 / 关 | 思考模式下仅流式 |
| `qwen3.6-flash` | 高吞吐轻量 | 可开 / 关 | 同上 |
| `qwen-mt-*` | 翻译特化 | ✗ | 否 |
| `qwen-coder-*` | 代码特化 | ✗ | 否 |
| `qwq-*` | 推理特化（数学 / 代码） | ✓ | ✓ |

### Qwen 视觉与多模态

| 模型 ID | 模态 | 思考 | 仅流式 |
| --- | --- | --- | --- |
| `qwen3.5-omni-plus` | 文本 + 图 + 视频 + 音频 | ✗ | 部分变体 |
| `qwen3.5-omni-plus-realtime` | + 实时音频 | ✗ | ✓ |
| `qwen-vl-*` | 文本 + 图像 | ✗ | ✗ |
| `qvq-*` | 视觉推理 | ✓ | ✓ |

### Qwen Embedding / Rerank / 3D

| 模型 ID | 用途 |
| --- | --- |
| `text-embedding-v4` | 文本向量 |
| `tongyi-embedding-vision-plus` | 多模态向量 |
| `qwen3-rerank` | 检索重排 |
| `Tripo-H3.1` / `Tripo-P1.0` | 3D 模型生成 |

### 图像 / 音频 / 视频 / 音乐

| 模型 ID | 用途 |
| --- | --- |
| `wan2.7-image-pro` | 文生图（万相） |
| `qwen-image-2.0-pro` | 千问文生图 |
| `cosyvoice-v3.5-plus` | 语音合成 |
| `fun-asr` / `fun-asr-realtime` | 语音识别 |
| `fun-music-v1` | 音乐生成 |

### 第三方聚合模型

通过百炼 API Key 统一调用，鉴权与 base URL 同 Qwen：

| 模型 ID | 来源 |
| --- | --- |
| `deepseek-v4-pro` / `deepseek-v4-flash` | DeepSeek |
| `kimi-k2.6` | Moonshot |
| `glm-5.1` | 智谱 |
| `MiniMax-M2.7` | MiniMax |
| `mimo-v2.5-pro` | 小米 MiMo |

> 第三方模型在百炼侧的字段命名 / 流式行为遵循百炼自家协议（OpenAI 兼容 / DashScope），与原厂端点不必然一致；切勿混用 base URL 与 Key。

## 上下文窗口、最大输出、定价

官方未在单一文档页集中列出全部模型的上下文 / 单价矩阵，且数值随版本调整频繁，请直接查阅模型广场或对应模型详情页。常见规律：

- 同一系列的 `*-plus` / `*-flash` / `*-turbo` 通常上下文窗口一致（数十万到上百万 token），价格分档；
- 思考模式产生的 `reasoning_tokens` 全部按 `output_tokens` 计费；
- 显式缓存命中按缓存输入价（通常为非命中价的 10%–25%）计费；
- 联网搜索按 `search_strategy` 单独计费（`turbo` < `max` < `agent` < `agent_max`）。

## 参考

- 模型清单页：https://help.aliyun.com/zh/model-studio/models
- 模型广场（含价格）：https://bailian.console.aliyun.com/?tab=model#/model-market
- API 参考入口：https://help.aliyun.com/zh/model-studio/model-api-reference/
