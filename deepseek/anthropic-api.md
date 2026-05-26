---
source: https://api-docs.deepseek.com/zh-cn/guides/anthropic_api
fetched_at: 2026-05-26
api_version: anthropic-version 兼容（请求中传入会被忽略）
---

# Anthropic API 兼容 · /anthropic/v1/messages

DeepSeek 提供与 Anthropic Messages API 兼容的入口，可让原本调用 Claude 的代码（含 `@anthropic-ai/sdk` / `anthropic` Python SDK）通过仅切换 base URL + API Key 直接迁移到 DeepSeek。

## Base URL

```text
https://api.deepseek.com/anthropic
```

SDK 配置示例：

```python
import anthropic
client = anthropic.Anthropic(
    base_url="https://api.deepseek.com/anthropic",
    api_key="<DEEPSEEK_API_KEY>",
)
```

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_API_KEY=<DEEPSEEK_API_KEY>
```

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` | ✓ | DeepSeek API Key。 |
| `Content-Type` | ✓ | `application/json`。 |
| `anthropic-version` | ✗ | 忽略；保持原值不会报错。 |
| `anthropic-beta` | ✗ | 忽略。 |

## 支持的端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Messages | POST | `/anthropic/v1/messages` |
| Messages 流式 | POST | `/anthropic/v1/messages`（`stream: true`） |

> 其余 Anthropic 端点（count_tokens / batches / files / models / skills / managed-agents 等）暂不提供。

## 字段映射

### 模型 ID

| Anthropic 模型名（请求） | 实际路由 |
| --- | --- |
| `claude-opus-*` | `deepseek-v4-pro` |
| `claude-sonnet-*` / `claude-haiku-*` | `deepseek-v4-flash` |
| 其余字符串 | `deepseek-v4-flash`（兜底） |

### 请求字段

参考 Anthropic Messages API 字段（详见 [anthropic/messages.md](../anthropic/messages.md)）。下表只列 **DeepSeek 兼容侧的差异**：

| 字段 | 支持情况 | 说明 |
| --- | --- | --- |
| `model` | ✓（映射后） | 见上表。 |
| `messages` | ✓ | 仅 `content` 为字符串或 `TextBlockParam` 数组；**图像 / 文档 / 工具结果 / 引用等其他 block 全部不支持**。 |
| `system` | ✓ | string 或 `TextBlockParam` 数组。 |
| `max_tokens` | ✓ | — |
| `temperature` / `top_p` | ✓ | — |
| `top_k` | ✗（忽略） | — |
| `stop_sequences` | ✓ | — |
| `stream` | ✓ | SSE 协议同 Anthropic 原生。 |
| `tools` / `tool_choice` | ✓ | 函数定义保留，不支持 server tools（web_search / code_execution 等）。 |
| `thinking` | ✓ | `type: enabled` 启用思考；**`budget_tokens` 字段被忽略**，由 DeepSeek 内部策略决定。 |
| `cache_control` | ✗（忽略） | DeepSeek 用自动硬盘缓存（见 [caching.md](./caching.md)）。 |
| `metadata.user_id` | ✓ | 用于按用户做限流隔离。 |
| `metadata.*` 其他键 | ✗（忽略） | — |
| `mcp_servers` | ✗（忽略） | — |
| `container` | ✗（忽略） | — |

### 响应字段

返回结构遵循 Anthropic Messages 响应：`id` / `type: "message"` / `role: "assistant"` / `content[]`（仅 `text` + `thinking` 两种 block）/ `model`（回显映射后的模型 ID）/ `stop_reason` / `usage.input_tokens` / `usage.output_tokens`。

> `usage` 中**不包含** `cache_creation_input_tokens` / `cache_read_input_tokens`；自动缓存命中的体现请改查 DeepSeek 原生 chat completions 的 `prompt_cache_hit_tokens` / `prompt_cache_miss_tokens`（详见 [caching.md](./caching.md)）。

## 最小请求

```bash
curl https://api.deepseek.com/anthropic/v1/messages \
  -H "x-api-key: $DEEPSEEK_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-4-7",
    "max_tokens": 1024,
    "messages": [{"role":"user","content":"Hello"}]
  }'
```

## 计费

按 `deepseek-v4-pro` / `deepseek-v4-flash` 实际命中的模型计费，单价见 [pricing.md](./pricing.md)。Anthropic 协议本身不收转换费。

## 限制

- 仅文本对话；多模态 / 文件 / 工具结果一律不支持。
- 限流与 DeepSeek 原生接口共享同一池（详见 [rate-limits.md](./rate-limits.md)）。
- `tools` 数组中只支持 `custom` / 函数定义，无 `type: "web_search_..."` 等 server tools。

## 参考

- 兼容指南：<https://api-docs.deepseek.com/zh-cn/guides/anthropic_api>
- DeepSeek 原生 chat 接口：[chat-completions.md](./chat-completions.md)
- Anthropic Messages：<https://platform.claude.com/docs/en/api/messages>
