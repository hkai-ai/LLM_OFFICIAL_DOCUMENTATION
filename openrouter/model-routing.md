---
source: https://openrouter.ai/docs/guides/routing/model-fallbacks
fetched_at: 2026-05-19
api_version: N/A
---

# 模型路由与回退

> OpenRouter 支持在一次请求中提供多个候选模型，或使用元模型 `openrouter/auto` 自动挑选合适模型。

## `models` + `route: "fallback"`：多模型回退

在请求体内传入 `models` 数组并设置 `route: "fallback"`：

```json
{
  "models": [
    "anthropic/claude-3.5-sonnet",
    "openai/gpt-4o",
    "google/gemini-1.5-pro"
  ],
  "route": "fallback",
  "messages": [{"role": "user", "content": "Hi"}]
}
```

### 行为

- 按数组顺序优先级尝试，首个模型失败则按顺序回退到下一个。
- 任何下列错误都会触发回退：
  - 上下文长度校验错误。
  - 内容审核被标记。
  - 速率限制（429）。
  - provider 服务下线（502/503）。
- 实际使用的模型在响应的 `model` 字段中返回。
- 计费按实际使用的模型计算。

### 与 `provider` 的关系

- `provider.allow_fallbacks` 控制 **provider 层** 是否回退；`route: "fallback"` 控制 **模型层** 是否回退。两层独立，可同时启用。
- 配合 `provider.sort.partition`：
  - 默认 `partition: "model"`：先尝试主模型所有 endpoint，再切到下一个模型。
  - `partition: "none"`：全部候选 endpoint 一起按 `sort` 全局排序，跨模型混排。

### 与 OpenAI SDK 的兼容

OpenAI SDK 没有 `models` 字段，需把它放进 `extra_body`（Python）或 `body` 透传字段：

```python
client.chat.completions.create(
    model="anthropic/claude-3.5-sonnet",
    extra_body={
        "models": ["anthropic/claude-3.5-sonnet", "openai/gpt-4o"],
        "route": "fallback",
    },
    messages=[...],
)
```

## `openrouter/auto`：Auto Router

将 `model` 设为 `openrouter/auto`，由 OpenRouter 基于 prompt 内容自动挑选合适模型（底层使用 NotDiamond）。

```json
{
  "model": "openrouter/auto",
  "messages": [{"role": "user", "content": "Write a haiku about TypeScript"}]
}
```

### 关键属性

- **无额外费用**：路由本身免费，仅按实际选中的模型计费。
- **结果可追溯**：响应 `model` 字段为最终选中的模型 ID。
- **候选限制**：通过 `plugins` 参数限制 Auto Router 可挑选的模型池，支持通配符：

  ```json
  {
    "model": "openrouter/auto",
    "plugins": [
      {
        "id": "auto-router",
        "allowed_models": ["anthropic/*", "openai/gpt-5*"]
      }
    ],
    "messages": [...]
  }
  ```

### 其他路由策略

`openrouter_metadata.strategy` 字段（在 `X-OpenRouter-Experimental-Metadata: enabled` 时返回）显示当次路由策略，可能取值：

- `direct`：直接路由到指定模型。
- `auto`：Auto Router 选择。
- `free`：路由到免费版本。
- `latest`：路由到最新版本。
- `alias`：通过别名解析。
- `fallback`：模型回退触发。
- `pareto`：Pareto-router 选择（多目标平衡）。
- `bodybuilder`：内部策略。
- `fusion`：Fusion 插件融合多模型输出。

## 参考

- 模型回退指南：<https://openrouter.ai/docs/guides/routing/model-fallbacks>
- Auto Router：<https://openrouter.ai/docs/guides/routing/routers/auto-router>
- Provider Routing：<https://openrouter.ai/docs/guides/routing/provider-selection>
