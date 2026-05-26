---
source: https://ai.google.dev/api/live?hl=zh-cn
fetched_at: 2026-05-26
api_version: v1beta
---

# Live API · BidiGenerateContent（WebSocket）

低延迟双向流式接口，用于语音 / 视频实时对话；客户端持续向 server 推流（音频 / 视频 / 文本），server 持续返回模型输出与工具调用事件。

## 端点

```text
wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1beta.GenerativeService.BidiGenerateContent
```

只在 `/v1beta` 下提供。

## 鉴权

| 方式 | 说明 |
| --- | --- |
| API Key | `?key=$GEMINI_API_KEY` 或 `x-goog-api-key` header。 |
| 短期 Access Token | 通过 `POST /v1beta/auth_tokens:create`（`AuthTokenService.CreateToken`）获取的临时 token，可放 `?access_token=` 查询参数或 `Authorization: Token <token>` header；适合浏览器 / 移动端避免直接暴露 API Key。 |

## 会话生命周期

1. 客户端连接 WebSocket。
2. **首条消息必须是 `BidiGenerateContentSetup`**；server 回复 `BidiGenerateContentSetupComplete` 表示握手完成。
3. 客户端发送 `clientContent` / `realtimeInput` / `toolResponse` 任一类型消息。
4. server 持续推送 `serverContent` / `toolCall` / `toolCallCancellation` / `usageMetadata` / `sessionResumptionUpdate` / `goAway`。
5. 客户端断开或 server 主动 `goAway`。

## 客户端 → server 消息

### 1. BidiGenerateContentSetup（首条）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | `models/{model_id}`，如 `models/gemini-2.5-flash-live-preview`。 |
| `generationConfig` | object | ✗ | 与 [generate-content.md](./generate-content.md) 的同名字段一致。**不支持** `responseLogprobs`、`responseMimeType`、`logprobs`、`responseSchema`、`stopSequence`、`routingConfig`、`audioTimestamp`。 |
| `systemInstruction` | object | ✗ | 仅文本；多个 part 表示多段。 |
| `tools` | array | ✗ | 函数 / 内置工具声明（结构同 generateContent）。 |
| `realtimeInputConfig` | object | ✗ | 实时输入开关：`automaticActivityDetection`（含 `disabled` / `startOfSpeechSensitivity` / `endOfSpeechSensitivity` / `prefixPaddingMs` / `silenceDurationMs`） + `activityHandling` + `turnCoverage`。 |
| `speechConfig` | object | ✗ | 输出语音配置（voice、language 等）。 |
| `sessionResumption` | object | ✗ | `{ "handle": "..." }` 复用上次会话状态。 |
| `contextWindowCompression` | object | ✗ | Sliding window 压缩策略。 |
| `inputAudioTranscription` | object | ✗ | 启用输入音频转写。 |
| `outputAudioTranscription` | object | ✗ | 启用输出音频转写。 |
| `proactivityConfig` | object | ✗ | 允许模型主动拒答。 |
| `historyConfig` | object | ✗ | 通过 `clientContent` 复用历史。 |

### 2. BidiGenerateContentClientContent（轮次更新）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `turns` | `array<Content>` | ✓ | 一次提交一个或多个完整轮次。 |
| `turnComplete` | boolean | ✗ | 是否声明本轮结束、可触发生成。 |

### 3. BidiGenerateContentRealtimeInput（流式音频 / 视频 / 文本）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `audio` | `Blob` | ✗ | 音频块，`mimeType` 通常 `audio/pcm;rate=16000`。 |
| `video` | `Blob` | ✗ | 视频帧，`mimeType` 如 `image/jpeg`、`image/png`。 |
| `text` | string | ✗ | 文本片段。 |
| `activityStart` | object | ✗ | 手动告知开始说话（automaticActivityDetection 关闭时）。 |
| `activityEnd` | object | ✗ | 手动告知结束。 |
| `audioStreamEnd` | boolean | ✗ | 输入音频流结束。 |

### 4. BidiGenerateContentToolResponse

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `functionResponses` | `array<FunctionResponse>` | ✓ | 与之前 `toolCall.functionCalls[].id` 一一对应。 |

## server → 客户端消息

| 消息 | 关键字段 |
| --- | --- |
| `setupComplete` | 空载荷；表示握手完成可发送 content。 |
| `serverContent` | `modelTurn`（`Content` 对象，含 `parts[]` 文本 / `inlineData` 音频帧）、`generationComplete`、`turnComplete`、`interrupted`、`inputTranscription` / `outputTranscription`（启用时）。 |
| `toolCall` | `functionCalls[]`（`id` / `name` / `args`）。 |
| `toolCallCancellation` | `ids[]` — 之前下发但被取消的 tool call。 |
| `sessionResumptionUpdate` | `newHandle` / `resumable` 表示当前可恢复 handle。 |
| `goAway` | `timeLeft`：server 即将断开前的剩余时间（ISO duration）。 |
| `usageMetadata` | `promptTokenCount` / `responseTokenCount` / `cachedContentTokenCount` 等；与 generateContent 一致。 |

## 音频 / 视频媒体格式

- 输入音频：默认 16 kHz / 16-bit / mono PCM，`mimeType: "audio/pcm;rate=16000"`。
- 输出音频：默认 24 kHz / 16-bit / mono PCM；通过 `speechConfig` 可改音色 / 语言。
- 视频帧：以 `image/*` mime 传单帧，建议低帧率（如 1 fps）以控成本。

## 最小示例（伪代码）

```js
const ws = new WebSocket(
  `wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1beta.GenerativeService.BidiGenerateContent?key=${API_KEY}`
);

ws.onopen = () => ws.send(JSON.stringify({
  setup: {
    model: "models/gemini-2.5-flash-live-preview",
    generationConfig: { responseModalities: ["AUDIO"] },
    speechConfig: { voiceConfig: { prebuiltVoiceConfig: { voiceName: "Aoede" } } }
  }
}));

ws.onmessage = (e) => {
  const msg = JSON.parse(e.data);
  if (msg.setupComplete) {
    ws.send(JSON.stringify({
      clientContent: {
        turns: [{ role: "user", parts: [{ text: "Hello!" }] }],
        turnComplete: true
      }
    }));
  } else if (msg.serverContent?.modelTurn) {
    // 处理 modelTurn.parts[].inlineData（音频帧）等
  } else if (msg.toolCall) {
    // 处理 toolCall.functionCalls[]
  }
};
```

## 限制与坑点

- Live API 仅支持特定带 `-live-` 后缀的模型（详见 [models.md](./models.md)）。
- 单条 WebSocket 会话有最长时长限制，超过会触发 `goAway`；使用 `sessionResumption` 可跨连接续接。
- `tools` 配置必须在 setup 中声明；建立连接后无法再追加。

## 参考

- 接口 reference：<https://ai.google.dev/api/live?hl=zh-cn>
- 指南：<https://ai.google.dev/gemini-api/docs/live?hl=zh-cn>
- 实时输入：<https://ai.google.dev/api/live#realtimeinputconfig>
- 会话续接：<https://ai.google.dev/gemini-api/docs/live-session>
