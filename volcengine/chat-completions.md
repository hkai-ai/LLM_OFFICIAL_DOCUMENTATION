---
source: https://www.volcengine.com/docs/82379/1494384
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3，无独立版本号）
---

# 对话(Chat) API · POST /api/v3/chat/completions

> 发送包含文本、图片、视频、音频等模态的消息列表，模型生成对话中的下一条消息。火山方舟（豆包 Doubao 系列）的核心文本生成端点，请求 / 响应字段与 OpenAI Chat Completions 高度兼容。

完整 URL：`POST https://ark.cn-beijing.volces.com/api/v3/chat/completions`

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY`，API Key 在[控制台 API Key 管理](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey)创建。 |
| `Content-Type` | ✓ | `application/json` |

## 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 调用的模型 ID（Model ID），需先开通模型服务。多应用 / 精细管理场景推荐用 Endpoint ID 调用。 |
| `messages` | object[] | ✓ | — | 消息列表。不同模型支持的消息 / 模态类型不同（文本、图片、视频、音频）。见 [`messages[]`](#messages)。 |
| `thinking` | object | ✗ | `{"type":"enabled"}` | 控制是否开启深度思考模式。不同模型是否支持及默认值不同。见 [`thinking`](#thinking)。 |
| `stream` | boolean \| null | ✗ | `false` | `false` 一次性返回；`true` 按 SSE 逐块返回，以 `data: [DONE]` 结束。 |
| `stream_options` | object \| null | ✗ | `null` | 流式选项，仅 `stream=true` 时生效。见 [`stream_options`](#stream_options)。 |
| `max_tokens` | integer \| null | ✗ | `4096` | 模型回答最大长度（token，不含思维链）。取值范围因模型而异，见模型列表。 |
| `max_completion_tokens` | integer \| null | ✗ | — | 模型输出最大长度（含回答 + 思维链）。取值范围 `[1, 65536]`。配置后 `max_tokens` 默认值失效；不可与 `max_tokens` 同时设置。 |
| `service_tier` | string \| null | ✗ | `auto` | 在线推理模式：`fast`（低延迟）/ `auto`（优先 TPM 保障包）/ `default`（仅常规）。详见下方枚举说明。 |
| `stop` | string \| string[] \| null | ✗ | `null` | 命中即停止生成（不输出该词），最多 4 个字符串。深度思考能力模型不支持。 |
| `reasoning_effort` | string \| null | ✗ | `medium` | 思考工作量：`minimal`（关闭思考直接答）/ `low` / `medium` / `high` / `max`（仅部分模型支持）。 |
| `response_format` | object | ✗ | `{"type":"text"}` | 指定回答格式（beta）。见 [`response_format`](#response_format)。 |
| `frequency_penalty` | float \| null | ✗ | `0` | 范围 `[-2.0, 2.0]`。频率惩罚。`doubao-seed-1.8`、`doubao-seed-2.0` 系列不支持。 |
| `presence_penalty` | float \| null | ✗ | `0` | 范围 `[-2.0, 2.0]`。存在惩罚。`doubao-seed-1.8`、`doubao-seed-2.0` 系列不支持。 |
| `temperature` | float \| null | ✗ | `1` | 范围 `[0, 2]`。`doubao-seed-2-0-pro-260215` / `doubao-seed-2-0-lite-260215` 固定为 `1`，手动值被忽略。 |
| `top_p` | float \| null | ✗ | `0.7` | 范围 `[0, 1]`。`doubao-seed-2-0-pro/lite-260215`、`doubao-seed-1-8-251228` 固定为 `0.95`，手动值被忽略。 |
| `logprobs` | boolean \| null | ✗ | `false` | 是否返回输出 token 对数概率。带深度思考能力模型不支持。 |
| `top_logprobs` | integer \| null | ✗ | `0` | 范围 `[0, 20]`。每个位置返回最可能的 token 数，仅 `logprobs=true` 时可设。带深度思考能力模型不支持。 |
| `logit_bias` | map \| null | ✗ | `null` | 调整指定 token ID 的出现概率，值范围 `[-100, 100]`（`-100` 禁止、`100` 仅可选该 token）。token ID 用分词接口获取。带深度思考能力模型不支持。 |
| `tools` | object[] \| null | ✗ | `null` | 可供模型调用的工具列表。见 [`tools[]`](#tools)。 |
| `parallel_tool_calls` | boolean | ✗ | `true` | 是否允许返回多个待调用工具。`false` 时返回 ≤1 个（`doubao-seed-1.6` 及之后系列生效）。 |
| `tool_choice` | string \| object | ✗ | 无工具时 `none`，有工具时 `auto` | 控制模型是否 / 如何调用工具。仅 `doubao-seed-1.6` 及之后系列支持。见 [`tool_choice`](#tool_choice)。 |

### `service_tier` 枚举

- `fast`：优先使用在线推理（低延迟）模式。有低延迟限流配额则优先使用获得更高服务等级；无配额或配额满则降级至常规模式。
- `auto`：优先使用在线推理（TPM 保障包）模式。有 TPM 保障包额度则优先使用；无额度 / 超额则降级至常规模式。
- `default`：只使用在线推理（常规）模式，即使有 TPM 保障包 / 低延迟额度也不使用。

### `messages[]`

按 `role` 分为四类消息：系统消息、用户消息、模型消息、工具消息。

#### 系统消息（`role` = `system`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `system`。 |
| `content` | string \| object[] | ✓ | 字符串（纯文本）或多模态内容块数组，见 [`messages[].content[]`](#messagescontent)。 |

#### 用户消息（`role` = `user`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `user`。 |
| `content` | string \| object[] | ✓ | 字符串（纯文本）或多模态内容块数组，见 [`messages[].content[]`](#messagescontent)。 |

#### 模型消息（`role` = `assistant`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `assistant`。 |
| `content` | string \| array | ✗ | 模型消息内容。`content` 与 `tool_calls` 至少填其一。 |
| `reasoning_content` | string | ✗ | 思维链内容。仅 `doubao-seed-1.8`、`deepseek-v3.2`、`doubao-seed-2.0` 支持回传。 |
| `encrypted_content` | string | ✗ | 加密压缩后的思考内容原文。自 `doubao-seed-2-0-lite-260428` 起支持回传。回传内容须有效（篡改 / 无法还原返回 `Invalid signature`）；优先级高于 `reasoning_content`，同时回传时忽略后者。 |
| `tool_calls` | object[] | ✗ | 工具调用部分，见下表。 |

##### `messages[].tool_calls[]`（assistant 回传）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `id` | string | ✓ | 待调用工具 ID，由模型生成。 |
| `type` | string | ✓ | 当前仅 `function`。 |
| `function` | object | ✓ | 包含 `name`（函数名）、`arguments`（JSON 字符串入参）。模型不保证生成有效 JSON，调用前需校验。 |

#### 工具消息（`role` = `tool`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | 固定 `tool`。 |
| `content` | string \| array | ✓ | 工具返回的消息。 |
| `tool_call_id` | string | ✓ | 关联模型发起调用时的 `tool_calls.id`，避免多工具调用信息混淆。 |

### `messages[].content[]`

`content` 为数组时，每个元素是一个模态块，由 `type` 区分。

#### 文本部分（`type` = `text`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `text`。 |
| `text` | string | ✓ | 文本内容。 |

#### 图片部分（`type` = `image_url`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `image_url`。 |
| `image_url` | object | ✓ | 见下表。 |

##### `messages[].content[].image_url`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `url` | string | ✓ | — | 图片链接或图片的 Base64 编码。 |
| `detail` | string | ✗ | — | 理解精细度：`low` / `high` / `xhigh`。默认值及对应像素区间因模型而异。 |
| `image_pixel_limit` | object \| null | ✗ | `null` | 输入图片像素范围，超出则等比缩放。优先级高于 `detail`。像素须在 `[196, 36000000]`，否则报错。 |
| `image_pixel_limit.max_pixels` | integer | ✗ | — | 最大像素，超出则缩小。`doubao-seed-1.8` 之前：`(min_pixels, 4014080]`；`doubao-seed-1.8`/`2.0`：`(min_pixels, 9031680]`。 |
| `image_pixel_limit.min_pixels` | integer | ✗ | — | 最小像素，不足则放大。`doubao-seed-1.8` 之前：`[3136, max_pixels)`；`doubao-seed-1.8`/`2.0`：`[1764, max_pixels)`。 |

#### 视频部分（`type` = `video_url`）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | string | ✓ | — | 固定 `video_url`。 |
| `video_url` | object | ✓ | — | 见下表。 |
| `video_url.url` | string | ✓ | — | 视频链接或视频的 Base64 编码。 |
| `video_url.fps` | float \| null | ✗ | `1` | 抽帧频率，范围 `[0.2, 5]`。越高对画面变化越敏感、token 花费越多。 |

#### 音频部分（`type` = `input_audio`）

音频输入 `url` / `data` 二选一传入。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 固定 `input_audio`。 |
| `input_audio` | object | ✓ | 见下表。 |
| `input_audio.url` | string | ✗ | 音频内容 URL。 |
| `input_audio.data` | string | ✗ | 音频 Base64 编码。 |
| `input_audio.format` | string | ✗ | 使用 `data` 时必填。支持 MIME：`mp3`=audio/mpeg、`wav`=audio/wav、`aac`=audio/aac、`m4a`=audio/mp4；视频内嵌音频另支持 `pcm`=audio/L16、`ac3`=audio/ac3、`alac`=audio/mp4。 |

> 音频限制：文件 ≤25 MB；单次请求音频总时长 ≤120 分钟（仅统计纯音频，视频内嵌音频不计入）。

### `thinking`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `enabled`（强制先思考再答）/ `disabled`（直接答不思考）/ `auto`（模型自主判断）。 |

### `stream_options`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `include_usage` | boolean \| null | ✗ | `false` | `true` 时在 `data: [DONE]` 前返回一个额外 chunk，其 `usage` 给出整请求 token 用量，`choices` 为空数组。 |
| `chunk_include_usage` | boolean \| null | ✗ | `false` | `true` 时每个 chunk 的 `usage` 给出到该 chunk 为止的累计 token 用量。 |

### `response_format`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | `text`（默认）/ `json_object` / `json_schema`。后两者 beta，需模型支持。 |
| `json_schema` | object | 当 `type=json_schema` 时 ✓ | JSON 结构定义，见下表。 |

##### `response_format.json_schema`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `name` | string | ✓ | — | 自定义 JSON 结构名称。 |
| `description` | string \| null | ✗ | — | 用途描述，模型据此决定如何回复。 |
| `schema` | object | ✓ | — | 以 JSON Schema 对象描述的回复格式定义。 |
| `strict` | boolean \| null | ✗ | `false` | `true` 严格遵循 schema；`false` 尽量遵循。 |

### `tools[]`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 工具类型，此处 `function`。 |
| `function` | object | ✓ | 见下表。 |

##### `tools[].function`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 函数名称。 |
| `description` | string | ✗ | 函数描述，模型据此判断是否调用。 |
| `parameters` | object | ✗ | 函数入参，JSON Schema 格式。字段名大小写敏感，建议英文字段名 + 中文置于 `description`。 |

### `tool_choice`

可为字符串（选择模式）或对象（指定工具）。

- 字符串：`none`（不调用工具）/ `required`（必须调用）/ `auto`（模型自行判断）。
- 对象（指定工具）：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 调用类型，此处 `function`。 |
| `function` | object | ✓ | 含 `name`（待调用工具名称）。 |

## 响应

### 非流式（`object` = `chat.completion`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 本次请求唯一标识。 |
| `model` | string | 实际使用的模型名称和版本。 |
| `service_tier` | string | 实际使用的模式：`scale`（TPM 保障包）/ `default`（常规）/ `fast`（低延迟）。 |
| `created` | integer | 创建时间 Unix 时间戳（秒）。 |
| `object` | string | 固定 `chat.completion`。 |
| `choices` | object[] | 模型输出内容，见 [`choices[]`](#choices)。 |
| `usage` | object | token 用量，见 [`usage`](#usage)。 |

### `choices[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `index` | integer | 在 `choices` 列表的索引。 |
| `finish_reason` | string | 停止原因：`stop`（自然结束 / 命中 stop）/ `length`（达到 max_tokens / max_completion_tokens / context_window 限制）/ `content_filter`（被内容审核拦截）/ `tool_calls`（调用了工具）。 |
| `message` | object | 模型输出内容，见下表。 |
| `logprobs` | object \| null | 对数概率信息，见下表。 |
| `moderation_hit_type` | string \| null | 命中的风险分类：`severe_violation`（严重违规）/ `violence`（激进行为）。仅视觉理解模型、且接入点 `ModerationStrategy` 设为 `Basic` 时返回。 |

##### `choices[].message`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `role` | string | 固定 `assistant`。 |
| `content` | string | 模型生成的消息内容。 |
| `reasoning_content` | string \| null | 思维链内容，仅深度推理模型返回。 |
| `tool_calls` | object[] \| null | 工具调用，元素含 `id`、`type`（`function`）、`function.name`、`function.arguments`（JSON 字符串）。 |

##### `choices[].logprobs`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `content` | object[] \| null | 每个输出 token 的对数概率信息。 |
| `content[].token` | string | 当前 token。 |
| `content[].bytes` | integer[] \| null | token 的 UTF-8 字节值列表，无则空。 |
| `content[].logprob` | float | 当前 token 对数概率。 |
| `content[].top_logprobs` | object[] | 该位置最可能的 token 列表，每项含 `token` / `bytes` / `logprob`。 |

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `total_tokens` | integer | 总 token（输入 + 输出）。 |
| `prompt_tokens` | integer | 输入 token 数。 |
| `prompt_tokens_details` | object | 输入 token 细节，见下表。 |
| `completion_tokens` | integer | 输出 token 数。 |
| `completion_tokens_details` | object | 输出 token 细节，见下表。 |

##### `usage.prompt_tokens_details`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `cached_tokens` | integer | 缓存命中的输入内容（含文本、音频等所有类型）消耗的 token 总数。 |
| `audio_tokens` | integer | 音频输入消耗的 token 数。 |
| `audio_cached_tokens` | integer | 缓存命中的音频输入消耗的 token 数。 |

##### `usage.completion_tokens_details`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `reasoning_tokens` | integer | 输出思维链消耗的 token 数（仅支持输出思维链的模型）。 |

## 流式响应（SSE）

`stream=true` 时按 SSE 逐块返回，每个 chunk 的 `object` 固定为 `chat.completion.chunk`，以 `data: [DONE]` 结束。顶层字段（`id` / `model` / `service_tier` / `created` / `object` / `choices` / `usage`）与非流式一致，差异：

- `choices[].delta` 替代 `choices[].message`，承载增量内容：
  - `delta.role`：固定 `assistant`。
  - `delta.content`：增量消息内容。
  - `delta.reasoning_content`：增量思考内容（深度推理模型）。自 `doubao-seed-2-0-lite-260428` 起返回思考摘要。
  - `delta.encrypted_content`：加密思考内容原文。自 `doubao-seed-2-0-lite-260428` 起，在思考内容输出完成后、应答输出前下发一条包含完整 `encrypted_content` 的数据（`content`、`reasoning_content` 均为空）。
  - `delta.tool_calls`：增量工具调用，结构同非流式 `message.tool_calls`。
- `usage`：流式默认为 `null`，需设 `stream_options.include_usage=true` 才统计。

> 长文本 / 深度推理等耗时场景建议调大首 token 超时（TTFT）与逐 token 超时（TPOT），避免请求超时中断。

## 示例

### 最小请求

```bash
curl https://ark.cn-beijing.volces.com/api/v3/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ARK_API_KEY" \
  -d '{
    "model": "doubao-seed-2-0-lite-260215",
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
  }'
```

### 最小响应

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "message": {
        "content": "Hello! How can I help you today?",
        "role": "assistant"
      }
    }
  ],
  "created": 1742631811,
  "id": "0217426318107460cfa43dc3f3683b1de1c09624ff49085a456ac",
  "model": "doubao-seed-2-0-lite-260215",
  "service_tier": "default",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 9,
    "prompt_tokens": 19,
    "total_tokens": 28,
    "prompt_tokens_details": { "cached_tokens": 0 },
    "completion_tokens_details": { "reasoning_tokens": 0 }
  }
}
```

### 多模态（图片理解）请求片段

```json
{
  "model": "doubao-seed-2-0-lite-260215",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "image_url", "image_url": {"url": "https://example.com/img.png"}},
        {"type": "text", "text": "这张图里是什么？"}
      ]
    }
  ]
}
```

## 参考

- 端点文档：https://www.volcengine.com/docs/82379/1494384
- 深度思考能力：https://www.volcengine.com/docs/82379/1956279
- 多模态理解：https://www.volcengine.com/docs/82379/1958521
- 上级目录（API 参考）：https://www.volcengine.com/docs/82379/1099455
