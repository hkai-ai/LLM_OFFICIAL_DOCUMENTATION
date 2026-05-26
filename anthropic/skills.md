---
source: https://platform.claude.com/docs/en/api/skills/create-skill
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01；anthropic-beta: skills-2025-10-02
---

# Skills（Agent Skills）API · /v1/skills（Beta）

将一组指令文件（`SKILL.md` + 可选脚本 / 资源）上传到 Anthropic，模型在 Messages 请求中通过 `container.skills[]` 引用，按需展开 SKILL 内容（progressive disclosure），与 `code_execution` 工具配合可生成 PPT/Word/Excel/PDF 等文件。

**Beta header**：`anthropic-beta: skills-2025-10-02`（创建 / 列出 / 引用都要带；与 `code-execution-2025-08-25` 经常叠加）。

## Anthropic-managed Skills

Anthropic 维护以下系统 skill，直接可用、无需上传：

| skill_id | 用途 |
| --- | --- |
| `pptx` | 创建 / 编辑 PowerPoint |
| `xlsx` | 创建 / 分析 Excel |
| `docx` | 创建 / 编辑 Word |
| `pdf` | 生成 PDF |

## 端点索引

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create skill | POST | `/v1/skills` |
| List skills | GET | `/v1/skills` |
| Retrieve skill | GET | `/v1/skills/{skill_id}` |
| Delete skill | DELETE | `/v1/skills/{skill_id}` |
| Create skill version | POST | `/v1/skills/{skill_id}/versions` |
| List skill versions | GET | `/v1/skills/{skill_id}/versions` |
| Retrieve skill version | GET | `/v1/skills/{skill_id}/versions/{version}` |
| Delete skill version | DELETE | `/v1/skills/{skill_id}/versions/{version}` |

> CRUD 端点中除创建 / 删除外，文档详尽程度较低；以下结构以官方 reference 为准。

## 公共鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` / `Authorization` | 二选一 | API Key 或 WIF Bearer。 |
| `anthropic-version` | ✓ | `2023-06-01`。 |
| `anthropic-beta` | ✓ | 必含 `skills-2025-10-02`。 |
| `content-type` | 上传时 ✓ | `multipart/form-data`。 |

## Skill 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 例如 `skill_01JAbcdefghijklmnopqrstuvw`。 |
| `type` | string | 固定 `skill`。 |
| `display_title` | string | 人类可读名（仅展示，不进入模型 prompt）。 |
| `source` | enum | `custom`（用户自建） / `anthropic`（系统）。 |
| `latest_version` | string | 最新版本号（Unix epoch 数字，例如 `1759178010641129`）。 |
| `created_at` / `updated_at` | string | ISO 8601。 |

## SkillVersion 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 例如 `skillver_...`。 |
| `type` | string | 固定 `skill_version`。 |
| `skill_id` | string | 所属 skill。 |
| `version` | string | Unix epoch 时间戳形式版本号。 |
| `name` | string | 解析自 `SKILL.md` frontmatter 的 `name` 字段（≤64 字符，仅小写字母 / 数字 / `-`，不得包含 XML 标签或保留词 `anthropic` / `claude`）。 |
| `description` | string | 解析自 `SKILL.md` frontmatter 的 `description`（≤1024 字符，非空，禁 XML 标签）。 |
| `directory` | string | 上传时所在顶层目录名。 |
| `created_at` | string | ISO 8601。 |

## SKILL.md 与目录结构

每个自建 skill 必须按以下结构打包：

```
my-skill/
├── SKILL.md             # 必需。YAML frontmatter + 自然语言指令
├── *.md                 # 可选。补充文档
├── scripts/             # 可选。可执行代码
└── resources/           # 可选。模板 / 数据
```

`SKILL.md` 顶部 YAML frontmatter 至少含 `name` 与 `description` 两个字段。

## Create Skill · POST /v1/skills

上传整个 skill 目录（打包为 multipart）。

```bash
curl https://api.anthropic.com/v1/skills \
  -X POST \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: skills-2025-10-02" \
  -F "files[]=@my-skill/SKILL.md" \
  -F "files[]=@my-skill/scripts/run.py"
```

返回 Skill 对象，并自动建立 `latest_version`。

## List Skills · GET /v1/skills

### Query 参数

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `limit` | number | `20` | `≤100`。 |
| `page` | string | — | 取上次响应的 `next_page`。 |
| `source` | enum | — | `custom` / `anthropic` 过滤。 |

### 响应

```json
{
  "data": [
    {
      "id": "skill_01JAbcdefghijklmnopqrstuvw",
      "type": "skill",
      "display_title": "My Custom Skill",
      "source": "custom",
      "latest_version": "1759178010641129",
      "created_at": "2024-10-30T23:58:27.427722Z",
      "updated_at": "2024-10-30T23:58:27.427722Z"
    }
  ],
  "has_more": true,
  "next_page": "page_..."
}
```

## Create Skill Version · POST /v1/skills/{skill_id}/versions

新增一个版本，文件结构同 create。返回 SkillVersion 对象，并把 skill 的 `latest_version` 推进到该版本。

```bash
curl https://api.anthropic.com/v1/skills/$SKILL_ID/versions \
  -X POST \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: skills-2025-10-02" \
  -F "files[]=@my-skill/SKILL.md"
```

## 在 Messages 中引用 Skill

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: code-execution-2025-08-25,skills-2025-10-02" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-4-7",
    "max_tokens": 16000,
    "container": {
      "skills": [
        {"type": "anthropic", "skill_id": "pptx", "version": "latest"}
      ]
    },
    "messages": [
      {"role": "user", "content": "Create a presentation about renewable energy with 5 slides"}
    ],
    "tools": [{"type": "code_execution_20250825", "name": "code_execution"}]
  }'
```

### container.skills[] 结构

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | enum | ✓ | `anthropic` / `custom`。 |
| `skill_id` | string | ✓ | 对应 Skills API 返回的 ID（系统 skill 用名字，如 `pptx`）。 |
| `version` | string | ✓ | 指定版本号或 `latest`。 |

> 通常需同时声明 `tools[] = [{ type: "code_execution_20250825", name: "code_execution" }]`，Skill 由模型在 code execution 容器内执行；产物以 `code_execution_tool_result` / `bash_code_execution_tool_result` block 返回，含 `file_id` 后再走 [files.md](./files.md) 下载。

## 限流

Skills 走 Messages 公共 RPM/ITPM/OTPM；Managed Agents 路线下的 skill 操作受 Managed Agents 限流（见 [rate-limits.md](./rate-limits.md)）。

## 参考

- Overview：<https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview>
- Quickstart：<https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart>
- API：<https://platform.claude.com/docs/en/api/skills/create-skill>
- Skill Version API：<https://platform.claude.com/docs/en/api/skills/create-skill-version>
- Authoring 最佳实践：<https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices>
- Skills + Managed Agents：<https://platform.claude.com/docs/en/managed-agents/skills>
