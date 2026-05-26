---
source: https://platform.minimaxi.com/docs/api-reference/text-prompt-caching
fetched_at: 2026-05-26
api_version: N/A
---

# Prompt 缓存（被动 + 主动）

MiniMax 提供两套并行的缓存机制：

| 模式 | 入口 | 控制方式 |
| --- | --- | --- |
| **被动缓存（Prompt Caching）** | OpenAI / Anthropic 兼容入口 | 服务端自动按前缀匹配，调用方无需任何字段。 |
| **主动缓存（Anthropic 主动缓存）** | Anthropic 兼容入口 | 客户端显式声明 `cache_control: { type: "ephemeral" }`，类似 Anthropic 原生。 |

> Base URL：OpenAI 兼容 `https://api.minimaxi.com/v1`、Anthropic 兼容 `https://api.minimaxi.com/anthropic`。

## 1. 被动缓存（自动）

### 工作机制

服务端按 **prompt 前缀** 自动匹配；典型拼接顺序为：

```text
tools 定义 → system 消息 → 多轮对话历史
```

任何前缀变更（哪怕一个字符）都会导致后续 token 重新计入「新输入」。

### 启用

OpenAI / Anthropic SDK 均**默认开启**，不需要新字段：

```python
from openai import OpenAI

client = OpenAI(api_key="<API_KEY>", base_url="https://api.minimaxi.com/v1")
resp = client.chat.completions.create(
    model="MiniMax-M2.7",
    messages=[...]
)
print(resp.usage.prompt_tokens_details.cached_tokens)
```

### 命中字段

| SDK / 协议 | 字段 |
| --- | --- |
| OpenAI 兼容 | `usage.prompt_tokens_details.cached_tokens` |
| Anthropic 兼容 | `usage.cache_read_input_tokens` |

> 被动缓存不在 `usage` 中体现「新创建」的 token 数；只能看「读取」字段。

## 2. 主动缓存（Anthropic 兼容入口）

### 启用方式

通过 Anthropic SDK 在 `system` / `messages[].content[]` / `tools[]` 的任一文本 / tool 块上加 `cache_control`：

```python
import anthropic

client = anthropic.Anthropic(
    base_url="https://api.minimaxi.com/anthropic",
    api_key="<API_KEY>",
)

resp = client.messages.create(
    model="MiniMax-M2.7",
    system=[
        {
            "type": "text",
            "text": "static instructions...",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": "..."}
    ]
)
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `cache_control.type` | enum | ✓ | 目前仅 `ephemeral`。 |

单次请求**最多 4 个 cache breakpoint**；超出报 `400 invalid_request_error`。

可缓存内容：

- `system` 块（text）
- `tools[]` 工具定义
- `messages[].content[]` 内任意 `text` / `tool_use` / `tool_result` part

### TTL

- 默认 **5 分钟** ephemeral；每次命中**自动续期** 5 分钟（不额外计费）。
- 不支持 1 小时档（Anthropic 原生独有）。

### 命中字段

响应 `usage` 内：

| 字段 | 说明 |
| --- | --- |
| `cache_creation_input_tokens` | 本轮新写入缓存的 token 数。 |
| `cache_read_input_tokens` | 本轮从缓存读到的 token 数。 |
| `input_tokens` | 最后一个 cache breakpoint **之后**的 token。 |

> 总输入 = `cache_read_input_tokens + cache_creation_input_tokens + input_tokens`。

## 3. 计费倍率

以 MiniMax-M2.7 为例（元 / 1M tokens）：

| 类型 | 单价 | 倍率 |
| --- | --- | --- |
| 标准输入 | `2.1` | `1.0×` |
| 输出 | `8.4` | — |
| 缓存读取（read） | `0.21`（部分模型）/ `0.42`（M2.7） | `0.1×–0.2×` |
| 缓存写入（write） | `2.625` | `1.25×` |

> 其他模型档位见 [pricing.md](./pricing.md)。被动缓存的 `cached_tokens` 与主动缓存的 `cache_read_input_tokens` 计费一致。

## 4. 与思考 / 工具调用

| 维度 | 说明 |
| --- | --- |
| 思考模式（M2.x reasoning） | 兼容；`reasoning_content` / `reasoning_details` 不计入 cache。 |
| 工具调用 | `tools[]` 定义本身可缓存；多任务复用同一组工具时收益最大。 |
| 流式（`stream: true`） | 缓存字段在最后一个 chunk 的 `usage` 中返回（需带 `stream_options.include_usage: true`）。 |

## 实战建议

- 对长 system prompt / 长工具定义 / RAG 检索回来的文档块加 `cache_control` 是收益最稳定的位置。
- 频繁变更对话历史时**不要**给末尾消息加 cache_control，会反复重写缓存反而更贵。
- 90% 命中率场景下整体成本可下降约 75%。

## 参考

- 被动缓存：<https://platform.minimaxi.com/docs/api-reference/text-prompt-caching>
- 主动缓存：<https://platform.minimaxi.com/docs/api-reference/anthropic-api-compatible-cache>
- 定价：[pricing.md](./pricing.md)
- Anthropic 兼容聊天接口：[messages.md](./messages.md)
- OpenAI 兼容聊天接口：[chat-completions.md](./chat-completions.md)
