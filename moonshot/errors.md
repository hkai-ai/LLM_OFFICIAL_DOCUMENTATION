---
source: https://platform.kimi.com/docs/api/errors
fetched_at: 2026-05-20
api_version: N/A
---

# 错误响应

所有失败请求返回 JSON：

```json
{
  "error": {
    "message": "...",
    "type": "...",
    "code": "..."
  }
}
```

`type` 与 HTTP 状态对应，`code` 是更细的子分类标识。

## 错误码清单

### 400 — 请求错误

| `code` / `type` | 含义 |
| --- | --- |
| `content_filter` | 输入或输出触发内容安全策略，请求被拒绝。 |
| `invalid_request_error` | 参数格式错误、超长、不合法（含 token 长度超限、文件上传字段缺失等）。 |

### 401 — 鉴权失败

| `code` / `type` | 含义 |
| --- | --- |
| `invalid_authentication_error` | `Authorization` header 缺失或格式错误。 |
| `incorrect_api_key_error` | API Key 无效或已被撤销。 |

### 403 — 权限不足

| `code` / `type` | 含义 |
| --- | --- |
| `permission_denied_error` | 当前 API Key 无权访问该模型 / 资源，或访问的用户信息属于他人。 |

### 404 — 资源不存在

| `code` / `type` | 含义 |
| --- | --- |
| `resource_not_found_error` | 模型 ID 不存在或当前组织未开通；文件 ID 不存在。 |

### 429 — 限流 / 配额

| `code` / `type` | 含义 |
| --- | --- |
| `engine_overloaded_error` | 推理引擎当前过载，可重试。 |
| `exceeded_current_quota_error` | 账户欠费或 token 余量不足。 |
| `rate_limit_reached_error` | 触发并发 / RPM / TPM / TPD 任一限速；详见 [rate-limits.md](./rate-limits.md)。 |

### 500 — 服务端错误

| `code` / `type` | 含义 |
| --- | --- |
| `server_error` | 服务端异常（如文件抽取失败），可重试。 |
| `unexpected_output` | `invalid state transition`，模型生成进入异常状态，建议联系支持。 |

## 排查建议

- `kimi.ai`（海外消费版）与 `platform.kimi.com`（开发者平台）使用相互独立的 API Key，切勿混用。
- 429 中 `rate_limit_reached_error` 由「并发 / RPM / TPM / TPD」任一维度触发，需结合自身请求模式与 Tier 上限判断；TPD 仅对 Tier0 设限。
- 500 系列错误建议指数退避重试 2–3 次；若持续出现请提供 `id` 与时间戳联系官方支持。

## 参考

- 端点：https://platform.kimi.com/docs/api/errors
- 限速：https://platform.kimi.com/docs/pricing/limits
- FAQ：https://platform.kimi.com/docs/guide/faq
