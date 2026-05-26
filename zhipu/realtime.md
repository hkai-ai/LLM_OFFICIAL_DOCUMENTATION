---
source: https://docs.bigmodel.cn/cn/asyncapi/realtime
fetched_at: 2026-05-26
api_version: v4（WebSocket，AsyncAPI）
---

# 实时 API · WSS /api/paas/v4/realtime

GLM-Realtime 系列模型的双向流式语音 / 视频 / 文本通话接口。Server 端持续接收音视频字节，模型实时输出语音 + 文本，并支持工具调用。

## 端点

```text
wss://open.bigmodel.cn/api/paas/v4/realtime
```

## 鉴权

| 方式 | Header / Query |
| --- | --- |
| Server 端 | `Authorization: Bearer $ZHIPU_API_KEY`。 |
| Browser / Mobile | 用 server 端 API Key 自签 JWT（同 [README.md](./README.md) 描述），再以 `Authorization: Bearer <jwt>` 建链。 |

> 与 OpenAI Realtime 类似，建议浏览器场景**不要**把 server API Key 直接发到前端。

## 会话生命周期

1. 客户端 WebSocket 连接。
2. 发送 `session.update`，server 回 `session.updated` 确认。
3. 客户端用 `input_audio_buffer.append` 推流，`commit` 触发模型生成。
4. server 持续 push `response.audio.delta` / `response.text.delta` 等事件；`response.done` 表示一轮结束。
5. 客户端断开或 server 在异常时发 `error` + 关闭。

## 客户端 → server 事件

| `type` | 关键字段 | 说明 |
| --- | --- | --- |
| `session.update` | `session` | 更新当前 session（详见下方字段表）。 |
| `input_audio_buffer.append` | `audio`（base64） | 增量推送音频块；`session.input_audio_format` 决定格式。 |
| `input_audio_buffer.commit` | — | 提交当前 buffer，标记一段说话结束（`turn_detection.type: client_vad` 时使用）。 |
| `input_audio_buffer.clear` | — | 清空。 |
| `conversation.item.create` | `item` | 追加文本 / 工具结果 item。 |
| `conversation.item.delete` | `item_id` | 删除。 |
| `response.create` | `response` | 触发模型生成；可覆盖 modalities / instructions / tools。 |
| `response.cancel` | — | 中断当前响应。 |

## server → 客户端事件

| `type` | 说明 |
| --- | --- |
| `session.created` / `session.updated` | session 状态变更。 |
| `conversation.created` / `conversation.item.created` | 会话状态。 |
| `input_audio_buffer.speech_started` / `.speech_stopped` / `.committed` | 服务端 VAD 状态。 |
| `response.created` / `response.done` | 一轮响应生命周期；`response.done` 携带 `usage`。 |
| `response.audio.delta` / `.done` | 音频字节流（base64 PCM 24 kHz / 16-bit / mono）。 |
| `response.audio_transcript.delta` / `.done` | 模型输出音频的转写。 |
| `response.text.delta` / `.done` | 文本流。 |
| `response.function_call_arguments.delta` / `.done` | 工具调用参数增量。 |
| `heartbeat` | 长连接保活；客户端可忽略。 |
| `error` | `{ "error": { "type", "code", "message" } }`。 |

## session 字段

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `model` | enum | — | `glm-realtime-flash` / `glm-realtime-air`。 |
| `modalities` | `array<enum>` | `["text"]` | `text` / `audio`，可组合。 |
| `instructions` | string | — | system prompt。 |
| `input_audio_format` | enum | `pcm16` | `wav` / `pcm16` / `pcm24`。 |
| `output_audio_format` | enum | `pcm24` | 服务端固定 24 kHz / 16-bit / mono PCM。 |
| `voice` | enum | `tongtong` | `tongtong` / `female-tianmei` / `male-qn-daxuesheng` / `male-qn-jingying` / `lovely_girl` / `female-shaonv` 等。 |
| `turn_detection.type` | enum | `server_vad` | `server_vad` 由服务端检测；`client_vad` 客户端通过 `commit` 控制。 |
| `turn_detection.threshold` | number | — | `server_vad` 触发阈值。 |
| `turn_detection.prefix_padding_ms` | integer | — | — |
| `turn_detection.silence_duration_ms` | integer | — | — |
| `temperature` | number | — | `0`–`1`。 |
| `max_response_output_tokens` | integer | — | `0`–`1024`。 |
| `tools` | array | — | OpenAI 风格 function 定义。 |
| `tool_choice` | enum / object | `auto` | — |
| `input_audio_noise_reduction.type` | enum | — | `near_field` / `far_field`。 |
| `greeting_config` | object | — | 开场白配置。 |
| `beta_fields.chat_mode` | enum | — | `audio` / `video_passive`（视频被动接收）。 |

## 最小示例（伪代码）

```js
const ws = new WebSocket(
  "wss://open.bigmodel.cn/api/paas/v4/realtime",
  ["Authorization.Bearer." + JWT]
);

ws.onopen = () => ws.send(JSON.stringify({
  type: "session.update",
  session: {
    model: "glm-realtime-air",
    modalities: ["text", "audio"],
    voice: "tongtong",
    input_audio_format: "pcm16",
    turn_detection: { type: "server_vad", threshold: 0.5 }
  }
}));

ws.onmessage = (e) => {
  const msg = JSON.parse(e.data);
  if (msg.type === "response.audio.delta") {
    // 把 msg.delta (base64) 解码后送到 AudioContext
  } else if (msg.type === "response.text.delta") {
    console.log(msg.delta);
  }
};
```

## 计费

按音频秒数 + 文本 token 实时计费；详见 [pricing.md](./pricing.md)。

## 参考

- AsyncAPI：<https://docs.bigmodel.cn/cn/asyncapi/realtime>
- GLM-Realtime 模型：<https://docs.bigmodel.cn/cn/guide/models/sound-and-video/glm-realtime>
- Chat 接口：[chat-completions.md](./chat-completions.md)
- 音频（同步）：[audio.md](./audio.md)
