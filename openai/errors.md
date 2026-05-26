---
source: https://developers.openai.com/api/docs/guides/error-codes
fetched_at: 2026-05-19
api_version: N/A
---

# 错误码

> OpenAI 所有 REST 端点共享相同的错误响应结构：HTTP 状态码 + JSON body 中的 `error` 对象。

## error 对象结构

```json
{
  "error": {
    "type": "invalid_request_error",
    "message": "...",
    "param": "messages.0.content",
    "code": "context_length_exceeded"
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 错误类型分类，见下表。 |
| `message` | string | 人类可读的描述。`error.message` 是排查首要依据，**会随服务端版本变化**，不要做精确字符串匹配，匹配前缀或 `code` 字段更稳。 |
| `param` | string \| null | 触发错误的字段路径（如 `messages.0.content`），若无字段相关则为 null。 |
| `code` | string \| null | 细粒度错误代码，比 `type` 更具体，例如 `model_not_found` / `context_length_exceeded` / `insufficient_quota`。 |

流式响应中错误以 `data: { "error": { ... } }` 事件下发（Chat Completions）或 `response.failed` / `response.error` 事件（Responses API）。

## 常见 HTTP 状态码

| HTTP | 含义 | 典型 `error.type` | 典型 `error.code` |
| --- | --- | --- | --- |
| `400` | 请求格式 / 参数错误 | `invalid_request_error` | `context_length_exceeded` / `model_not_found` / `invalid_value` / `unknown_parameter` / `unsupported_parameter` |
| `400` | 内容违规 | `invalid_request_error` | `content_policy_violation` |
| `401` | 未通过鉴权 | `invalid_api_key` / `authentication_error` | `invalid_api_key` / `missing_authorization` |
| `401` | 用户未加入 organization | `authentication_error` | `organization_required` |
| `401` | IP 不在 allowlist | `authentication_error` | `ip_address_not_allowed` |
| `403` | 区域不支持 | `permission_error` | `unsupported_country_region_territory` |
| `403` | 项目无权使用资源 | `permission_error` | — |
| `404` | 资源不存在 / 模型不可见 | `not_found_error` | `model_not_found` |
| `409` | 状态冲突（重复取消、对象处于终态等） | `conflict` | — |
| `413` | 请求体过大 | `invalid_request_error` | — |
| `422` | 部分子系统对参数语义校验失败 | `invalid_request_error` | — |
| `429` | 触发限流 | `rate_limit_error` | `rate_limit_exceeded` / `tokens_per_min` |
| `429` | 配额耗尽 | `insufficient_quota` | `insufficient_quota` |
| `500` | 服务端异常 | `server_error` / `api_error` | — |
| `502` | 网关错误 | `server_error` | — |
| `503` | 模型过载 | `engine_overloaded` | — |
| `504` | 上游超时 | `timeout` | — |

## error.type 总览

| `type` | 触发场景 |
| --- | --- |
| `invalid_request_error` | 参数缺失、类型不符、值越界、内容违反 policy。 |
| `authentication_error` | API key 缺失 / 失效 / organization 不匹配 / IP 不允许。 |
| `permission_error` | 区域 / project / 资源访问权限不足。 |
| `not_found_error` | 资源不存在或当前账户不可见。 |
| `rate_limit_error` | RPM / TPM / 并发限流。 |
| `insufficient_quota` | 余额或 quota 不足。 |
| `server_error` | OpenAI 服务端异常，通常可重试。 |
| `api_error` | 与 `server_error` 类似，部分老端点继续使用此名。 |
| `engine_overloaded` | 模型过载。 |
| `timeout` | 上游处理超时。 |
| `conflict` | 状态冲突。 |
| `content_policy_violation` | 输入或输出被 moderation 拦截（也可能以 `invalid_request_error.code = content_policy_violation` 形式返回）。 |

## SDK 异常映射（参考）

| Python (`openai`) | JS (`openai`) | HTTP | error.type |
| --- | --- | --- | --- |
| `APIConnectionError` | `APIConnectionError` | — | 网络层，未必有 body。 |
| `APITimeoutError` | `APITimeoutError` | — | 客户端超时。 |
| `BadRequestError` | `BadRequestError` | 400 | `invalid_request_error` |
| `AuthenticationError` | `AuthenticationError` | 401 | `authentication_error` |
| `PermissionDeniedError` | `PermissionDeniedError` | 403 | `permission_error` |
| `NotFoundError` | `NotFoundError` | 404 | `not_found_error` |
| `ConflictError` | `ConflictError` | 409 | `conflict` |
| `UnprocessableEntityError` | `UnprocessableEntityError` | 422 | `invalid_request_error` |
| `RateLimitError` | `RateLimitError` | 429 | `rate_limit_error` / `insufficient_quota` |
| `InternalServerError` | `InternalServerError` | 5xx | `server_error` |

## 重试建议

- **400 / 401 / 403 / 404** 一律不要重试。
- **409** 通常说明请求与状态机冲突，不应该简单重试。
- **429** 根据 `Retry-After` 响应头（秒）或指数退避；`insufficient_quota` 不要重试，需补充额度。
- **5xx** 与 `engine_overloaded` 用指数退避 + 抖动，限定最大重试次数。

## 流式错误特例

- Chat Completions：发送中途出错通常在最后一个 chunk 内以 `data: {"error": {...}}` 单独下发；客户端必须解析 `[DONE]` 之前的每条 data。
- Responses：发出 `response.failed` 或 `response.error` 事件，对应 `response.status: "failed"`。

## 参考

- 指南：<https://developers.openai.com/api/docs/guides/error-codes>
- 生产实践：<https://developers.openai.com/api/docs/guides/production-best-practices>
