---
name: volcengine-api-docs
description: 需要查阅、更新或扩充本仓库 `volcengine/` 目录下火山方舟（Volcengine Ark，豆包 Doubao 系列）API 中文文档时使用。覆盖数据面 API：对话(Chat) `/api/v3/chat/completions`、Responses `/api/v3/responses`（创建 / 查询 / 上下文 / 删除 + 内置工具 web_search / image_process / knowledge_search / mcp / doubao_app）、多模态向量化 `/api/v3/embeddings/multimodal`、图片生成 Seedream `/api/v3/images/generations`、视频生成 Seedance `/api/v3/contents/generations/tasks`、上下文缓存 `/api/v3/context/*`、批量 `/api/v3/batch/chat/completions`、文件 `/api/v3/files`、分词 `/api/v3/tokenization`、应用 bot `/api/v3/bots/chat/completions`、兼容 OpenAI SDK、错误码、模型清单、定价。触发场景示例："火山方舟怎么用"、"豆包 API"、"doubao-seed-2.0 思考模式"、"Seedance 视频生成参数"、"Seedream 图片生成"、"火山方舟 thinking 字段"、"豆包 Responses API"、"火山方舟上下文缓存"、"火山方舟价格"、"doubao 定价"、"刷新火山方舟文档"、"火山方舟错误码 429"、"Ark service_tier"。
---

# 火山方舟（Volcengine Ark）API 文档撰写与维护

## 1. 文档站全貌

- 产品：火山方舟（Volcengine Ark），火山引擎大模型平台，主力模型为豆包 Doubao 系列（含视频 Seedance、图片 Seedream），并托管 GLM / DeepSeek 等第三方模型。
- 文档库 ID 固定为 `82379`，单篇 URL 形如 `https://www.volcengine.com/docs/82379/<docId>?lang=zh`。
- 数据面 Base URL：`https://ark.cn-beijing.volces.com/api/v3`；管控面：`https://ark.cn-beijing.volcengineapi.com/`。
- 文档分两大类导航：「文档指南」（教程类）与「API 参考」（端点字段，撰写本仓库主要看这里）。
- 公开访问，无需登录、无 paywall。

## 2. 抓取要点（关键，务必先读）

火山引擎文档站是 **Modern.js 服务端壳 + 客户端（CSR）渲染**，正文用火山自研富文本编辑器 `volc-doceditor` 呈现。直接抓 HTML 拿不到正文，必须用真实浏览器渲染。

- **WebFetch 不可用**：`www.volcengine.com` 会被 claude.ai 拦截（"Unable to verify if domain is safe to fetch"）。`curl` 能拿到 HTML 但**正文为空**（CSR）。
- **必须用 chrome-devtools MCP**（真实浏览器）：`new_page` / `navigate_page` 打开 `https://www.volcengine.com/docs/82379/<docId>?lang=zh`。
- **正文容器**：参数 / 字段渲染在 `.volc-doceditor-viewer`（或 `[class*="volc-doceditor"]`）里，表格被渲染成逐行 `div.ace-line`，**不是 `<table>`**，所以语义化表格提取拿不到。最可靠的方式是取**最长的** `[class*="volc-doceditor"]` 元素的 `innerText`（保留换行，参数名 / 类型 / 说明会逐行出现，可重建表格）。
- **嵌套字段默认折叠**：`messages` 的消息类型、`tools` 的属性、`choices`、`usage` 等子结构默认折叠成可点击的"属性"/"消息类型"/"可选类型"链接，innerText 抓不到内容。页面右上角有"一键展开折叠"开关 `button[role="switch"].arco-switch`，**必须先点击它**（`aria-checked` 从 `false` → `true`），等 ~600ms 重渲染后再取 innerText，才能拿到全部嵌套字段。
- **通用提取脚本**（evaluate_script，建议存到临时文件再 Read）：

  ```js
  async () => {
    const sleep = ms => new Promise(r => setTimeout(r, ms));
    const getViewer = () => [...document.querySelectorAll('[class*="volc-doceditor"]')]
      .sort((a,b)=>b.innerText.length-a.innerText.length)[0];
    let v;
    for (let i=0;i<40;i++){ v=getViewer(); if(v && v.innerText.length>300) break; await sleep(250); }
    const sw = document.querySelector('button[role="switch"].arco-switch');
    if (sw && sw.getAttribute('aria-checked')==='false'){ sw.click(); await sleep(700); }
    v = getViewer();
    return v ? v.innerText : '(none)';
  }
  ```

- **端点 URL 与方法**在正文首行（如 `POST https://ark.cn-beijing.volces.com/api/v3/...`），列表 / 删除等简单端点只取前几百字符即可确认路径。
- **不要抓** GitHub / 知乎 / 第三方教程，会引入未经核实字段。

## 3. 更新流程

1. 打开「快速入门」`/docs/82379/1399008` 与「模型列表」`/docs/82379/1330310`，确认在售模型 ID、上下文窗口、能力维度是否变动，同步 `volcengine/README.md` 与 `volcengine/models.md`。
2. 打开「模型价格」`/docs/82379/1544106`，比对 `volcengine/pricing.md`（人民币，分段计费）。
3. 逐端点重抓「API 参考」分类下页面（见下方链接表），diff 参数 / 响应字段表。
4. 同步示例（请求 / 响应 JSON）。
5. 所有改动文件统一更新 frontmatter 的 `fetched_at`，并把 `volcengine/README.md` 的 `last_updated` 改为同一日期（北京时间）。

## 3.1 定价维护（`volcengine/pricing.md`）

- 权威源：`https://www.volcengine.com/docs/82379/1544106`。
- 货币人民币（元），文本类单位「元/百万 token」，缓存存储「元/百万 token/小时」，图片「元/张」，视频「元/百万 token」。
- 大语言模型多为**分段计费**：按单次请求输入长度区间（`[0,32]` / `(32,128]` / `(128,256]`，单位 k token），部分模型再按输出长度区间。撰写时务必保留区间维度。
- 四档分别成表：在线推理（常规）/ 在线推理（低延迟）/ TPM 保障包 / 批量推理。
- 视频按 token 估算（公式见 pricing.md），Seedance 2.0 含视频输入有最低 token 用量限制。

## 4. 坑点清单

- **CSR + 自研富文本编辑器**：见第 2 节，这是抓取本站的核心难点。不展开折叠开关就会丢掉所有嵌套字段（多模态 content、tools 定义、usage 明细等）。
- **Model ID vs Endpoint ID**：`model` 字段两者都可填；但 **Access Key 鉴权、上下文缓存、批量、bot** 等场景要求用 Endpoint ID（`ep-xxx` / Bot ID）。鉴权方式：数据面支持 API Key（`Bearer`）与 Access Key（HMAC-SHA256，Service=ark / Region=cn-beijing）。
- **Chat API vs Responses API 字段命名不同**：
  - Chat 的 content type 是 `text` / `image_url` / `video_url` / `input_audio`；Responses 是 `input_text` / `input_image` / `input_video` / `input_file` / `input_audio`（OpenAI Responses 风格）。
  - Chat 用 `reasoning_effort`（顶层）；Responses 用 `reasoning.effort`（对象）。
  - Chat 响应 `usage`；bot 响应是 `bot_usage.model_usage[]`；Responses 是 `usage.input_tokens/output_tokens`（非 prompt/completion）。
- **service_tier**：请求侧 `fast`（低延迟）/ `auto`（TPM 保障包优先）/ `default`（常规）；响应侧返回 `scale`（TPM 保障包）/ `default` / `fast`。注意请求枚举与响应枚举不完全相同。
- **深度思考字段**：`thinking.type`（enabled/disabled/auto）+ `reasoning_effort`（minimal~max，`minimal`=关闭思考）。输出有 `reasoning_content`（Chat）/ `output.summary`（Responses，自 doubao-seed-2-0-lite-260428 起为摘要）；加密思考原文 `encrypted_content` 需在 Responses `include` 指定 `reasoning.encrypted_content`。
- **固定参数模型**：`doubao-seed-2-0-pro/lite-260215` 的 `temperature` 固定 1、`top_p` 固定 0.95，手动值被忽略；`doubao-seed-1.8`/`2.0` 系列不支持 `frequency_penalty`/`presence_penalty`。
- **上下文缓存对话**（`/api/v3/context/chat/completions`）限制多：仅 Endpoint ID、messages 末条不能是 assistant、不支持 tools / thinking、service_tier 不支持 auto。
- **向量化只有多模态端点**：`/api/v3/embeddings/multimodal`，**不支持 OpenAI SDK**，需用方舟 SDK；文本向量化模型已逐步下线。`data` 是单个对象而非数组（即使 `object=list`）。
- **视频生成是异步**：`POST /api/v3/contents/generations/tasks` 返回 `id`，需轮询 `GET .../tasks/{id}`。视频 / 图片 URL 仅 24 小时有效；视频任务记录仅保留 7 天。
- **视频参数双传法**：`resolution`/`ratio`/`duration` 等可在 request body 传（强校验），也可在提示词后加 `--rs 720p --dur 5`（弱校验）。
- **OpenAI 兼容靠 extra_body / extra_headers**：方舟特有字段（如 `thinking`）通过 OpenAI SDK 的 `extra_body` 透传；自定义头用 `extra_headers`。
- **docId 与端点强绑定**：文档无统一 OpenAPI 导出，靠 docId → 端点对照（见下表）。新端点需先在「API 参考」导航里找 docId。

## 5. 关键链接表（docId → 仓库文件）

| 主题 | 官方 URL | 对应仓库文件 |
| --- | --- | --- |
| 产品文档首页 | https://www.volcengine.com/docs/82379/1099455 | `volcengine/README.md` |
| 快速入门 | https://www.volcengine.com/docs/82379/1399008 | `volcengine/README.md` |
| Base URL 及鉴权 | https://www.volcengine.com/docs/82379/1298459 | `volcengine/README.md` |
| 模型列表 | https://www.volcengine.com/docs/82379/1330310 | `volcengine/models.md` |
| 模型价格 | https://www.volcengine.com/docs/82379/1544106 | `volcengine/pricing.md` |
| 对话(Chat) API | https://www.volcengine.com/docs/82379/1494384 | `volcengine/chat-completions.md` |
| 创建模型响应（Responses） | https://www.volcengine.com/docs/82379/1569618 | `volcengine/responses.md` |
| response object | https://www.volcengine.com/docs/82379/1783703 | `volcengine/responses.md` |
| 查询模型响应 | https://www.volcengine.com/docs/82379/1783709 | `volcengine/responses.md` |
| 获取响应上下文 | https://www.volcengine.com/docs/82379/1783719 | `volcengine/responses.md` |
| 删除模型响应 | https://www.volcengine.com/docs/82379/1584286 | `volcengine/responses.md` |
| Responses 流式响应 | https://www.volcengine.com/docs/82379/1599499 | `volcengine/responses.md` |
| 向量化（多模态 Embeddings） | https://www.volcengine.com/docs/82379/1523520 | `volcengine/embeddings.md` |
| 图片生成（Seedream） | https://www.volcengine.com/docs/82379/1541523 | `volcengine/images.md` |
| 图片生成流式响应 | https://www.volcengine.com/docs/82379/1824137 | `volcengine/images.md` |
| 创建视频生成任务（Seedance） | https://www.volcengine.com/docs/82379/1520757 | `volcengine/video.md` |
| 查询视频生成任务 | https://www.volcengine.com/docs/82379/1521309 | `volcengine/video.md` |
| 查询视频生成任务列表 | https://www.volcengine.com/docs/82379/1521675 | `volcengine/video.md` |
| 取消 / 删除视频生成任务 | https://www.volcengine.com/docs/82379/1521720 | `volcengine/video.md` |
| 创建上下文缓存 | https://www.volcengine.com/docs/82379/1528789 | `volcengine/context-cache.md` |
| 上下文缓存对话 | https://www.volcengine.com/docs/82379/1529329 | `volcengine/context-cache.md` |
| 批量(Chat) API | https://www.volcengine.com/docs/82379/1528783 | `volcengine/batch.md` |
| 上传文件 | https://www.volcengine.com/docs/82379/1870405 | `volcengine/files.md` |
| 检索文件 | https://www.volcengine.com/docs/82379/1870406 | `volcengine/files.md` |
| 查询文件列表 | https://www.volcengine.com/docs/82379/1870407 | `volcengine/files.md` |
| 删除文件 | https://www.volcengine.com/docs/82379/1870408 | `volcengine/files.md` |
| file object | https://www.volcengine.com/docs/82379/1873424 | `volcengine/files.md` |
| 分词 API | https://www.volcengine.com/docs/82379/1528728 | `volcengine/tokenization.md` |
| 应用(bot) API | https://www.volcengine.com/docs/82379/1526787 | `volcengine/bot.md` |
| 联网插件数据结构 | https://www.volcengine.com/docs/82379/1285209 | `volcengine/bot.md` |
| 知识库插件数据结构 | https://www.volcengine.com/docs/82379/1285210 | `volcengine/bot.md` |
| 兼容 OpenAI SDK | https://www.volcengine.com/docs/82379/1330626 | `volcengine/openai-compat.md` |
| 错误码 | https://www.volcengine.com/docs/82379/1299023 | `volcengine/errors.md` |
