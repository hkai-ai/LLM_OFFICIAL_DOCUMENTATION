---
source: https://openrouter.ai/docs/use-cases/oauth-pkce
fetched_at: 2026-05-19
api_version: N/A
---

# OAuth PKCE 鉴权

> OpenRouter 提供 PKCE 流程，第三方应用可一键代用户创建并获取专属 API key，不需要 OpenRouter 颁发的 client secret。

## 流程概要

1. 应用本地生成 `code_verifier`（推荐 43–128 字符的随机串）。
2. 计算 `code_challenge`：
   - `code_challenge_method = S256`：`code_challenge = base64url(sha256(code_verifier))`。
   - `code_challenge_method = plain`：`code_challenge = code_verifier`。
3. 服务端调用 `POST /api/v1/auth/keys/code`，传入 `callback_url`、`code_challenge`、`code_challenge_method` 等参数，获取 `id`（即授权 code）。
4. 将用户重定向到 OpenRouter 的授权页（用户登录授权），授权完成后浏览器带 `?code=...` 回跳到 `callback_url`。
5. 应用用 `code` 调用 `POST /api/v1/auth/keys`，传入对应的 `code_verifier`，换取真正的 API key。

## POST /api/v1/auth/keys/code

> 创建一个授权码（authorization code），用于后续 PKCE 流程换 API key。

### 请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | Bearer token（management/普通 key 视具体集成场景）。 |
| `Content-Type` | ✓ | `application/json` |

### 请求体

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `callback_url` | string (URI) | ✓ | — | HTTPS URL；允许的端口为 443 和 3000。 |
| `code_challenge` | string | ✗ | — | PKCE 挑战值。 |
| `code_challenge_method` | string | ✗ | — | `S256` 或 `plain`。 |
| `expires_at` | string (datetime) | ✗ | — | 该授权码生成的 API key 的过期时间。 |
| `key_label` | string | ✗ | — | 生成 API key 的自定义标签。 |
| `limit` | number | ✗ | — | API key 的支出限额（USD）。 |
| `usage_limit_type` | string | ✗ | — | `daily` / `weekly` / `monthly`。 |

### 响应（`data`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 授权码 ID（即后续交换用的 `code`）。 |
| `app_id` | integer | 与该授权码关联的应用 ID。 |
| `created_at` | string | ISO 8601 时间戳。 |

## 授权页（用户跳转）

用户重定向到 OpenRouter 授权 URL（形如 `https://openrouter.ai/auth?callback_url=...&code=<id>&...`），完成登录与授权后，OpenRouter 将带 `code` 查询参数回跳到应用提供的 `callback_url`。

## POST /api/v1/auth/keys

> 用授权码交换真正的用户级 API key。

### 请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <token>` |
| `Content-Type` | ✓ | `application/json` |

### 请求体

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `code` | string | ✓ | — | 从 OAuth 回跳中拿到的授权码。 |
| `code_verifier` | string | ✗ | — | 若上一步使用了 `code_challenge`，此处必填。 |
| `code_challenge_method` | string | ✗ | — | `S256` 或 `plain`，与生成 `code_challenge` 时一致。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `key` | string | 用户授权产生的 API key。 |
| `user_id` | string \| null | 对应的用户 ID。 |

错误响应结构含 `error.code` / `error.message`，并可能附带 `user_id` 与 `openrouter_metadata`。

## 安全建议

- 生产环境务必使用 `S256`，避免 `plain`。
- `code_verifier` 由客户端持有，**不要**通过 URL 明文传递。
- `callback_url` 必须 HTTPS；本地调试可走 `http://localhost:3000`（OpenRouter 单独放行 3000 端口）。
- `code` 单次使用；交换 API key 后立即失效。

## 参考

- OAuth PKCE 使用指南：<https://openrouter.ai/docs/use-cases/oauth-pkce>
- Create Auth Code 端点：<https://openrouter.ai/docs/api/api-reference/o-auth/create-auth-keys-code>
- Exchange Auth Code 端点：<https://openrouter.ai/docs/api/api-reference/o-auth/exchange-auth-code-for-api-key>
- API 鉴权总览：<https://openrouter.ai/docs/api/reference/authentication>
