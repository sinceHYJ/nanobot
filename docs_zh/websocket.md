# WebSocket 服务器频道

nanobot 可以作为 WebSocket 服务器运行，允许外部客户端（Web 应用、CLI、脚本）通过持久连接与智能体进行实时交互。

## 特性

- 基于 WebSocket 的双向实时通信
- 支持流式传输 —— 逐 token 接收智能体响应
- 基于 token 的身份验证（静态 token 和短期签发 token）
- 多聊天多路复用 —— 单个连接可以运行多个并发的 `chat_id`
- 支持 TLS/SSL（WSS），强制最低 TLSv1.2
- 通过 `allowFrom` 实现客户端白名单
- 自动清理失效连接

## 快速开始

### 1. 配置

WebSocket 频道默认启用。只需在 `channels.websocket` 下添加你要覆盖的字段：

```json
{
  "channels": {
    "websocket": {
      "host": "127.0.0.1",
      "port": 8765,
      "path": "/",
      "tokenIssueSecret": "your-webui-password",
      "websocketRequiresToken": true,
      "allowFrom": ["*"],
      "streaming": true
    }
  }
}
```

### 2. 启动 nanobot

```bash
nanobot gateway
```

你应该会看到：

```text
WebSocket server listening on ws://127.0.0.1:8765/
```

### 3. 连接客户端

```bash
# Using websocat
websocat ws://127.0.0.1:8765/?client_id=alice

# Using Python
import asyncio, json, websockets

async def main():
    async with websockets.connect("ws://127.0.0.1:8765/?client_id=alice") as ws:
        ready = json.loads(await ws.recv())
        print(ready)  # {"event": "ready", "chat_id": "...", "client_id": "alice"}
        await ws.send(json.dumps({"content": "Hello nanobot!"}))
        reply = json.loads(await ws.recv())
        print(reply["text"])

asyncio.run(main())
```

## 连接 URL

```text
ws://{host}:{port}{path}?client_id={id}&token={token}
```

| 参数 | 是否必填 | 描述 |
|-----------|----------|-------------|
| `client_id` | 否 | 用于 `allowFrom` 授权的标识符。省略时自动生成为 `anon-xxxxxxxxxxxx`。截断至 128 个字符。 |
| `token` | 条件必填 | 认证 token。当 `websocketRequiresToken` 为 `true` 或已配置 `token`（静态密钥）时必填。 |

## 通信协议

所有帧均为 JSON 文本。每条消息都有一个 `event` 字段。

### 服务器 → 客户端

**`ready`** —— 连接建立后立即发送：

```json
{
  "event": "ready",
  "chat_id": "uuid-v4",
  "client_id": "alice"
}
```

**`message`** —— 完整的智能体响应：

```json
{
  "event": "message",
  "chat_id": "uuid-v4",
  "text": "Hello! How can I help?",
  "media": ["/tmp/image.png"],
  "reply_to": "msg-id"
}
```

`media` 和 `reply_to` 仅在适用时出现。

**`delta`** —— 流式文本片段（仅当 `streaming: true` 时）：

```json
{
  "event": "delta",
  "chat_id": "uuid-v4",
  "text": "Hello",
  "stream_id": "s1"
}
```

**`stream_end`** —— 表示流式片段的结束：

```json
{
  "event": "stream_end",
  "chat_id": "uuid-v4",
  "stream_id": "s1"
}
```

**`reasoning_delta`** —— 当前助手轮次的增量模型推理/思考片段。与 `delta` 类似，但指向答案主体上方的推理气泡，而不是答案主体本身：

```json
{
  "event": "reasoning_delta",
  "chat_id": "uuid-v4",
  "text": "Let me decompose ",
  "stream_id": "r1"
}
```

**`reasoning_end`** —— 当前推理流的结束标记。WebUI 使用该标记锁定原位气泡，并将闪烁的表头切换为静态折叠状态：

```json
{
  "event": "reasoning_end",
  "chat_id": "uuid-v4",
  "stream_id": "r1"
}
```

只有当频道的 `showReasoning` 为 `true`（默认值）且模型返回推理内容（DeepSeek-R1 / Kimi / MiMo / OpenAI 推理模型、Anthropic extended thinking，或内联 `<think>` / `<thought>` 标签）时，才会推送推理帧。不具备推理能力的模型不会产生任何 `reasoning_delta` 帧。

**`runtime_model_updated`** —— 当网关运行时模型发生变更时广播，例如在执行 `/model <preset>` 之后：

```json
{
  "event": "runtime_model_updated",
  "model_name": "openai/gpt-4.1-mini",
  "model_preset": "fast"
}
```

未激活命名预设时，`model_preset` 字段会被省略。WebUI 客户端使用该事件在斜杠命令、配置重载和设置变更时保持所显示的模型徽章同步。

**`attached`** —— 对 `new_chat` / `attach` 入站信封的确认（参见 [多聊天多路复用](#multi-chat-multiplexing)）：

```json
{"event": "attached", "chat_id": "uuid-v4"}
```

**`error`** —— 针对格式错误的入站信封的软错误。连接保持打开：

```json
{"event": "error", "detail": "invalid chat_id"}
```

### 客户端 → 服务器

**旧版（默认聊天）：** 发送纯字符串，或者带有可识别文本字段的 JSON 对象：

```json
"Hello nanobot!"
```

```json
{"content": "Hello nanobot!"}
```

可识别的字段：`content`、`text`、`message`（按此顺序检查）。无效的 JSON 会被当作纯文本处理。这些帧会路由到该连接的默认 `chat_id`（即在 `ready` 中公布的那一个）。

**类型化信封（多聊天）：** 任何带有字符串 `type` 字段的 JSON 对象都是一个类型化信封：

| `type` | 字段 | 效果 |
|--------|--------|--------|
| `new_chat` | — | 服务器铸造一个新的 `chat_id`，将此连接订阅到该 chat，并回复 `attached`。 |
| `attach` | `chat_id` | 订阅已存在的 `chat_id`（例如页面重载后）。回复 `attached`。 |
| `message` | `chat_id`, `content` | 在 `chat_id` 上发送 `content`。首次使用会自动附加；无需显式 `attach`。 |

完整流程见 [多聊天多路复用](#multi-chat-multiplexing)。

## 配置参考

所有字段都位于 `config.json` 中的 `channels.websocket` 下。

### 连接

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `enabled` | bool | `true` | 启用 WebSocket 服务器。仅在你明确不希望使用捆绑的 WebUI/WebSocket 界面时设为 `false`。 |
| `host` | string | `"127.0.0.1"` | 绑定地址。使用 `"0.0.0.0"` 以接受外部连接。 |
| `port` | int | `8765` | 监听端口。 |
| `path` | string | `"/"` | WebSocket 升级路径。尾部斜杠会被规范化（根路径 `/` 会被保留）。 |
| `maxMessageBytes` | int | `37748736` | 入站消息的最大字节数（1 KB – 40 MB）。默认值（36 MB）按容纳最多 4 个 base64 编码图像附件（每个 8 MB）设计；如果该频道只承载文本，可以调低。 |

### 身份验证

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `token` | string | `""` | 静态共享密钥。设置后，客户端必须提供匹配的 `?token=<value>`（使用时间安全比较）。签发的 token 也会作为回退被接受。 |
| `websocketRequiresToken` | bool | `true` | 当为 `true` 且未配置静态 `token` 时，客户端仍必须出示有效的签发 token。设为 `false` 可允许未认证连接（仅在本地/受信任的网络下安全）。 |
| `tokenIssuePath` | string | `""` | 签发短期 token 的 HTTP 路径。必须与 `path` 不同。参见 [Token 签发](#token-issuance)。 |
| `tokenIssueSecret` | string | `""` | 通过签发端点获取 token 所需的密钥。为空时，任何客户端都可以从 `tokenIssuePath` 获取 WebSocket 连接 token（会记录为警告）。`/webui/bootstrap` 仍会向同机 localhost 浏览器请求签发 WebUI REST API token；远程或转发的 bootstrap 请求则需要 `tokenIssueSecret` 或 `token`。 |
| `tokenTtlS` | int | `300` | 签发 token 的存活时间（秒），范围 30 – 86,400。 |

### 访问控制

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `allowFrom` | list of string | `["*"]` | 允许的 `client_id` 值。`"*"` 允许全部；`[]` 拒绝全部。 |

### 流式传输

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `streaming` | bool | `true` | 启用流式模式。智能体将发送 `delta` + `stream_end` 帧，而非单个 `message`。 |

### 保活

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `pingIntervalS` | float | `20.0` | WebSocket ping 间隔（秒），范围 5 – 300。 |
| `pingTimeoutS` | float | `20.0` | 等待 pong 后关闭连接的时长（秒），范围 5 – 300。 |

### TLS/SSL

| 字段 | 类型 | 默认值 | 描述 |
|-------|------|---------|-------------|
| `sslCertfile` | string | `""` | TLS 证书文件路径（PEM）。启用 WSS 需同时设置 `sslCertfile` 和 `sslKeyfile`。 |
| `sslKeyfile` | string | `""` | TLS 私钥文件路径（PEM）。强制的最低 TLS 版本为 TLSv1.2。 |

## Token 签发

对于 `websocketRequiresToken: true` 的生产部署，请使用短期 token，而不是在客户端中嵌入静态密钥。

### 工作原理

1. 客户端携带 `Authorization: Bearer {tokenIssueSecret}`（或 `X-Nanobot-Auth` 头）发送 `GET {tokenIssuePath}`。
2. 服务器返回一个一次性使用的 token：

```json
{"token": "nbwt_aBcDeFg...", "expires_in": 300}
```

3. 客户端使用 `?token=nbwt_aBcDeFg...&client_id=...` 打开 WebSocket。
4. token 会被消费（一次性），不可重复使用。

内置 WebUI 的 `/webui/bootstrap` 路由也会返回一个 WebSocket token。对于同机 localhost 浏览器请求，或请求已证明知道 `tokenIssueSecret` 或静态 `token` 后，它还会返回一个用于 REST 路由的独立 `api_token`。

### 示例配置

```json
{
  "channels": {
    "websocket": {
      "port": 8765,
      "path": "/ws",
      "tokenIssuePath": "/auth/token",
      "tokenIssueSecret": "your-secret-here",
      "tokenTtlS": 300,
      "websocketRequiresToken": true,
      "allowFrom": ["*"],
      "streaming": true
    }
  }
}
```

客户端流程：

```bash
# 1. Obtain a token
curl -H "Authorization: Bearer your-secret-here" http://127.0.0.1:8765/auth/token

# 2. Connect using the token
websocat "ws://127.0.0.1:8765/ws?client_id=alice&token=nbwt_aBcDeFg..."
```

### 限制

- 签发的 token 是一次性的 —— 每个 token 只能完成一次握手。
- 未使用的 token 上限为 10,000。超出该上限的请求会返回 HTTP 429。
- 过期 token 会在每次签发或校验请求时被惰性清理。

## 多聊天多路复用

单个 WebSocket 可以承载多个并发的聊天。服务器以扇出集合的方式维护 `chat_id -> {connections}`，因此同一个聊天也可以在多个连接之间镜像（例如两个浏览器标签页）。

### 典型流程（带侧边栏的 Web UI）

```text
client                                server
  | --- connect -------------------->  |
  | <-- {"event":"ready",              |
  |      "chat_id":"d3..."}   (default)|
  |                                     |
  | --- {"type":"new_chat"} --------->  |
  | <-- {"event":"attached",            |
  |      "chat_id":"a1..."}             |
  |                                     |
  | --- {"type":"message",              |
  |      "chat_id":"a1...",             |
  |      "content":"hi"} ------------>  |
  | <-- {"event":"delta", ...}          |
  | <-- {"event":"stream_end", ...}     |
  |                                     |
  | --- {"type":"attach",               |  # after page reload
  |      "chat_id":"a1..."} --------->  |
  | <-- {"event":"attached", ...}       |
```

### 规则

- 每个出站事件都携带 `chat_id`。客户端必须根据该字段进行分发。
- `chat_id` 格式：`^[A-Za-z0-9_:-]{1,64}$`。不匹配的值会返回 `error`。
- `message` 首次使用时会自动附加 —— 对于同一连接上服务器铸造（`new_chat`）的聊天，无需单独执行 `attach`。
- 错误（无效信封、未知 `type`、错误的 `chat_id`）为软错误：服务器回复 `{"event":"error","detail":"..."}` 并保持连接打开。

### 向后兼容

只发送纯文本或 `{"content": ...}` 的旧版客户端可以继续无缝工作：这些帧会路由到该连接的默认 `chat_id`（来自 `ready` 中的那一个）。无需任何配置开关。

### 安全边界

`chat_id` 是一种*能力*：任何持有有效 WebSocket 认证凭据和 chat_id 的人都可以附加到该会话并查看其输出。这对于 nanobot 的本地、单用户模型是安全的。多租户部署应按用户对 chat_id 进行命名空间隔离（或引入按租户的鉴权关卡）—— nanobot 目前不会自动这样做。

## 安全说明

- **时间安全比较**：静态 token 校验使用 `hmac.compare_digest` 以防止定时攻击。
- **纵深防御**：`allowFrom` 会在 HTTP 握手和消息两个层级都进行检查。
- **chat_id 作为能力**：见 [多聊天多路复用](#multi-chat-multiplexing)。WebSocket 握手时的认证是唯一的防线；通过认证的调用方可以附加到任意已知的 chat_id。
- **强制 TLS**：启用 SSL 时，允许的最低版本为 TLSv1.2。
- **默认安全**：`websocketRequiresToken` 默认为 `true`。仅在受信任的网络中显式设为 `false`。

## 媒体文件

出站的 `message` 事件可能包含一个 `media` 字段，其中是本地文件系统路径。远程客户端无法直接访问这些文件 —— 它们需要以下之一：

- 一个共享的文件系统挂载，或
- 一个用于提供 nanobot 媒体目录的 HTTP 文件服务器

## 常见模式

### 受信任的本地网络（无鉴权）

```json
{
  "channels": {
    "websocket": {
      "host": "0.0.0.0",
      "port": 8765,
      "websocketRequiresToken": false,
      "allowFrom": ["*"],
      "streaming": true
    }
  }
}
```

### 静态 token（简单鉴权）

```json
{
  "channels": {
    "websocket": {
      "token": "my-shared-secret",
      "allowFrom": ["alice", "bob"]
    }
  }
}
```

客户端使用 `?token=my-shared-secret&client_id=alice` 进行连接。

### 使用签发 token 的公开端点

```json
{
  "channels": {
    "websocket": {
      "host": "0.0.0.0",
      "port": 8765,
      "path": "/ws",
      "tokenIssuePath": "/auth/token",
      "tokenIssueSecret": "production-secret",
      "websocketRequiresToken": true,
      "sslCertfile": "/etc/ssl/certs/server.pem",
      "sslKeyfile": "/etc/ssl/private/server-key.pem",
      "allowFrom": ["*"]
    }
  }
}
```

### 自定义路径

```json
{
  "channels": {
    "websocket": {
      "path": "/chat/ws",
      "allowFrom": ["*"]
    }
  }
}
```

客户端连接到 `ws://127.0.0.1:8765/chat/ws?client_id=...`。尾部斜杠会被规范化，因此 `/chat/ws/` 效果相同。
