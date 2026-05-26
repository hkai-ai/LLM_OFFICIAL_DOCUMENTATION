---
name: alibaba-bailian-api-docs
description: 当需要查阅、更新或扩充阿里云大模型服务平台百炼（Model Studio）官方 API 文档（OpenAI 兼容 chat/completions、Responses、DashScope 原生 generation、Anthropic 兼容 messages、enable_thinking / thinking_budget、enable_search / search_options、cache_control、qwen3.6 系列、qwen-omni、QVQ / QwQ、错误码、地域 base URL 等）在本仓库 `alibaba-bailian/` 目录下的中文整理版本时使用。触发场景包括："百炼 API 怎么用"、"qwen-plus 调用"、"通义千问 OpenAI 兼容"、"DashScope generation 接口"、"百炼联网搜索"、"百炼思考模式 enable_thinking"、"百炼显式缓存 cache_control"、"百炼错误码 ModelNotFound"、"刷新百炼文档"、"qwen3.6-max"。
---

# 阿里百炼 Model Studio API 文档撰写与维护

## 1. 文档站全貌

- 域名：`https://help.aliyun.com/zh/model-studio/`（中文站）；英文站 `https://www.alibabacloud.com/help/en/model-studio/`（结构同步、URL slug 一致）。
- 文档由阿里云帮助中心承载，左侧导航树非常深；同一主题可能在「用户指南（模型）」、「API 参考（模型）」、「错误码」三个分支重复出现，权威以 **API 参考（模型）** 子树为准。
- 顶级目录（左侧导航）：
  - **用户指南（模型）**：`/zh/model-studio/model-user-guide/`
  - **用户指南（应用）**：`/zh/model-studio/application-user-guide/`
  - **API 参考（模型）**：`/zh/model-studio/model-api-reference/`
    - 「对话」「图像生成」「视频生成」「3D」「专项模型」「实时多模态」「语音合成」「音乐生成」「语音识别」「语音翻译」「文本向量」「多模态向量」「更多模型」「工具包 / 框架」「模型部署」「模型调优」等一级章节
  - **API 参考（应用）**：`/zh/model-studio/applicantion-api-reference/`（注意 typo `applicantion` 是官方原拼）
  - **错误码**：`/zh/model-studio/error-code`
- 百炼提供 **4 套并列协议**，文档对每一套都单独建页：
  - OpenAI 兼容 Chat Completions：`/qwen-api-via-openai-chat-completions`
  - OpenAI 兼容 Responses：`/compatibility-with-openai-responses-api`、`/qwen-api-via-openai-responses`（同一主题不同 slug，内容互补）
  - Anthropic 兼容 Messages：（位于 API 参考 - 对话子树）
  - DashScope 原生 generation：`/qwen-api-via-dashscope`
- 公开访问，无需登录；个别页面包含 SDK 代码块（Python / Java / Go / Node），但参数说明全部 SSR。

## 2. 抓取要点

- **入口选择**：直接抓 `https://help.aliyun.com/zh/model-studio/<slug>` 即可，无需走索引页。
- **WebFetch 限制**：阿里云帮助中心页面较长且导航嵌套深；**第一次抓取容易只拿到导航/概述**，宜针对每个 endpoint 单独抓 + 在 prompt 中明确列出要提取的字段（例如「请列出 `enable_thinking` / `thinking_budget` / `enable_search` 的字段类型、默认值、约束」）。
- **404 概率高**：不同时期 slug 改过名（如 `use-bailian-in-openai-sdk` 实际不存在），抓 404 时改用 WebSearch 查 `site:help.aliyun.com <关键词>` 找当前 slug。
- **模型清单不集中**：`/models` 页只显示模型分类卡片，单个模型的上下文 / 单价需要进入「模型广场」（控制台，非帮助文档）或单个模型详情页。**写文档时不要臆测数字，未在公开页明示的字段一律写「文档未说明」**。
- **DashScope vs OpenAI 兼容差异**：流式触发方式、字段命名（input_tokens vs prompt_tokens）、`seed` 范围、错误结构都不同，文档须分两边写。
- **不要抓** Github / CSDN / 知乎，避免引入 SDK 包装语义混进 API 字段。

## 3. 更新流程

1. **抓 `/qwen-api-reference/`**：核对左侧导航是否新增模态 / 协议子树。
2. **抓 4 套协议主页**：`qwen-api-via-openai-chat-completions` / `compatibility-with-openai-responses-api` / `qwen-api-via-openai-responses` / `qwen-api-via-dashscope`，diff 请求 / 响应字段表，同步 `alibaba-bailian/chat-completions.md`、`alibaba-bailian/responses.md`、`alibaba-bailian/dashscope-generation.md`。
3. **抓 `/error-code`**：核对错误码清单与排查段落，同步 `alibaba-bailian/errors.md`。
4. **抓 `/get-api-key` 与 `/first-api-call-to-qwen`**：核对各地域 base URL 是否新增 / 调整，更新 README 表格。
5. **抓 `/models` 与单个模型详情页**：仅同步官方明示数据（上下文窗口 / 价格 / 思考支持），未明示者保留「文档未说明」。
6. **更新 `fetched_at`**：所有改动文件统一改为新日期（北京时间）。

## 4. 坑点清单

- **地域 API Key 隔离**：北京 / 新加坡 / 美西 / 法兰克福各自独立 Key，base URL 必须与 Key 同地域，否则 401 `InvalidApiKey`。法兰克福 base URL 含 `{WorkspaceId}.eu-central-1.maas.aliyuncs.com` 子域，注意替换占位符。
- **OpenAI 兼容前缀 `/compatible-mode/v1`，DashScope 前缀 `/api/v1`**：两套路径都基于 `dashscope.aliyuncs.com` 同一域名，区别只在前缀；新人极易把二者搞混。
- **流式触发方式不同**：OpenAI 兼容用 `stream: true`；DashScope 用 header `X-DashScope-SSE: enable` + `parameters.incremental_output`。
- **`incremental_output`**：DashScope 流式默认 `false`，每个 chunk 携带**累积**输出（非增量），这与 OpenAI 习惯相反；不显式开 true 会导致客户端拼接重复。
- **token 字段命名不一致**：DashScope `usage.input_tokens` / `output_tokens`；OpenAI 兼容 `usage.prompt_tokens` / `usage.completion_tokens`。统计接入时务必区分。
- **思考模式与 `response_format: json_object` 互斥**：同用会报错 `InvalidParameter`；要结构化 JSON 与 reasoning 兼得，需走 prompt 工程或拆两次请求。
- **`enable_thinking` 在不同模型默认不同**：Qwen3 商业版 / 开源版 / QwQ / QVQ / 思考模式版本默认开；普通 Qwen3.6 系列默认关。
- **仅流式模型**：Qwen3 商业版（思考模式）、Qwen3 开源版、QwQ、QVQ 等不支持非流式调用，非流式直接报错。
- **`seed` 取值范围**：DashScope `[0, 2⁶³-1]`，OpenAI 兼容 `[0, 2³¹-1]`，跨协议迁移要截断。
- **多模态 content key 不同**：OpenAI 兼容用 `image_url` / `input_audio` / `video_url`；DashScope 用裸 `{"image": "..."}` / `{"video": [...]}` / `{"audio": "..."}`。
- **Responses API 的 `previous_response_id`**：有效期 7 天，过期后必须重新提供完整 messages。
- **会话缓存 vs 显式缓存**：Responses 用 header `x-dashscope-session-cache: enable` 实现会话缓存；Chat Completions 用 `cache_control` 字段实现显式 prompt 缓存，二者不同机制不同字段。
- **`cache_control` 命中字段**：成功命中后 `usage.prompt_tokens_details.cached_tokens > 0`；首次写入显示 `usage.cache_creation.ephemeral_5m_input_tokens`。
- **Anthropic 兼容模式**：百炼也提供 `/anthropic` 前缀的 Messages 兼容（与 DeepSeek 类似），但本仓库未单独整理；如用户问到需在 SKILL 中提醒「百炼也支持 Anthropic SDK 调用」。
- **聚合的第三方模型**：通过百炼调用 `kimi-k2.6` / `deepseek-v4-pro` 等仍走百炼协议字段，不要套用原厂特有字段（例如 DeepSeek 的 `prefix: true` 在百炼侧未必兼容）。
- **官方页 typo**：「应用」拼写为 `applicantion-api-reference` 是阿里云原拼，不是抓取错误。

## 5. 关键链接表

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| 入口 / API 参考 | https://help.aliyun.com/zh/model-studio/model-api-reference/ | `alibaba-bailian/README.md` |
| 首次调用 | https://help.aliyun.com/zh/model-studio/first-api-call-to-qwen | `alibaba-bailian/README.md` |
| 获取 API Key | https://help.aliyun.com/zh/model-studio/get-api-key | `alibaba-bailian/README.md` |
| OpenAI 兼容总览 | https://help.aliyun.com/zh/model-studio/compatibility-of-openai-with-dashscope | `alibaba-bailian/README.md` |
| OpenAI 兼容 Chat | https://help.aliyun.com/zh/model-studio/qwen-api-via-openai-chat-completions | `alibaba-bailian/chat-completions.md` |
| OpenAI 兼容 Responses | https://help.aliyun.com/zh/model-studio/compatibility-with-openai-responses-api | `alibaba-bailian/responses.md` |
| OpenAI 兼容 Responses（另一 slug） | https://help.aliyun.com/zh/model-studio/qwen-api-via-openai-responses | `alibaba-bailian/responses.md` |
| OpenAI 兼容 Batch Chat | https://help.aliyun.com/zh/model-studio/openai-compatible-batch-chat | （未单独整理） |
| OpenAI 兼容 Embedding | https://help.aliyun.com/zh/model-studio/embedding-interfaces-compatible-with-openai | （未单独整理） |
| OpenAI 兼容 VL | https://help.aliyun.com/zh/model-studio/qwen-vl-compatible-with-openai | （未单独整理） |
| DashScope 原生 | https://help.aliyun.com/zh/model-studio/qwen-api-via-dashscope | `alibaba-bailian/dashscope-generation.md` |
| 文本生成总览 | https://help.aliyun.com/zh/model-studio/text-generation | `alibaba-bailian/README.md` |
| 多轮对话 | https://help.aliyun.com/zh/model-studio/multi-round-conversation | `alibaba-bailian/chat-completions.md` |
| 结构化输出 | https://help.aliyun.com/zh/model-studio/qwen-structured-output | `alibaba-bailian/chat-completions.md` §response_format |
| 错误码 | https://help.aliyun.com/zh/model-studio/error-code | `alibaba-bailian/errors.md` |
| 模型清单 | https://help.aliyun.com/zh/model-studio/models | `alibaba-bailian/models.md` |
| 模型广场（控制台） | https://bailian.console.aliyun.com/?tab=model#/model-market | `alibaba-bailian/models.md` |
| API Key 控制台 | https://bailian.console.aliyun.com/?tab=model#/api-key | `alibaba-bailian/README.md` |
