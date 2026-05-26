# 撰写规范

本仓库收集各 LLM 厂商官方 API 文档的中文整理版本。所有文档需遵循以下规范。

## 通用原则

1. **正文使用中文**，但所有字段名、枚举值、HTTP header、URL、错误码、模型 ID 等技术标识符必须保留**英文原文**，不翻译。
2. 文档以**官方原文事实**为准，遇到歧义优先保留官方原表述（必要时附原文引用）。
3. **不臆测、不补全**。官方没说的字段不要凭直觉补默认值；如果文档中确实缺失，明确写「文档未说明」。
4. **不写赘述性序言**（不要"在本文档中我们将介绍……"），开门见山。

## 目录结构

```
<vendor>/
├── README.md           # 厂商概览 + 端点索引
├── <endpoint>.md       # 单个端点的详细参数文档（如 messages.md / chat-completions.md）
└── models.md           # 该厂商的模型清单（如有需要）
```

厂商目录命名：用小写 + 连字符，例如 `anthropic/`、`openai/`、`google-gemini/`、`deepseek/`。

## 文件顶部元数据

每个 endpoint 文档**必须**以以下格式开头：

```markdown
---
source: <官方文档完整 URL>
fetched_at: YYYY-MM-DD
api_version: <API 版本号或日期版本，例如 anthropic-version: 2023-06-01；若无可写 N/A>
---

# <端点中文名> · <METHOD> <PATH>
```

`fetched_at` 用文档撰写当天的日期（北京时间）。

### 厂商主文档（`<vendor>/README.md`）追加字段：`last_updated`

每个厂商目录的 `README.md` 在 frontmatter 中**额外**追加一个 `last_updated` 字段，表示**该厂商目录下任一文档最近一次有意义更新的日期（北京时间）**：

```markdown
---
source: <厂商官方文档入口 URL>
fetched_at: YYYY-MM-DD       # 本 README 本身的抓取/撰写日期
api_version: <版本号 / N/A>
last_updated: YYYY-MM-DD     # 厂商目录下任一文档最近一次更新的日期
---
```

约定：

- `last_updated` 由维护者每次更新该厂商目录下任一文件（包括 `pricing.md` 在内）时同步刷新。
- 它**只出现在每个厂商目录的 `README.md`**，不出现在单端点文档里——单端点文档用自己的 `fetched_at`。
- 用途：让读者一眼看出"上次尝试同步这个厂商文档大概是什么时候"，无需逐文件查看。
- `fetched_at` 与 `last_updated` 在 README 上经常一致，但语义不同：前者是本 README 文件本身的抓取日期，后者是整个厂商目录的"最后活动时间"。两者可独立存在与维护。

### 定价文档（`<vendor>/pricing.md`）

每个厂商目录下建议提供一份 `pricing.md`，汇总该厂商所有官方在售模型的最新定价。撰写要点：

- 顶部 frontmatter 与其他端点文档一致（`source` 指向厂商官方定价页，`fetched_at` 为本次抓取日期）。
- **必须在文档第一行正文中显式说明货币**（USD / 人民币 元 等）与计价单位（每 1M tokens / 每秒 / 每张图等），任何后续表格的数字以此为准。
- 区分不同档位（标准 / 批处理 / Flex / Priority / 缓存命中 / 缓存未命中 等）时单独成列或单独成表，不要把不同档位混在一个数字里。
- 优惠活动须标注**截止日期**（如「75% off 截至 2026-05-31」），并保留原价。
- 不臆测：官方未给出的子型号价格写「文档未列出」，不要从其他档位推导。
- 同步主文档 README 中的 `last_updated`。

## 参数表格式

请求参数统一用 markdown 表格：

| 字段 | 类型 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `model` | string | ✓ | — | 简要说明 |
| `temperature` | number | ✗ | `1.0` | 取值范围、影响…… |

约定：
- 必填用 `✓` / `✗`。
- 默认值用反引号包住的字面值，无默认写 `—`。
- 类型用 `string` / `integer` / `number` / `boolean` / `object` / `array<T>` / `string \| array<TextBlock>` 等。
- 枚举类参数在「说明」列列出全部允许值，例如：`role` 取 `system` / `user` / `assistant` / `tool`。
- 嵌套对象**单独再起一个表格**，标题用三级标题 `### <字段路径>`，例如 `### messages[].content[]`。

## 响应字段

同样用表格，并区分顶层对象与子结构。如有 SSE 流式响应，单独一节描述事件类型与各事件结构。

## 示例

每个端点至少给出一个**最小请求示例**（JSON）和一个**最小响应示例**（JSON）。代码块用 \`\`\`json 标注语言。

复杂特性（工具调用、流式、多模态、缓存等）可单独再加示例。

## 引用与外链

- 关键说明可在末尾用「> 引用」块附原文片段，便于核对。
- 末尾「参考」一节列出本文档参考的官方 URL（每个 endpoint 文档至少包含其源 URL 与上级目录索引 URL）。

## 撰写 Skill

每个厂商在 `.claude/skills/<vendor>-api-docs/SKILL.md` 撰写一份 skill，内容须包含：

1. **frontmatter**：`name` 与 `description`（description 写清楚触发条件，如「需要查阅或更新 <厂商> 官方 API 文档时使用」）。
2. **文档站全貌**：官方文档的目录组织（一级/二级分类）、入口 URL、是否需登录、是否有重定向。
3. **抓取要点**：哪些 URL 可被 WebFetch 直接抓取、哪些需要换源、是否需要先 WebSearch 找子页。
4. **更新流程**：如何检查文档变动（版本号、changelog 页、模型清单页等）、本仓库对应文件应如何同步更新。
5. **坑点清单**：踩过的具体问题（301 重定向到新域名、403 反爬、文档站是 CSR 渲染导致内容缺失、字段在 OpenAPI 与人工写的页面不一致等）。
6. **关键链接表**：endpoint 名 → 官方文档 URL 的对照表，方便日后定向更新。

## 不要做的事

- 不要在 markdown 中加 emoji（用户偏好）。
- 不要为没有必要的"亮点"加粗。粗体只用于强调字段关系或关键约束。
- 不要把 OpenAI SDK 的语法当作 API 字段写进去（如 `client.chat.completions.create()` 不是 API 字段，是 SDK）。文档对象是 HTTP API。
- 不要为单端点写"使用场景介绍"等营销文案，直接写参数。
