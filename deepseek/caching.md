---
source: https://api-docs.deepseek.com/zh-cn/guides/kv_cache
fetched_at: 2026-05-19
api_version: N/A
---

# 上下文硬盘缓存（Context Caching）

> 自动缓存请求的输入前缀，命中部分按更低单价计费。无需修改代码即可享受。

## 工作机制

- 默认全用户、全请求开启，无需开关参数或额外标记。
- 系统以请求边界（每条 user / assistant 消息的开始与结束）以及固定 token 间隔为切分点，将公共前缀切成可被复用的缓存单元。
- 当后续请求与已缓存前缀「完全匹配」时即命中缓存；同一前缀被多个请求共享。
- 缓存「best-effort」：受集群负载与缓存生命周期影响，不保证 100% 命中率。
- 缓存构建在秒级完成；长时间不使用的缓存会在数小时至若干天内自动淘汰。
- 缓存仅作用于输入。输出仍正常计算并受 `temperature` 等参数影响随机性。

## usage 中的相关字段

`chat.completion` 响应 `usage` 中提供两项：

| 字段 | 含义 |
| --- | --- |
| `prompt_cache_hit_tokens` | 本次请求 `prompt_tokens` 中命中缓存的 token 数，按缓存命中价计费。 |
| `prompt_cache_miss_tokens` | 本次请求 `prompt_tokens` 中未命中缓存的 token 数，按标准输入价计费。 |

两者之和等于 `prompt_tokens`。

## 命中条件

1. 前缀必须与已缓存内容字节级一致，包括 system prompt、历史 messages、role 字段顺序、空白与 JSON 序列化形式。
2. 在最近的活跃窗口内（一般数小时至数天），相同前缀曾被请求并被系统选中缓存。
3. 仅前缀部分命中，遇到首个不匹配的 token 起即开始未命中。

## 计费

- 命中部分按「输入命中缓存」价计费，远低于标准输入价（具体倍率以官方价格页为准）。
- 未命中部分按「输入未命中缓存」价计费。
- 输出与思维链 token 不受缓存影响。

## 实践建议

- 保持公共前缀（system prompt、长上下文文档）稳定不变。
- 把易变内容（用户最新输入）放在 messages 末尾，使前部最大化可被缓存。
- 多轮对话天然受益：每一轮的历史都是下一轮的前缀。
- 流式与非流式均支持缓存。

## 参考

- 指南：https://api-docs.deepseek.com/zh-cn/guides/kv_cache
- chat completion `usage`：[chat-completions.md](./chat-completions.md#usage)
