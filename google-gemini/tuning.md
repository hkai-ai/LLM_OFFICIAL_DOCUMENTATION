---
source: https://ai.google.dev/api/tuning
fetched_at: 2026-05-26
api_version: v1beta
---

# Tuning · /v1beta/tunedModels

Gemini Developer API 提供基于「输入 / 输出示例」的监督微调。建模过程异步：`create` 返回 `Operation` 句柄，通过 `google.longrunning.Operations` 服务轮询进度。

> 微调资源名形如 `tunedModels/{tunedmodel}`；微调完成后可像基础模型一样在 `tunedModels/*:generateContent` 上调用。

## 鉴权

| Header | 必填 | 说明 |
| --- | --- | --- |
| `x-goog-api-key` 或 `?key=` | ✓ | API Key。 |
| `Content-Type` | 写操作 ✓ | `application/json`。 |

## 端点

### CRUD

| 动作 | METHOD | PATH | 说明 |
| --- | --- | --- | --- |
| Create | POST | `/v1beta/tunedModels` | 返回 `Operation`。 |
| Retrieve | GET | `/v1beta/{name=tunedModels/*}` | 单个查询。 |
| List | GET | `/v1beta/tunedModels` | 支持分页 `pageSize` / `pageToken`、`filter`。 |
| Update | PATCH | `/v1beta/{tunedModel.name=tunedModels/*}` | 通过 `updateMask` 指定字段；典型可更新 `displayName` / `description` / 默认采样参数。 |
| Delete | DELETE | `/v1beta/{name=tunedModels/*}` | — |
| Transfer ownership | POST | `/v1beta/{name=tunedModels/*}:transferOwnership` | body `{ "emailAddress": "..." }`；原 owner 降级为 writer。 |

### 推理（与基础模型同形）

| 端点 | PATH |
| --- | --- |
| Generate | POST `/v1beta/{model=tunedModels/*}:generateContent` |
| Stream | POST `/v1beta/{model=tunedModels/*}:streamGenerateContent` |
| Count tokens | POST `/v1beta/{model=tunedModels/*}:countTokens` |

请求 / 响应字段与 [generate-content.md](./generate-content.md) / [stream-generate-content.md](./stream-generate-content.md) / [count-tokens.md](./count-tokens.md) 一致。

## TunedModel 对象

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `name` | string | `tunedModels/{id}`，create 后赋值。 |
| `displayName` | string | 人类可读名。 |
| `description` | string | 描述。 |
| `baseModel` | string | 基础模型 ID（如 `models/gemini-2.5-flash`）。 |
| `tunedModelSource` | object | 若基于现有 tunedModel 继续微调：`{ "tunedModel": "tunedModels/...", "baseModel": "..." }`。 |
| `state` | enum | `CREATING` / `ACTIVE` / `FAILED`。 |
| `createTime` / `updateTime` | string | RFC 3339。 |
| `temperature` | number | 默认采样温度。 |
| `topP` | number | 默认 top-p。 |
| `topK` | integer | 默认 top-k。 |
| `tuningTask` | object | 见下。 |
| `readerProjectNumbers` | `array<int64>` | 可读取该模型的 GCP 项目号列表。 |

### tuningTask

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `startTime` / `completeTime` | string | RFC 3339。 |
| `snapshots` | `array<TuningSnapshot>` | 每个 checkpoint 的 `step` / `epoch` / `meanLoss` / `computeTime`。 |
| `trainingData` | object | `{ "examples": { "examples": [{ "textInput": "...", "output": "..." }, ...] } }`；最少几十条、上限以官方文档为准。 |
| `hyperparameters` | object | `{ "learningRate": number, "learningRateMultiplier": number, "batchSize": int, "epochCount": int }`，可省略走默认。 |

## Create 请求 body 示例

```json
{
  "displayName": "polite-poetry",
  "baseModel": "models/gemini-2.5-flash",
  "tuningTask": {
    "hyperparameters": { "epochCount": 5, "batchSize": 4, "learningRate": 1e-3 },
    "trainingData": {
      "examples": {
        "examples": [
          { "textInput": "Write a haiku about rain.", "output": "..." },
          { "textInput": "Write a haiku about night.", "output": "..." }
        ]
      }
    }
  }
}
```

### Create 响应（Operation）

```json
{
  "name": "tunedModels/polite-poetry-abc/operations/op-xyz",
  "metadata": {
    "@type": "type.googleapis.com/google.ai.generativelanguage.v1beta.CreateTunedModelMetadata",
    "tunedModel": "tunedModels/polite-poetry-abc",
    "totalSteps": 250,
    "completedSteps": 0
  },
  "done": false
}
```

可通过 `GET /v1beta/{name=tunedModels/*/operations/*}` 轮询；完成后 `result.response` 为 TunedModel 对象。

## 在推理时引用

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/tunedModels/polite-poetry-abc:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{ "parts": [{ "text": "Write a haiku about coffee." }] }]
  }'
```

## 限制

- 仅部分基础模型支持 Tuning（如 `gemini-2.5-flash` 子集），具体清单看 [models.md](./models.md) 与官方[微调指南](https://ai.google.dev/gemini-api/docs/model-tuning?hl=zh-cn)。
- 单 tunedModel 数据集大小、训练步数等以官方配额为准；超过会在 `state: FAILED` 中给 `error` 字段。
- Update 仅支持改默认采样参数与 displayName / description，不能改训练数据；需要新数据时基于现有 tunedModel `tunedModelSource` 重新 create。

## 参考

- 接口 reference：<https://ai.google.dev/api/tuning>
- 微调指南：<https://ai.google.dev/gemini-api/docs/model-tuning?hl=zh-cn>
- transferOwnership：<https://ai.google.dev/api/rest/v1beta/tunedModels/transferOwnership>
- 模型清单：[models.md](./models.md)
