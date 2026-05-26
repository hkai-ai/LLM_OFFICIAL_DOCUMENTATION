---
source: https://platform.kimi.com/docs/guide/use-web-search
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · 联网搜索（Web Search）

通过内置工具 `$web_search` 让模型在生成回答前先搜索互联网。

## 启用方式

声明为 [chat-completions.md](./chat-completions.md) 的 `tools` 数组中的 `builtin_function` 类型：

```json
{
  "model": "kimi-k2.6",
  "messages": [{"role": "user", "content": "最近一周特斯拉股价怎么样？"}],
  "tools": [
    { "type": "builtin_function", "function": { "name": "$web_search" } }
  ]
}
```

可与用户自定义 `function` 工具混用，详见 [tool-use.md](./tool-use.md)。

## 工作流程

1. 模型判断需要联网，产生 `tool_calls`，其中 `function.name == "$web_search"`，`function.arguments` 为 JSON 字符串（包含 query 等参数）。
2. **客户端把 arguments 原样作为 `role=tool` 的 `content` 回传**——Kimi 服务端会**代为执行**实际检索，模型在下一轮看到检索结果并生成最终答复。
3. 调用方**无需自己实现搜索 / 爬取逻辑**。

```json
{
  "role": "tool",
  "tool_call_id": "call_search_001",
  "content": "{\"query\":\"特斯拉股价 最近一周\"}"
}
```

## 推荐模型

- 推荐 `kimi-k2.6`（搜索能力与上下文承载最强；返回结果会显著增加 input token 用量）。
- 其他支持模型详见官方页。

## 计费

| 项 | 价格 |
| --- | --- |
| `$web_search` 单次触发 | **¥0.03 / 次** |
| 搜索内容引入的 token | 按对应模型 input 价单独计费 |

详见 [pricing.md §4 工具定价](./pricing.md)。

## 与其他能力的组合

| 组合 | 兼容性 |
| --- | --- |
| 思考模式（thinking） | ✓ 模型在思考阶段决定是否触发 `$web_search` |
| JSON Mode | ✓ 检索内容作为输入；最终输出仍受 JSON 格式约束 |
| 自定义 function tool | ✓ 可与 `$web_search` 在同一请求内共存 |
| Partial Mode | 不推荐：partial 是续写 assistant 内容，与工具调用流程冲突 |

## 完整示例

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "user", "content": "查一下今天北京天气"}
  ],
  "tools": [
    {"type": "builtin_function", "function": {"name": "$web_search"}}
  ]
}
```

模型可能首轮返回：

```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "tool_calls": [{
        "id": "call_ws_001",
        "type": "builtin_function",
        "function": {"name": "$web_search", "arguments": "{\"query\":\"北京 今天 天气\"}"}
      }]
    },
    "finish_reason": "tool_calls"
  }]
}
```

客户端原样回传 arguments：

```json
{
  "role": "tool",
  "tool_call_id": "call_ws_001",
  "content": "{\"query\":\"北京 今天 天气\"}"
}
```

模型下一轮基于服务端检索结果生成最终回答。

## 错误码

| 场景 | 现象 |
| --- | --- |
| 搜索服务暂时不可用 | 工具调用 finish；`content` 中说明"暂时无法访问" |
| 触发限流 | 标准 `1002` 错误，见 [errors.md](./errors.md) |

## 参考

- Web Search Guide：https://platform.kimi.com/docs/guide/use-web-search
- 工具调用 API：[tool-use.md](./tool-use.md)
- Web Search 定价：https://platform.kimi.com/docs/pricing/tools
