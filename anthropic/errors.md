---
source: https://platform.claude.com/docs/en/api/errors
fetched_at: 2026-05-19
api_version: anthropic-version: 2023-06-01
---

# 错误处理

## HTTP 状态码与 `error.type` 对照

| HTTP | `error.type` | 含义 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 请求格式或内容有误。文档备注：4XX 中未单列的状态码也可能复用该 type。 |
| `401` | `authentication_error` | API key 无效。在 Claude Platform on AWS 上还可能是 AWS 凭证 / SigV4 签名问题。 |
| `402` | `billing_error` | 账单或支付信息异常。 |
| `403` | `permission_error` | API key 没有访问该资源的权限。 |
| `404` | `not_found_error` | 资源不存在。 |
| `413` | `request_too_large` | 请求体超过端点上限（见下方表格）。在直连 Claude API 上由 Cloudflare 在到达 API 服务器之前返回。 |
| `429` | `rate_limit_error` | 触发租户限流或加速限流（acceleration limits）。 |
| `500` | `api_error` | Anthropic 内部异常。 |
| `504` | `timeout_error` | 请求处理超时。长任务建议改用 streaming。 |
| `529` | `overloaded_error` | API 整体过载。高峰期所有用户都可能命中；如果是单租户用量陡升触发的「加速限流」，可能反而看到 `429`。 |

## 请求体积上限

| 端点 | 上限 |
| --- | --- |
| Messages API | `32 MB` |
| Token Counting API | `32 MB` |
| Batch API | `256 MB` |
| Files API | `500 MB` |

超过上限触发 `413 request_too_large`。

## 错误响应 JSON

所有错误（无论 HTTP 状态）都以 JSON 返回，顶层固定：

```json
{
  "type": "error",
  "error": {
    "type": "not_found_error",
    "message": "The requested resource could not be found."
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | `"error"` | 固定值。 |
| `error.type` | string | 见上方枚举表。 |
| `error.message` | string | 错误描述。 |
| `request_id` | string | 请求 ID，等同响应头中的 `request-id`。 |

`error.type` 取值可能随版本扩展，客户端需优雅处理未知值。

## 流式响应中的错误

启用 `stream: true` 时，HTTP 状态码可能是 200，但事件流中仍可能在任意时刻出现 `event: error`，载荷与上方 JSON 相同（无外层 `request_id`）：

```sse
event: error
data: {"type": "error", "error": {"type": "overloaded_error", "message": "Overloaded"}}
```

调用方在事件解析层必须处理该事件。

## 请求 ID

每个响应都返回 `request-id` 头，例如 `req_018EeWyXxfu5pfWkrYcMdjWG`。联系 support 时附上此 ID。

- Claude Platform on AWS：响应包含两个 ID，AWS 的 `x-amzn-requestid`（在 CloudTrail 中索引）与 Anthropic 的 `request-id`。用前者查 CloudTrail，用后者发 Anthropic support 工单。
- 官方 SDK 会把 `request-id` 暴露为顶层响应对象的属性。

## 长请求建议

- 大 `max_tokens` 时优先使用 streaming Messages API 或 Message Batches API，避免空闲连接被网络中间环节断开。
- 直连 API 时设置 TCP socket keep-alive 可降低断连概率。
- 官方 SDK 会校验非流式请求不应超过 10 分钟超时，并默认开启 TCP keep-alive。
- 如果只想拿最终 `Message`，可用 SDK 的 `.stream()` + `.get_final_message()`（Python） / `.finalMessage()`（TypeScript）。

## 常见校验错误

### 不支持 prefill 的模型

Claude Mythos Preview、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 4.6 不支持在最后一条 assistant 消息上 prefill，否则返回 400：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Prefilling assistant messages is not supported for this model."
  }
}
```

需要结构化输出时，改用 `output_config.format` 的 JSON schema 或 system prompt 指令。

### 非法 beta header

`anthropic-beta` 取值不在白名单内时：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Unsupported beta header: invalid-beta-name"
  }
}
```

## 参考

- 端点文档：`https://platform.claude.com/docs/en/api/errors`
- 版本与 beta：[versioning.md](./versioning.md)
- 上级目录：`https://platform.claude.com/docs/en/api/overview`
