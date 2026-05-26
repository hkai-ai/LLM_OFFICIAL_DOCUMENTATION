---
source: https://platform.kimi.com/docs/api/partial
fetched_at: 2026-05-20
api_version: N/A
---

# Partial Mode（前缀续写）

在 `/v1/chat/completions` 的最后一条 `assistant` 消息内附 `"partial": true`，模型将以该消息的 `content` 作为输出**前缀**继续生成。响应中不会重复返回该前缀，调用方需自行拼接。

## 字段

| 字段 | 位置 | 类型 | 说明 |
| --- | --- | --- | --- |
| `partial` | `messages[].assistant` | boolean | 仅可写在 `messages` 数组**最后一条** assistant 消息中。`true` 启用 Partial Mode。 |
| `content` | 同上 | string | 想让模型续写的前缀文本。 |
| `name` | 同上 | string | 可选；角色扮演时用于强化人设。 |

## 限制

- **必须是最后一条消息**且 `role` 必须为 `assistant`，否则被忽略或报错。
- **不可与 `response_format.type: "json_object"` 组合**：组合行为未定义，结果不可预期；如需 JSON 结构化输出，使用 `response_format.type: "json_schema"` 或不开 Partial Mode。
- **前缀不计入响应 content**：客户端拼回完整文本时需要自己手工 `prefix + delta`。

## 典型用法

### 引导 JSON 结构

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "user", "content": "给我一段人物简介，返回 JSON：{name, age, bio}"},
    {"role": "assistant", "content": "{", "partial": true}
  ]
}
```

返回 `content` 形如 `"name": "..."  ...}`；最终结构 = `"{" + content`。

### 角色扮演锁定语气

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "user", "content": "继续讲故事。"},
    {"role": "assistant", "name": "narrator", "content": "在那个雪夜，", "partial": true}
  ]
}
```

## 参考

- 端点：https://platform.kimi.com/docs/api/partial
- 指南：https://platform.kimi.com/docs/guide/use-partial-mode-feature-of-kimi-api
