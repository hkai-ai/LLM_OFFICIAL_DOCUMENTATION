---
source: https://www.volcengine.com/docs/82379/1299023
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 错误码

火山方舟数据面 API 的错误响应包含 HTTP 状态码、错误类型（Type）、错误码（Code）、错误信息（Message）。消息中通常带 `Request ID: {{id}}`，排查时请提供该 ID。

## 错误响应结构

错误对象含 `code`（错误码）与 `message`（错误描述）。例如 Responses / 视频任务的 `error` 字段、图片生成的 `error` / `data[].error`。常见 HTTP 状态：`400` / `401` / `403` / `429` / `500`。

## 4xx · 请求错误（400 BadRequest）

| Code | 含义 |
| --- | --- |
| `MissingParameter` / `MissingParameter.{{Parameter}}` | 缺少必要参数。 |
| `InvalidParameter` / `InvalidParameter.{{Parameter}}` | 参数非法或值不合法。 |
| `InvalidParameter.UnsupportedParameter` | 该参数在此推理接入点不可用。 |
| `InvalidEndpoint.ClosedEndpoint` | 推理接入点已关闭或暂时不可用。 |
| `InvalidArgumentError`（MissingRole） | `messages` 中有消息缺少 `role`。 |
| `InvalidArgumentError.UnknownRole` | `role` 值不支持，或 `inference_role` 未定义。 |
| `InvalidArgumentError.InvalidImageDetail` | `image_url.detail` 仅接受 `auto`/`high`/`low`。 |
| `InvalidArgumentError.InvalidPixelLimit` | 自定义像素限制无效（如 `min_pixels > max_pixels`）。 |
| `InvalidImageURL.EmptyURL` / `InvalidImageURL.InvalidFormat` | 图片 URL 为空 / Base64 格式错误或不支持。 |
| `OutofContextError` | 文本 + 图片编码后总 token 超过模型上下文限制。 |
| `Duplicate.Tags.Key` | 标签存在重复 Key。 |

### 内容安全 / 审核（400 BadRequest）

| Code | 含义 |
| --- | --- |
| `SensitiveContentDetected`（`.SevereViolation` / `.Violence`） | 输入文本含敏感 / 严重违规 / 激进信息。 |
| `Input{Text/Image/Video/Audio}SensitiveContentDetected` | 输入文本 / 图片 / 视频 / 音频含敏感信息。 |
| `Output{Text/Image/Video/Audio}SensitiveContentDetected` | 生成内容含敏感信息。 |
| `Input{...}SensitiveContentDetected.PolicyViolation` | 输入可能涉及版权限制。 |
| `Input{Image/Video}SensitiveContentDetected.PrivacyInformation` | 输入图片 / 视频可能包含真人。 |
| `Input{Text/Image}RiskDetection` / `Output{Text/Image}RiskDetection` | 风险识别产品检测到敏感内容（含 `Label` / `SubLabel`）。 |
| `ContentSecurityDetectionError` | 风险识别产品请求失败。 |

## 401 · 鉴权错误

| Code | Type | 含义 |
| --- | --- | --- |
| `AuthenticationError` | Unauthorized | API Key 或 AK/SK 缺失 / 无效。 |
| `InvalidAccountStatus` | Forbidden | 账号状态异常。 |

## 403 · 禁止

| Code | 含义 |
| --- | --- |
| `InvalidSubscription` | Coding Plan 套餐未订阅或已过期。 |
| `OperationDenied.InvalidState` | 关联的 Context ID 处于非空闲（如 InProgress）状态。 |
| `OperationDenied.PermissionDenied` | 无权限访问基础模型配置。 |
| `OperationDenied.ConflictedValidationSet` | 不支持同时配置 ValidationSet 与 ValidationPercentage。 |

## 429 · 限流 / 配额（TooManyRequests）

| Code | 含义 |
| --- | --- |
| `RateLimitExceeded.EndpointRPMExceeded` | 推理接入点超过 RPM 限制。 |
| `RateLimitExceeded.EndpointTPMExceeded` | 推理接入点超过 TPM 限制。 |
| `ModelAccountRpmRateLimitExceeded` / `ModelAccountTpmRateLimitExceeded` | 账户模型 RPM / TPM 超限。 |
| `ModelAccountIpmRateLimitExceeded` | 账户模型 IPM（Images Per Minute）超限。 |
| `APIAccountRpmRateLimitExceeded` | 账户该接口 RPM 超限。 |
| `AccountRateLimitExceeded` | 请求过于频繁，超出 RPM/TPM。 |
| `QuotaExceeded` | 免费试用额度耗尽 / 排队任务数超限 / 5 小时·周·月用量超限。 |
| `ServerOverloaded` | 服务资源紧张（doubao-seed-1.8 及之前版本突增流量限制）。 |
| `RequestBurstTooFast` | 请求量激增触发系统保护（doubao-seed-2.0 及之后版本突增流量限制）。 |
| `SetLimitExceeded` | 达到设置的推理限额（安心体验模式）。 |
| `InflightBatchsizeExceeded` | 达到当前充值金额下的最大并发数限制。 |

## 500 · 服务端错误

| Code | Type | 含义 |
| --- | --- | --- |
| `InternalServiceError` | InternalServerError | 内部系统异常，请稍后重试。 |

## 模型精调错误码（节选）

| Code | 含义 |
| --- | --- |
| `InvalidData.MissingKey` / `InvalidData.UnknownKey` | 数据集缺少 / 含错误的列键。 |
| `InvalidData.InvalidValue` | 不支持的数据集类型（仅 `pretrain`/`dialog`/`dialog-dpo`/`multimodal`）。 |
| `InvalidData.InvalidJsonl` / `InvalidData.InvalidJson` | 文件非 `.jsonl` 格式 / JSON 解析失败。 |
| `InvalidData` | 数据集 TOS 地址不存在。 |
| `UnknownError` / `InternalError` | 平台服务错误 / 训练失败。 |

## 参考

- 错误码：https://www.volcengine.com/docs/82379/1299023
- 突发流量处理最佳实践：https://www.volcengine.com/docs/82379/1848593
- 火山引擎公共错误码：见官方文档站「公共错误码」。
