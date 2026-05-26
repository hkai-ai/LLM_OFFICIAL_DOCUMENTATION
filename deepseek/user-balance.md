---
source: https://api-docs.deepseek.com/zh-cn/api/get-user-balance
fetched_at: 2026-05-19
api_version: N/A
---

# 账户余额 · GET /user/balance

> 查询账号余额。

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
| `is_available` | boolean | 当前账户是否有余额可供 API 调用。 |
| `balance_infos` | array | 各币种余额信息列表，详见 [`balance_infos[]`](#balance_infos)。 |

### `balance_infos[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `currency` | string | 币种，取值 `CNY` / `USD`。 |
| `total_balance` | string | 总的可用余额，含赠金与充值余额（字符串表示，避免浮点误差）。 |
| `granted_balance` | string | 未过期的赠金余额。 |
| `topped_up_balance` | string | 充值余额。 |

## 示例

### 请求

```http
GET /user/balance HTTP/1.1
Host: api.deepseek.com
Authorization: Bearer ${DEEPSEEK_API_KEY}
```

### 响应

```json
{
  "is_available": true,
  "balance_infos": [
    {
      "currency": "CNY",
      "total_balance": "110.00",
      "granted_balance": "10.00",
      "topped_up_balance": "100.00"
    }
  ]
}
```

## 错误码

| HTTP | 含义 | 触发原因 |
| --- | --- | --- |
| `401` | authentication_error | API key 错误。 |
| `500` / `503` | server_error / service_unavailable | 服务端故障或繁忙。 |

## 参考

- 端点文档：https://api-docs.deepseek.com/zh-cn/api/get-user-balance
