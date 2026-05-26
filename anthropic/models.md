---
source: https://platform.claude.com/docs/en/api/models-list
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# Models · GET /v1/models 与 GET /v1/models/{model_id}

> 列出可用模型，或根据 ID / alias 查询单个模型详情。可用于解析 alias 到具体模型 ID、读取上下文窗口和能力矩阵。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` | 二选一 | API Key。 |
| `anthropic-version` | ✓ | 例如 `2023-06-01`。 |
| `anthropic-beta` | ✗ | 可选 beta 名（数组形式，多个用逗号）。 |

---

## List 模型 · `GET /v1/models`

按发布时间倒序返回可用模型。

### 查询参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `after_id` | string | ✗ | — | 游标，返回此 ID 之后的一页。 |
| `before_id` | string | ✗ | — | 游标，返回此 ID 之前的一页。 |
| `limit` | integer | ✗ | `20` | 每页数量，范围 `1`–`1000`。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `data` | array&lt;ModelInfo&gt; | 模型对象数组。详见 [`ModelInfo`](#modelinfo)。 |
| `has_more` | boolean | 是否还有下一页。 |
| `first_id` | string | 当前页第一个 ID（可作为下一页向前翻的 `before_id`）。 |
| `last_id` | string | 当前页最后一个 ID（可作为下一页向后翻的 `after_id`）。 |

### 示例

```bash
curl https://api.anthropic.com/v1/models \
  -H 'anthropic-version: 2023-06-01' \
  -H "X-Api-Key: $ANTHROPIC_API_KEY"
```

---

## Retrieve 模型 · `GET /v1/models/{model_id}`

按 ID 或 alias 获取单个模型。Alias 会被解析为具体的快照 ID。

### 路径参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model_id` | string | ✓ | 模型 ID 或 alias，如 `claude-opus-4-7` / `claude-haiku-4-5-20251001`。 |

### 响应

返回单个 [`ModelInfo`](#modelinfo) 对象。

### 示例

```bash
curl https://api.anthropic.com/v1/models/claude-opus-4-7 \
  -H 'anthropic-version: 2023-06-01' \
  -H "X-Api-Key: $ANTHROPIC_API_KEY"
```

---

## `ModelInfo`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 唯一模型 ID。 |
| `type` | `"model"` | 固定值。 |
| `display_name` | string | 人类可读名称。 |
| `created_at` | string | RFC 3339 时间字符串。未知则可能是 epoch 值。 |
| `max_input_tokens` | integer | 上下文窗口大小（输入 token）。 |
| `max_tokens` | integer | 调用时 `max_tokens` 参数允许的最大值。 |
| `capabilities` | object | 能力矩阵，详见 [`ModelCapabilities`](#modelcapabilities)。 |

### `ModelCapabilities`

各字段统一形式为 `{ "supported": boolean }`（即 `CapabilitySupport`），表示该能力在此模型上是否可用：

| 字段 | 含义 |
| --- | --- |
| `batch` | 是否支持 Message Batches API。 |
| `citations` | 是否支持 citations。 |
| `code_execution` | 是否支持 code_execution 内置工具。 |
| `image_input` | 是否支持 image content block。 |
| `pdf_input` | 是否支持 PDF document content block。 |
| `structured_outputs` | 是否支持结构化输出 / JSON mode / strict 工具 schema。 |
| `thinking` | 嵌套 `ThinkingCapability`（见下）。 |
| `effort` | 嵌套 `EffortCapability`（见下）。 |
| `context_management` | 嵌套 `ContextManagementCapability`（见下）。 |

#### `ThinkingCapability`

| 字段 | 说明 |
| --- | --- |
| `supported` | 是否整体支持 thinking。 |
| `types.adaptive` | 是否支持 `thinking.type = "adaptive"`。 |
| `types.enabled` | 是否支持 `thinking.type = "enabled"`。 |

#### `EffortCapability`

| 字段 | 说明 |
| --- | --- |
| `supported` | 是否支持 `output_config.effort`。 |
| `low` / `medium` / `high` / `xhigh` / `max` | 各档位是否支持。 |

#### `ContextManagementCapability`

| 字段 | 说明 |
| --- | --- |
| `supported` | 是否整体支持 context management。 |
| `clear_thinking_20251015` | 是否支持清理 thinking 策略。 |
| `clear_tool_uses_20250919` | 是否支持清理 tool_uses 策略。 |
| `compact_20260112` | 是否支持 compact 策略。 |

---

## 模型清单（截至 2026-05-19）

### 主推模型

| 模型 | API ID | API alias | 上下文窗口 | 最大输出 | 扩展思考 | adaptive thinking |
| --- | --- | --- | --- | --- | --- | --- |
| Claude Opus 4.7 | `claude-opus-4-7` | `claude-opus-4-7` | 1M tokens | 128k | No | Yes |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | `claude-sonnet-4-6` | 1M tokens | 64k | Yes | Yes |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | `claude-haiku-4-5` | 200k tokens | 64k | Yes | No |

注：

- Opus 4.7 使用新 tokenizer，1M tokens 约对应 555k 词。
- Opus 4.7、Opus 4.6、Sonnet 4.6 在 Message Batches API 上可通过 `output-300k-2026-03-24` beta header 输出至 300k token。
- Claude Mythos Preview 为研究预览模型（Project Glasswing），邀请制，无自助开通。

### 历史 / 兼容模型

| 模型 | API ID | API alias | 上下文窗口 | 最大输出 | 备注 |
| --- | --- | --- | --- | --- | --- |
| Claude Opus 4.6 | `claude-opus-4-6` | `claude-opus-4-6` | 1M tokens | 128k | — |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | `claude-sonnet-4-5` | 200k | 64k | — |
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | `claude-opus-4-5` | 200k | 64k | — |
| Claude Opus 4.1 | `claude-opus-4-1-20250805` | `claude-opus-4-1` | 200k | 32k | — |
| Claude Sonnet 4 | `claude-sonnet-4-20250514` | `claude-sonnet-4-0` | 200k | 64k | deprecated，2026-06-15 退役。 |
| Claude Opus 4 | `claude-opus-4-20250514` | `claude-opus-4-0` | 200k | 32k | deprecated，2026-06-15 退役。 |
| Claude 3 Haiku | `claude-3-haiku-20240307` | — | 200k | 文档未给出 | 列在 messages 模型枚举中。 |

> 模型 ID 与 alias 的规则：4.5 及更早世代，alias 是指向带日期的快照 ID 的便捷指针；4.6 起，dateless ID 本身也是 pinned snapshot，不再是 evergreen。详见 `https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions`。

## 参考

- List：`https://platform.claude.com/docs/en/api/models-list`
- Retrieve：`https://platform.claude.com/docs/en/api/models` （Retrieve 与 List 在同一参考页下）
- 模型对比：`https://platform.claude.com/docs/en/about-claude/models/overview`
- 弃用计划：`https://platform.claude.com/docs/en/about-claude/model-deprecations`
