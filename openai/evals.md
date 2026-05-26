---
source: https://developers.openai.com/api/reference/resources/evals
fetched_at: 2026-05-26
api_version: v1
---

# Evals · /v1/evals

定义评测集 + grader，针对某个模型 / Responses 配置批量跑评测；典型用于上线前模型回归、Fine-tuning 调参的指标基线、RFT 的 reward 调试。

| 资源 | 路径前缀 |
| --- | --- |
| Evals | `/v1/evals` |
| Runs | `/v1/evals/{eval_id}/runs` |
| Output items | `/v1/evals/{eval_id}/runs/{run_id}/output_items` |

## 鉴权与请求头

| Header | 必填 | 说明 |
| --- | --- | --- |
| `Authorization` | ✓ | `Bearer $OPENAI_API_KEY`。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 1. Evals CRUD

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/evals` |
| Retrieve | GET | `/v1/evals/{eval_id}` |
| Update | POST | `/v1/evals/{eval_id}` |
| Delete | DELETE | `/v1/evals/{eval_id}` |
| List | GET | `/v1/evals` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✗ | 人类可读名。 |
| `data_source_config` | object | ✓ | 评测样本的数据形态；type 决定字段，见下。 |
| `testing_criteria` | `array<Grader>` | ✓ | 评分标准；每个 grader 对每条样本独立打分。 |
| `metadata` | object | ✗ | KV。 |

### data_source_config 三种 type

| `type` | 说明 |
| --- | --- |
| `custom` | 自定义 JSONL，`item_schema` 描述每条记录的 JSON Schema；可选 `include_sample_schema: true` 让 run 时模型采样也带 schema。 |
| `stored_completions` | （**deprecated**）从已有 chat completions 抽取。 |
| `logs` | 从 Responses / Chat logs 取样，通过 `metadata` 过滤。 |

### testing_criteria · grader

每条 testing criterion 是一个 grader 对象，`type` 取以下之一：

| `type` | 关键字段 |
| --- | --- |
| `label_model` | 用 LLM 判定离散标签：`name`, `model`, `input`（messages，可引用 `{{sample.output_text}}` 等模板）, `labels[]`, `passing_labels[]`。 |
| `string_check` | `name`, `operation`（`eq`/`ne`/`like`/`ilike`）, `input`, `reference`。 |
| `text_similarity` | `name`, `evaluation_metric`（`bleu` / `rouge_*` 等）, `input`, `reference`, `pass_threshold`。 |
| `python` | `name`, `source`（Python 代码字符串）, `image_tag`, `pass_threshold`。 |
| `score_model` | `name`, `model`, `input`, `range[]`, `pass_threshold`, `sampling_params`。 |

### Eval 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `eval_...`。 |
| `object` | string | 固定 `eval`。 |
| `name` | string | — |
| `data_source_config` | object | 同请求。 |
| `testing_criteria` | array | 同请求。 |
| `metadata` | object | — |
| `created_at` | integer | Unix 秒。 |

## 2. Runs · /v1/evals/{eval_id}/runs

### 端点

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Create | POST | `/v1/evals/{eval_id}/runs` |
| Retrieve | GET | `/v1/evals/{eval_id}/runs/{run_id}` |
| List | GET | `/v1/evals/{eval_id}/runs` |
| Delete | DELETE | `/v1/evals/{eval_id}/runs/{run_id}` |
| Cancel | POST | `/v1/evals/{eval_id}/runs/{run_id}/cancel` |

### Create body

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `name` | string | ✗ | — |
| `data_source` | object | ✓ | 实际数据源 + 采样配置，与 eval 的 `data_source_config.type` 必须一致；type 字段：`completions` / `responses` / `jsonl`。 |
| `metadata` | object | ✗ | KV。 |

`data_source` 关键子字段：

| 字段 | 说明 |
| --- | --- |
| `type` | `completions` / `responses` / `jsonl`。 |
| `model` | 用于采样的模型 ID。 |
| `input_messages` / `input` | 模板化 prompt，可引用 `{{item.field}}`。 |
| `sampling_params` | `{ temperature, top_p, max_tokens, seed }` 等。 |
| `source` | 嵌套对象：`{ "type": "file_id", "id": "file-..." }` / `{ "type": "file_content", "content": [...]}` / `{ "type": "stored_completions"|"responses", "metadata": {...} }`。 |

### Run 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `evalrun_...`。 |
| `object` | string | 固定 `eval.run`。 |
| `eval_id` | string | — |
| `status` | enum | `queued` / `in_progress` / `completed` / `failed` / `canceled`。 |
| `model` | string | 实际使用的模型。 |
| `name` | string | — |
| `data_source` | object | 同请求。 |
| `result_counts` | object | `{ total, errored, failed, passed }`。 |
| `per_testing_criteria_results` | array | 每个 criterion 的 `{ testing_criteria, passed, failed }`。 |
| `report_url` | string | Console 中可视化报告 URL。 |
| `error` | object \| null | — |
| `created_at` | integer | — |

## 3. Output Items · /v1/evals/{eval_id}/runs/{run_id}/output_items

| 动作 | METHOD | PATH |
| --- | --- | --- |
| Retrieve | GET | `.../output_items/{output_item_id}` |
| List | GET | `.../output_items` |

每个 output item 对应数据集中的一条样本，字段：`id` / `object: "eval.run.output_item"` / `created_at` / `run_id` / `eval_id` / `status` / `datasource_item`（原始输入）/ `sample`（模型采样结果，含 `usage`） / `results[]`（每条 grader 结果：`name` / `score` / `passed`）。

## 最小请求（创建 + 跑 run）

```bash
# 1. 创建 eval
curl https://api.openai.com/v1/evals \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "qa-baseline",
    "data_source_config": {
      "type": "custom",
      "item_schema": {
        "type": "object",
        "properties": { "question": {"type":"string"}, "answer": {"type":"string"} },
        "required": ["question","answer"]
      }
    },
    "testing_criteria": [{
      "type": "string_check",
      "name": "exact-match",
      "operation": "eq",
      "input": "{{sample.output_text}}",
      "reference": "{{item.answer}}"
    }]
  }'

# 2. 起 run
curl https://api.openai.com/v1/evals/$EVAL_ID/runs \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "run-1",
    "data_source": {
      "type": "completions",
      "model": "gpt-4.1",
      "input_messages": { "template": [{ "role": "user", "content": "{{item.question}}" }] },
      "source": { "type": "file_id", "id": "file-abc" }
    }
  }'
```

## 错误码

| HTTP | `error.type` | 触发 |
| --- | --- | --- |
| `400` | `invalid_request_error` | data_source 与 data_source_config 不匹配、grader 模板里引用了不存在的字段 |
| `404` | `not_found_error` | eval / run / output_item 不存在 |
| `409` | `invalid_request_error` | 对已完成 run 调用 cancel |

## 参考

- API：<https://developers.openai.com/api/reference/resources/evals>
- 指南：<https://developers.openai.com/api/docs/guides/evals>
- Fine-tuning 关联：[fine-tuning.md](./fine-tuning.md)
