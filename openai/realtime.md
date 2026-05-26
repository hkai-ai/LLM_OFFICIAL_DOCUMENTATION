---
source: https://developers.openai.com/api/reference/resources/realtime
fetched_at: 2026-05-26
api_version: v1
---

# Realtime API · /v1/realtime

实时双向语音 / 文本 / 工具调用，主要走 WebSocket；REST 端点用来发放短期 token（client secret）与控制 SIP 电话呼叫。

| 通道 | 用途 |
| --- | --- |
| WebSocket | 双向事件流，承载音视频字节与模型输出。 |
| REST · Client Secrets | 浏览器 / 移动端用 server 颁发短期 token 鉴权。 |
| REST · Calls (SIP) | 电话呼入控制（accept / hangup / refer / reject）。 |

## 鉴权与请求头（REST）

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。WebSocket 既可用 server API Key，也可用通过 client secret 端点发放的短期 token。 |
| `Content-Type` | ✓ | `application/json`。 |
| `OpenAI-Beta` | 部分 beta 阶段需要 | `realtime=v1`。新版可省略。 |

## 1. Client Secrets

### Create client secret · POST /v1/realtime/client_secrets

用 server 端 API Key 换一份**短期 token**，前端直接拿它建 WebSocket，避免泄漏 server Key。

#### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `session` | object | ✓ | 与 WebSocket `session.update` 等价的 session 配置。 |
| `expires_after` | object | ✗ | `{ "anchor": "created_at", "seconds": 600 }`，默认 ~1 min。 |

#### session 主要字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | enum | 固定 `realtime`。 |
| `model` | string | 例如 `gpt-realtime`、`gpt-4o-realtime-preview`。 |
| `modalities` | `array<string>` | `["text"]` / `["audio"]` / `["text","audio"]`。 |
| `instructions` | string | 类似 system prompt。 |
| `audio.input.format` | object | 输入 `pcm16` / `g711_ulaw` / `g711_alaw` 等。 |
| `audio.input.transcription` | object | `{ "model": "whisper-1" \| "gpt-4o-transcribe-..." , "language": "zh" }`。 |
| `audio.input.turn_detection` | object | `{ "type": "server_vad" \| "semantic_vad" , "threshold": 0.5, "prefix_padding_ms": 300, "silence_duration_ms": 200 }`。 |
| `audio.output.format` | object | 输出格式。 |
| `audio.output.voice` | string | 系统 voice 名（如 `alloy` / `verse`）。 |
| `audio.output.speed` | number | 输出速率 0.25–1.5。 |
| `tools` | array | 函数 / 内置工具列表。 |
| `tool_choice` | string \| object | `auto` / `required` 等。 |
| `temperature` | number | 采样温度。 |
| `max_response_output_tokens` | integer \| `"inf"` | — |
| `mcp_servers` | array | MCP 服务器声明（部分新版本支持）。 |

#### 响应

```json
{
  "id": "ephemeral_abc",
  "object": "realtime.client_secret",
  "value": "eph_secret_xxx",
  "expires_at": 1716700600,
  "session": { /* 同请求 */ }
}
```

`value` 直接放在 WebSocket 的 `Authorization: Bearer <value>` 中使用。

### Translation 专用 · POST /v1/realtime/translations/client_secrets

请求 / 响应结构同上，但 session 内多出 `translation.input_language` / `translation.output_language` 等翻译专属字段，且事件协议为翻译子集。

## 2. WebSocket 协议

### 端点

```text
wss://api.openai.com/v1/realtime?model=gpt-realtime
```

可选 query 参数：`?model=<model_id>`、`?intent=transcription`（仅做转写）等。

### 鉴权

| 方式 | Header |
| --- | --- |
| Server 端 | `Authorization: Bearer $OPENAI_API_KEY` |
| 浏览器 | `Authorization: Bearer <client_secret_value>` |

部分 beta 阶段还需 `OpenAI-Beta: realtime=v1`。

### Client → Server 事件

| `type` | 关键字段 | 说明 |
| --- | --- | --- |
| `session.update` | `session` | 更新当前 session 配置（同 client_secret 的 session）。 |
| `input_audio_buffer.append` | `audio`（base64） | 增量推送音频片段。 |
| `input_audio_buffer.commit` | — | 提交当前 buffer 作为一个 user turn。 |
| `input_audio_buffer.clear` | — | 清空未提交音频。 |
| `conversation.item.create` | `item`、`previous_item_id` | 追加 user / assistant / tool message。 |
| `conversation.item.truncate` | `item_id`、`content_index`、`audio_end_ms` | 截断之前生成的音频。 |
| `conversation.item.delete` | `item_id` | 删除。 |
| `response.create` | `response` | 触发模型生成；可覆盖 modalities / instructions / tools 等。 |
| `response.cancel` | — | 中断当前 response。 |
| `mcp_approval_response` | `approval_request_id`、`approve` | 回复 MCP 工具批准请求。 |

### Server → Client 事件

| `type` | 说明 |
| --- | --- |
| `session.created` / `session.updated` | session 状态变更回执。 |
| `conversation.created` | 会话建立。 |
| `conversation.item.added` / `.done` | item 进出状态。 |
| `conversation.item.input_audio_transcription.delta` / `.completed` / `.failed` | 输入音频 ASR 结果。 |
| `input_audio_buffer.speech_started` / `.speech_stopped` / `.committed` / `.cleared` | VAD 状态。 |
| `response.created` / `.done` | 响应生命周期。 |
| `response.output_item.added` / `.done` | 输出 item 增量。 |
| `response.content_part.added` / `.done` | content part 增量。 |
| `response.text.delta` / `.done` | 文本流。 |
| `response.audio.delta` / `.done` | 音频字节流（base64）。 |
| `response.audio_transcript.delta` / `.done` | 模型输出语音的转写。 |
| `response.function_call_arguments.delta` / `.done` | 函数调用增量。 |
| `rate_limits.updated` | 当前 rate-limit 配额。 |
| `error` | `{ "error": { "type", "code", "message", "param" } }`。 |

## 3. SIP Calls

适用于让模型作为电话坐席处理 SIP 入呼。

| 端点 | 说明 |
| --- | --- |
| `POST /v1/realtime/calls/{call_id}/accept` | body: 接听时使用的 `session`（同上）。 |
| `POST /v1/realtime/calls/{call_id}/reject` | body: `{ "status_code": 486 }` 等。 |
| `POST /v1/realtime/calls/{call_id}/hangup` | 主动挂断；空 body。 |
| `POST /v1/realtime/calls/{call_id}/refer` | body: `{ "target_uri": "tel:+1234..." }` 转接。 |

呼叫接听后，依然走 WebSocket 事件协议；call ID 体现在 `call_id` 字段中。

## 计费

按 audio / text token 实时计费；详见 [pricing.md](./pricing.md) §Realtime。Client secret 自身免费。

## 错误码

| HTTP / event | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | session 字段不合法、超出 token 上限 |
| `401` | `authentication_error` | client_secret 已过期 |
| `429` | `rate_limit_error` | Realtime 独立配额 |
| event `error` | `server_error` | 推理端异常 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/realtime>
- 指南：<https://developers.openai.com/api/docs/guides/realtime>
- Client secret 创建：<https://developers.openai.com/api/reference/resources/realtime/methods/create_client_secret>
- WebSocket 协议参考：<https://developers.openai.com/api/docs/guides/realtime-websocket>
- SIP 接入：<https://developers.openai.com/api/docs/guides/realtime-sip>
