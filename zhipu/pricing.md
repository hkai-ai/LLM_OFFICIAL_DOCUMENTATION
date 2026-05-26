---
source: https://www.bigmodel.cn/pricing
fetched_at: 2026-05-26
api_version: paas/v4
---

# 智谱 BigModel · 计费要点

> 智谱平台的具体单价以官方[计费页](https://www.bigmodel.cn/pricing)为准；目前本仓库**只汇总规则与字段**，不冻结具体数字（因为智谱定价随订阅 / 套餐 / 模型版本调整较频繁，且部分模型按字数 / 张 / 秒分别计价）。
>
> 接下来同步本文档时，请按 SKILL `zhipu-api-docs` §定价文档维护 的清单逐项核对，并把官方表格转写到本文（含货币单位 `人民币 元 / 1M tokens`）。

## 计费维度

### 文本 / 视觉 / 思考模型（chat）

按 input / output token 计费，二者价格不同。
- 思考产生的 `reasoning_content` 计入 **output**。
- 上下文缓存命中部分按"缓存命中价"计费，约为标准 input 价的 **50%**（具体折扣以官方[缓存指南](https://docs.bigmodel.cn/cn/guide/capabilities/cache)和计费页为准）；命中 token 反映在 `usage.prompt_tokens_details.cached_tokens`。
- 工具调用（function calling）token 计入 prompt；内置 `web_search` 工具按"次"另收费。

### 嵌入模型（embedding）

按 input token 计费，单价较低；`embedding-3` 不同 `dimensions` 价格相同。

### 图像生成

按"张"计费；`quality="hd"` 单价高于 `standard`。

### 视频生成

按"秒"计费（与 model × resolution × quality 组合相关）。Vidu / CogVideoX 子型号价格不同。

### 语音 ASR

按"分钟"计费。

### 语音 TTS

按"千字符"计费。

### 音色复刻（voice clone）

按"个音色 / 月"或"试听字符数"两种模式之一计费，具体见官方。

### 工具 API

- Web Search：按"次"。`search_pro` / `search_pro_sogou` / `search_pro_quark` 单价高于 `search_std`。
- 文件解析：按"页"或"文件"，按 `tool_type`（`lite` / `expert` / `prime`）档位不同价。
- OCR：按"张"。
- 内容安全：按"次"。

### 批处理（Batch API）

`POST /paas/v4/batches` 享受批量折扣（官方约 50%），具体折扣以计费页为准；详见 [batch.md](./batch.md)。

### 知识库

按"字数"独立计费（与对话 token 分离），详见 https://docs.bigmodel.cn/cn/guide/tools/knowledge/price 。

## 免费模型

下列模型按官方政策**完全免费**（仍受限速 / 用量上限限制）：

- 文本：`glm-4-flash-250414` / `glm-4.7-flash` / `glm-4.5-flash`
- 视觉：`glm-4v-flash` / `glm-4.6v-flash` / `glm-4.1v-thinking-flash`
- 图像：`cogview-3-flash`
- 视频：`cogvideox-flash`

## 套餐订阅

智谱另提供 **GLM Coding Plan**、**Token Plan** 等订阅档位（详见官方）。订阅相关错误码：

- `1309` GLM Coding Plan 套餐已到期
- `1310` 已达周 / 月使用上限
- `1311` 订阅套餐未开放模型权限

## 同步建议

下次刷新本文档时建议：

1. 抓 https://www.bigmodel.cn/pricing 完整价格表，按模型 × 单位整理；
2. 抓 https://docs.bigmodel.cn/cn/guide/tools/knowledge/price 同步知识库价格；
3. 同步 `pricing.md` frontmatter 的 `fetched_at`，更新 [README.md](./README.md) 的 `last_updated`。

## 参考

- 计费总览：https://www.bigmodel.cn/pricing
- 上下文缓存：https://docs.bigmodel.cn/cn/guide/capabilities/cache
- 知识库价格：https://docs.bigmodel.cn/cn/guide/tools/knowledge/price
- 费用 FAQ：https://docs.bigmodel.cn/cn/faq/fee-issues
