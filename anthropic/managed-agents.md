---
source: https://platform.claude.com/docs/en/managed-agents/overview
fetched_at: 2026-05-26
api_version: anthropic-version: 2023-06-01；anthropic-beta: managed-agents-2026-04-01
---

# Managed Agents API（Beta）

Managed Agents 把「带容器的 agent 服务」托管在 Anthropic 侧。开发者只声明 agent 配置和云端环境，Anthropic 负责拉起带网络 / 文件系统 / 工具的容器，并以 SSE 事件流暴露 agent 的执行过程。

**Beta header**：所有端点必须带 `anthropic-beta: managed-agents-2026-04-01`。SDK 自动注入。

涉及三组核心资源：

| 资源 | 路径前缀 | 含义 |
| --- | --- | --- |
| Agents | `/v1/agents` | 可复用、带版本号的 agent 配置（model / system / tools / MCP / skills 等） |
| Environments | `/v1/environments` | 容器配置（packages / networking 等） |
| Sessions | `/v1/sessions` | 单个 agent 在某个 environment 中的运行实例 |

> 限流见 [rate-limits.md](./rate-limits.md) §Managed Agents（create 类 300 RPM，read 类 600 RPM）。

## 公共鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-api-key` / `Authorization` | 二选一 | API Key 或 WIF Bearer。 |
| `anthropic-version` | ✓ | `2023-06-01`。 |
| `anthropic-beta` | ✓ | 必含 `managed-agents-2026-04-01`。 |
| `content-type` | 写操作时 ✓ | `application/json`。 |

请求体上限：Agents / Sessions / Environments 各为 **32 MB**。

## 1. Agents · /v1/agents

### Agent 配置字段（请求 body）

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 人类可读名称。 |
| `model` | `string \| ModelObject` | ✓ | 模型 ID（Claude 4.5 及以后均支持）。用 `{"id":"claude-opus-4-7","speed":"fast"}` 开 fast mode。 |
| `system` | string | ✗ | system prompt。 |
| `description` | string | ✗ | 描述。 |
| `tools` | `array<Tool>` | ✗ | pre-built agent 工具（如 `{"type":"agent_toolset_20260401"}`）、MCP 工具、自定义工具的组合。 |
| `mcp_servers` | `array<MCPServer>` | ✗ | MCP 服务器声明（type=`url`/`sse` + `name` + `url`）。 |
| `skills` | `array<SkillRef>` | ✗ | 引用 [skills.md](./skills.md) 中已上传的 skill。 |
| `multiagent` | object | ✗ | Multi-agent 协调器，列出可委派的子 agent。 |
| `metadata` | `map<string,string>` | ✗ | 自定义 KV。 |

### Agent 对象（响应）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 例如 `agent_01HqR2k7vXbZ9mNpL3wYcT8f`。 |
| `type` | string | 固定 `agent`。 |
| `version` | integer | 从 `1` 起步，每次配置变更自动 +1。 |
| `created_at` / `updated_at` | string | ISO 8601。 |
| `archived_at` | string | 归档时间；归档后只读，旧 session 仍可运行，但不能创建新 session。 |
| `name` / `model` / `system` / `description` / `tools` / `mcp_servers` / `skills` / `metadata` | — | 回显请求配置。 |

### 端点

| 动作 | METHOD | PATH | 备注 |
| --- | --- | --- | --- |
| Create | POST | `/v1/agents` | 返回 `version: 1`。 |
| Retrieve | GET | `/v1/agents/{agent_id}` | — |
| List | GET | `/v1/agents` | — |
| Update | POST | `/v1/agents/{agent_id}` | body 须带 `version` 做乐观锁；变更字段会生成新版本。 |
| List versions | GET | `/v1/agents/{agent_id}/versions` | 返回所有历史版本。 |
| Archive | POST | `/v1/agents/{agent_id}/archive` | 写入 `archived_at`。 |

### Update 语义

- 省略的字段保留原值。
- 标量字段（`model` / `system` / `name` / `description`）整体替换；`system` / `description` 可传 `null` 清空，`model` / `name` 不可清空。
- 数组字段（`tools` / `mcp_servers` / `skills`）整体替换；传 `null` 或 `[]` 清空。
- `multiagent` 整体替换。
- `metadata` 按 key 合并；传 `""` 删除指定 key。
- 无差异更新不生成新版本。

### Create 示例

```bash
curl https://api.anthropic.com/v1/agents \
  -X POST \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -H "content-type: application/json" \
  -d '{
    "name": "Coding Assistant",
    "model": "claude-opus-4-7",
    "system": "You are a helpful coding agent.",
    "tools": [{"type": "agent_toolset_20260401"}]
  }'
```

## 2. Environments · /v1/environments

### Environment 配置字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 在 organization + workspace 内唯一。 |
| `config.type` | enum | ✓ | 目前 `cloud`；自托管走 [Self-hosted sandboxes](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes)。 |
| `config.packages` | object | ✗ | 预装包；键为包管理器（`apt` / `cargo` / `gem` / `go` / `npm` / `pip`），值为 `array<string>`，可带版本号（`pandas==2.2.0`、`express@4.18.0` 等）。多个键时按字母顺序执行。 |
| `config.networking.type` | enum | ✓ | `unrestricted`（默认）或 `limited`。 |
| `config.networking.allowed_hosts` | `array<string>` | `limited` 时 ✗ | HTTPS 域名白名单。 |
| `config.networking.allow_mcp_servers` | boolean | `limited` 时 ✗ | 是否额外放行 agent 上声明的 MCP server endpoint。默认 `false`。 |
| `config.networking.allow_package_managers` | boolean | `limited` 时 ✗ | 是否放行 PyPI / npm 等公共注册中心。默认 `false`。 |

### Environment 对象（响应）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 例如 `env_...`。 |
| `type` | string | 固定 `environment`。 |
| `name` | string | 同请求。 |
| `config` | object | 回显。 |
| `created_at` | string | ISO 8601。 |
| `archived_at` | string | 归档时间，归档后只读。 |

> Environments **不带版本**；如频繁更新，建议自行记录变更日志。

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/environments` |
| Retrieve | GET | `/v1/environments/{environment_id}` |
| List | GET | `/v1/environments` |
| Archive | POST | `/v1/environments/{environment_id}/archive` |
| Delete | DELETE | `/v1/environments/{environment_id}` |

### Create 示例

```bash
curl https://api.anthropic.com/v1/environments \
  -X POST \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -H "content-type: application/json" \
  -d '{
    "name": "python-dev",
    "config": {
      "type": "cloud",
      "packages": { "pip": ["pandas", "numpy", "scikit-learn"] },
      "networking": { "type": "unrestricted" }
    }
  }'
```

## 3. Sessions · /v1/sessions

Session = 一个 agent 在一个 environment 中的运行实例，保留对话历史。**生命周期分两步**：

1. `POST /v1/sessions` 创建容器；
2. `POST /v1/sessions/{id}/events` 发 `user.message` 事件触发执行。

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `agent` | `string \| { type:"agent", id, version }` | ✓ | 字符串使用最新版本；对象固定到具体版本。 |
| `environment_id` | string | ✓ | environment 资源 ID。 |
| `vault_ids` | `array<string>` | ✗ | OAuth 凭证库 ID 列表（MCP 工具认证用，详见 [Vaults](https://platform.claude.com/docs/en/managed-agents/vaults)）。 |
| `metadata` | `map<string,string>` | ✗ | 自定义 KV。 |

### Session 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 例如 `sess_...`。 |
| `type` | string | 固定 `session`。 |
| `agent_id` | string | — |
| `agent_version` | integer | 实际固定到的 agent 版本。 |
| `environment_id` | string | — |
| `vault_ids` | `array<string>` | — |
| `status` | enum | `idle` / `running` / `rescheduling` / `terminated`。 |
| `archived_at` | string | 归档时间。 |
| `created_at` / `updated_at` | string | ISO 8601。 |

### 端点

| 动作 | METHOD | PATH | 备注 |
| --- | --- | --- | --- |
| Create | POST | `/v1/sessions` | — |
| Retrieve | GET | `/v1/sessions/{session_id}` | — |
| List | GET | `/v1/sessions?agent_id=...` | 按 agent 过滤。 |
| Update agent override | POST | `/v1/sessions/{session_id}` | 仅 `idle` 时可改 `agent.tools` / `agent.mcp_servers`（包括 permission policy），不回传给 agent 资源。 |
| Send events | POST | `/v1/sessions/{session_id}/events` | body 形如 `{"events":[{"type":"user.message","content":[{"type":"text","text":"..."}]}]}`。 |
| Stream events | GET | `/v1/sessions/{session_id}/stream` | SSE 流式接收 agent 输出 + 工具事件。 |
| Archive | POST | `/v1/sessions/{session_id}/archive` | 保留历史，禁止新事件。 |
| Delete | DELETE | `/v1/sessions/{session_id}` | `running` 状态不可删除，需先发 interrupt 事件。文件 / memory / vault / skill / environment / agent 等独立资源不受影响。 |

### 事件类型（events-and-streaming）

| 类型 | 方向 | 说明 |
| --- | --- | --- |
| `user.message` | 客户端 → session | 文本 / 图片等用户输入；触发 agent 执行。 |
| `user.interrupt` | 客户端 → session | 打断 `running` 会话以便更新配置或删除。 |
| `assistant.message` | session → 客户端 | 模型文本输出。 |
| `tool.use` / `tool.result` | session → 客户端 | 工具调用与回执（含 permission policy 触发的确认请求）。 |
| `error` | session → 客户端 | 不可恢复错误，最终 `status: terminated`。 |

> 完整事件 schema 见 <https://platform.claude.com/docs/en/managed-agents/events-and-streaming>。

### Create + 发起任务示例

```bash
# 1. 创建 session
SESSION_ID=$(curl https://api.anthropic.com/v1/sessions \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -H "content-type: application/json" \
  -d "{\"agent\":\"$AGENT_ID\",\"environment_id\":\"$ENVIRONMENT_ID\"}" \
  | jq -r .id)

# 2. 发送 user.message 事件
curl https://api.anthropic.com/v1/sessions/$SESSION_ID/events \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -H "content-type: application/json" \
  -d '{
    "events": [
      {"type":"user.message","content":[{"type":"text","text":"List the files in the working directory."}]}
    ]
  }'

# 3. SSE 订阅事件流
curl -N https://api.anthropic.com/v1/sessions/$SESSION_ID/stream \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01"
```

## 错误码

公共错误结构见 [errors.md](./errors.md)。Managed Agents 特有：

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | `agent.version` 冲突 / `name` 重复 / 配置 schema 错误 |
| `404` | `not_found_error` | `agent_id` / `environment_id` / `session_id` 不存在或不在当前 Workspace |
| `409` | `invalid_request_error` | 对 `running` session 调用 delete / 对非 `idle` session 调用 update |
| `429` | `rate_limit_error` | 触发 Managed Agents 限流（300/600 RPM） |

## 参考

- Overview：<https://platform.claude.com/docs/en/managed-agents/overview>
- Agents：<https://platform.claude.com/docs/en/managed-agents/agent-setup>
- Environments：<https://platform.claude.com/docs/en/managed-agents/environments>
- Sessions：<https://platform.claude.com/docs/en/managed-agents/sessions>
- Events & streaming：<https://platform.claude.com/docs/en/managed-agents/events-and-streaming>
- Tools：<https://platform.claude.com/docs/en/managed-agents/tools>
- Vaults：<https://platform.claude.com/docs/en/managed-agents/vaults>
- Multi-agent：<https://platform.claude.com/docs/en/managed-agents/multi-agent>
