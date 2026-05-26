---
source: https://api-docs.deepseek.com/zh-cn/quick_start/error_codes
fetched_at: 2026-05-19
api_version: N/A
---

# 错误响应与错误码

DeepSeek API 错误响应沿用 OpenAI 协议格式：HTTP 状态码 + JSON 错误体。

## 错误响应结构

```json
{
  "error": {
    "message": "Invalid model specified.",
    "type": "invalid_request_error",
    "code": "invalid_model",
    "param": "model"
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `error.message` | string | 错误的可读说明。 |
| `error.type` | string | 错误类型，与 HTTP 状态码对应。 |
| `error.code` | string \| null | 细分错误代码（可能为空）。 |
| `error.param` | string \| null | 引发错误的参数名（仅参数类错误）。 |

> 官方错误码页对错误体字段未逐项列出，以上结构基于 OpenAI 兼容契约与官方示例。

## 状态码

| HTTP | type | 含义 | 触发原因 | 处理建议 |
| --- | --- | --- | --- | --- |
| `400` | invalid_request_error | 格式错误 | 请求体格式错误（非法 JSON、字段类型错误等）。 | 按错误提示修改请求体结构。 |
| `401` | authentication_error | 认证失败 | API key 错误。 | 核对 Key，必要时在控制台重新生成。 |
| `402` | insufficient_balance | 余额不足 | 账户余额不足以支付该请求。 | 前往充值；可先查询 [user-balance](./user-balance.md)。 |
| `422` | invalid_request_error | 参数错误 | 请求体参数取值不合法（如非法枚举、范围超界）。 | 按错误提示修改对应参数。 |
| `429` | rate_limit_reached | 限速 | TPM / RPM / 并发达到上限。 | 客户端节流，指数退避重试；见 [rate-limits.md](./rate-limits.md)。 |
| `500` | server_error | 服务器故障 | 服务端内部错误。 | 稍后重试；持续出现请联系 `api-service@deepseek.com`。 |
| `503` | service_unavailable | 服务繁忙 | 服务器负载过高。 | 稍后重试，建议指数退避。 |

## 流式请求中的错误

- 错误同样以 SSE `data: { "error": { ... } }` 单事件返回，然后连接关闭。
- 等待推理期间，服务器会发送 `: keep-alive` 注释行（流式）或空行（非流式）以保持连接；如果 10 分钟内仍未开始推理，服务器将主动关闭连接。

## 重试建议

| HTTP | 是否建议重试 | 备注 |
| --- | --- | --- |
| `400` / `422` | 否 | 请求自身问题。 |
| `401` / `402` | 否 | 需用户介入。 |
| `429` | 是 | 指数退避，建议从 1s 起，最多 5 次。 |
| `500` / `503` | 是 | 指数退避；优先在网关侧重试。 |

## 参考

- 官方错误码页：https://api-docs.deepseek.com/zh-cn/quick_start/error_codes
- 限速：https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit
