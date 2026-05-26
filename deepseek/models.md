---
source: https://api-docs.deepseek.com/zh-cn/api/list-models
fetched_at: 2026-05-19
api_version: N/A
---

# 模型列表 · GET /models

> 列出当前账号可用的模型。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer ${DEEPSEEK_API_KEY}` |

## 请求参数

无。

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `list`。 |
| `data` | array | 模型对象列表。 |

### `data[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 模型标识符，用作 `chat/completions` 请求中的 `model`。 |
| `object` | string | 固定 `model`。 |
| `owned_by` | string | 模型所属组织，通常为 `deepseek`。 |

## 当前模型清单（截至 fetched_at）

| 模型 ID | 上下文窗口 | 最大输出 | 思考模式 | 备注 |
| --- | --- | --- | --- | --- |
| `deepseek-v4-pro` | 1M tokens | 384K tokens | 支持 | 旗舰模型，适用于复杂推理与 Agent 任务。 |
| `deepseek-v4-flash` | 1M tokens | 384K tokens | 支持 | 高吞吐、低成本通用模型。 |
| `deepseek-chat` | 1M tokens | 384K tokens | 默认关闭 | 兼容别名，映射至 `deepseek-v4-flash` 非思考模式；2026-07-24 下线。 |
| `deepseek-reasoner` | 1M tokens | 384K tokens | 默认开启 | 兼容别名，映射至 `deepseek-v4-flash` 思考模式；2026-07-24 下线。 |

> 上下文窗口与最大输出以官方[模型与价格页](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)为准。历史版本（V3、V3.1、V3.2、R1 等）已通过 `deepseek-chat` / `deepseek-reasoner` 别名指向新模型；如需固定调用旧版本须使用厂商提供的特殊 base_url，常规情况下不可用。

## 示例

### 请求

```http
GET /models HTTP/1.1
Host: api.deepseek.com
Authorization: Bearer ${DEEPSEEK_API_KEY}
```

### 响应

```json
{
  "object": "list",
  "data": [
    { "id": "deepseek-chat",     "object": "model", "owned_by": "deepseek" },
    { "id": "deepseek-reasoner", "object": "model", "owned_by": "deepseek" },
    { "id": "deepseek-v4-flash", "object": "model", "owned_by": "deepseek" },
    { "id": "deepseek-v4-pro",   "object": "model", "owned_by": "deepseek" }
  ]
}
```

## 参考

- 端点文档：https://api-docs.deepseek.com/zh-cn/api/list-models
- 模型与价格：https://api-docs.deepseek.com/zh-cn/quick_start/pricing
- 更新日志：https://api-docs.deepseek.com/zh-cn/updates
