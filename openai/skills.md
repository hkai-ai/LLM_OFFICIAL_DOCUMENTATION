---
source: https://developers.openai.com/api/reference/resources/skills
fetched_at: 2026-05-26
api_version: v1
---

# Skills · /v1/skills

把一组 SKILL 文件（`SKILL.md` + 资源）上传到 OpenAI，模型在 Responses / code_interpreter 容器里按需加载、执行。每个 skill 有 immutable 版本，可指定 `default_version` 做 rollout。

> 注意：OpenAI Skills 与 Anthropic Skills 是**不同厂商各自的概念**，结构相似但 schema 不共用。

| 资源 | 路径前缀 |
| --- | --- |
| Skills | `/v1/skills` |
| Versions | `/v1/skills/{skill_id}/versions` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | JSON 接口为 `application/json`；上传走 `multipart/form-data`。 |

## 1. Skills CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/skills` |
| Retrieve | GET | `/v1/skills/{skill_id}` |
| Update | POST | `/v1/skills/{skill_id}` |
| Delete | DELETE | `/v1/skills/{skill_id}` |
| List | GET | `/v1/skills` |
| Retrieve content（zip） | GET | `/v1/skills/{skill_id}/content` |

### Create body（multipart）

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `files[]` | ✓ | skill 目录所有文件；其中必须含一份 `SKILL.md`（YAML frontmatter 含 `name` / `description`）。 |
| `name` | ✗ | 覆盖 SKILL.md 中的 name。 |
| `description` | ✗ | 覆盖 description。 |

### Skill 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `skill_...`。 |
| `object` | string | 固定 `skill`。 |
| `name` | string | — |
| `description` | string | — |
| `default_version` | string | 当前默认指针；不指定则等于 `latest_version`。 |
| `latest_version` | string | 最新版本号。 |
| `created_at` / `updated_at` | integer | Unix 秒。 |

### Update body

仅允许 `default_version` 指针切换（与 metadata / 描述合并）。**内容修改一律通过新建 version**。

### List query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` / `before` | string | — | 游标。 |
| `limit` | integer | `20` | `1`–`100`。 |
| `order` | enum | `desc` | — |

### Retrieve content

返回 `Content-Type: application/zip` 的字节流，是当前 `default_version` 的完整 skill 目录。

## 2. Skill Versions · /v1/skills/{skill_id}/versions

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create version | POST | `/v1/skills/{skill_id}/versions` |
| Retrieve | GET | `/v1/skills/{skill_id}/versions/{version_id}` |
| List | GET | `/v1/skills/{skill_id}/versions` |
| Delete | DELETE | `/v1/skills/{skill_id}/versions/{version_id}` |
| Retrieve content | GET | `/v1/skills/{skill_id}/versions/{version_id}/content` |

### Create body（multipart）

同 Create Skill 的 `files[]` 字段；上传成功后 `latest_version` 推进。

### SkillVersion 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `skillver_...`。 |
| `object` | string | 固定 `skill.version`。 |
| `skill_id` | string | 所属 skill。 |
| `version` | string | 版本号（递增整数 / 时间戳形式）。 |
| `name` / `description` | string | 解析自 SKILL.md。 |
| `created_at` | integer | Unix 秒。 |

## 在 Responses 中引用

```json
{
  "model": "gpt-4.1",
  "tools": [
    { "type": "skill_use", "skill_id": "skill_abc", "version": "latest" }
  ],
  "input": "用 SKILL 处理这个 CSV"
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `tools[].type` | string | `skill_use`。 |
| `tools[].skill_id` | string | skill ID。 |
| `tools[].version` | string | 版本号或 `latest` / `default`。 |

Skill 内若声明了脚本，需要同时启用 `code_interpreter` 工具，由 [containers.md](./containers.md) 中的容器执行。

## SKILL.md 约定

```markdown
---
name: my-skill
description: A skill that does something useful.
---

# Detailed instructions for the model
...
```

| 字段 | 限制 |
| --- | --- |
| `name` | ≤64 字符；仅小写字母、数字、`-`；不得含保留词 `openai`。 |
| `description` | ≤1024 字符。 |

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | 缺少 SKILL.md、字段超长、保留词 |
| `404` | `not_found_error` | skill / version 不存在 |
| `409` | `invalid_request_error` | 删除被 default_version 指向的 version |

## 参考

- API：<https://developers.openai.com/api/reference/resources/skills>
- 指南：<https://developers.openai.com/api/docs/guides/skills>
- Containers：[containers.md](./containers.md)
