---
source: https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · 思考模式（Thinking）

K2 系列在生成最终回答前进行一段"内部思考"，通过 `reasoning_content` 单独返回，便于排查模型推理过程并支持多步工具规划。

## 支持模型与默认行为

| 模型 | 是否支持 | 默认开关 |
| --- | --- | --- |
| `kimi-k2.6` | ✓ | **默认开启**，可通过 `thinking.type: disabled` 关闭 |
| `kimi-k2.5` | ✓ | **默认开启**，可关 |
| `moonshot-v1-*` 系列 | ✗ | 不支持 |
| `kimi-k2-*-preview` 系列 | ✗ | 不支持 |

## `thinking` 参数

```json
{
  "thinking": {
    "type": "enabled",
    "keep": "all"
  }
}
```

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `type` | string | `enabled` | `enabled` / `disabled` |
| `keep` | string | — | **K2.6 专属**：多轮对话中保留历史 `reasoning_content` 的策略：`all`（保留全部）、`last_turn`（仅上一轮）、`none`（不保留）。`keep: all` 等价于 "Preserved Thinking"，命中缓存率更高但占 context |

## 响应字段

```json
{
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "reasoning_content": "用户在问北京天气。我需要先调用 $web_search ...",
      "content": "根据搜索结果，今天北京晴朗，气温 20°C..."
    },
    "finish_reason": "stop"
  }]
}
```

- `reasoning_content` 出现在 `content` **之前**：流式响应中先到达的是 `delta.reasoning_content`，思考完成后才开始 `delta.content`。可据此区分阶段并在 UI 上分块展示。
- `reasoning_content` 的 token 计入 `completion_tokens`（按 output 价计费）。

## 与采样参数的关系

| 参数 | 思考模式开启时 | 思考模式关闭时 |
| --- | --- | --- |
| `temperature` | 强制 `1.0`（K2.6 / K2.5 自动） | `0.6` 默认（K2 系列） |
| `top_p` | 强制 `0.95` | `0.95` 默认 |
| `presence_penalty` / `frequency_penalty` / `top_k` | 静默忽略 | 生效 |
| `max_tokens` 推荐 | ≥ `16000`（思考 + 回答都很费 token） | 普通值 |

> 思考模式不会因为传入被忽略的参数而报错，但参数不会生效。

## 多轮对话中的 `reasoning_content` 回传

| 触发条件 | 是否必须把上一轮的 `reasoning_content` 写回 `messages` |
| --- | --- |
| 上一轮包含 `tool_calls` | **必须**回传，否则模型可能丢失工具规划上下文 |
| 上一轮普通文本结尾（`finish_reason: stop`） | 可省略；K2.6 配合 `thinking.keep` 时影响命中率 |

写回结构（K2.6 / K2.5 通用）：

```json
{
  "role": "assistant",
  "reasoning_content": "...",
  "content": "...",
  "tool_calls": [...]
}
```

## 流式事件格式

`stream: true` 时按 OpenAI 风格分块下发，`delta` 中分别出现：

```json
{ "delta": { "reasoning_content": "首先..." } }
{ "delta": { "reasoning_content": " 然后..." } }
{ "delta": { "content": "基于以上分析，" } }
{ "delta": { "content": "答案是..." } }
{ "delta": {}, "finish_reason": "stop" }
```

最后一个 chunk 携带完整 `usage`（含 `cached_tokens` 等）。

## 关闭思考

```json
{
  "model": "kimi-k2.6",
  "messages": [{"role": "user", "content": "..."}],
  "thinking": {"type": "disabled"}
}
```

关闭后：
- 不返回 `reasoning_content`。
- `temperature` 默认 `0.6`，可自定义（K2 系列）。
- 计费仅按 `content` 部分。

## 与其他能力的组合

| 能力 | 与 thinking 兼容性 |
| --- | --- |
| 工具调用（function / `$web_search`） | ✓ 模型在 `reasoning_content` 中规划工具调用 |
| JSON Mode | ✓ 思考中再输出 JSON 格式正文 |
| Partial Mode | ⚠ 不推荐组合：partial 接续 assistant 内容，思考过程不能预先填充 |
| Prompt Caching | ✓ 命中后 `cached_tokens` 体现；`thinking.keep: all` 会让历史思考也参与命中 |

## 计费提示

- `reasoning_content` token 按 **output 价**计费。
- K2.6 / K2.5 的输出价见 [pricing.md](./pricing.md)。
- `thinking.keep: all` 会让历史 reasoning 持续占用 context 长度并产生费用，长会话需评估成本。

## 参考

- 思考模式 Guide：https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model
- K2.6 快速开始：https://platform.kimi.com/docs/guide/kimi-k2-6-quickstart
- Chat Completions：[chat-completions.md](./chat-completions.md)
