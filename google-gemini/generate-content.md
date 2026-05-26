---
source: https://ai.google.dev/api/generate-content?hl=zh-CN
fetched_at: 2026-05-19
api_version: v1beta
---

# 生成内容 · POST /v1beta/{model=models/*}:generateContent

> 同步生成单次模型回复，支持文本、图像、音频、视频、PDF 多模态输入，可启用工具调用、代码执行、Google Search Grounding、结构化输出、思考（thinking）等能力。

`/v1` 路径同名端点存在，但不包含 `thinkingConfig`、`urlContext` 等较新字段；下表标 `v1beta only` 的字段在 `/v1` 中不可用。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` | ✓（或 query `key`） | API Key。 |
| `Content-Type` | ✓ | `application/json`。 |
| `x-goog-user-project` | ✗ | 计费项目覆盖，仅 Vertex 模式相关，Developer API 通常不需要。 |

## Path 参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型资源名，格式 `models/{model}`，如 `models/gemini-2.5-pro`。允许值见 [models.md](./models.md)。 |

## 请求 Body

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `contents` | array&lt;Content&gt; | ✓ | — | 当前会话内容。单轮可只放一条 user 消息；多轮按时间顺序排列。 |
| `tools` | array&lt;Tool&gt; | ✗ | — | 模型可调用的工具列表（function calling、Google Search、代码执行、URL Context 等）。 |
| `toolConfig` | ToolConfig | ✗ | — | 工具调用配置。 |
| `safetySettings` | array&lt;SafetySetting&gt; | ✗ | — | 每个 `HarmCategory` 独立的拦截阈值与方法。 |
| `systemInstruction` | Content | ✗ | — | 系统指令；仅含 `parts[].text`，role 字段服务端忽略。 |
| `generationConfig` | GenerationConfig | ✗ | — | 解码、输出格式、思考、语音等通用控制参数。 |
| `cachedContent` | string | ✗ | — | 已缓存上下文资源名，格式 `cachedContents/{id}`。 |
| `labels` | map&lt;string, string&gt; | ✗ | — | 调用方自定义标签，用于账单维度划分。 |

### `contents[]`（Content）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `role` | string | ✗ | — | 取 `user` / `model` / `system`（`system` 仅在 `systemInstruction` 中使用）；多轮中 user/model 必须交替。 |
| `parts` | array&lt;Part&gt; | ✓ | — | 内容片段，按出现顺序拼接。 |

### `contents[].parts[]`（Part，oneof）

每个 Part 只能填以下 `data` 字段之一；可独立携带 `thought`、`thoughtSignature`、`videoMetadata` 等修饰字段。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `text` | string | 文本片段。 |
| `inlineData` | Blob | 内联二进制数据，含 `mimeType`、`data`（base64）。 |
| `fileData` | FileData | 引用 Files API 的资源，含 `mimeType`、`fileUri`。 |
| `functionCall` | FunctionCall | 模型发起的函数调用，含 `name`、`args`、可选 `id`。 |
| `functionResponse` | FunctionResponse | 调用方回传的函数执行结果，含 `name`、`response`、可选 `id`、`willContinue`、`scheduling`。 |
| `executableCode` | ExecutableCode | 代码执行工具产出的代码，含 `language`（`PYTHON` 等）、`code`。 |
| `codeExecutionResult` | CodeExecutionResult | 代码执行结果，含 `outcome`（`OUTCOME_OK` / `OUTCOME_FAILED` / `OUTCOME_DEADLINE_EXCEEDED`）、`output`。 |
| `thought` | boolean | 标记该 Part 是模型思考链；启用 `thinkingConfig.includeThoughts` 后可能出现。 |
| `thoughtSignature` | string (bytes) | 思考签名，用于多轮维持思考状态。 |
| `videoMetadata` | VideoMetadata | 仅在 `fileData`/`inlineData` 为视频时使用，含 `startOffset`、`endOffset`、`fps`。 |

### `tools[]`（Tool）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `functionDeclarations` | array&lt;FunctionDeclaration&gt; | 可被模型调用的函数声明列表。 |
| `googleSearch` | object | 启用 Google Search Tool（Gemini 2.x+ 推荐）。 |
| `googleSearchRetrieval` | GoogleSearchRetrieval | 旧 1.5 系搜索 Grounding；含 `dynamicRetrievalConfig.mode` / `dynamicThreshold`。**已弃用**，新模型用 `googleSearch`。 |
| `codeExecution` | object | 启用内置 Python 代码执行。 |
| `urlContext` | object | v1beta only。启用 URL Context（自动抓取 prompt 中 URL）。 |
| `enterpriseWebSearch` | object | 企业版 Search Grounding，仅特定接入开放。 |

`FunctionDeclaration` 字段：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 函数名，正则 `[A-Za-z_][A-Za-z0-9_.-]{0,63}`。 |
| `description` | string | ✗ | 函数用途。 |
| `parameters` | Schema | ✗ | 入参 JSON Schema（OpenAPI 3.0 子集）。 |
| `response` | Schema | ✗ | 出参 Schema（供 grounding/校验）。 |
| `behavior` | string | ✗ | `BLOCKING` / `NON_BLOCKING`，控制是否阻塞主响应。 |

### `toolConfig`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `functionCallingConfig.mode` | string | `MODE_UNSPECIFIED` / `AUTO`（默认，模型自决） / `ANY`（必须调用，可配合 `allowedFunctionNames`） / `NONE`（禁止调用） / `VALIDATED`（v1beta 新增，强制按 schema 校验）。 |
| `functionCallingConfig.allowedFunctionNames` | array&lt;string&gt; | `mode=ANY` 时限定可调用的函数名子集。 |

### `safetySettings[]`（SafetySetting）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `category` | HarmCategory | ✓ | 见下方枚举。 |
| `threshold` | HarmBlockThreshold | ✓ | 见下方枚举。 |
| `method` | HarmBlockMethod | ✗ | `HARM_BLOCK_METHOD_UNSPECIFIED` / `SEVERITY`（按严重度） / `PROBABILITY`（按概率）。默认按服务端配置。 |

**HarmCategory 全部枚举**：

| 值 | 说明 |
| --- | --- |
| `HARM_CATEGORY_UNSPECIFIED` | 默认。 |
| `HARM_CATEGORY_DEROGATORY` | PaLM 旧分类。 |
| `HARM_CATEGORY_TOXICITY` | PaLM 旧分类。 |
| `HARM_CATEGORY_VIOLENCE` | PaLM 旧分类。 |
| `HARM_CATEGORY_SEXUAL` | PaLM 旧分类。 |
| `HARM_CATEGORY_MEDICAL` | PaLM 旧分类。 |
| `HARM_CATEGORY_DANGEROUS` | PaLM 旧分类。 |
| `HARM_CATEGORY_HARASSMENT` | Gemini 当前类别。 |
| `HARM_CATEGORY_HATE_SPEECH` | Gemini 当前类别。 |
| `HARM_CATEGORY_SEXUALLY_EXPLICIT` | Gemini 当前类别。 |
| `HARM_CATEGORY_DANGEROUS_CONTENT` | Gemini 当前类别。 |
| `HARM_CATEGORY_CIVIC_INTEGRITY` | Gemini 当前类别。 |

> 设置时只需对实际启用的 4 个 Gemini 类别（HARASSMENT / HATE_SPEECH / SEXUALLY_EXPLICIT / DANGEROUS_CONTENT，部分模型还含 CIVIC_INTEGRITY）传值；PaLM 旧分类对 Gemini 模型无效。

**HarmBlockThreshold 全部枚举**：

| 值 | 说明 |
| --- | --- |
| `HARM_BLOCK_THRESHOLD_UNSPECIFIED` | 服务端默认。 |
| `BLOCK_LOW_AND_ABOVE` | 拦截低及以上。 |
| `BLOCK_MEDIUM_AND_ABOVE` | 拦截中及以上（多数模型默认）。 |
| `BLOCK_ONLY_HIGH` | 仅拦截高。 |
| `BLOCK_NONE` | 不拦截，但仍返回安全评级。 |
| `OFF` | 完全关闭该分类的安全过滤（需账户具备相应权限）。 |

### `generationConfig`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `stopSequences` | array&lt;string&gt; | ✗ | — | 最多 5 个，命中后停止生成且不包含在输出中。 |
| `responseMimeType` | string | ✗ | `text/plain` | 取 `text/plain` / `application/json` / `text/x.enum`。设为 JSON 时通常配合 `responseSchema`。 |
| `responseSchema` | Schema | ✗ | — | 结构化输出 Schema（OpenAPI 3.0 子集）。仅在 `responseMimeType=application/json` 或 `text/x.enum` 时生效。 |
| `responseJsonSchema` | object | ✗ | — | 标准 JSON Schema 形式（与 `responseSchema` 二选一）。 |
| `responseModalities` | array&lt;Modality&gt; | ✗ | `["TEXT"]` | 取值 `TEXT` / `IMAGE` / `AUDIO`，由模型能力决定可组合性。 |
| `candidateCount` | integer | ✗ | `1` | 候选数；多数模型仅支持 1。 |
| `maxOutputTokens` | integer | ✗ | 模型默认 | 单次最大输出 token 数。 |
| `temperature` | number | ✗ | 模型默认 | 范围 `[0.0, 2.0]`，越大越随机。 |
| `topP` | number | ✗ | 模型默认 | Nucleus 采样。 |
| `topK` | integer | ✗ | 模型默认 | Top-K 采样；部分模型不支持。 |
| `seed` | integer | ✗ | — | 可复现采样种子（best-effort，并非严格确定）。 |
| `presencePenalty` | number | ✗ | `0.0` | 与 OpenAI 同义。 |
| `frequencyPenalty` | number | ✗ | `0.0` | 与 OpenAI 同义。 |
| `responseLogprobs` | boolean | ✗ | `false` | 是否在响应中返回 logprobs。 |
| `logprobs` | integer | ✗ | — | 每个位置返回的 top logprobs 数量。 |
| `enableEnhancedCivicAnswers` | boolean | ✗ | `false` | 增强公民信息答复（受地域限制）。 |
| `speechConfig` | SpeechConfig | ✗ | — | TTS / 音频输出配置。 |
| `thinkingConfig` | ThinkingConfig | ✗ | — | 思考预算与是否在响应中包含思考链。 |
| `mediaResolution` | string | ✗ | — | 取 `MEDIA_RESOLUTION_UNSPECIFIED` / `MEDIA_RESOLUTION_LOW` / `MEDIA_RESOLUTION_MEDIUM` / `MEDIA_RESOLUTION_HIGH`，影响图像/视频 token 消耗。 |
| `routingConfig` | object | ✗ | — | v1beta 实验性，模型路由控制。 |

### `generationConfig.speechConfig`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `voiceConfig.prebuiltVoiceConfig.voiceName` | string | 预制音色名（如 `Kore`、`Charon`、`Aoede` 等，由模型决定可选集合）。 |
| `multiSpeakerVoiceConfig.speakerVoiceConfigs[].speaker` | string | 多说话人标签。 |
| `multiSpeakerVoiceConfig.speakerVoiceConfigs[].voiceConfig` | VoiceConfig | 每个标签的音色。 |
| `languageCode` | string | BCP-47，如 `en-US`、`zh-CN`。 |

### `generationConfig.thinkingConfig`

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `thinkingBudget` | integer | 模型默认 | 允许的思考 token 上限；设 `0` 关闭（部分模型不支持关闭）。 |
| `includeThoughts` | boolean | `false` | 是否在响应 `parts[]` 中返回带 `thought=true` 的思考片段。 |

> `thinkingConfig` 仅 `gemini-2.5-*`、`gemini-3.*` 等具备思考能力的模型支持；对其他模型设置会被忽略或报错。

## 响应（GenerateContentResponse）

### 顶层

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `candidates` | array&lt;Candidate&gt; | 候选响应列表，长度受 `candidateCount` 控制。 |
| `promptFeedback` | PromptFeedback | 输入侧的安全反馈；输入被拦截时 `candidates` 可能为空。 |
| `usageMetadata` | UsageMetadata | token 用量。 |
| `modelVersion` | string | 实际服务版本，如 `gemini-2.5-pro-002`。 |
| `responseId` | string | 服务端唯一响应 ID。 |

### `candidates[]`（Candidate）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `content` | Content | 模型返回内容，`role=model`。 |
| `finishReason` | string | 见下方枚举。 |
| `safetyRatings` | array&lt;SafetyRating&gt; | 各 HarmCategory 的概率/严重度评级；含 `category`、`probability`、`severity`、`probabilityScore`、`severityScore`、`blocked`。 |
| `citationMetadata` | CitationMetadata | 引用片段列表（`citationSources[].startIndex` 等）。 |
| `tokenCount` | integer | 该候选输出 token 数。 |
| `groundingAttributions` | array | 旧 grounding 字段（1.5 系）。 |
| `groundingMetadata` | GroundingMetadata | Search Grounding 元数据，含 `webSearchQueries`、`searchEntryPoint`、`groundingChunks`、`groundingSupports`、`retrievalMetadata`。 |
| `urlContextMetadata` | UrlContextMetadata | 启用 `urlContext` 时的 URL 抓取结果。 |
| `avgLogprobs` | number | 该候选的平均 logprob。 |
| `logprobsResult` | LogprobsResult | 含 `topCandidates[]`、`chosenCandidates[]`。 |
| `index` | integer | 候选下标。 |
| `finishMessage` | string | 终止补充说明（如安全拦截原因文本）。 |

**FinishReason 全部枚举**：

| 值 | 说明 |
| --- | --- |
| `FINISH_REASON_UNSPECIFIED` | 默认占位。 |
| `STOP` | 自然结束或命中 `stopSequences`。 |
| `MAX_TOKENS` | 达到 `maxOutputTokens`。 |
| `SAFETY` | 安全分类拦截。 |
| `RECITATION` | 复述训练数据被拦截。 |
| `LANGUAGE` | 输出语言不被支持。 |
| `OTHER` | 其他原因。 |
| `BLOCKLIST` | 命中术语黑名单。 |
| `PROHIBITED_CONTENT` | 禁止内容。 |
| `SPII` | 敏感个人信息。 |
| `MALFORMED_FUNCTION_CALL` | 模型生成的 functionCall 不合法。 |
| `IMAGE_SAFETY` | 图像安全拦截。 |
| `UNEXPECTED_TOOL_CALL` | 在不允许的工具上发起调用。 |

### `promptFeedback`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `blockReason` | string | `BLOCK_REASON_UNSPECIFIED` / `SAFETY` / `OTHER` / `BLOCKLIST` / `PROHIBITED_CONTENT` / `IMAGE_SAFETY`。 |
| `safetyRatings` | array&lt;SafetyRating&gt; | 输入侧的安全评级。 |
| `blockReasonMessage` | string | 文本描述（部分场景）。 |

### `usageMetadata`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `promptTokenCount` | integer | 输入 token 总数。 |
| `cachedContentTokenCount` | integer | 来自 `cachedContent` 的 token 数。 |
| `candidatesTokenCount` | integer | 所有候选输出 token 数。 |
| `totalTokenCount` | integer | 计费 token 总数。 |
| `toolUsePromptTokenCount` | integer | 工具调用引入的 prompt 增量 token。 |
| `thoughtsTokenCount` | integer | 思考 token 数。 |
| `promptTokensDetails` | array&lt;ModalityTokenCount&gt; | 输入分模态拆分。 |
| `cacheTokensDetails` | array&lt;ModalityTokenCount&gt; | 缓存命中分模态拆分。 |
| `candidatesTokensDetails` | array&lt;ModalityTokenCount&gt; | 输出分模态拆分。 |
| `toolUsePromptTokensDetails` | array&lt;ModalityTokenCount&gt; | 工具引入分模态拆分。 |

`ModalityTokenCount`：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `modality` | string | `MODALITY_UNSPECIFIED` / `TEXT` / `IMAGE` / `VIDEO` / `AUDIO` / `DOCUMENT`。 |
| `tokenCount` | integer | 对应模态 token 数。 |

## 示例

### 最小请求

```json
{
  "contents": [
    { "role": "user", "parts": [{ "text": "用一句话介绍量子纠缠。" }] }
  ]
}
```

`POST /v1beta/models/gemini-2.5-flash:generateContent`，`x-goog-api-key: $GEMINI_API_KEY`。

### 最小响应

```json
{
  "candidates": [
    {
      "content": {
        "role": "model",
        "parts": [{ "text": "量子纠缠是指两个或多个粒子的状态相互关联，无论相距多远，对其一的测量会瞬时决定另一者的对应观测结果。" }]
      },
      "finishReason": "STOP",
      "index": 0
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 12,
    "candidatesTokenCount": 38,
    "totalTokenCount": 50
  },
  "modelVersion": "gemini-2.5-flash-002",
  "responseId": "abc123..."
}
```

### 工具调用（function calling）请求

```json
{
  "contents": [
    { "role": "user", "parts": [{ "text": "明天北京天气？" }] }
  ],
  "tools": [
    {
      "functionDeclarations": [
        {
          "name": "get_weather",
          "description": "查询指定城市某日天气",
          "parameters": {
            "type": "OBJECT",
            "properties": {
              "city": { "type": "STRING" },
              "date": { "type": "STRING", "format": "date" }
            },
            "required": ["city", "date"]
          }
        }
      ]
    }
  ],
  "toolConfig": {
    "functionCallingConfig": { "mode": "AUTO" }
  }
}
```

### 结构化输出请求

```json
{
  "contents": [{ "role": "user", "parts": [{ "text": "列出 3 部宫崎骏代表作。" }] }],
  "generationConfig": {
    "responseMimeType": "application/json",
    "responseSchema": {
      "type": "ARRAY",
      "items": {
        "type": "OBJECT",
        "properties": {
          "title": { "type": "STRING" },
          "year": { "type": "INTEGER" }
        },
        "required": ["title", "year"]
      }
    }
  }
}
```

## 错误

| HTTP | `error.status` | 触发原因 |
| --- | --- | --- |
| 400 | `INVALID_ARGUMENT` | 字段缺失/格式错误（如 `contents` 为空、`role` 非法）。 |
| 400 | `FAILED_PRECONDITION` | Free tier 不可用、模型不接受该模态等。 |
| 403 | `PERMISSION_DENIED` | API Key 无权限或被拒。 |
| 404 | `NOT_FOUND` | 模型 ID 不存在。 |
| 429 | `RESOURCE_EXHAUSTED` | 触发 RPM/TPM/RPD 限额。 |
| 500 | `INTERNAL` | 服务端异常。 |
| 503 | `UNAVAILABLE` | 模型过载或维护。 |
| 504 | `DEADLINE_EXCEEDED` | 请求处理超时（长上下文/复杂工具链常见）。 |

完整错误结构见 [errors.md](./errors.md)。

## 参考

- 端点文档：<https://ai.google.dev/api/generate-content?hl=zh-CN>
- 安全设置说明：<https://ai.google.dev/gemini-api/docs/safety-settings>
- Thinking：<https://ai.google.dev/gemini-api/docs/thinking>
- 结构化输出：<https://ai.google.dev/gemini-api/docs/structured-output>
