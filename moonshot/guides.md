---
source: https://platform.kimi.com/docs/guide/start-using-kimi-api
fetched_at: 2026-05-26
api_version: N/A
---

# Moonshot Kimi · 入门指南索引

> 把官方 `/guide/` 下所有页面归类汇总，方便对照本仓库各 endpoint md 速查"功能怎么用"。每个条目附官方原文 URL 与本仓库对应文件。

## 1. 快速开始

| 主题 | 官方 URL | 本仓库 |
| --- | --- | --- |
| 开始使用 Kimi API | https://platform.kimi.com/docs/guide/start-using-kimi-api | [README.md](./README.md) |
| 快速开始 | https://platform.kimi.com/docs/guide/quickstart | [README.md](./README.md) |
| API 快速开始（控制台） | https://platform.kimi.com/docs/api/quickstart | [README.md](./README.md) |
| K2.6 快速开始 | https://platform.kimi.com/docs/guide/kimi-k2-6-quickstart | [thinking.md](./thinking.md) |
| 从 OpenAI 迁移 | https://platform.kimi.com/docs/guide/migrating-from-openai-to-openai | [README.md §协议兼容](./README.md) |

## 2. Chat 能力

| 主题 | 官方 URL | 本仓库 |
| --- | --- | --- |
| 流式输出 | https://platform.kimi.com/docs/guide/utilize-the-streaming-output-feature-of-kimi-api | [chat-completions.md §流式](./chat-completions.md) |
| 多轮对话实现 | https://platform.kimi.com/docs/guide/engage-in-multi-turn-conversations-using-kimi-api | [chat-completions.md](./chat-completions.md) |
| JSON Mode | https://platform.kimi.com/docs/guide/use-json-mode-feature-of-kimi-api | [json-mode.md](./json-mode.md) |
| Partial Mode（assistant 续写） | https://platform.kimi.com/docs/guide/use-partial-mode-feature-of-kimi-api | [partial-mode.md](./partial-mode.md) |
| 思考模式 | https://platform.kimi.com/docs/guide/use-kimi-k2-thinking-model | [thinking.md](./thinking.md) |
| 视觉模型 | https://platform.kimi.com/docs/guide/use-kimi-vision-model | [vision.md](./vision.md) |
| 工具调用 | https://platform.kimi.com/docs/guide/use-kimi-api-to-complete-tool-calls | [tool-use.md](./tool-use.md) |
| 联网搜索 | https://platform.kimi.com/docs/guide/use-web-search | [web-search.md](./web-search.md) |
| 官方工具集成 | https://platform.kimi.com/docs/guide/use-official-tools | [tool-use.md](./tool-use.md) |
| 文件问答 | https://platform.kimi.com/docs/guide/use-kimi-api-for-file-based-qa | [files.md](./files.md) |
| 自动断线重连 | https://platform.kimi.com/docs/guide/auto-reconnect | [chat-completions.md](./chat-completions.md) |

## 3. 批量与文件

| 主题 | 官方 URL | 本仓库 |
| --- | --- | --- |
| 批量任务（推荐入口） | https://platform.kimi.com/docs/guide/use-batch-api | [batch.md](./batch.md) |
| 批量推理控制台 | https://platform.kimi.com/docs/guide/use-batch-inference | [batch.md](./batch.md) |

## 4. 编程工具与集成

| 主题 | 官方 URL | 备注 |
| --- | --- | --- |
| Kimi CLI | https://platform.kimi.com/docs/guide/kimi-cli-support | 终端命令行工具 |
| MoonPalace 调试 | https://platform.kimi.com/docs/guide/use-moonpalace | 抓包与请求重放 |
| Playground | https://platform.kimi.com/docs/guide/use-playground-to-debug-the-model | 浏览器调试 |
| OpenClaw 集成 | https://platform.kimi.com/docs/guide/use-kimi-in-openclaw | 浏览器自动化 |
| ModelScope MCP | https://platform.kimi.com/docs/guide/configure-the-modelscope-mcp-server | MCP server 配置 |
| 编程工具集成 | https://platform.kimi.com/docs/guide/agent-support | Cursor / Cline / Continue / 其他 IDE |

## 5. 最佳实践

| 主题 | 官方 URL | 备注 |
| --- | --- | --- |
| Prompt 最佳实践 | https://platform.kimi.com/docs/guide/prompt-best-practice | — |
| Benchmark 最佳实践 | https://platform.kimi.com/docs/guide/benchmark-best-practice | — |
| 组织认证最佳实践 | https://platform.kimi.com/docs/guide/org-best-practice | 企业用户开通 |

## 6. 排查与 FAQ

| 主题 | 官方 URL | 本仓库 |
| --- | --- | --- |
| 常见问题 | https://platform.kimi.com/docs/guide/faq | [errors.md](./errors.md) §排查 |
| 错误说明 | https://platform.kimi.com/docs/api/errors | [errors.md](./errors.md) |
| 定价 FAQ | https://platform.kimi.com/docs/pricing/faq | [pricing.md](./pricing.md) |
| 充值与限速 | https://platform.kimi.com/docs/pricing/limits | [rate-limits.md](./rate-limits.md) |

## 7. 协议与服务条款

| 主题 | 官方 URL |
| --- | --- |
| 服务协议 | https://platform.kimi.com/docs/agreement/modeluse |
| 充值协议 | https://platform.kimi.com/docs/agreement/payment |
| 用户服务协议 | https://platform.kimi.com/docs/agreement/userservice |
| 隐私政策 | https://platform.kimi.com/docs/agreement/userprivacy |

## 维护说明

- 文档总索引：https://platform.kimi.com/docs/llms.txt
- 本入门索引随官方 `/guide/` 路径变动同步刷新；新增的 guide 在抓 `llms.txt` 时优先加到这里，再视需要展开为独立 md。
- 详细 endpoint 字段以仓库各 md 为准（带 frontmatter 标注 `fetched_at`）。
