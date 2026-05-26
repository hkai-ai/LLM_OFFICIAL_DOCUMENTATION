---
source: https://developers.openai.com/api/reference/resources/uploads
fetched_at: 2026-05-26
api_version: v1
---

# Uploads · /v1/uploads

大文件分片上传协议；单次 `POST /v1/files` 限 512 MB，需要更大或断点续传时走 Uploads 流程：

1. `POST /v1/uploads`：声明文件大小与 purpose，拿到 `upload_id`。
2. 逐片 `POST /v1/uploads/{upload_id}/parts` 上传（每片可 ≤64 MB）。
3. `POST /v1/uploads/{upload_id}/complete`：按 part_ids 顺序拼接、可选 md5 校验，得到最终 File 对象（同 Files API 返回）。
4. 出错或弃用：`POST /v1/uploads/{upload_id}/cancel`。

> 单次 Upload 总大小上限 **8 GB**；upload 在创建后约 1 小时过期。

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | JSON 接口为 `application/json`；上传 part 走 `multipart/form-data`。 |

## 1. Create upload · POST /v1/uploads

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `filename` | string | ✓ | 文件原名。 |
| `purpose` | string | ✓ | 与 Files API 一致：`assistants` / `batch` / `fine-tune` / `vision` / `user_data` 等。 |
| `bytes` | integer | ✓ | 文件总字节数；上传完成后需匹配。 |
| `mime_type` | string | ✓ | MIME。 |

### Upload 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `upload_...`。 |
| `object` | string | 固定 `upload`。 |
| `created_at` | integer | Unix 秒。 |
| `expires_at` | integer | Unix 秒，通常为 created_at + 3600。 |
| `filename` | string | 同请求。 |
| `bytes` | integer | 同请求。 |
| `purpose` | string | 同请求。 |
| `status` | enum | `pending` / `completed` / `cancelled` / `expired`。 |
| `file` | object \| null | 完成后填入完整 File 对象（与 Files API 返回字段一致）。 |

## 2. Add upload part · POST /v1/uploads/{upload_id}/parts

`multipart/form-data` 表单：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `data` | ✓ | 当前片字节流；单片 `≤64 MB`。 |

返回：

```json
{
  "id": "upart_abc",
  "object": "upload.part",
  "upload_id": "upload_...",
  "created_at": 1716700000
}
```

## 3. Complete upload · POST /v1/uploads/{upload_id}/complete

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `part_ids` | `array<string>` | ✓ | 按拼接顺序排列的 part ID。 |
| `md5` | string | ✗ | 文件 md5 hex，用于服务端校验；不一致返回 `400`。 |

完成后 status 转为 `completed`，`file` 字段填入；可直接当作 `file_id` 提交给 Batches / Fine-tuning / Assistants 等下游接口。

## 4. Cancel upload · POST /v1/uploads/{upload_id}/cancel

将 status 设为 `cancelled`；已上传的 parts 自动清理。

## 完整示例

```bash
# 1) 创建
UPLOAD=$(curl https://api.openai.com/v1/uploads \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"filename":"big.jsonl","purpose":"batch","bytes":2147483648,"mime_type":"application/jsonl"}')
UID=$(echo $UPLOAD | jq -r .id)

# 2) 分片
PARTS=()
for chunk in chunks/*.bin; do
  P=$(curl https://api.openai.com/v1/uploads/$UID/parts \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -F "data=@$chunk" | jq -r .id)
  PARTS+=($P)
done

# 3) 完成
curl https://api.openai.com/v1/uploads/$UID/complete \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"part_ids\": $(printf '"%s",' ${PARTS[@]} | sed 's/,$//' | awk '{print "["$0"]"}')}"
```

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | `bytes` 与实际累计 part 字节不一致 / md5 校验失败 / part 顺序错乱 |
| `404` | `not_found_error` | upload_id 不存在或已过期 |
| `409` | `invalid_request_error` | 在 `completed` / `cancelled` 状态再次 complete |

## 参考

- API：<https://developers.openai.com/api/reference/resources/uploads>
- Files API：[files-and-batches.md](./files-and-batches.md)
