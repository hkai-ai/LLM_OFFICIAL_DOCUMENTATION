---
source: https://platform.claude.com/docs/en/about-claude/pricing
fetched_at: 2026-05-26
api_version: N/A
---

# Anthropic Claude · 定价

> 全部价格以 **USD** 计，token 计价单位为 **MTok（百万 tokens）**；工具与会话类计价按其各自单位（次 / 小时 / 搜索）。所有数字源自官方 `https://platform.claude.com/docs/en/about-claude/pricing`，本仓库不做汇率换算与推算。

## 1. 模型定价（按 token，标准档）

| 模型 | Base Input | 5m Cache Write | 1h Cache Write | Cache Hits & Refreshes | Output |
| --- | --- | --- | --- | --- | --- |
| `claude-opus-4-7` | $5 / MTok | $6.25 / MTok | $10 / MTok | $0.50 / MTok | $25 / MTok |
| `claude-opus-4-6` | $5 / MTok | $6.25 / MTok | $10 / MTok | $0.50 / MTok | $25 / MTok |
| `claude-opus-4-5` | $5 / MTok | $6.25 / MTok | $10 / MTok | $0.50 / MTok | $25 / MTok |
| `claude-opus-4-1` | $15 / MTok | $18.75 / MTok | $30 / MTok | $1.50 / MTok | $75 / MTok |
| `claude-opus-4` *(deprecated)* | $15 / MTok | $18.75 / MTok | $30 / MTok | $1.50 / MTok | $75 / MTok |
| `claude-sonnet-4-6` | $3 / MTok | $3.75 / MTok | $6 / MTok | $0.30 / MTok | $15 / MTok |
| `claude-sonnet-4-5` | $3 / MTok | $3.75 / MTok | $6 / MTok | $0.30 / MTok | $15 / MTok |
| `claude-sonnet-4` *(deprecated)* | $3 / MTok | $3.75 / MTok | $6 / MTok | $0.30 / MTok | $15 / MTok |
| `claude-haiku-4-5` | $1 / MTok | $1.25 / MTok | $2 / MTok | $0.10 / MTok | $5 / MTok |
| `claude-haiku-3-5` *(retired, except on Bedrock/Vertex AI)* | $0.80 / MTok | $1 / MTok | $1.60 / MTok | $0.08 / MTok | $4 / MTok |

> Opus 4.7 使用新 tokenizer，对相同文本可能多消耗最多 35% tokens。

### 缓存价倍率（相对 Base Input）

| 操作 | 倍率 | 有效期 |
| --- | --- | --- |
| 5-minute cache write | 1.25× | 5 分钟 |
| 1-hour cache write | 2× | 1 小时 |
| Cache hit / refresh | 0.1× | 同上 |

> Cache write tokens 在内容首次落缓存时计费；cache read tokens 在后续命中读取时计费。Batch / data residency / fast mode 等倍率可叠加在缓存价之上。

## 2. Batch API（异步批处理，标准档 50% 折扣）

| 模型 | Batch Input | Batch Output |
| --- | --- | --- |
| `claude-opus-4-7` | $2.50 / MTok | $12.50 / MTok |
| `claude-opus-4-6` | $2.50 / MTok | $12.50 / MTok |
| `claude-opus-4-5` | $2.50 / MTok | $12.50 / MTok |
| `claude-opus-4-1` | $7.50 / MTok | $37.50 / MTok |
| `claude-opus-4` *(deprecated)* | $7.50 / MTok | $37.50 / MTok |
| `claude-sonnet-4-6` | $1.50 / MTok | $7.50 / MTok |
| `claude-sonnet-4-5` | $1.50 / MTok | $7.50 / MTok |
| `claude-sonnet-4` *(deprecated)* | $1.50 / MTok | $7.50 / MTok |
| `claude-haiku-4-5` | $0.50 / MTok | $2.50 / MTok |
| `claude-haiku-3-5` *(retired)* | $0.40 / MTok | $2 / MTok |

> Fast mode 与 Batch API 不可同时启用。

## 3. Fast Mode 价（仅 Opus 4.6 / 4.7，beta）

Fast mode 为 6× 标准价，跨完整上下文窗口生效（含 >200k 输入）。

| Input | Output |
| --- | --- |
| $30 / MTok | $150 / MTok |

> Prompt caching、data residency 倍率可叠加于 fast mode 之上；Fast mode 不可与 Batch API 同时使用，也不在 Claude Platform on AWS 提供。

## 4. Long context（1M 上下文窗口）

适用模型：Claude Mythos Preview、Opus 4.7、Opus 4.6、Sonnet 4.6。1M 窗口统一按标准价计费（900k 与 9k 请求同价），Prompt caching 与 Batch 折扣均按标准价叠加。

## 5. Data residency（地区路由）

仅 Opus 4.6 / Sonnet 4.6 及之后模型支持 `inference_geo` 参数。`inference_geo: "us"` 全部 token 类别（输入、输出、cache write、cache read）×1.1；`global`（默认）按标准价。早期模型不接受 `inference_geo`，传入返回 400。

## 6. 工具相关附加费

### Web Search

- $10 / 1,000 次搜索；搜索得到的内容按对应模型 input token 价计费。
- 错误失败的搜索不计费。

### Web Fetch

- 无附加费，仅按抓取内容的 input tokens 计费。

### Code Execution

- 搭配 `web_search_*` 或 `web_fetch_*` 工具使用时**免费**。
- 单独使用：每组织每月免费 1,550 容器小时；超出按 **$0.05 / 小时 / 容器** 计费。
- 最小计费单位 5 分钟；带 file 的请求即使未触发工具也按容器时间计费。

### Bash Tool / Text Editor / Computer Use 系统 token 开销

| 工具 | 额外 input tokens |
| --- | --- |
| Bash tool | 245 tokens |
| Text editor (`text_editor_20250429`, Claude 4.x) | 700 tokens |
| Computer use (Claude 4.x) | 735 tokens + 系统 prompt 加 466–499 tokens |

### Tool use 系统 prompt token 开销（按 tool_choice）

| 模型 | `auto` / `none` | `any` / `tool` |
| --- | --- | --- |
| Opus 4.7 / 4.6 / 4.5 / 4.1 / 4 / Sonnet 4.6 / 4.5 / 4 / Haiku 4.5 | 346 tokens | 313 tokens |
| Haiku 3.5 | 264 tokens | 340 tokens |

## 7. Claude Managed Agents

| SKU | Rate | 计费方式 |
| --- | --- | --- |
| Session runtime | $0.08 / session-hour | `status: running` 时长（毫秒级累加） |
| Tokens | 同 §1 标准价 | 缓存倍率适用 |
| Web search | $10 / 1,000 searches | 同标准计费 |

> Session 内 `idle` / `rescheduling` / `terminated` 时段不计 runtime 费。
>
> Managed Agents **不适用** Batch / Fast mode / Data residency 倍率，也不在 partner-operated cloud platforms 提供。

## 8. Claude Platform on AWS

- 计费单位：Claude Consumption Unit（CCU）；1 CCU = $0.01 USD。
- token 用量先按上表 USD 价率计费，应用折扣后换算为 CCU 计入 AWS Marketplace。
- 仅后付费；折扣以「计入 CCU 较少」的方式生效，CCU 单价不变。

## 9. Inference geography（仅 Claude Platform on AWS）

Opus 4.6 / Sonnet 4.6 及之后模型在 Claude Platform on AWS 使用 `inference_geo: "us"` 时所有费用 ×1.1；其他场景见 §5。

## 参考

- Pricing 主页：https://platform.claude.com/docs/en/about-claude/pricing
- Prompt caching：https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Batch processing：https://platform.claude.com/docs/en/build-with-claude/batch-processing
- Fast mode：https://platform.claude.com/docs/en/build-with-claude/fast-mode
- Model deprecations：https://platform.claude.com/docs/en/about-claude/model-deprecations
- Managed Agents overview：https://platform.claude.com/docs/en/managed-agents/overview
