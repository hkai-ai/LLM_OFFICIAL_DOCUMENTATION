---
source: https://developers.openai.com/api/docs/guides/webhooks
fetched_at: 2026-05-26
api_version: v1
---

# Webhooks

异步任务（Batches / Fine-tuning / Evals / 后台 Responses / Realtime SIP 等）完成时，OpenAI 主动向用户配置的 URL POST 事件。免去客户端轮询。

## 配置入口

在 Console <https://platform.openai.com/settings/project/webhooks> 配置：

| 字段 | 说明 |
| --- | --- |
| Endpoint name | 用户标签 |
| Public server URL | 必须公网可达；建议带路径前缀做项目隔离 |
| Event types | 多选订阅 |

> 创建后会一次性显示 **signing secret**，立即保存；后续无法再读取。

## OpenAI → 你的服务

POST 到配置 URL：

```text
POST https://yourserver.com/webhook
Content-Type: application/json
webhook-id: wh_<identifier>
webhook-timestamp: 1716700000
webhook-signature: v1,<base64_hmac>
```

要求服务在数秒内返回 **2xx**，否则会按指数退避重试。

## 通用 payload 结构

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `object` | string | 固定 `event`。 |
| `id` | string | `evt_...`，唯一；用于去重。 |
| `type` | string | 事件类型，见下表。 |
| `created_at` | integer | Unix 秒。 |
| `data` | object | 事件相关数据；通常含 `id`（目标资源 ID）等。 |

示例：

```json
{
  "object": "event",
  "id": "evt_01J...",
  "type": "batch.completed",
  "created_at": 1716700000,
  "data": { "id": "batch_abc123" }
}
```

## 签名验证

签名遵循 [Standard Webhooks](https://www.standardwebhooks.com/) 规范。

```text
to_sign = webhook-id . "." . webhook-timestamp . "." . raw_body
signature_v1 = base64( HMAC-SHA256(secret, to_sign) )
```

`webhook-signature` 头格式：`v1,<sig1> v1,<sig2> ...`（用空格分隔允许多重签名以支持密钥滚动）。验证时与本地 secret 计算结果做**常量时间比较**。

> Python / Node / Go SDK 均提供 `unwrap()` 工具自动校验并返回事件对象。

## 已知事件类型

事件名采用 `<resource>.<state>` 命名，新事件随能力发布持续追加；以下为已知主类目。

### Batches（[messages-batches.md 的 OpenAI 端]：见 [files-and-batches.md](./files-and-batches.md)）

| 事件 | 触发 |
| --- | --- |
| `batch.completed` | 批次全部请求处理完成。 |
| `batch.failed` | 批次因输入校验等失败。 |
| `batch.cancelled` | 主动取消。 |
| `batch.expired` | 超出 24h 窗口。 |

`data` 含 `{ "id": "batch_..." }`。

### Fine-tuning Jobs（见 [fine-tuning.md](./fine-tuning.md)）

| 事件 | 触发 |
| --- | --- |
| `fine_tuning.job.succeeded` | 训练成功。 |
| `fine_tuning.job.failed` | 训练失败。 |
| `fine_tuning.job.cancelled` | 用户取消。 |

`data` 含 `{ "id": "ftjob_..." }`。

### Eval Runs（见 [evals.md](./evals.md)）

| 事件 | 触发 |
| --- | --- |
| `eval.run.succeeded` | run 完成。 |
| `eval.run.failed` | run 失败。 |
| `eval.run.canceled` | 取消。 |

`data` 含 `{ "id": "evalrun_..." }`。

### Background Responses（见 [responses.md](./responses.md)）

后台模式（`background: true`）的 Response 完成时触发。

| 事件 | 触发 |
| --- | --- |
| `response.completed` | 正常完成。 |
| `response.failed` | 失败。 |
| `response.cancelled` | 取消。 |
| `response.incomplete` | 超 max_output_tokens / max_tool_calls 等导致提前结束。 |

`data` 含 `{ "id": "resp_..." }`。

### Realtime SIP Calls（见 [realtime.md](./realtime.md)）

| 事件 | 触发 |
| --- | --- |
| `realtime.call.incoming` | 有入呼到达，需要 server 决定 accept / reject。 |

`data` 含 `{ "call_id": "...", "from": "...", "to": "..." }` 等。

## 最佳实践

- 用 `webhook-id` 做幂等去重，重试时同 ID 会重复送达。
- 在请求线程内只做**验签 + 落库**；业务处理走后台 worker，避免超时。
- 滚动密钥：在 Console 添加新 secret 后让两个 secret 同时生效一段时间，待客户端切换完成再删除旧密钥。
- 本地测试用 ngrok / Cloudflare Tunnel 等隧道暴露。

## 参考

- 指南：<https://developers.openai.com/api/docs/guides/webhooks>
- 事件清单：<https://developers.openai.com/api/reference/resources/webhooks>
- Standard Webhooks 规范：<https://www.standardwebhooks.com/>
