---
source: https://ai.google.dev/gemini-api/docs/troubleshooting?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 错误响应

> Gemini Developer API 沿用 [Google API 设计规范](https://cloud.google.com/apis/design/errors) 的错误结构，所有非 2xx 响应都返回统一形态的 JSON。

## 顶层结构

```json
{
  "error": {
    "code": 400,
    "message": "Invalid value at 'contents[0].role' ...",
    "status": "INVALID_ARGUMENT",
    "details": [ /* google.rpc.* 类型数组 */ ]
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `error.code` | integer | HTTP 状态码。 |
| `error.message` | string | 人类可读的错误描述（英文）。 |
| `error.status` | string | Google 规范的 canonical code 名称，见下表。 |
| `error.details` | array&lt;object&gt; | 详细信息数组，元素含 `@type` 字段（如 `type.googleapis.com/google.rpc.BadRequest`、`google.rpc.QuotaFailure`、`google.rpc.ErrorInfo`、`google.rpc.RetryInfo`、`google.rpc.Help`、`google.rpc.LocalizedMessage`），各自字段不同。 |

### 常见 `error.details` 元素

| `@type` | 关键字段 | 说明 |
| --- | --- | --- |
| `google.rpc.BadRequest` | `fieldViolations[].field` / `description` | 字段级校验错误。 |
| `google.rpc.QuotaFailure` | `violations[].subject` / `description` | 配额具体维度。 |
| `google.rpc.ErrorInfo` | `reason` / `domain` / `metadata` | 机器可读错误代码，常见 `reason=API_KEY_INVALID` / `MODEL_NOT_FOUND` / `BILLING_DISABLED` 等。 |
| `google.rpc.RetryInfo` | `retryDelay` | 建议重试间隔。 |
| `google.rpc.Help` | `links[].description` / `url` | 帮助链接。 |
| `google.rpc.LocalizedMessage` | `locale` / `message` | 本地化文本。 |

## HTTP 与 canonical status 映射

| HTTP | `error.status`（canonical code） | 典型触发场景 |
| --- | --- | --- |
| 200 | `OK` | 成功。 |
| 400 | `INVALID_ARGUMENT` | 字段缺失/格式错误、`responseSchema` 非法、`safetySettings.category` 错填。 |
| 400 | `FAILED_PRECONDITION` | Free tier 在某些地区不可用、模型要求开启计费、prompt 触发了不可处理的前置条件。 |
| 400 | `OUT_OF_RANGE` | 数值超界（如 `temperature > 2.0`）。 |
| 401 | `UNAUTHENTICATED` | 未提供 API Key 或 Key 不可识别。 |
| 403 | `PERMISSION_DENIED` | Key 未启用该 API、被 IP/Referer 限制、调用方账号被禁用。 |
| 404 | `NOT_FOUND` | 模型 ID、`files/...`、`cachedContents/...` 不存在或已过期。 |
| 409 | `ABORTED` | 并发修改冲突（如 `cachedContents.patch`）。 |
| 409 | `ALREADY_EXISTS` | 资源已存在（如指定 `name` 的 file 重复上传）。 |
| 429 | `RESOURCE_EXHAUSTED` | RPM / TPM / RPD / 存储 / 并发配额超限；常见 `reason=RATE_LIMIT_EXCEEDED`。 |
| 499 | `CANCELLED` | 客户端主动断开。 |
| 500 | `INTERNAL` | 服务端未预期错误。 |
| 501 | `UNIMPLEMENTED` | 方法在该模型或该 API 版本（如 `/v1`）上未实现。 |
| 503 | `UNAVAILABLE` | 模型暂时不可用、容量过载，建议指数退避重试。 |
| 504 | `DEADLINE_EXCEEDED` | 服务端处理超时（长上下文 / 复杂工具链常见）。 |
| — | `UNKNOWN` | 服务器返回了无法映射到 canonical code 的错误。 |
| — | `DATA_LOSS` | 数据完整性失败，极少见。 |

## 流式响应中的错误

`streamGenerateContent` 在 chunk 中可能直接返回带 `error` 的对象，结构同上；调用方需在解析时兼容。响应头阶段的错误（4xx/5xx）则不会进入流模式，直接以普通 JSON 返回。

## 业务侧"软错误"

下列情况 HTTP 仍是 200，需要通过响应字段判断：

| 场景 | 判定字段 |
| --- | --- |
| Prompt 被安全策略拦截 | `promptFeedback.blockReason` 非空，`candidates` 为空。 |
| Candidate 被安全/复述拦截 | `candidates[].finishReason ∈ {SAFETY, RECITATION, BLOCKLIST, PROHIBITED_CONTENT, SPII, IMAGE_SAFETY}` |
| 函数调用结构不合法 | `candidates[].finishReason = MALFORMED_FUNCTION_CALL` |
| 工具被非法触发 | `candidates[].finishReason = UNEXPECTED_TOOL_CALL` |
| File 异步处理失败 | `File.state = FAILED`，详情在 `File.error`（结构同顶层 `error`） |

## 重试建议

| 状态 | 建议 |
| --- | --- |
| `RESOURCE_EXHAUSTED`(429) | 指数退避，参考 `details[].retryDelay`；或申请提升配额。 |
| `UNAVAILABLE`(503) | 指数退避重试，建议 jitter。 |
| `DEADLINE_EXCEEDED`(504) | 缩短上下文 / 关闭 thinking / 拆分请求后重试。 |
| `INTERNAL`(500) | 短时间内可重试 1-2 次。 |
| `INVALID_ARGUMENT` / `FAILED_PRECONDITION` / `PERMISSION_DENIED` / `NOT_FOUND` | 不可重试，应修复请求或权限。 |

## 示例

### 配额超限

```json
{
  "error": {
    "code": 429,
    "message": "Resource has been exhausted (e.g. check quota).",
    "status": "RESOURCE_EXHAUSTED",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.QuotaFailure",
        "violations": [
          { "quotaMetric": "generativelanguage.googleapis.com/generate_content_requests", "quotaId": "GenerateContentRequestsPerMinutePerProjectPerModel" }
        ]
      },
      {
        "@type": "type.googleapis.com/google.rpc.RetryInfo",
        "retryDelay": "32s"
      }
    ]
  }
}
```

### Key 无效

```json
{
  "error": {
    "code": 400,
    "message": "API key not valid. Please pass a valid API key.",
    "status": "INVALID_ARGUMENT",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.ErrorInfo",
        "reason": "API_KEY_INVALID",
        "domain": "googleapis.com",
        "metadata": { "service": "generativelanguage.googleapis.com" }
      }
    ]
  }
}
```

## 参考

- 错误故障排查：<https://ai.google.dev/gemini-api/docs/troubleshooting?hl=zh-CN>
- Google API 错误模型：<https://cloud.google.com/apis/design/errors>
