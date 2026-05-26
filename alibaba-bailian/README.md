---
source: https://help.aliyun.com/zh/model-studio/model-api-reference/
fetched_at: 2026-05-20
api_version: N/A（DashScope 路径含 `/api/v1`；OpenAI 兼容模式仅用 `/compatible-mode/v1` 前缀）
last_updated: 2026-05-20
---

# 阿里百炼 Model Studio API 概览

阿里云大模型服务平台「百炼」（Model Studio）以 DashScope 为底座，对外暴露 4 套并列协议：

1. **OpenAI 兼容 - Chat Completions**（推荐迁移现有 OpenAI 代码）
2. **OpenAI 兼容 - Responses**（带内置 web_search / code_interpreter / web_extract 与会话续接）
3. **Anthropic 兼容 - Messages**（兼容 Anthropic SDK，支持 thinking / tools）
4. **DashScope 原生 generation**（功能最全，参数最完整，含多模态、视频、3D、向量等所有能力）

百炼托管自家通义千问（Qwen）系列，也聚合第三方模型（DeepSeek、Kimi、GLM、MiniMax 等）。本仓库以**OpenAI 兼容模式**与 **DashScope 原生模式**两条线为主。

## Base URL（按地域）

| 地域 | OpenAI 兼容 | DashScope 原生 |
| --- | --- | --- |
| 华北 2（北京） | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `https://dashscope.aliyuncs.com/api/v1` |
| 新加坡 | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` | `https://dashscope-intl.aliyuncs.com/api/v1` |
| 美西（弗吉尼亚） | `https://dashscope-us.aliyuncs.com/compatible-mode/v1` | `https://dashscope-us.aliyuncs.com/api/v1` |
| 德国（法兰克福） | `https://{WorkspaceId}.eu-central-1.maas.aliyuncs.com/compatible-mode/v1` | 同左 base / `api/v1` |
| Batch 专用 | `https://batch.dashscope.aliyuncs.com/compatible-mode/v1` | — |

> **不同地域 API Key 不互通**；切换地域必须同时切换 API Key 与 base URL。

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DASHSCOPE_API_KEY}` |
| `Content-Type` | ✓ | `application/json` |
| `X-DashScope-SSE` | DashScope 流式 ✓ | `enable`，开启 SSE 流（OpenAI 兼容模式用 `stream: true`，不需要此 header）。 |
| `X-DashScope-WorkSpace` | 多业务空间时 ✓ | 指定使用的子业务空间 ID。 |
| `x-dashscope-session-cache` | Responses 可选 | `enable` 开启会话缓存，自动缓存 `previous_response_id` 上下文。 |

API Key 在 https://bailian.console.aliyun.com/?tab=model#/api-key 创建，按业务空间隔离；子用户需「管理员」或 API-Key 权限。

## 端点索引

### OpenAI 兼容模式

| 端点 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| Chat Completions | POST | `/compatible-mode/v1/chat/completions` | [chat-completions.md](./chat-completions.md) |
| Responses | POST | `/compatible-mode/v1/responses` | [responses.md](./responses.md) |
| Embeddings | POST | `/compatible-mode/v1/embeddings` | [embeddings.md](./embeddings.md) |
| Batch Chat | POST | `/compatible-mode/v1/batches` | [batch.md](./batch.md) |
| Models | GET | `/compatible-mode/v1/models` | [models.md](./models.md) |

### DashScope 原生

| 端点 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 文本生成 | POST | `/api/v1/services/aigc/text-generation/generation` | [dashscope-generation.md](./dashscope-generation.md) |
| 多模态生成 | POST | `/api/v1/services/aigc/multimodal-generation/generation` | [dashscope-generation.md](./dashscope-generation.md) §多模态 |
| 通用文本向量 | POST | `/api/v1/services/embeddings/text-embedding/text-embedding` | [embeddings.md](./embeddings.md) §dashscope |

辅助主题：

| 主题 | 文档 |
| --- | --- |
| 错误码（含 DashScope 与 OpenAI 兼容差异） | [errors.md](./errors.md) |
| 模型清单与定价 | [models.md](./models.md) |
| 思考模式 / Reasoning | [chat-completions.md](./chat-completions.md) §enable_thinking |
| 联网搜索 | [chat-completions.md](./chat-completions.md) §enable_search |
| 显式缓存（cache_control） | [chat-completions.md](./chat-completions.md) §cache_control |

## 与 OpenAI SDK 的兼容性

- 完整字段名、SSE 格式、`messages[]` 结构与 OpenAI Chat Completions 一致。
- Python：`OpenAI(api_key=..., base_url="https://dashscope.aliyuncs.com/compatible-mode/v1")`。
- 百炼特有字段（`enable_thinking` / `thinking_budget` / `enable_search` / `search_options` / `translation_options` / `cache_control` 等）通过 SDK 的 `extra_body` 透传。
- 思考模式开启时，模型在 `choices[].message.reasoning_content` 输出思维链（与 DeepSeek、Moonshot 同名）。
- Qwen3 商业版（思考模式）、Qwen3 开源版、QwQ、QVQ 等**仅支持流式**返回，必须 `stream: true`。

## 模型清单速查

> 完整清单与价格见 [models.md](./models.md)。下表仅为旗舰系列摘要，价格、上下文以官方[模型广场](https://bailian.console.aliyun.com/?tab=model#/model-market)为准。

| 模型 ID | 系列 | 上下文 | 思考 | 多模态 | 用途 |
| --- | --- | --- | --- | --- | --- |
| `qwen3.6-max-preview` | Qwen 旗舰 | 文档未明确 | 可开 | 文本 | 通用复杂推理 |
| `qwen3.6-plus` | Qwen 主力 | 文档未明确 | 可开 | 文本 | 性价比对话 |
| `qwen3.6-flash` | Qwen 轻量 | 文档未明确 | 可开 | 文本 | 高吞吐低成本 |
| `qwen3.5-omni-plus` | 多模态 | 文档未明确 | — | 文本 / 图 / 视频 / 音频 | Omni 全模态 |
| `qwen3.5-omni-plus-realtime` | 实时多模态 | — | — | 同上 + 流式音频 | 实时语音交互 |
| `qwen-vl-*` | 视觉系列 | — | — | 文本 + 图像 | 视觉理解 |
| `qwen-coder-*` | 代码系列 | — | — | 代码 | 编程辅助 |
| `qvq-*` | 视觉推理 | — | ✓（仅流式） | 文本 + 图像 | 多模态推理 |
| `qwq-*` | 推理特化 | — | ✓（仅流式） | 文本 | 数学 / 代码推理 |
| `text-embedding-v4` | 向量 | — | — | 文本 | Embedding |

聚合的第三方模型示例：`deepseek-v4-pro` / `deepseek-v4-flash` / `kimi-k2.6` / `glm-5.1` / `MiniMax-M2.7` 等，统一通过百炼 endpoint 调用，鉴权使用 `DASHSCOPE_API_KEY`。

## 计费要点

- 按模型计价，输入 / 输出分档，部分模型支持显式缓存（`cache_control` + `cache_creation.ephemeral_5m_input_tokens` 字段）。
- 思考模式产生的 `reasoning_tokens` 计入 `output_tokens`。
- 联网搜索（`enable_search`）按 `search_strategy` 单独计费。
- Batch Chat（异步批量）享受 50% 折扣，独立 endpoint。

## 参考

- API 参考入口：https://help.aliyun.com/zh/model-studio/model-api-reference/
- 文本生成总览：https://help.aliyun.com/zh/model-studio/text-generation
- 首次调用千问：https://help.aliyun.com/zh/model-studio/first-api-call-to-qwen
- 获取 API Key：https://help.aliyun.com/zh/model-studio/get-api-key
- 错误码：https://help.aliyun.com/zh/model-studio/error-code
- 模型广场：https://bailian.console.aliyun.com/?tab=model#/model-market
