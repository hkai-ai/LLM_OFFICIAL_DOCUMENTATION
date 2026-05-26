---
source: https://platform.minimaxi.com/docs/api-reference/errorcode
fetched_at: 2026-05-26
api_version: N/A
---

# MiniMax · 错误码

> MiniMax 的错误结构不直接复用 HTTP 状态码，而是在响应体顶层附带 `base_resp` 对象（`status_code` + `status_msg`）。HTTP 状态通常为 200，业务成功与否需读 `base_resp.status_code`，**0 表示成功**。

## 响应结构

```json
{
  "base_resp": {
    "status_code": 0,
    "status_msg": "success"
  }
}
```

业务错误时其他业务字段可能缺失或为空，必须先校验 `base_resp.status_code`。

## 错误码列表

| `status_code` | 含义 | 触发场景 / 排查建议 |
| --- | --- | --- |
| `0` | 成功 | — |
| `1000` | 未知错误 / 系统默认错误 | 稍后重试。 |
| `1001` | 请求超时 | 稍后重试；检查网络与上游负载。 |
| `1002` | 触发限流 | 降低并发 / 提升 Tier；详见 [pricing.md](./pricing.md)。 |
| `1004` | 未授权 / Token 不匹配 | 检查 API Key（或 Anthropic 路径下的 `X-Api-Key`）。 |
| `1008` | 余额不足 | 充值或检查 Token Plan。 |
| `1024` | 内部错误 | 稍后重试。 |
| `1026` | 输入内容涉敏 | 调整 input；详见敏感词分类（同 `input_sensitive_type` 枚举）。 |
| `1027` | 输出内容涉敏 | 调整 prompt 或重试。 |
| `1033` | 系统 / 下游服务错误 | 稍后重试。 |
| `1039` | 触发 token 限制 | 降低 `max_tokens` / `max_completion_tokens`。 |
| `1041` | 触发连接限制 | 联系客服扩容。 |
| `1042` | 非法字符 / 不可见字符占比超过 10% | 清洗输入内容。 |
| `1043` | ASR 相似度校验失败（语音相关） | 核对 `file_id` 与 `text_validation` 一致性。 |
| `1044` | 复刻 prompt 相似度校验失败 | 检查音频与 prompt 是否对应同一发声内容。 |
| `2013` | 参数错误 | 核对请求参数类型与必填项。 |
| `20132` | 复刻样本 / `voice_id` 错误 | 检查 voice cloning 参数。 |
| `2037` | 语音时长不符合要求 | 范围 10s–5min。 |
| `2038` | 未开通声音复刻权限 | 完成实名认证 / 个人或企业认证。 |
| `2039` | `voice_id` 重复 | 使用全局唯一的 `voice_id`。 |
| `2042` | 无权访问该 `voice_id` | 仅创建者可使用 / 删除。 |
| `2045` | 请求速率突增超过限制 | 平滑发送，避免短时尖峰。 |
| `2048` | 复刻 prompt 音频过长 | prompt 音频应 < 8 秒。 |
| `2049` | API Key 非法 | 重新生成密钥。 |
| `2056` | Token Plan 限制 | 等待资源释放或升级订阅。 |

> 文档未明确给出对应的 HTTP code；多数情况下业务错误仍返回 HTTP 200，错误信息全部由 `base_resp` 体现。如需进一步细节，官方建议联系 `api@minimaxi.com`。

## 排查清单

1. **先看 `base_resp.status_code`**：非 0 才说明业务失败。
2. **再看 `input_sensitive` / `output_sensitive`**：chat 接口附带的敏感词命中标志（独立于 `status_code` 1026 / 1027）。
3. **重试策略**：对 `1000` / `1001` / `1024` / `1033` 之类的非确定性错误使用指数退避；`1002` / `2045` 限流类错误使用 jitter 重试或扩容 Tier。
4. **鉴权类（`1004` / `2049`）**：不要重试，直接联系账户管理员。
5. **配额类（`1008` / `2056`）**：检查账户余额或 Token Plan 用量。

## 参考

- 错误码原文：https://platform.minimaxi.com/docs/api-reference/errorcode
- 速率限制：https://platform.minimaxi.com/docs/guides/rate-limits
- 接口 FAQ：https://platform.minimaxi.com/docs/faq/about-apis
