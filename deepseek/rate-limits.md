---
source: https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit
fetched_at: 2026-05-19
api_version: N/A
---

# 限速策略

DeepSeek API 不公开固定的 TPM / RPM 数值，而是基于实时负载进行**动态限流**。

## 关键行为

- **动态限流**：DeepSeek API 会根据负载情况，动态限制用户并发量。
- **触发限速**：达到上限时立即返回 HTTP `429`。
- **无静态配额**：文档未公开静态 TPM / RPM 与并发数；不同时段、不同模型可能不同。
- **超时**：如果 10 分钟内服务器仍未开始推理，连接会被关闭。

## 等待期间的占位响应

为防止网络中间件超时关闭连接，服务器在排队等待推理时会发送占位数据：

| 模式 | 占位形式 | 说明 |
| --- | --- | --- |
| 非流式 | 多个空行 | 客户端 JSON 解析器需先去掉前导空白后再 parse。 |
| 流式（SSE） | `: keep-alive` 注释行 | SSE 规范规定 `:` 开头为注释，正常 SDK 会忽略。 |

> 这些占位符不属于最终响应；最终响应仍是一段完整 JSON（非流式）或以 `data: [DONE]` 收尾的 SSE 流（流式）。

## 处理 429 的建议

- 退避重试：建议指数退避，从 1s 起，最多重试 5 次。
- 控制并发：客户端侧限制同模型并发请求数。
- 拆分请求：长上下文拆分，或将高频小请求合并为少量大请求。
- 缓存命中：尽量复用相同前缀，使大部分 token 走低价命中路径（见 [caching.md](./caching.md)），降低 TPM 占用。

## 与计费的关系

限速与计费独立：未命中缓存的输入与思维链 token 均计入计费量；处于 `429` 状态的请求不计费（请求被拒绝）。

## 参考

- 官方限速页：https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit
- 错误码：https://api-docs.deepseek.com/zh-cn/quick_start/error_codes
