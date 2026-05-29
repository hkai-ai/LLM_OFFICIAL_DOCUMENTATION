---
source: https://www.volcengine.com/docs/82379/1569618
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# Responses API · POST /api/v3/responses

> 火山方舟的新一代有状态对话端点，对齐 OpenAI Responses API。相比 Chat API 提供更简洁的多轮上下文管理（`previous_response_id` / `store`）、更强的内置工具（豆包助手、联网搜索、图像处理、MCP、私域知识库）与上下文缓存能力。新用户推荐使用。

包含以下端点：

| 操作 | 方法 | 路径 | 鉴权 |
| --- | --- | --- | --- |
| 创建模型响应 | POST | `/api/v3/responses` | API Key / Access Key |
| 查询模型响应 | GET | `/api/v3/responses/{response_id}` | 仅 API Key |
| 获取响应上下文 | GET | `/api/v3/responses/{response_id}/input_items` | 仅 API Key |
| 删除模型响应 | DELETE | `/api/v3/responses/{response_id}` | 仅 API Key |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $ARK_API_KEY` |
| `Content-Type` | ✓ | `application/json` |

> Responses API 支持开启数据上报（统计分析以排查问题），通过额外 header 设置，详见官方「开启数据上报」。

## 创建模型响应 · 请求参数

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 模型 ID 或 Endpoint ID。 |
| `input` | string \| array | ✓ | — | 输入内容。字符串等同于一条 `user` 文本；数组为输入元素列表，见 [`input[]`](#input)。 |
| `instructions` | string \| null | ✗ | — | 作为首项插入的 system / developer 指令。与 `previous_response_id` 同用时不继承上一轮指令。**不可与缓存同用**（配置后本轮无法写入 / 使用缓存）。 |
| `previous_response_id` | string \| null | ✗ | — | 上一轮响应 ID，用于多轮对话。会引入上一轮输入与回答，增加输入 token。连续多轮间建议加 ~100ms 延迟。 |
| `expire_at` | integer | ✗ | 创建时刻+259200 | 存储 / 缓存过期 UTC 时间戳（秒）。范围 `(创建时刻, 创建时刻+604800]`（最多 7 天）。缓存存储不满 1 小时按 1 小时计费。 |
| `max_output_tokens` | integer \| null | ✗ | — | 输出最大 token（含回答 + 思维链）。 |
| `thinking` | object | ✗ | 取决于模型 | 深度思考开关，`thinking.type` 取 `enabled` / `disabled` / `auto`。 |
| `reasoning` | object | ✗ | `{"effort":"medium"}` | `reasoning.effort` 取 `minimal` / `low` / `medium` / `high` / `max`（仅部分模型）。 |
| `include` | array | ✗ | — | 额外返回字段。当前支持 `reasoning.encrypted_content`（加密思考内容原文，可手动回传复用）。 |
| `caching` | object | ✗ | `{"type":"disabled"}` | 缓存开关。`caching.type`（`enabled`/`disabled`）、`caching.prefix`（`true` 仅创建公共前缀缓存且不回复）。不可与 `instructions`、非 Function 的 `tools` 同用。 |
| `store` | boolean \| null | ✗ | `true` | 是否存储响应供后续检索。 |
| `stream` | boolean \| null | ✗ | `false` | 是否 SSE 流式返回，以 `data: [DONE]` 结束。 |
| `temperature` | float \| null | ✗ | `1` | 范围 `[0, 2]`。`doubao-seed-2-0-pro/lite-260215` 固定 `1`。 |
| `top_p` | float \| null | ✗ | `0.7` | 范围 `[0, 1]`。`doubao-seed-2-0-pro/lite-260215`、`doubao-seed-1-8-251228` 固定 `0.95`。 |
| `text` | object | ✗ | `{"format":{"type":"text"}}` | 输出格式定义，见 [`text.format`](#textformat)。 |
| `tools` | array | ✗ | — | 可调用工具（内置工具 / MCP / 函数），见 [`tools[]`](#tools)。 |
| `tool_choice` | string \| object | ✗ | 无工具 `none`，有工具 `auto` | 工具选择策略。仅 Doubao Seed 1.8 / 2.0 系列支持。见 [`tool_choice`](#tool_choice)。 |
| `max_tool_calls` | integer | ✗ | 因工具而异 | 最大工具调用轮次，范围 `[1, 10]`。best effort。Web Search 默认 3、Image Process 默认 10（不可改）、Knowledge Search 默认 3；豆包助手不支持。 |
| `context_management` | object | ✗ | — | 上下文管理策略（思考块清除 / 工具调用内容清除），见 [`context_management`](#context_management)。 |

### `input[]`

数组元素可为多种类型，由 `type` 区分。常见为「输入的消息」。

#### 输入的消息（`type` = `message`）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `role` | string | ✓ | `user` / `system` / `assistant` / `developer`。`developer`/`system` 指令优先于 `user`。 |
| `content` | string \| array | ✓ | 文本字符串，或内容项列表，见 [`input[].content[]`](#inputcontent)。 |
| `type` | string | ✗ | 固定 `message`。 |
| `partial` | boolean | ✗ | 续写模式。最后一条 `role=assistant` 且 `partial=true` 时模型基于 `content` 续写（续写模式下必填）。 |

其他 `input` 元素类型（用于手动管理历史上下文）：

| `type` | 说明 | 关键字段 |
| --- | --- | --- |
| `message`（上下文元素） | 历史发给模型的信息 | `content`、`role`（`system`/`user`/`developer`）、`status`（`in_progress`/`completed`/`incomplete`） |
| `function_call` | 工具函数调用 | `arguments`、`call_id`、`name`、`type=function_call`、`status` |
| `function_call_output` | 工具返回结果 | `call_id`、`output`、`type=function_call_output`、`status` |
| `reasoning` | 思维链信息（仅 doubao-seed-1.8 / deepseek-v3.2 / doubao-seed-2.0；推荐改用 `previous_response_id` 自动管理） | `id`、`type=reasoning`、`summary[]`（`text`、`type=summary_text`）、`status` |

### `input[].content[]`

内容项由 `type` 区分。

| `type` | 说明 | 关键字段 |
| --- | --- | --- |
| `input_text` | 文本 | `text`；翻译模型另支持 `translation_options`（`source_language` / `target_language`，仅 `doubao-seed-translation-250728`） |
| `input_image` | 图片（`file_id` / `image_url` 二选一） | `file_id`、`image_url`、`detail`（`low`/`high`/`xhigh`）、`image_pixel_limit`（`max_pixels`/`min_pixels`，像素须在 `[196, 36000000]`） |
| `input_video` | 视频（`file_id` / `video_url` 二选一） | `file_id`、`video_url`、`fps`（`[0.2, 5]`，用 `file_id` 时失效） |
| `input_file` | 文件，当前仅 PDF（`file_id` / `file_data` / `file_url` 三选一） | `file_id`、`file_data`（Base64，≤50 MB）、`filename`（用 `file_data` 时必填）、`file_url`（≤50 MB） |
| `input_audio` | 音频（`file_id` / `audio_url` 二选一） | `file_id`、`audio_url` |

### `text.format`

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | string | ✓ | `text` | `text` / `json_object` / `json_schema`（后两者 beta）。 |
| `name` | string | `json_schema` 时 ✓ | — | 自定义 JSON 结构名称。 |
| `schema` | object | `json_schema` 时 ✓ | — | JSON Schema 对象。 |
| `description` | string \| null | ✗ | — | 用途描述。 |
| `strict` | boolean \| null | ✗ | `false` | 是否严格遵循 schema。 |

### `tools[]`

由 `type` 区分工具类型。

#### 函数调用（`type` = `function`）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `name` | string | ✓ | — | 函数名。 |
| `parameters` | object | ✓ | — | JSON Schema 描述的入参。 |
| `description` | string | ✗ | — | 函数描述。 |
| `strict` | boolean | ✓ | `true` | 是否强制严格参数验证。 |

#### 联网搜索（`type` = `web_search`，需开通「联网内容插件」）

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `sources` | string[] | — | 附加内容源：`toutiao`（头条图文）/ `douyin`（抖音百科）/ `moji`（墨迹天气）。 |
| `limit` | integer | `10` | 单轮搜索最大召回条数，`[1, 50]`（单次搜索最多返回 20 条）。 |
| `user_location` | object | `{"type":"approximate"}` | 用户地理位置（`type`/`country`/`region`/`city`），填 `type` 后后三者至少一个有效。 |
| `max_keyword` | integer | — | 一轮最大并行搜索关键词数，`[1, 50]`。 |

#### 图像处理（`type` = `image_process`）

子功能开关均为对象 `{"type":"enabled"/"disabled"}`：`point`（画点/连线，默认 enabled）、`grounding`（框选/裁剪，默认 enabled）、`zoom`（缩放 0.5–2.0 倍，默认 enabled）、`rotate`（旋转 0–359°，默认 enabled）。

#### 豆包助手（`type` = `doubao_app`，需开通「豆包助手」）

子功能 `feature` 下：`chat`（日常沟通）、`deep_chat`（深度沟通）、`ai_search`（联网搜索）、`reasoning_search`（边想边搜），各含 `type`（默认 `disabled`）与 `role_description`（角色设定，与 system prompt / instructions 互斥）。另含 `user_location`。

#### MCP 工具（`type` = `mcp`）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `server_label` | string | ✓ | — | MCP Server 标签。 |
| `server_url` | string | ✓ | — | MCP Server 访问地址。 |
| `headers` | object | ✗ | — | 发往 MCP Server 的 HTTP 头（如 `Authorization`，不存储）。 |
| `require_approval` | object \| string | ✗ | `always` | 工具授权策略：`always` / `never`，或对象 `{always:{tool_names:[]}, never:{tool_names:[]}}`。 |
| `allowed_tools` | array \| object | ✗ | 全部 | 工具加载范围：字符串数组或 `{tool_names:[]}`。 |

#### 私域知识库搜索（`type` = `knowledge_search`）

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `knowledge_resource_id` | string | ✓ | — | 私域知识库 ID。 |
| `limit` | integer | ✗ | `10` | 最大采用结果数，`[1, 200]`。 |
| `max_keyword` | integer | ✗ | — | 一轮最大并行关键词数，`[1, 50]`。 |
| `doc_filters` | object | ✗ | — | 文档字段级检索过滤（`op`：`must`/`must_not`/`and`/`or` 等 + `field` + `conds`）。 |
| `description` | string | ✗ | — | 知识库描述。 |
| `dense_weight` | float | ✗ | `0.5` | 稠密向量权重，`[0.2, 1]`（仅 `hnsw_hybrid` 混合检索有效）。 |
| `ranking_options` | object | ✗ | — | 检索后处理：`rerank_switch`(默认 false)、`retrieve_count`(默认 25，≥limit)、`get_attachment_link`、`chunk_diffusion_count`(`[0,5]`)、`chunk_group`、`rerank_model`(`base-multilingual-rerank`/`m3-v2-rerank`)、`rerank_only_chunk`。 |

### `tool_choice`

- 字符串：`none` / `required` / `auto`。
- 对象：`type`（`function` 时 `name` 必填；内置工具填工具名称）、`name`（待调用工具名称）。

### `context_management`

`context_management.edits[]` 数组，策略由 `type` 区分：

- `clear_thinking`（思考块清除）：`keep` 为 `{type:"thinking_turns", value:N}`（保留最近 N 轮，默认 1）或字符串 `"all"`（保留全部）。
- `clear_tool_uses`（工具调用内容清除）：`keep`（`{type:"tool_uses", value:N}`，默认 3）、`exclude_tools[]`（不清除的工具名）、`clear_tool_input`（默认 false）、`trigger`（`{type:"tool_uses", value:N}`，达 N 轮触发）。

## response object（响应）

非流式返回一个 response 对象。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 请求唯一标识。 |
| `created_at` | integer | 创建 Unix 时间戳（秒）。 |
| `object` | string | 固定 `response`。 |
| `model` | string | 实际使用的模型名称和版本。 |
| `status` | string | `completed` / `failed` / `in_progress` / `incomplete`。 |
| `error` | object \| null | 失败时返回，含 `code`、`message`。 |
| `incomplete_details` | object \| null | 未完成细节，含 `reason`。 |
| `instructions` | string \| null | 本轮插入的系统 / 开发者指令。 |
| `max_output_tokens` | integer \| null | 输出最大 token（含回答 + 思维链）。 |
| `output` | array | 输出消息列表（回答 / 思维链 / 工具调用），见 [`output[]`](#output)。 |
| `previous_response_id` | string \| null | 传入的历史响应 ID。 |
| `thinking` | object \| null | 深度思考开关回显（`thinking.type`）。 |
| `service_tier` | string | 实际模式（`default` 表示未使用 TPM 保障包额度）。 |
| `text` | object | 输出格式定义回显（`text.format`）。 |
| `tools` | array | 可调用工具回显。 |
| `tool_choice` | string \| object \| null | 工具选择回显。 |
| `temperature` | float \| null | 采样温度回显。 |
| `top_p` | float \| null | 核采样回显。 |
| `usage` | object | token 用量，见 [`usage`](#usage)。 |
| `store` | boolean | 是否存储（默认 true）。 |
| `caching` | object | 缓存开关回显（`caching.type`）。 |
| `expire_at` | integer \| null | 存储有效期。 |
| `context_management` | object | 已应用的上下文管理策略（`applied_edits[]`，含 `cleared_thinking_turns` / `cleared_tool_uses`）。 |

> 通过「查询模型响应」获取的 response 对象**不包含思维链内容**。

### `output[]`

由 `type` 区分：

#### 模型回答（`type` = `message`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 此回答唯一标识。 |
| `role` | string | 固定 `assistant`。 |
| `content` | array | 内容项，`type=output_text` 时含 `text`。 |
| `status` | string | 输出消息状态。 |
| `partial` | boolean | 续写模式时返回，固定 `true`。 |

#### 模型思维链（`type` = `reasoning`）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 思维链消息唯一标识。 |
| `summary` | array | 思考内容摘要（自 doubao-seed-2-0-lite-260428 起为摘要），元素含 `text`、`type=summary_text`。 |
| `content` | array | 思考内容原文，元素含 `text`、`type=reasoning_text`。 |
| `encrypted_content` | string | 加密压缩后的思考原文，仅 `include` 指定 `reasoning.encrypted_content` 时返回（自 doubao-seed-2-0-lite-260428 起支持）。 |
| `status` | string | 思维链返回状态。 |

#### 工具调用（`type` = `function_call`）

含 `id`、`call_id`、`name`、`arguments`（JSON 字符串）、`status`。内置工具另有 `mcp_call` / `mcp_list_tools` / `web_search_call` / `image_process` 等输出类型，含各自的 `server_label`、`action`、`output`、`error` 等字段。

### `usage`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `input_tokens` | integer | 输入 token 数。 |
| `input_tokens_details` | object | `cached_tokens`（缓存命中总数）、`audio_tokens`、`audio_cached_tokens`。 |
| `output_tokens` | integer | 输出 token 数。 |
| `output_tokens_details` | object | `reasoning_tokens`（思考 token 数）。 |
| `total_tokens` | integer | 总 token 数。 |
| `tool_usage` | object | 工具调用次数：`image_process` / `mcp` / `web_search`。 |
| `tool_usage_details` | object | 工具调用明细，如 `web_search:{toutiao:1, moji:1, search_engine:1}`。 |

## 查询模型响应 · GET /api/v3/responses/{response_id}

仅 API Key 鉴权。路径参数 `response_id`（必填）。响应已完成时返回对应 response object；未完成则返回错误码。

## 获取响应上下文 · GET /api/v3/responses/{response_id}/input_items

仅 API Key 鉴权。返回某次响应的全部上下文元素（分页）。

| 参数 | 位置 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `response_id` | path | string | — | 必填，响应 ID。 |
| `after` | query | string \| null | — | 返回该 ID 之后的输入项。 |
| `before` | query | string \| null | — | 返回该 ID 之前的输入项。 |
| `include[]` | query | array \| null | — | 额外返回字段，如 `message.input_image.image_url`。 |
| `limit` | query | integer | `20` | 单次最大项目数，`1~100`。 |
| `order` | query | string | `desc` | `asc` / `desc`。 |

响应：`object`=`list`、`data[]`（结构同创建时的 `input`，引用 `previous_response_id` 时含其上下文）、`first_id`、`last_id`、`has_more`。

## 删除模型响应 · DELETE /api/v3/responses/{response_id}

仅 API Key 鉴权。路径参数 `response_id`（必填）。响应：`{ "id", "object": "response", "deleted": true/false }`。

## 流式响应（SSE）

`stream=true` 时服务器通过 SSE 实时推送生成过程事件，以 `data: [DONE]` 结束。事件类型与各事件结构详见 [streaming（流式响应）](https://www.volcengine.com/docs/82379/1599499)。

## 示例

### 最小请求

```bash
curl --location "https://ark.cn-beijing.volces.com/api/v3/responses" \
  --header "Authorization: Bearer $ARK_API_KEY" \
  --header "Content-Type: application/json" \
  --data '{
    "model": "doubao-seed-2-0-lite-260215",
    "input": "你好呀。"
  }'
```

### 最小响应（节选）

```json
{
  "id": "resp_0217****",
  "created_at": 1756280722.0,
  "model": "doubao-seed-2-0-lite-260215",
  "object": "response",
  "output": [
    { "id": "rs_...", "type": "reasoning", "summary": [{ "text": "...", "type": "summary_text" }], "status": "completed" },
    { "id": "msg_...", "type": "message", "role": "assistant", "status": "completed",
      "content": [{ "text": "你好呀！很高兴见到你～", "type": "output_text", "annotations": null }] }
  ],
  "max_output_tokens": 32768,
  "service_tier": "default",
  "status": "completed",
  "usage": {
    "input_tokens": 88,
    "input_tokens_details": { "cached_tokens": 0 },
    "output_tokens": 230,
    "output_tokens_details": { "reasoning_tokens": 211 },
    "total_tokens": 318
  },
  "caching": { "type": "disabled" },
  "store": true,
  "expire_at": 1756539922
}
```

### 多模态输入

```json
{
  "model": "doubao-seed-1-6-250615",
  "input": [
    { "role": "user", "content": [
      { "type": "input_text", "text": "你看见了什么？" },
      { "type": "input_image", "image_url": "https://example.com/img.png" }
    ]}
  ],
  "thinking": { "type": "enabled" }
}
```

## 参考

- 创建模型响应：https://www.volcengine.com/docs/82379/1569618
- response object：https://www.volcengine.com/docs/82379/1783703
- 查询模型响应：https://www.volcengine.com/docs/82379/1783709
- 获取响应上下文：https://www.volcengine.com/docs/82379/1783719
- 删除模型响应：https://www.volcengine.com/docs/82379/1584286
- 流式响应：https://www.volcengine.com/docs/82379/1599499
