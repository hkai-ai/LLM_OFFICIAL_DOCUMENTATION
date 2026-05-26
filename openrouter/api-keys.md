---
source: https://openrouter.ai/docs/api/api-reference/api-keys/list
fetched_at: 2026-05-19
api_version: N/A
---

# API Keys 管理端点

> 用于程序化管理 OpenRouter 账户下的 API key。所有 `/api/v1/keys` 路径下的端点（List / Create / Get by hash / Update / Delete）都需要 **management key**（也叫 provisioning key）作为 Bearer。management key 不能调用 `/chat/completions` 等推理端点。
>
> `/api/v1/key`（单数）是一个特例，用于查询「当前调用所用 key」的使用与额度信息，接受任意 API key。

## GET /api/v1/key

> 返回当前 Authorization header 所用 key 的元信息。

### 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <API_KEY>`（可以是普通 key 或 management key）。 |

### 响应（`data`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `label` | string | 人类可读的标签。 |
| `usage` | number | 累计使用额（USD）。 |
| `usage_daily` | number | 当前 UTC 日使用额。 |
| `usage_weekly` | number | 当前 UTC 周使用额（周一到周日）。 |
| `usage_monthly` | number | 当前 UTC 月使用额。 |
| `limit` | number \| null | 美元支出限额，`null` 表示无限制。 |
| `limit_remaining` | number \| null | 剩余可用额度。 |
| `limit_reset` | string \| null | 限额重置周期：`daily` / `weekly` / `monthly` / `null`。 |
| `is_free_tier` | boolean | 是否为免费层。 |
| `is_provisioning_key` | boolean | 是否为 provisioning（management）key。 |
| `is_management_key` | boolean | 是否为 management key（等价字段）。 |
| `rate_limit` | object | 旧字段，固定返回 `-1`。新版速率限制查 [rate-limits.md](./rate-limits.md)。 |
| `creator_user_id` | string \| null | 创建者用户 ID。 |
| `expires_at` | string \| null | ISO 8601 UTC 过期时间。 |
| `include_byok_in_limit` | boolean | 是否把 BYOK 用量计入 limit。 |
| `byok_usage` | number | 累计 BYOK 用量（USD）。 |
| `byok_usage_daily` | number | 当日 BYOK 用量。 |
| `byok_usage_weekly` | number | 当周 BYOK 用量。 |
| `byok_usage_monthly` | number | 当月 BYOK 用量。 |

### 错误码

| HTTP | 含义 |
| --- | --- |
| `401` | key 无效。 |
| `500` | 服务器错误。 |

## GET /api/v1/keys

> 列出当前 management key 所属账户/工作区下的全部 API key。

### 请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <MANAGEMENT_KEY>` |

### 请求参数（query string）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `include_disabled` | string | ✗ | — | 是否包含已禁用 key。 |
| `offset` | integer | ✗ | `0` | 分页偏移。 |
| `workspace_id` | string (UUID) | ✗ | — | 按工作区筛选。 |

### 响应

返回 `{"data": [Key, ...]}`，每个 `Key`：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `hash` | string | 唯一标识，用于删除/更新。 |
| `name` | string | key 名称。 |
| `label` | string | 显示标签。 |
| `disabled` | boolean | 是否禁用。 |
| `limit` | number \| null | 美元支出限额。 |
| `limit_remaining` | number \| null | 剩余额度。 |
| `limit_reset` | string \| null | 重置周期。 |
| `created_at` | string | ISO 8601 创建时间。 |
| `updated_at` | string | ISO 8601 最后更新时间。 |
| `usage` | number | 累计使用额（USD）。 |
| `usage_daily` | number | 当日使用额。 |
| `usage_weekly` | number | 当周使用额。 |
| `usage_monthly` | number | 当月使用额。 |
| `byok_usage` | number | BYOK 累计使用额。 |
| `expires_at` | string \| null | 过期时间。 |
| `workspace_id` | string \| null | 工作区 ID。 |
| `creator_user_id` | string \| null | 创建者用户 ID。 |

## POST /api/v1/keys

> 创建新的 API key。

### 请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <MANAGEMENT_KEY>` |
| `Content-Type` | ✓ | `application/json` |

### 请求体

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `name` | string | ✓ | — | key 名称。 |
| `label` | string | ✗ | — | 显示标签。 |
| `limit` | number | ✗ | — | 美元支出限额。 |
| `include_byok_in_limit` | boolean | ✗ | — | 是否将 BYOK 用量计入 limit。 |
| `limit_reset` | string | ✗ | — | `daily` / `weekly` / `monthly`，UTC 0 点重置。 |
| `expires_at` | string | ✗ | — | ISO 8601 UTC 过期时间。 |
| `creator_user_id` | string | ✗ | — | 创建者用户 ID。 |
| `workspace_id` | string (UUID) | ✗ | — | 关联的工作区 ID。 |

### 响应（201）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `key` | string | **实际的 API key 字符串**（只在创建时返回一次，需要立即保存）。 |
| `data` | object | 含 `name` / `hash` / `created_at` / `disabled` / `usage` / `limit` / `limit_remaining` / `limit_reset` / `expires_at` / `workspace_id` 等字段。 |

## DELETE /api/v1/keys/{hash}

> 删除指定的 API key。

### 请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <MANAGEMENT_KEY>` |

### 路径参数

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `hash` | string | ✓ | 待删除 key 的 hash。 |

### 响应

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `deleted` | boolean | 固定 `true`。 |

### 错误码

| HTTP | 含义 |
| --- | --- |
| `200` | 删除成功。 |
| `401` | 未授权。 |
| `404` | 未找到对应 key。 |
| `429` | 触发限流。 |
| `500` | 服务器错误。 |

## 示例：创建 key

### 请求

```json
{
  "name": "tenant-123",
  "limit": 10.0,
  "include_byok_in_limit": false,
  "limit_reset": "monthly"
}
```

### 响应

```json
{
  "key": "sk-or-v1-...",
  "data": {
    "hash": "abc123def...",
    "name": "tenant-123",
    "created_at": "2026-05-19T08:00:00Z",
    "disabled": false,
    "usage": 0,
    "limit": 10.0,
    "limit_remaining": 10.0,
    "limit_reset": "monthly"
  }
}
```

## 参考

- 列出 keys：<https://openrouter.ai/docs/api/api-reference/api-keys/list>
- 创建 key：<https://openrouter.ai/docs/api/api-reference/api-keys/create-keys>
- 删除 key：<https://openrouter.ai/docs/api/api-reference/api-keys/delete-keys>
- 当前 key 信息：<https://openrouter.ai/docs/api/api-reference/api-keys/get-current-key>
- Management API Keys 指南：<https://openrouter.ai/docs/guides/overview/auth/provisioning-api-keys>
