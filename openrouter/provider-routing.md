---
source: https://openrouter.ai/docs/guides/routing/provider-selection
fetched_at: 2026-05-19
api_version: N/A
---

# Provider 路由

> OpenRouter 在请求体的 `provider` 字段下接收 provider 路由偏好，控制使用哪些 provider、按何种顺序、用什么量化等级、走什么定价/吞吐/延迟策略。

`provider` 是请求体顶层的可选 object，未传时按 OpenRouter 默认排序。

## `provider` 字段

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `order` | array&lt;string&gt; | — | 按顺序优先尝试的 provider 名称列表。 |
| `allow_fallbacks` | boolean | `true` | 当 `order` 中的 provider 全部不可用时，是否回退到其他 provider。 |
| `require_parameters` | boolean | `false` | 仅使用支持本次请求中**所有**参数的 provider（否则可能丢弃不支持参数）。 |
| `data_collection` | string | `allow` | `allow`（允许使用可能记录数据的 provider）/ `deny`（仅使用不留存数据的 provider）。 |
| `only` | array&lt;string&gt; | — | 白名单：只允许这些 provider。 |
| `ignore` | array&lt;string&gt; | — | 黑名单：跳过这些 provider。 |
| `quantizations` | array&lt;string&gt; | — | 允许的量化等级，取值见下。 |
| `sort` | string \| object | — | 排序策略，详见下。 |
| `max_price` | object | — | 价格上限，详见下。 |
| `preferred_max_latency` | number \| object | — | 期望的最大延迟（秒），或 `{p50, p75, p90, p99}` 分位数对象。 |
| `preferred_min_throughput` | number \| object | — | 期望的最低吞吐（tokens/sec），或 `{p50, p75, p90, p99}` 分位数对象。 |
| `zdr` | boolean | — | 仅路由到零数据保留（Zero Data Retention）的 endpoint。 |
| `enforce_distillable_text` | boolean | — | 仅路由到允许文本蒸馏的模型。 |

## `quantizations` 允许值

`int4`、`int8`、`fp4`、`fp6`、`fp8`、`fp16`、`bf16`、`fp32`、`unknown`。

## `sort`

可作为字符串简写或对象指定：

| 形式 | 说明 |
| --- | --- |
| `"price"` | 价格优先（最低）。 |
| `"throughput"` | 吞吐量优先（最高）。 |
| `"latency"` | 延迟优先（最低）。 |
| `"exacto"` | 工具调用质量优先（详见 exacto variant 文档）。 |
| `{"by": "<...>", "partition": "model" \| "none"}` | 完整对象形式。 |

### `sort.partition`

| 取值 | 说明 |
| --- | --- |
| `model`（默认） | 当指定多个候选模型时，按模型分组排序，主模型的 endpoint 始终先尝试。 |
| `none` | 取消分组，跨模型全局排序所有 endpoint。 |

## `max_price`

每百万 token 的价格上限（USD/M tokens）：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `prompt` | number | 输入价格上限。 |
| `completion` | number | 输出价格上限。 |
| `request` | number | 单次请求上限。 |
| `image` | number | 单张图像上限。 |
| `audio` | number | 音频上限（如适用）。 |

## 动态变体（model ID 后缀）

可在 model ID 末尾追加变体后缀，等价于设置 `sort`：

| 变体 | 等价含义 |
| --- | --- |
| `:nitro` | `sort: "throughput"`，吞吐量优先。 |
| `:floor` | `sort: "price"`，价格最低。 |

示例：`meta-llama/llama-3.3-70b-instruct:nitro`。

## 示例

```json
{
  "model": "meta-llama/llama-3.3-70b-instruct",
  "provider": {
    "order": ["Fireworks", "Together"],
    "allow_fallbacks": true,
    "require_parameters": true,
    "data_collection": "deny",
    "quantizations": ["fp16", "bf16"],
    "sort": "throughput",
    "max_price": {"prompt": 0.5, "completion": 1.0},
    "preferred_max_latency": {"p95": 5},
    "zdr": true
  },
  "messages": [{"role": "user", "content": "Hi"}]
}
```

## 参考

- Provider Routing 指南：<https://openrouter.ai/docs/guides/routing/provider-selection>
- Exacto 变体（工具调用质量优先）：<https://openrouter.ai/docs/guides/routing/model-variants/exacto>
- Uptime 优化建议：<https://openrouter.ai/docs/guides/best-practices/uptime-optimization>
