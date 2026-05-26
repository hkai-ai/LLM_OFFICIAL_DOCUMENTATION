---
source: https://developers.openai.com/api/reference/resources/organization
fetched_at: 2026-05-26
api_version: v1
---

# Administration · /v1/organization

组织 / 项目级管理 API，与日常推理 API 分开鉴权：**必须使用 Admin API Key**（在 Console 单独创建，与项目 API Key 是两套），且只能在 server 端使用。

所有 Admin 端点位于 `/v1/organization/...`，覆盖：

- Audit Logs · 审计日志
- Admin API Keys · 管理员 key 自管理
- Usage · 用量统计
- Invites · 组织邀请
- Users · 组织成员
- Groups · 用户组
- Roles · 自定义角色
- Data Retention · 数据保留策略
- Spend Alerts · 消费告警
- Certificates · mTLS 证书
- Projects · 项目（包含项目内 users / service_accounts / api_keys / rate_limits / model_permissions / groups / roles / data_retention / spend_alerts / certificates）

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_ADMIN_API_KEY`（注意：与项目 API Key 不同）。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

---

## 1. Audit Logs

| 动作 | METHOD | PATH |
| --- | --- | --- |
| List | GET | `/v1/organization/audit_logs` |

### Query

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `effective_at[gt|gte|lt|lte]` | integer | Unix 秒区间过滤。 |
| `project_ids[]` | string | 项目过滤。 |
| `event_types[]` | string | 事件类型过滤（如 `api_key.created` / `project.created`）。 |
| `actor_ids[]` / `actor_emails[]` | string | 操作者过滤。 |
| `resource_ids[]` | string | 资源过滤。 |
| `limit` / `after` / `before` | — | 分页。 |

### 响应字段

每条 log：`id` / `type` / `effective_at` / `actor` / `project` / 事件特定 `<event_type>` 字段。

---

## 2. Admin API Keys

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/admin_api_keys` |
| Retrieve | GET | `/v1/organization/admin_api_keys/{key_id}` |
| Delete | DELETE | `/v1/organization/admin_api_keys/{key_id}` |
| List | GET | `/v1/organization/admin_api_keys` |

Create body：`{ "name": "..." }`；返回 `value`（明文 key 仅本次可见）+ 元数据。

---

## 3. Usage · /v1/organization/usage/*

按业务类型独立子端点；统一 query：`start_time`（必填，Unix 秒）/ `end_time` / `bucket_width`（`1m` / `1h` / `1d`）/ `project_ids[]` / `user_ids[]` / `api_key_ids[]` / `models[]` / `group_by[]` / `limit` / `page`。

| 端点 | 统计对象 |
| --- | --- |
| `/v1/organization/usage/completions` | Chat / Responses |
| `/v1/organization/usage/embeddings` | Embeddings |
| `/v1/organization/usage/audio_speeches` | TTS |
| `/v1/organization/usage/audio_transcriptions` | STT |
| `/v1/organization/usage/code_interpreter_sessions` | code_interpreter 容器 |
| `/v1/organization/usage/images` | Images |
| `/v1/organization/usage/moderations` | Moderations |
| `/v1/organization/usage/vector_stores` | Vector stores（按存储字节） |
| `/v1/organization/usage/file_search_calls` | file_search 调用 |
| `/v1/organization/usage/web_search_calls` | web_search 调用 |
| `/v1/organization/usage/costs` | 费用汇总（USD） |

返回结构统一为 `{ "object": "page", "data": [{ "object": "bucket", "start_time", "end_time", "results": [...] }], "has_more": true, "next_page": "..." }`。

---

## 4. Invites

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/invites` |
| Retrieve | GET | `/v1/organization/invites/{invite_id}` |
| Delete | DELETE | `/v1/organization/invites/{invite_id}` |
| List | GET | `/v1/organization/invites` |

Create body：`{ "email": "...", "role": "owner" \| "reader" \| ..., "projects": [{"id":"proj_...","role":"member"}] }`。

---

## 5. Users · /v1/organization/users

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Retrieve | GET | `/v1/organization/users/{user_id}` |
| Update | POST | `/v1/organization/users/{user_id}` |
| Delete | DELETE | `/v1/organization/users/{user_id}` |
| List | GET | `/v1/organization/users` |

### 子资源：Roles

绑定到具体 user 的角色。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/users/{user_id}/roles` |
| Retrieve | GET | `.../roles/{role_id}` |
| Delete | DELETE | `.../roles/{role_id}` |
| List | GET | `/v1/organization/users/{user_id}/roles` |

---

## 6. Groups · /v1/organization/groups

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create / Retrieve / Update / Delete / List | POST / GET / POST / DELETE / GET | `/v1/organization/groups[/{group_id}]` |

子资源：

| 子资源 | 路径 |
| --- | --- |
| Users | `/v1/organization/groups/{group_id}/users[/{user_id}]` |
| Roles | `/v1/organization/groups/{group_id}/roles[/{role_id}]` |

---

## 7. Roles · /v1/organization/roles

自定义组织级角色：

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/roles` |
| Retrieve | GET | `/v1/organization/roles/{role_id}` |
| Update | POST | `/v1/organization/roles/{role_id}` |
| Delete | DELETE | `/v1/organization/roles/{role_id}` |
| List | GET | `/v1/organization/roles` |

Role 字段：`id` / `name` / `description` / `permissions[]` / `scope`。

---

## 8. Data Retention · /v1/organization/data_retention

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Retrieve | GET | `/v1/organization/data_retention` |
| Update | POST | `/v1/organization/data_retention` |

字段：`api_inputs_outputs_retention_days` / `evals_retention_days` 等。

---

## 9. Spend Alerts · /v1/organization/spend_alerts

按月预算告警。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/spend_alerts` |
| Update | POST | `/v1/organization/spend_alerts/{alert_id}` |
| Delete | DELETE | `/v1/organization/spend_alerts/{alert_id}` |
| List | GET | `/v1/organization/spend_alerts` |

字段：`threshold_usd` / `notify_emails[]` / `webhook_url` / `status`。

---

## 10. Certificates · /v1/organization/certificates

mTLS 证书自管理。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/organization/certificates` |
| Retrieve | GET | `/v1/organization/certificates/{cert_id}` |
| Update | POST | `/v1/organization/certificates/{cert_id}` |
| Delete | DELETE | `/v1/organization/certificates/{cert_id}` |
| List | GET | `/v1/organization/certificates` |
| Activate | POST | `/v1/organization/certificates/{cert_id}/activate` |
| Deactivate | POST | `/v1/organization/certificates/{cert_id}/deactivate` |

---

## 11. Projects · /v1/organization/projects

项目本身：

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create / Retrieve / Update | POST / GET / POST | `/v1/organization/projects[/{project_id}]` |
| List | GET | `/v1/organization/projects` |
| Archive | POST | `/v1/organization/projects/{project_id}/archive` |

> 项目**不可删除**，只能 archive。

### 项目内子资源（全部位于 `/v1/organization/projects/{project_id}/...`）

| 资源 | 端点 |
| --- | --- |
| Users | `.../users[/{user_id}]` + `.../users/{user_id}/roles[/{role_id}]` |
| Service Accounts | `.../service_accounts[/{sa_id}]`（含 `value` 仅 create 时返回） |
| API Keys | `.../api_keys[/{key_id}]`（**只能 list / retrieve / delete**，create 需在 Console 完成） |
| Rate Limits | `.../rate_limits` GET（list） + `.../rate_limits` POST（update，按 model 设定 RPM / TPM） |
| Model Permissions | `.../model_permissions/{model}` GET / POST / DELETE |
| Hosted Tool Permissions | `.../hosted_tool_permissions/{tool}` GET / POST |
| Groups | `.../groups[/{group_id}]` + `.../groups/{group_id}/roles[/{role_id}]` |
| Roles | `.../roles[/{role_id}]` |
| Data Retention | `.../data_retention` GET / POST |
| Spend Alerts | `.../spend_alerts[/{alert_id}]` |
| Certificates | `.../certificates` GET + `.../certificates/{cert_id}/activate` POST + `.../certificates/{cert_id}/deactivate` POST |

> 项目级与组织级同名资源是**独立** scope；项目级覆盖组织级的设置。

---

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `401` | `authentication_error` | 使用项目级 API Key 调用 Admin 端点 |
| `403` | `permission_error` | Admin Key 权限不足 |
| `404` | `not_found_error` | 资源不存在 |
| `409` | `invalid_request_error` | 试图删除项目（应 archive） |
| `429` | `rate_limit_error` | Admin 端点独立配额 |

## 参考

- 组织 API：<https://developers.openai.com/api/reference/resources/organization>
- 项目 API：<https://developers.openai.com/api/reference/resources/organization/subresources/projects>
- Usage：<https://developers.openai.com/api/reference/resources/organization/subresources/usage>
- Admin Keys 指南：<https://developers.openai.com/api/docs/guides/admin-api-keys>
