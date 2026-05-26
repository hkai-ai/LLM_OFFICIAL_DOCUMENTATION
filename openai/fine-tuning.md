---
source: https://developers.openai.com/api/reference/resources/fine_tuning
fetched_at: 2026-05-26
api_version: v1
---

# Fine-tuning · /v1/fine_tuning

监督微调（SFT）、偏好优化（DPO）与强化微调（RFT）的统一接口。所有任务都通过 `POST /v1/fine_tuning/jobs` 创建，按 `method.type` 区分。任务异步执行，可读 events 流监控、随时 cancel / pause / resume。

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create job | POST | `/v1/fine_tuning/jobs` |
| Retrieve job | GET | `/v1/fine_tuning/jobs/{fine_tuning_job_id}` |
| List jobs | GET | `/v1/fine_tuning/jobs` |
| List events | GET | `/v1/fine_tuning/jobs/{id}/events` |
| Cancel | POST | `/v1/fine_tuning/jobs/{id}/cancel` |
| Pause | POST | `/v1/fine_tuning/jobs/{id}/pause` |
| Resume | POST | `/v1/fine_tuning/jobs/{id}/resume` |
| List checkpoints | GET | `/v1/fine_tuning/jobs/{id}/checkpoints` |
| Create checkpoint permission | POST | `/v1/fine_tuning/checkpoints/{checkpoint_id}/permissions` |
| List checkpoint permissions | GET | `/v1/fine_tuning/checkpoints/{checkpoint_id}/permissions` |
| Retrieve checkpoint permission | GET | `/v1/fine_tuning/checkpoints/{checkpoint_id}/permissions/{perm_id}` |
| Delete checkpoint permission | DELETE | `/v1/fine_tuning/checkpoints/{checkpoint_id}/permissions/{perm_id}` |
| Run grader (alpha) | POST | `/v1/fine_tuning/alpha/graders/run` |
| Validate grader (alpha) | POST | `/v1/fine_tuning/alpha/graders/validate` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## Create Job · POST /v1/fine_tuning/jobs

### 请求 body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | string | ✓ | 基础模型 ID（如 `gpt-4o-mini`、`gpt-4.1`、`babbage-002`、`davinci-002`、`gpt-3.5-turbo`）。 |
| `training_file` | string | ✓ | `purpose: "fine-tune"` 上传的 file ID；JSONL 内容。 |
| `validation_file` | string | ✗ | 验证集 file ID；提供后训练过程中会跑评估。 |
| `suffix` | string | ✗ | ≤64 字符；附在 `fine_tuned_model` 名末尾，如 `ft:gpt-4o-mini:openai:custom-name:7p4lURel`。 |
| `seed` | integer | ✗ | 随机种子；同种子 + 同参数尽量复现结果，未指定时系统自动生成。 |
| `hyperparameters` | object | ✗（**deprecated**） | 已被 `method.supervised.hyperparameters` 取代；包含 `batch_size` / `learning_rate_multiplier` / `n_epochs`，三者均可设 `"auto"`。 |
| `method` | object | ✗ | 微调方法配置；见下。 |
| `integrations` | `array<Integration>` | ✗ | 训练监控集成，目前只支持 wandb。 |
| `metadata` | object | ✗ | 最多 16 个 key-value，key ≤64 字符、value ≤512 字符。 |

### method

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | enum | ✓ | `supervised` / `dpo` / `reinforcement`。 |
| `supervised.hyperparameters` | object | ✗ | `{batch_size, learning_rate_multiplier, n_epochs}`，全部可 `"auto"`。 |
| `dpo.hyperparameters` | object | ✗ | 同上额外含 `beta`（DPO 温度系数）。 |
| `reinforcement.hyperparameters` | object | ✗ | RFT 专属（如 `reasoning_effort` / `compute_multiplier` 等，详见官方）。 |
| `reinforcement.grader` | object | RFT ✓ | 评分器，type 取值：`string_check_grader` / `text_similarity_grader` / `python_grader` / `score_model_grader`。 |

### integrations[]

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `type` | string | ✓ | 目前固定 `wandb`。 |
| `wandb.project` | string | ✓ | wandb 项目名。 |
| `wandb.entity` | string | ✗ | wandb 团队 / 用户名。 |
| `wandb.name` | string | ✗ | run 名。 |
| `wandb.tags` | `array<string>` | ✗ | 标签。 |

## FineTuningJob 对象（响应）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `ftjob-...`。 |
| `object` | string | 固定 `fine_tuning.job`。 |
| `created_at` | integer | Unix 秒。 |
| `finished_at` | `integer \| null` | 终态时填写。 |
| `model` | string | 基础模型。 |
| `fine_tuned_model` | `string \| null` | 成功后的模型 ID，例如 `ft:gpt-4o-mini:openai:custom:7p4lURel`。 |
| `organization_id` | string | — |
| `result_files` | `array<string>` | 训练输出 file ID（含 metrics CSV 等）。 |
| `status` | enum | `validating_files` / `queued` / `running` / `succeeded` / `failed` / `cancelled`。 |
| `error` | `object \| null` | `{ "code", "message", "param" }`。 |
| `trained_tokens` | integer | 累计训练 token 数。 |
| `hyperparameters` | object | 实际生效（含解析后的 `auto` 数值）。 |
| `training_file` / `validation_file` | string | 同请求。 |
| `integrations` | array | 同请求。 |
| `seed` | integer | 实际使用的种子。 |
| `estimated_finish` | integer | Unix 秒估算完成时刻。 |
| `method` | object | 同请求。 |
| `metadata` | object | 同请求。 |
| `eval_id` | `string \| null` | 关联的 Evals 资源 ID（详见 [evals.md](./evals.md)）。 |

## 最小请求

```bash
curl https://api.openai.com/v1/fine_tuning/jobs \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "training_file": "file-abc123",
    "method": {
      "type": "supervised",
      "supervised": { "hyperparameters": { "n_epochs": 3 } }
    }
  }'
```

## 最小响应

```json
{
  "id": "ftjob-abc",
  "object": "fine_tuning.job",
  "created_at": 1716700000,
  "model": "gpt-4o-mini",
  "fine_tuned_model": null,
  "status": "validating_files",
  "training_file": "file-abc123",
  "validation_file": null,
  "method": { "type": "supervised", "supervised": { "hyperparameters": { "n_epochs": 3 } } }
}
```

## List jobs · GET /v1/fine_tuning/jobs

### Query

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `after` | string | — | 游标。 |
| `limit` | integer | `20` | `1`–`100`。 |
| `metadata` | object | — | 过滤 metadata 子集，例如 `metadata[key]=value`。 |

### 响应

```json
{ "object": "list", "data": [ /* FineTuningJob[] */ ], "has_more": true }
```

## List events · GET /v1/fine_tuning/jobs/{id}/events

返回 jsonl-like 事件列表，每条字段：`id` / `object: "fine_tuning.job.event"` / `created_at` / `level`（`info` / `warn` / `error`）/ `message` / `type`（`message` / `metrics`）/ `data`（指标对象，含 `step`、`train_loss`、`valid_loss` 等）。

## Cancel / Pause / Resume

| 端点 | 语义 |
| --- | --- |
| `POST /v1/fine_tuning/jobs/{id}/cancel` | 永久终止；status 走向 `cancelled`。 |
| `POST /v1/fine_tuning/jobs/{id}/pause` | 暂停（保留 checkpoint）。 |
| `POST /v1/fine_tuning/jobs/{id}/resume` | 从暂停状态恢复。 |

均返回更新后的 FineTuningJob 对象。

## Checkpoints · GET /v1/fine_tuning/jobs/{id}/checkpoints

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `ftckpt_...`。 |
| `object` | string | `fine_tuning.job.checkpoint`。 |
| `fine_tuned_model_checkpoint` | string | 可用于推理的 checkpoint model ID。 |
| `created_at` | integer | — |
| `step_number` | integer | — |
| `metrics` | object | `{ step, train_loss, train_mean_token_accuracy, valid_loss, full_valid_loss, ... }`。 |
| `fine_tuning_job_id` | string | 所属 job。 |

### Checkpoint permissions

允许把 checkpoint 共享给其他 OpenAI Organization。

| 端点 | 说明 |
| --- | --- |
| `POST /v1/fine_tuning/checkpoints/{id}/permissions` | body：`{ "project_ids": ["proj_..."] }`。返回 permission 对象数组。 |
| `GET /v1/fine_tuning/checkpoints/{id}/permissions` | 列出。 |
| `GET /v1/fine_tuning/checkpoints/{id}/permissions/{perm_id}` | 详情。 |
| `DELETE /v1/fine_tuning/checkpoints/{id}/permissions/{perm_id}` | 删除。 |

Permission 对象字段：`id` / `object: "checkpoint.permission"` / `created_at` / `project_id`。

## Alpha graders（用于 RFT 调试）

| 端点 | 说明 |
| --- | --- |
| `POST /v1/fine_tuning/alpha/graders/run` | 本地对一条样本运行 grader；body：`{ "grader": {...}, "model_sample": "...", "reference_answer": "..." }`；返回 `{ "reward": number, "metadata": {...}, "sub_rewards": {...} }`。 |
| `POST /v1/fine_tuning/alpha/graders/validate` | 仅校验 grader schema 合法性；body：`{ "grader": {...} }`。返回 `{ "grader": ... }` 或 `{ "error": ... }`。 |

### grader 子类型

| `type` | 说明 |
| --- | --- |
| `string_check_grader` | 简单字符串匹配；字段：`name` / `operation`（`eq` / `ne` / `like` / `ilike`）/ `input` / `reference`。 |
| `text_similarity_grader` | 文本相似度；字段：`name` / `evaluation_metric`（`bleu` / `rouge_1` / `rouge_2` / `rouge_3` / `rouge_4` / `rouge_5` / `rouge_l`）/ `input` / `reference`。 |
| `python_grader` | 自定义 Python；字段：`name` / `source`（Python 代码字符串）/ `image_tag`。 |
| `score_model_grader` | 用 LLM 做 grader；字段：`name` / `model` / `input`（messages）/ `range[]` / `sampling_params`。 |

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | training_file JSONL 校验失败、模型不支持、hyperparameter 越界 |
| `404` | `not_found_error` | job_id / file_id 不存在 |
| `409` | `invalid_request_error` | 在非 paused 状态调用 resume |
| `429` | `rate_limit_error` | 触发 fine-tuning 配额 |

## 参考

- API：<https://developers.openai.com/api/reference/resources/fine_tuning>
- 创建 job：<https://developers.openai.com/api/reference/resources/fine_tuning/subresources/jobs/methods/create>
- SFT 指南：<https://developers.openai.com/api/docs/guides/supervised-fine-tuning>
- RFT 指南：<https://developers.openai.com/api/docs/guides/reinforcement-fine-tuning>
- 模型优化总览：<https://developers.openai.com/api/docs/guides/model-optimization>
