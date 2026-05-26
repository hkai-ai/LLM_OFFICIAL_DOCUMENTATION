# 使用说明

本仓库是各 LLM 厂商**官方 API 文档**的中文整理副本，定位是「快速查参数、查字段、查定价、对照差异」，不是教程，也不替代官方原文。本说明告诉你**怎么用、怎么查、怎么和 Claude Code 配合**。

> 撰写规范见 [CONVENTIONS.md](./CONVENTIONS.md)，厂商索引与能力横向对照见 [README.md](./README.md)。

## 这个仓库适合谁

- 在多家 LLM 厂商之间做选型 / 接入对比，需要快速看清字段差异（思考模式、缓存、工具调用、流式协议等）。
- 接 SDK 时只想确认某个字段的类型 / 默认值 / 取值范围，不想点开十几个官方文档站。
- 在 Claude Code 里写跨厂商代码，希望 Claude 直接读到带 frontmatter 与抓取日期的本地副本，而不是从训练知识里猜。
- 给团队内部留一份「上次抓取的官方事实快照」，方便核对官方文档悄悄改动的字段。

## 三种典型查阅方式

### 1) 按厂商查

从根目录 [README.md 的「厂商索引」](./README.md#厂商索引)进入对应目录，比如 [`anthropic/`](./anthropic/)、[`openai/`](./openai/)、[`google-gemini/`](./google-gemini/)。每个厂商目录的固定文件：

| 文件 | 内容 |
| --- | --- |
| `README.md` | 厂商概览、Base URL、鉴权、端点索引；frontmatter 的 `last_updated` 字段告诉你**该厂商目录上次同步是什么时候** |
| `<endpoint>.md` | 单个端点的请求参数表 + 响应字段表 + 最小示例 |
| `errors.md` | 错误响应结构、错误码清单 |
| `models.md` | 模型清单（如有 List Models 端点也在这里） |
| `pricing.md` | 该厂商当前在售模型 / 工具 / 套餐定价（含货币、计价单位、优惠截止日期） |

每个 endpoint 文档顶部都有 frontmatter：

```markdown
---
source: <官方文档完整 URL>
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01
---
```

读文档前先看 `fetched_at`，判断它和官方原文的可能时差。

### 2) 按能力横向对比

直接看 [README.md 的「能力横向对照」](./README.md#能力横向对照)，一张表覆盖：Base URL / 鉴权 / 协议风格 / 推理 / 视觉 / 音频 / 函数调用 / 内置工具 / JSON 输出 / 流式 / 上下文缓存 / 缓存计费字段 / 中断续写 / 路由 / 旗舰模型。

**用途**：选型阶段直接对照差异，定位到具体厂商后再钻进对应 endpoint 文档看细节。

### 3) 按端点速查

[README.md 的「端点速查」](./README.md#端点速查)按能力分组（文本生成 / Token 计数 / 嵌入向量 / 文件缓存批处理 / 音频图像 / 定价表）列出了每家对应端点的链接，跨厂商找「同一件事的不同 API」就走这里。

## 字段层级约定

读端点文档前先了解三层约定，省得来回翻：

1. **顶层参数表**用 markdown 表格，列依次是「字段 / 类型 / 必填 / 默认 / 说明」。必填用 `✓` / `✗`，无默认值写 `—`，类型用 `string \| array<TextBlock>` 这种联合写法。
2. **嵌套对象**单起一个三级标题表格，标题写字段路径，如 `### messages[].content[]`。
3. **流式响应**单起一节，按事件类型 + 各事件结构来描述，不和非流式响应混在一起。

> 文档以**官方原文事实**为准。官方没说的字段不会被臆测补全；遇到「文档未说明」字样，意味着官方文档本身就没说。

## 配合 Claude Code 使用

`.claude/skills/` 下为每个厂商维护了一份 skill（如 [anthropic-api-docs](./.claude/skills/anthropic-api-docs/SKILL.md)、[openai-api-docs](./.claude/skills/openai-api-docs/SKILL.md)），skill 记录了：

- 该厂商官方文档站的全貌（一级目录、是否需登录、是否有 SPA 渲染问题）
- 抓取要点（哪些 URL 可 WebFetch、哪些要换源）
- 更新流程与坑点清单（重定向、字段命名差异、SDK 与 HTTP 不一致等）
- endpoint → 官方 URL 的对照表

### 常见调用场景

- **「帮我查 Claude 的 thinking 字段怎么传」** —— Claude 直接读 [`anthropic/messages.md`](./anthropic/messages.md)；本地副本有 frontmatter 的 `fetched_at`，能判断新鲜度。
- **「帮我把 OpenAI 文档刷一下到最新」** —— 触发 [openai-api-docs](./.claude/skills/openai-api-docs/SKILL.md) skill，它知道 platform.openai.com / developers.openai.com 两个站点的差异和抓取坑。
- **「Kimi K2.6 和 DeepSeek V4-pro 的缓存定价谁便宜」** —— 直接对比 [`moonshot/pricing.md`](./moonshot/pricing.md) 与 [`deepseek/pricing.md`](./deepseek/pricing.md)，注意两家都是人民币计价。
- **「给我一个跨厂商的 chat completion 调用样例」** —— 让 Claude 同时读多家的 `chat-completions.md`，输出按字段名对齐的对照表。

### 把仓库作为上下文丢给 Claude

如果你在另一个项目里写 LLM 接入代码，可以让 Claude Code 把本仓库当成知识库：

```
请参考 D:\Backup\Documents\GitHub\LLM_OFFICIAL_DOCUMENTATION\anthropic\messages.md
为这段代码补全 Anthropic 的 tool_use 字段
```

Claude 会读取本地文件，比直接依赖训练知识更准确，因为本地副本明确标注了 `fetched_at`。

## 怎么贡献 / 维护

### 更新一个已有 endpoint

1. 找到对应 `<vendor>/<endpoint>.md`，比对官方文档（必要时用 skill 协助抓取）。
2. 同步参数 / 响应字段 / 示例。
3. 更新顶部 frontmatter 的 `fetched_at` 为当天日期（北京时间）。
4. 同时把厂商 `README.md` 的 `last_updated` 改为同一日期。

### 同步定价

1. 编辑 `<vendor>/pricing.md`。
2. **第一行正文**必须显式说明货币（USD / 人民币 元）与计价单位（每 1M tokens / 每秒 / 每张图）。
3. 不同档位（标准 / 批处理 / Flex / Priority / 缓存命中 / 缓存未命中）单独成列或单独成表。
4. 优惠活动标注**截止日期**并保留原价。
5. 同步厂商 `README.md` 的 `last_updated`。

### 新增一个厂商

1. 复制现有厂商目录结构（推荐参考 [`anthropic/`](./anthropic/) 或 [`openai/`](./openai/)）。
2. 用 [`_template.md`](./_template.md) 起单端点文档。
3. 在 [`.claude/skills/`](./.claude/skills/) 下新建 `<vendor>-api-docs/SKILL.md`，按 [CONVENTIONS.md 的「撰写 Skill」节](./CONVENTIONS.md#撰写-skill)的六项内容填写。
4. 在根 README 的「厂商索引」「目录结构」「能力横向对照」「端点速查」「定价表」「当前状态」六处都补上对应行 / 列。

### 不要做的事

- 不要加 emoji（用户偏好，统一约定）。
- 不要把 SDK 语法（如 `client.chat.completions.create()`）当作 API 字段写进去——文档对象是 HTTP API。
- 不要为单 endpoint 写「使用场景介绍」等营销文案，直接写参数。
- 不要臆测官方没说的字段默认值，明确写「文档未说明」。

完整规范见 [CONVENTIONS.md](./CONVENTIONS.md)。

## FAQ

**Q：为什么不直接看官方文档？**
A：可以。但本仓库的价值是（a）中文整理、（b）按统一表格结构呈现、（c）跨厂商横向对照、（d）frontmatter 标注抓取日期可追溯、（e）适合作为 Claude Code 的本地上下文。

**Q：`fetched_at` 和 `last_updated` 有什么区别？**
A：`fetched_at` 是**当前文件**本身的抓取/撰写日期；`last_updated` 只出现在每个厂商目录的 `README.md`，表示**整个厂商目录**最近一次任意文件被更新的日期。详见 [CONVENTIONS.md 「文件顶部元数据」](./CONVENTIONS.md#文件顶部元数据)。

**Q：文档里说「文档未说明」是什么意思？**
A：字面意思——官方原文就没说。本仓库不臆测补全，需要时请去官方原文或抓包确认。

**Q：阿里百炼 / OpenRouter 为什么没单独的 `pricing.md`？**
A：前者价格随上游变动频繁，建议查官方[模型广场](https://bailian.console.aliyun.com/?tab=model#/model-market)；后者通过 `GET /api/v1/models` 实时返回 `pricing` 字段。详见 [README.md 「定价表」节](./README.md#定价表)。

**Q：还会新增哪些厂商？**
A：待补：xAI Grok、字节豆包、Mistral 等，见 README 末尾。
