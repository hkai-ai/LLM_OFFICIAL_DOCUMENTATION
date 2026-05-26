---
source: https://help.aliyun.com/zh/model-studio/error-code
fetched_at: 2026-05-20
api_version: N/A
---

# 错误响应

## 两种返回结构

| 模式 | 失败响应结构 |
| --- | --- |
| OpenAI 兼容（`/compatible-mode/v1/...`） | HTTP 标准状态码 + `{"error": {"message": "...", "type": "...", "code": "..."}}` |
| DashScope 原生（`/api/v1/...`） | HTTP 标准状态码 + `{"code": "...", "message": "...", "request_id": "..."}`；`code` 用自有命名空间。 |

## 错误码清单

### 400 — 请求错误

| `code` | 含义 |
| --- | --- |
| `InvalidParameter` | 参数不合法。常见：`enable_thinking` 与 `response_format=json_object` 同时设置；`max_tokens` 超模型上限；多模态字段命名错位。 |
| `DataInspectionFailed` | 输入或输出触发内容安全审核。 |
| `Arrearage` | 账户欠费：「Access denied, please make sure your account is in good standing」。 |

### 401 — 鉴权

| `code` | 含义 |
| --- | --- |
| `InvalidApiKey` | API Key 错误、格式不对，或与当前 base URL 地域不匹配。 |

### 403 — 权限

| `code` | 含义 |
| --- | --- |
| `AccessDenied` | 当前账号 / 子账号无权访问该接口。 |
| `Model.AccessDenied` | 未在模型广场开通该模型。 |

### 404 — 资源不存在

| `code` | 含义 |
| --- | --- |
| `ModelNotFound` | 模型 ID 拼写错误或当前地域未上架；注意区分百炼托管 ID 与开源仓库 ID。 |

### 429 — 限流

| `code` | 含义 |
| --- | --- |
| `Throttling` | 触发 RPM / 并发 限制。 |
| `Throttling.AllocationQuota` | 触发 token 配额（TPM / TPD）限制。 |

### 500 — 服务端错误

| `code` | 含义 |
| --- | --- |
| `InternalError` | 内部算法 / 服务异常，可指数退避重试。 |
| `RequestTimeOut` | 请求超时（默认 300 秒）；建议改用流式或拆分请求。 |

## 与 OpenAI 协议的字段差异

| 维度 | DashScope | OpenAI 兼容 |
| --- | --- | --- |
| `seed` 范围 | `[0, 2⁶³-1]` | `[0, 2³¹-1]` |
| 错误结构 | 顶层 `code` + `message` + `request_id` | 包裹在 `error` 对象中，附 `type` |
| token 用量字段 | `input_tokens` / `output_tokens` | `prompt_tokens` / `completion_tokens` |
| Tool calls 命名 | 同 OpenAI | 同 OpenAI |
| 流式触发 | header `X-DashScope-SSE: enable` | `stream: true` |

## 常见排查

- **欠费 `400-Arrearage`**：阿里云主账户余额不足，控制台充值后通常 1–2 分钟生效。
- **`ModelNotFound`**：同名开源模型与百炼托管模型 ID 不同（例：开源 `Qwen3-32B` ≠ 百炼 `qwen3.6-plus`）；以模型广场为准。
- **超时 `500-RequestTimeOut`**：单次请求最长 300 秒；长上下文 + 思考模式高 budget 容易触发，建议拆分或流式。
- **地域 Key 不通**：北京 Key 不能用于新加坡 / 美西 / 法兰克福，反之亦然，需切换 `base_url`。

## 参考

- 错误码：https://help.aliyun.com/zh/model-studio/error-code
- 模型广场：https://bailian.console.aliyun.com/?tab=model#/model-market
