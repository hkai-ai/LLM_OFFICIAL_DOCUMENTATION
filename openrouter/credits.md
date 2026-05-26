---
source: https://openrouter.ai/docs/api/api-reference/credits/get-credits
fetched_at: 2026-05-19
api_version: N/A
---

# 查询额度 · GET /api/v1/credits

> 查询当前账户已购买与已使用的 credits。需使用 management key（即 provisioning key），普通 API key 调用会返回 403。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer <MANAGEMENT_KEY>` |

## 请求参数

无。

## 响应

### 顶层对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `data` | object | 详见下。 |

### `data`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `total_credits` | number | 已购买的总额度（USD）。 |
| `total_usage` | number | 累计已使用额度（USD）。 |

> 剩余额度可通过 `total_credits - total_usage` 计算。

## 示例

### 请求

```
GET /api/v1/credits HTTP/1.1
Host: openrouter.ai
Authorization: Bearer sk-or-mgmt-...
```

### 响应

```json
{
  "data": {
    "total_credits": 100.0,
    "total_usage": 12.34
  }
}
```

## 错误码

| HTTP | 含义 |
| --- | --- |
| `401` | 未授权或 key 无效。 |
| `403` | 权限不足（使用了普通 key 而非 management key）。 |
| `500` | 服务器错误。 |

## 相关端点

- 单个 API key 的使用情况、限额、剩余额度查询：`GET /api/v1/key`，详见 [api-keys.md](./api-keys.md)。

## 参考

- 端点文档：<https://openrouter.ai/docs/api/api-reference/credits/get-credits>
- API Reference 总入口：<https://openrouter.ai/docs/api/reference/overview>
