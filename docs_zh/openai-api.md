# Nanobot OpenAI 兼容 API：在 /v1/chat/completions 背后运行本地智能体

nanobot 可以为本地集成暴露一个精简的 OpenAI 兼容端点：

```bash
nanobot plugins enable api
nanobot agent -m "Hello!"
nanobot serve
```

请先运行 CLI 检查。如果 `nanobot agent -m "Hello!"` 失败，请在调试 API 服务器之前先修复提供商或配置设置。默认情况下，API 绑定到 `127.0.0.1:8900`。你可以在 `config.json` 中更改此设置。

获取设置帮助，请参阅 [`quick-start.md`](./quick-start.md)、[`providers.md`](./providers.md) 和 [`troubleshooting.md`](./troubleshooting.md)。

## 认证

仅本地 `127.0.0.1` 使用无需 API 密钥。如果你通过 `api.host: "0.0.0.0"` 或 `"::"` 将 API
服务器绑定到所有网络接口，nanobot 会要求
`api.apiKey`；否则启动会失败，以避免在网络上暴露未经身份验证的智能体
端点。

```json
{
  "api": {
    "host": "0.0.0.0",
    "port": 8900,
    "apiKey": "${NANOBOT_API_KEY}"
  }
}
```

当设置了 `api.apiKey` 时，请将其作为 Bearer 令牌发送到 API 路由。健康检查
端点保持无需认证，以便本地探针和负载均衡器仍然可以
检查进程健康状态。

```bash
curl http://127.0.0.1:8900/v1/models \
  -H "Authorization: Bearer $NANOBOT_API_KEY"
```

## 行为

- 会话隔离：在请求体中传入 `"session_id"` 来隔离对话；省略则使用共享的默认会话（`api:default`）
- 单条消息输入：每个请求必须仅包含一条 `user` 消息
- 固定模型：省略 `model`，或传入 `/v1/models` 显示的相同模型
- 流式传输：设置 `stream=true` 以接收带有 OpenAI 兼容 delta 分块的服务器发送事件（`text/event-stream`），以 `data: [DONE]` 终止；省略或设置 `stream=false` 则返回单个 JSON 响应
- **文件上传**：通过 JSON base64 或 `multipart/form-data` 支持图片、PDF、Word（.docx）、Excel（.xlsx）、PowerPoint（.pptx）（每个文件最大 10MB）
- API 请求在合成的 `api` 频道中运行，因此 `message` 工具**不会**自动投递到 Telegram/Discord/等平台。要主动发送到其他聊天，请调用 `message` 并为已启用的频道显式指定 `channel` 和 `chat_id`。

从 API 会话进行跨频道投递的工具调用示例：

```json
{
  "content": "Build finished successfully.",
  "channel": "telegram",
  "chat_id": "123456789"
}
```

如果 `channel` 指向的频道在你的配置中未启用，nanobot 会将出站事件入队，但不会发生平台投递。

## 端点

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`

## curl

```bash
curl http://127.0.0.1:8900/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "hi"}],
    "session_id": "my-session"
  }'
```

## 文件上传（JSON base64）

使用 OpenAI 多模态内容格式内联发送图片：

```bash
curl http://127.0.0.1:8900/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": [
      {"type": "text", "text": "Describe this image"},
      {"type": "image_url", "image_url": {"url": "data:image/png;base64,iVBOR..."}}
    ]}]
  }'
```

## 文件上传（multipart/form-data）

通过 multipart 上传任何受支持的文件类型（图片、PDF、Word、Excel、PPT）：

```bash
# Single file
curl http://127.0.0.1:8900/v1/chat/completions \
  -F "message=Summarize this report" \
  -F "files=@report.docx"

# Multiple files with session isolation
curl http://127.0.0.1:8900/v1/chat/completions \
  -F "message=Compare these files" \
  -F "files=@chart.png" \
  -F "files=@data.xlsx" \
  -F "session_id=my-session"
```

支持的文件类型：
- **图片**：PNG、JPEG、GIF、WebP（以 base64 形式发送给 AI 用于视觉分析）
- **文档**：PDF、Word（.docx）、Excel（.xlsx）、PowerPoint（.pptx）（提取文本并发送给 AI）
- **文本**：TXT、Markdown、CSV、JSON 等（直接读取）

## Python (`requests`)

```python
import requests

resp = requests.post(
    "http://127.0.0.1:8900/v1/chat/completions",
    json={
        "messages": [{"role": "user", "content": "hi"}],
        "session_id": "my-session",  # optional: isolate conversation
    },
    timeout=120,
)
resp.raise_for_status()
print(resp.json()["choices"][0]["message"]["content"])
```

## Python (`openai`)

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:8900/v1",
    api_key="dummy",
)

resp = client.chat.completions.create(
    model="MiniMax-M2.7",
    messages=[{"role": "user", "content": "hi"}],
    extra_body={"session_id": "my-session"},  # optional: isolate conversation
)
print(resp.choices[0].message.content)
```
