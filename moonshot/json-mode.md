---
source: https://platform.kimi.com/docs/guide/use-json-mode-feature-of-kimi-api
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · JSON Mode

强制模型输出合法 JSON。通过 [chat-completions.md](./chat-completions.md) 的 `response_format` 字段启用。

## `response_format`

| `type` | 说明 |
| --- | --- |
| `text` | 默认，无格式约束 |
| `json_object` | 启用 JSON Mode，输出必须可被 `JSON.parse` |
| `json_schema` | 进一步约束 JSON 结构（详见官方页 §Structured Outputs） |

### 启用方式

```json
{
  "model": "kimi-k2.6",
  "messages": [
    {"role": "system", "content": "你是 JSON 数据生成助手，输出格式为 JSON Object。"},
    {"role": "user", "content": "请生成一个包含 text / image / url 三个字段的对象"}
  ],
  "response_format": {"type": "json_object"}
}
```

## 使用注意

1. **必须在 prompt 中显式描述 JSON 结构**：列出每个字段的 key 名与数据类型，附带 1–2 个示例，否则模型可能返回 schema 不一致的 JSON。
2. **只支持 JSON Object，不支持 JSON Array**：要求顶层是 array 时，请包成 `{ "items": [...] }`。
3. **`max_tokens` 要给足**：JSON Mode 输出被截断（`finish_reason: "length"`）会得到不完整 / 不合法 JSON；建议预估 token 后留出余量。
4. **检查 `finish_reason`**：仅 `stop` 才是完整输出；遇到 `length` 必须重试或扩大上限。
5. **思考模式 / 联网搜索 / partial mode** 与 JSON Mode 的组合行为以官方实际为准：
   - 与 **partial mode** 不兼容（partial 续写 assistant 消息内容，不应再强制 JSON）。
   - 与 **thinking** 可叠加；模型在 `reasoning_content` 中思考、在 `content` 中输出 JSON。
   - 与 **$web_search** 可叠加；联网检索作为输入辅助，最终 `content` 仍受 JSON 约束。

## 示例输出

```json
{
  "text": "今天天气晴朗",
  "image": "https://example.com/sunny.png",
  "url": "https://weather.example.com/today"
}
```

## 参考

- JSON Mode Guide：https://platform.kimi.com/docs/guide/use-json-mode-feature-of-kimi-api
- Chat Completions：[chat-completions.md](./chat-completions.md)
- Partial Mode（与 JSON Mode 互斥）：[partial-mode.md](./partial-mode.md)
- Thinking Mode：[thinking.md](./thinking.md)
