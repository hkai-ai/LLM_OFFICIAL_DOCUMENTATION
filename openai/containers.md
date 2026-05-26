---
source: https://developers.openai.com/api/reference/resources/containers
fetched_at: 2026-05-26
api_version: v1
---

# Containers · /v1/containers

为 `code_interpreter` 工具（Responses / Assistants）提供持久化执行容器：可挂文件、跨多轮调用保留工作目录、按需配置内存与网络。

| 资源 | 路径前缀 |
| --- | --- |
| Containers | `/v1/containers` |
| Files（容器内文件） | `/v1/containers/{container_id}/files` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`；上传文件走 `multipart/form-data`。 |

## 1. Containers CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/containers` |
| Retrieve | GET | `/v1/containers/{container_id}` |
| List | GET | `/v1/containers` |
| Delete | DELETE | `/v1/containers/{container_id}` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✓ | 人类可读名。 |
| `file_ids` | `array<string>` | ✗ | 创建时直接挂入的 Files API file ID。 |
| `expires_after` | object | ✗ | `{ "anchor": "last_active_at", "minutes": 20 }`。超时自动销毁。 |
| `memory_limit` | enum | ✗ | `1g` / `4g` / `16g` / `64g`，默认 `1g`。 |
| `network_policy` | object | ✗ | `{ "type": "allowlist" \| "disabled", "allowed_domains": ["pypi.org", ...] }`。 |

### Container 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `cntr_...`。 |
| `object` | string | 固定 `container`。 |
| `name` | string | — |
| `status` | enum | `active` / `deleted`。 |
| `created_at` / `last_active_at` | integer | Unix 秒。 |
| `expires_after` | object | 同请求。 |
| `memory_limit` | enum | 同请求。 |
| `network_policy` | object | 同请求。 |

## 2. Container Files · /v1/containers/{container_id}/files

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/containers/{container_id}/files` |
| Retrieve | GET | `/v1/containers/{container_id}/files/{file_id}` |
| List | GET | `/v1/containers/{container_id}/files` |
| Delete | DELETE | `/v1/containers/{container_id}/files/{file_id}` |
| Retrieve content | GET | `/v1/containers/{container_id}/files/{file_id}/content` |

### Create

支持两种方式：

1. **Multipart 直接上传**：表单字段 `file` + `path`（容器内目标路径，可选）。
2. **引用 Files API 的 file**：`{"file_id":"file-...","path":"/optional/path"}`。

### ContainerFile 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `cfile_...`。 |
| `object` | string | 固定 `container.file`。 |
| `container_id` | string | 所属容器。 |
| `path` | string | 容器内绝对路径，例如 `/mnt/data/report.csv`。 |
| `source` | enum | `user`（用户上传）/ `assistant`（模型生成）。 |
| `bytes` | integer | — |
| `created_at` | integer | — |

### Retrieve content

`GET .../content` 返回字节流，`Content-Type` 与原文件 MIME 一致。

## 在 Responses 中引用

```json
{
  "model": "gpt-4.1",
  "tools": [{ "type": "code_interpreter", "container": "cntr_..." }],
  "input": "分析挂载的 report.csv 并画一张折线图"
}
```

模型在 container 内执行代码，生成的文件作为 `code_interpreter_tool_result` 输出，`file_id` 指向 ContainerFile，可再调 `GET .../content` 下载。

## 计费

按容器活跃时长 + 内存档计费；详见 [pricing.md](./pricing.md) §Code Interpreter / Containers。

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | network_policy.type 与 allowed_domains 冲突、memory_limit 越界 |
| `404` | `not_found_error` | container / file 不存在 |
| `409` | `invalid_request_error` | 对 `deleted` container 操作 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/containers>
- Code Interpreter 指南：<https://developers.openai.com/api/docs/guides/tools-code-interpreter>
