---
source: https://platform.kimi.com/docs/api/balance
fetched_at: 2026-05-20
api_version: N/A
---

# 查询账户余额 · GET /v1/users/me/balance

返回当前 API Key 所属账户的现金 + 代金券余额（单位：人民币元）。

## 请求

无参数；仅需 `Authorization: Bearer ${MOONSHOT_API_KEY}` header。

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `code` | integer | 响应码，`0` 表示成功；其他值表示业务错误。 |
| `status` | boolean | 请求是否成功执行。 |
| `scode` | string | 状态码字符串形式。 |
| `data` | object | 详见 §data。 |

### data

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `available_balance` | number | 可用余额总额 = `cash_balance` + `voucher_balance`（保留负值含义）。 |
| `voucher_balance` | number | 代金券余额，不计入 Tier 阶梯累计充值。 |
| `cash_balance` | number | 现金余额；可为负值，表示账户当前欠费。 |

## 响应示例

```json
{
  "code": 0,
  "status": true,
  "scode": "0",
  "data": {
    "available_balance": 123.45,
    "voucher_balance": 20.00,
    "cash_balance": 103.45
  }
}
```

## 参考

- 端点：https://platform.kimi.com/docs/api/balance
