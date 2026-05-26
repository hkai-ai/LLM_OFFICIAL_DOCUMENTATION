---
source: https://platform.claude.com/docs/en/api/versioning
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# API 版本与 beta header

## `anthropic-version` header

所有 API 请求都必须携带 `anthropic-version` 请求头，例如：

```
anthropic-version: 2023-06-01
```

使用官方 SDK 时由 SDK 自动注入。

### 兼容性承诺

对于给定的版本，Messages API 会保留：

- 现有输入参数；
- 现有输出参数。

但仍可能：

- 新增可选输入；
- 新增输出字段；
- 调整特定错误类型的触发条件；
- 为枚举型输出值（如流式事件类型）增加新变体。

> 按文档原文使用时，不会被 break。

### 版本历史

| 版本 | 主要变化 |
| --- | --- |
| `2023-06-01` | 1) SSE 流式事件改为命名事件（`event: <name>`），不再是 data-only。2) 文本流改为真正的增量（如 `"Hello"`, `" my"`, `" name"`），不再发送累计串。3) 移除 `data: [DONE]` 哨兵。4) 移除响应中废弃的 `exception` / `truncated` 字段。 |
| `2023-01-01` | 初始版本。 |

> 建议始终使用最新的稳定版本。旧版本视为 deprecated，可能对新用户不可用。

## `anthropic-beta` header

`anthropic-beta` 用于启用尚未进入稳定通道的实验性能力。单条 header 可同时声明多个 beta，逗号分隔：

```
anthropic-beta: feature1,feature2,feature3
```

部分 SDK 也支持在请求体里通过 `betas` 字段传入（等价于该 header）。

### 命名约定

通常为 `feature-name-YYYY-MM-DD`，日期是该 beta 的发布日。务必精确匹配。

### 端点级别 beta

某些 beta 不是单个参数级别，而是必须每次请求都带的 header。已知映射：

| 端点 | 必带 beta header |
| --- | --- |
| `/v1/agents` / `/v1/sessions` / `/v1/environments` | `managed-agents-2026-04-01` |

### 当前已发布的 beta 名清单（截至 2026-05-19）

来源：`/v1/models` 端点 `anthropic-beta` 参数的枚举值列表。

| beta | 用途（按名字推断或对应文档） |
| --- | --- |
| `message-batches-2024-09-24` | Message Batches API。 |
| `prompt-caching-2024-07-31` | Prompt caching。 |
| `computer-use-2024-10-22` | Computer use。 |
| `computer-use-2025-01-24` | Computer use 新版本。 |
| `pdfs-2024-09-25` | PDF 输入。 |
| `token-counting-2024-11-01` | Token counting。 |
| `token-efficient-tools-2025-02-19` | 紧凑的工具调用编码。 |
| `output-128k-2025-02-19` | 输出扩展到 128k。 |
| `files-api-2025-04-14` | Files API。 |
| `mcp-client-2025-04-04` | MCP client（旧）。 |
| `mcp-client-2025-11-20` | MCP client（新）。 |
| `dev-full-thinking-2025-05-14` | 完整思考输出（开发者）。 |
| `interleaved-thinking-2025-05-14` | 交错思考。 |
| `code-execution-2025-05-22` | 代码执行工具。 |
| `extended-cache-ttl-2025-04-11` | 扩展 cache TTL（如 `1h`）。 |
| `context-1m-2025-08-07` | 1M token 上下文窗口。 |
| `context-management-2025-06-27` | Context management 策略。 |
| `model-context-window-exceeded-2025-08-26` | 上下文越界错误处理。 |
| `skills-2025-10-02` | Skills API。 |
| `fast-mode-2026-02-01` | Fast mode。 |
| `output-300k-2026-03-24` | 批处理上 300k 输出 token（Opus 4.7、Opus 4.6、Sonnet 4.6）。 |
| `user-profiles-2026-03-24` | 用户画像。 |
| `advisor-tool-2026-03-01` | Advisor 工具。 |
| `managed-agents-2026-04-01` | Managed Agents 端点。 |
| `cache-diagnosis-2026-04-07` | 缓存诊断。 |

> 文档原文未一一解释每个 beta 的语义。具体含义以对应能力页面为准。

### 错误处理

非法或不可用的 beta 名返回 400：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Unsupported beta header: invalid-beta-name"
  }
}
```

### 风险提示（官方原文）

beta 特性可能：

- 在通知前发生 breaking changes；
- 被弃用或移除；
- 限流策略或定价与稳定通道不同；
- 在部分区域不可用。

## 参考

- Versioning：`https://platform.claude.com/docs/en/api/versioning`
- Beta headers：`https://platform.claude.com/docs/en/api/beta-headers`
- 上级目录：`https://platform.claude.com/docs/en/api/overview`
