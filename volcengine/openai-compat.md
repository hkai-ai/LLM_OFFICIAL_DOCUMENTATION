---
source: https://www.volcengine.com/docs/82379/1330626
fetched_at: 2026-05-28
api_version: v3（路径前缀 /api/v3）
---

# 兼容 OpenAI SDK

火山方舟数据面 API 尽可能兼容 OpenAI API：仅修改 `base_url`、`model`、`api_key` 即可将方舟模型服务接入已有系统。

## 接入要点

| 配置 | 值 |
| --- | --- |
| `base_url` | `https://ark.cn-beijing.volces.com/api/v3` |
| `api_key` | 方舟 API Key（`$ARK_API_KEY`） |
| `model` | Model ID 或 Endpoint ID（如 `doubao-seed-1-6-251015`） |

- OpenAI SDK 需 `>=1.0`（Python ≥3.7）。请求 / 响应字段、SSE 流式格式与 OpenAI Chat Completions 一致。
- 通过 `GET /api/v3/models` 列模型行为见各 SDK；模型清单见 [models.md](./models.md)。

## 传入方舟特有字段：`extra_body`

OpenAI SDK 不认识的字段（如 `thinking`、`reasoning_effort` 等）通过 `extra_body` 透传到请求体：

```python
from openai import OpenAI
import os

client = OpenAI(
    base_url="https://ark.cn-beijing.volces.com/api/v3",
    api_key=os.environ.get("ARK_API_KEY"),
)

completion = client.chat.completions.create(
    model="doubao-seed-1-6-251015",
    messages=[{"role": "user", "content": "Hello"}],
    extra_body={
        "thinking": {"type": "disabled"}  # 或 {"type": "enabled"} 开启深度思考
    },
)
print(completion.choices[0].message.content)
```

## 自定义请求头：`extra_headers`

用于传递配置 ID 串联日志、启用数据加密等：

```python
completion = client.chat.completions.create(
    model="doubao-seed-1-6-251015",
    messages=[{"role": "user", "content": "Hello"}],
    extra_headers={"X-Client-Request-Id": "202406251728190000B7EA7A9648AC08D9"},
)
```

## 兼容性边界

- **多模态向量化**（`doubao-embedding-vision`）**不支持** OpenAI API，需使用方舟 SDK（`volcengine-python-sdk[ark]`），见 [embeddings.md](./embeddings.md)。文本向量化模型已逐步下线。
- **Responses API**、**视频生成**、**上下文缓存**等方舟特有端点不在 OpenAI Chat Completions 协议范围内，需用方舟 SDK 或直接 HTTP 调用。
- **LangChain**：可用 `langchain-openai` 的 `ChatOpenAI`，设置 `openai_api_base` 与 `openai_api_key` 即可。

> 社区第三方 SDK 不由火山引擎团队维护，仅供参考。

## 参考

- 兼容 OpenAI SDK：https://www.volcengine.com/docs/82379/1330626
- 安装及升级 SDK：https://www.volcengine.com/docs/82379/1541595
- 对话(Chat) API：https://www.volcengine.com/docs/82379/1494384
