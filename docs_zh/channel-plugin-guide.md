# 频道插件指南

分三步构建自定义的 nanobot 频道：继承、打包、安装。

> **注意：** 我们建议针对 nanobot 的源码检出（`python -m pip install -e .`）开发频道插件，而不是使用 PyPI 发布版，这样你总能获得最新的基础频道功能和 API。

## 工作原理

nanobot 通过 Python 的[入口点](https://packaging.python.org/en/latest/specifications/entry-points/)发现频道插件。当 `nanobot gateway` 启动时，它会扫描：

1. `nanobot/channels/` 下的内置频道
2. 在 `nanobot.channels` 入口点组下注册的外部包

如果匹配的配置节包含 `"enabled": true`，就会实例化并启动该频道。

## 快速开始

我们将构建一个最小的 webhook 频道，它通过 HTTP POST 接收消息，并回传响应。

### 项目结构

```text
nanobot-channel-webhook/
├── nanobot_channel_webhook/
│   ├── __init__.py          # 重新导出 WebhookChannel
│   └── channel.py           # 频道实现
└── pyproject.toml
```

### 1. 创建你的频道

```python
# nanobot_channel_webhook/__init__.py
from nanobot_channel_webhook.channel import WebhookChannel

__all__ = ["WebhookChannel"]
```

```python
# nanobot_channel_webhook/channel.py
import asyncio
from typing import Any

from aiohttp import web
from loguru import logger
from pydantic import Field

from nanobot.channels.base import BaseChannel
from nanobot.bus.events import OutboundMessage
from nanobot.bus.queue import MessageBus
from nanobot.config.schema import Base


class WebhookConfig(Base):
    """Webhook channel configuration."""
    enabled: bool = False
    port: int = 9000
    allow_from: list[str] = Field(default_factory=list)


class WebhookChannel(BaseChannel):
    name = "webhook"
    display_name = "Webhook"

    def __init__(self, config: Any, bus: MessageBus):
        if isinstance(config, dict):
            config = WebhookConfig(**config)
        super().__init__(config, bus)

    @classmethod
    def default_config(cls) -> dict[str, Any]:
        return WebhookConfig().model_dump(by_alias=True)

    async def start(self) -> None:
        """Start an HTTP server that listens for incoming messages.

        IMPORTANT: start() must block forever (or until stop() is called).
        If it returns, the channel is considered dead.
        """
        self._running = True
        port = self.config.port

        app = web.Application()
        app.router.add_post("/message", self._on_request)
        runner = web.AppRunner(app)
        await runner.setup()
        site = web.TCPSite(runner, "0.0.0.0", port)
        await site.start()
        logger.info("Webhook listening on :{}", port)

        # Block until stopped
        while self._running:
            await asyncio.sleep(1)

        await runner.cleanup()

    async def stop(self) -> None:
        self._running = False

    async def send(self, msg: OutboundMessage) -> None:
        """Deliver an outbound message.

        msg.content  — markdown text (convert to platform format as needed)
        msg.media    — list of local file paths to attach
        msg.chat_id  — the recipient (same chat_id you passed to _handle_message)
        msg.metadata — channel routing context such as message/thread ids
        msg.event    — typed runtime event for progress/status messages
        """
        logger.info("[webhook] -> {}: {}", msg.chat_id, msg.content[:80])
        # In a real plugin: POST to a callback URL, send via SDK, etc.

    async def _on_request(self, request: web.Request) -> web.Response:
        """Handle an incoming HTTP POST."""
        body = await request.json()
        sender = body.get("sender", "unknown")
        chat_id = body.get("chat_id", sender)
        text = body.get("text", "")
        media = body.get("media", [])       # list of URLs

        # This is the key call: validates allowFrom, then puts the
        # message onto the bus for the agent to process.
        await self._handle_message(
            sender_id=sender,
            chat_id=chat_id,
            content=text,
            media=media,
        )

        return web.json_response({"ok": True})
```

### 2. 注册入口点

```toml
# pyproject.toml
[project]
name = "nanobot-channel-webhook"
version = "0.1.0"
dependencies = ["nanobot-ai", "aiohttp"]

[project.entry-points."nanobot.channels"]
webhook = "nanobot_channel_webhook:WebhookChannel"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["nanobot_channel_webhook"]
```

键（`webhook`）会成为配置节名，值指向你的 `BaseChannel` 子类。

### 3. 安装并配置

```bash
python -m pip install -e .
nanobot plugins list      # 确认已安装的示例插件以 "webhook" 显示
nanobot onboard           # 自动为检测到的插件添加默认配置
```

编辑 `~/.nanobot/config.json`：

```json
{
  "channels": {
    "webhook": {
      "enabled": true,
      "port": 9000,
      "allowFrom": ["*"]
    }
  }
}
```

### 4. 运行并测试

```bash
nanobot gateway
```

在另一个终端：

```bash
curl -X POST http://localhost:9000/message \
  -H "Content-Type: application/json" \
  -d '{"sender": "user1", "chat_id": "user1", "text": "Hello!"}'
```

智能体接收消息并处理它。回复会送达到你的 `send()` 方法。

## BaseChannel API

### 必需（抽象方法）

| 方法 | 描述 |
|--------|-------------|
| `async start()` | **必须永远阻塞。** 连接到平台，监听消息，对每条消息调用 `_handle_message()`。如果它返回，频道就被视为死亡。 |
| `async stop()` | 设置 `self._running = False` 并进行清理。网关关闭时调用。 |
| `async send(msg: OutboundMessage)` | 将出站消息投递到平台。 |

### 交互式登录

如果你的频道需要交互式认证（例如扫码），请重写 `login(force=False)`：

```python
async def login(self, force: bool = False) -> bool:
    """
    Perform channel-specific interactive login.

    Args:
        force: If True, ignore existing credentials and re-authenticate.

    Returns True if already authenticated or login succeeds.
    """
    # For QR-code-based login:
    # 1. If force, clear saved credentials
    # 2. Check if already authenticated (load from disk/state)
    # 3. If not, show QR code and poll for confirmation
    # 4. Save token on success
```

不需要交互式登录的频道（例如使用机器人 token 的 Telegram、使用机器人 token 的 Discord）会继承默认的 `login()`，它只返回 `True`。

用户通过以下命令触发交互式登录：
```bash
nanobot channels login <channel_name>
nanobot channels login <channel_name> --force  # 重新认证
```

### 基类提供的能力

| 方法 / 属性 | 描述 |
|-------------------|-------------|
| `_handle_message(sender_id, chat_id, content, media?, metadata?, session_key?)` | **接收到消息时调用。** 会检查 `is_allowed()`，然后发布到总线。如果 `supports_streaming` 为真，会自动设置 `_wants_stream`。 |
| `is_allowed(sender_id)` | 对照 `config.allow_from` 进行检查；`"*"` 允许所有，`[]` 拒绝所有。 |
| `default_config()` (classmethod) | 为 `nanobot onboard` 返回默认配置字典。重写以声明你自己的字段。 |
| `transcribe_audio(file_path)` | 通过共享的顶层 `transcription` 配置转写音频（如已配置）。 |
| `supports_streaming` (property) | 当配置有 `"streaming": true` **且**子类重写了 `send_delta()` 时为 `True`。 |
| `is_running` | 返回 `self._running`。 |
| `login(force=False)` | 执行交互式登录（例如扫码）。若已认证或登录成功则返回 `True`。在支持交互式登录的子类中重写。 |
| `send_reasoning_delta(chat_id, delta, metadata?, *, stream_id?)` | 用于流式模型推理/思考内容的可选钩子。默认无操作。 |
| `send_reasoning_end(chat_id, metadata?, *, stream_id?)` | 标记推理块结束的可选钩子。默认无操作。 |
| `send_reasoning(msg)` | 可选的一次性推理回退。默认会翻译为 `send_reasoning_delta()` + `send_reasoning_end()`。 |

### 可选（流式）

| 方法 | 描述 |
|--------|-------------|
| `async send_delta(chat_id, delta, metadata?, *, stream_id?, stream_end=False, resuming=False)` | 重写以接收流式片段。详见 [流式支持](#streaming-support)。 |

### 消息类型

```python
@dataclass
class OutboundMessage:
    channel: str        # your channel name
    chat_id: str        # recipient (same value you passed to _handle_message)
    content: str        # markdown text — convert to platform format as needed
    media: list[str]    # local file paths to attach (images, audio, docs)
    metadata: dict      # channel routing context, e.g. "message_id" for threading
    event: object | None # typed runtime/UI event; usually inspect with isinstance()
```

运行时/UI 语义位于 `msg.event` 上。插件编写的出站消息应使用类型化事件，而不是老的 metadata 标志，例如 `_progress`、`_stream_delta`、`_stream_end`、`_reasoning_delta`、`_turn_end` 或 `_goal_status`。nanobot 仍会接受这些旧标志，作为现有进程内扩展的兼容桥，但新插件代码不应对它们产生新的依赖。

## 流式支持

频道可以选择加入实时流式——智能体一段一段地发送内容，而不是最终一次性发送。这完全是可选的；不启用流式频道也能正常工作。

### 工作原理

当**同时**满足以下两个条件时，智能体通过你的频道以流式发送内容：

1. 配置中有 `"streaming": true`
2. 你的子类重写了 `send_delta()`

若缺少任一条件，智能体会回退到普通的一次性 `send()` 路径。

### 实现 `send_delta`

重写 `send_delta` 以处理两种调用：

```python
async def send_delta(
    self,
    chat_id: str,
    delta: str,
    metadata: dict[str, Any] | None = None,
    *,
    stream_id: str | None = None,
    stream_end: bool = False,
    resuming: bool = False,
) -> None:
    buffer_key = stream_id or chat_id
    if stream_end:
        # Streaming finished — do final formatting, cleanup, etc.
        return

    # Regular delta — append text, update the message on screen
    # delta contains a small chunk of text (a few tokens)
```

流式状态通过仅关键字参数传递，而不是 `_stream_delta` 或 `_stream_end` metadata 标志。用 `stream_id` 作为按流缓冲的键；若缺失则回退到 `chat_id`。

### 示例：带流式的 Webhook

```python
class WebhookChannel(BaseChannel):
    name = "webhook"
    display_name = "Webhook"

    def __init__(self, config: Any, bus: MessageBus):
        if isinstance(config, dict):
            config = WebhookConfig(**config)
        super().__init__(config, bus)
        self._buffers: dict[str, str] = {}

    async def send_delta(
        self,
        chat_id: str,
        delta: str,
        metadata: dict[str, Any] | None = None,
        *,
        stream_id: str | None = None,
        stream_end: bool = False,
        resuming: bool = False,
    ) -> None:
        buffer_key = stream_id or chat_id
        if stream_end:
            text = self._buffers.pop(buffer_key, "")
            # Final delivery — format and send the complete message
            await self._deliver(chat_id, text, final=True)
            return

        self._buffers.setdefault(buffer_key, "")
        self._buffers[buffer_key] += delta
        # Incremental update — push partial text to the client
        await self._deliver(chat_id, self._buffers[buffer_key], final=False)

    async def send(self, msg: OutboundMessage) -> None:
        # Non-streaming path — unchanged
        await self._deliver(msg.chat_id, msg.content, final=True)
```

### 配置

为每个频道启用流式：

```json
{
  "channels": {
    "webhook": {
      "enabled": true,
      "streaming": true,
      "allowFrom": ["*"]
    }
  }
}
```

当 `streaming` 为 `false`（默认）或未设置时，只会调用 `send()`——没有流式开销。

### BaseChannel 流式 API

| 方法 / 属性 | 描述 |
|-------------------|-------------|
| `async send_delta(chat_id, delta, metadata?, *, stream_id?, stream_end=False, resuming=False)` | 重写以处理流式片段。默认无操作。 |
| `supports_streaming` (property) | 当配置有 `streaming: true` **且**子类重写了 `send_delta` 时返回 `True`。 |

## 进度、工具提示与推理

除了正常的助手文本，nanobot 还可以发出低强调的追踪块。它们是为 UI 组件而设计的，例如状态行、可折叠的 "used tools" 分组，或推理/思考块。没有合适地方来展示它们的平台可以安全地忽略它们。

### 进度和工具提示

进度和工具提示会通过正常的 `send(msg)` 路径到达。渲染之前先检查 `msg.event`：

```python
from nanobot.bus.outbound_events import ProgressEvent

async def send(self, msg: OutboundMessage) -> None:
    event = msg.event

    if isinstance(event, ProgressEvent) and event.tool_hint:
        # A short tool breadcrumb, e.g. read_file("config.json")
        await self._send_trace(msg.chat_id, msg.content, kind="tool")
        return

    if isinstance(event, ProgressEvent):
        # Generic non-final status, e.g. "Thinking..." or "Running command..."
        await self._send_trace(msg.chat_id, msg.content, kind="progress")
        return

    await self._send_message(msg.chat_id, msg.content, media=msg.media)
```

对大多数频道来说，工具提示默认关闭。用户可以全局或按频道启用：

```json
{
  "channels": {
    "sendToolHints": true,
    "webhook": {
      "enabled": true,
      "sendToolHints": true
    }
  }
}
```

### 推理块

推理通过专用的可选钩子交付，而不是 `send()`。如果你的平台可以把模型推理展示为一个弱化/可折叠的块，请重写 `send_reasoning_delta()` 和 `send_reasoning_end()`。默认实现是空的，因此不支持的频道会直接丢弃推理内容。

```python
class WebhookChannel(BaseChannel):
    name = "webhook"
    display_name = "Webhook"

    def __init__(self, config: Any, bus: MessageBus):
        if isinstance(config, dict):
            config = WebhookConfig(**config)
        super().__init__(config, bus)
        self._reasoning_buffers: dict[str, str] = {}

    async def send_reasoning_delta(
        self,
        chat_id: str,
        delta: str,
        metadata: dict[str, Any] | None = None,
        *,
        stream_id: str | None = None,
    ) -> None:
        buffer_key = stream_id or chat_id
        self._reasoning_buffers[buffer_key] = self._reasoning_buffers.get(buffer_key, "") + delta
        await self._update_reasoning_block(chat_id, self._reasoning_buffers[buffer_key], final=False)

    async def send_reasoning_end(
        self,
        chat_id: str,
        metadata: dict[str, Any] | None = None,
        *,
        stream_id: str | None = None,
    ) -> None:
        buffer_key = stream_id or chat_id
        text = self._reasoning_buffers.pop(buffer_key, "")
        if text:
            await self._update_reasoning_block(chat_id, text, final=True)
```

**推理参数：**

| 参数 | 含义 |
|------|---------|
| `delta` | `send_reasoning_delta()` 的一段推理/思考片段。 |
| `stream_id` | 当前助手轮/段的稳定 id。使用它作为缓冲的键，而不仅是 `chat_id`。 |
| `send_reasoning_end()` | 当前推理块结束。 |

推理可见性由 `showReasoning` 全局或按频道控制：

```json
{
  "channels": {
    "showReasoning": true,
    "webhook": {
      "enabled": true,
      "showReasoning": true
    }
  }
}
```

推荐的渲染方式：

- 把工具提示和进度渲染为追踪/状态 UI，而不是普通的助手回复。
- 用较弱的视觉强调渲染推理，并在平台支持时于完成后折叠。
- 把推理与最终答案文本分开。最终答案仍会通过 `send()` 或 `send_delta()` 到达。

## 配置

### 为什么必须使用 Pydantic 模型

`BaseChannel.is_allowed()` 通过 `getattr(self.config, "allow_from", [])` 读取权限列表。这在 Pydantic 模型上是可行的，因为 `allow_from` 是一个真正的 Python 属性；但对于普通的 `dict` **会静默失败**——`dict` 没有 `allow_from` 属性，所以 `getattr` 总是返回默认值 `[]`，导致所有消息被拒绝。

内置频道使用 Pydantic 配置模型（继承自 `nanobot.config.schema` 的 `Base`）。插件频道**必须做同样的事**。

### 模式

1. 定义一个继承自 `nanobot.config.schema.Base` 的 Pydantic 模型：

```python
from pydantic import Field
from nanobot.config.schema import Base

class WebhookConfig(Base):
    """Webhook channel configuration."""
    enabled: bool = False
    port: int = 9000
    allow_from: list[str] = Field(default_factory=list)
```

`Base` 配置了 `alias_generator=to_camel` 和 `populate_by_name=True`，因此 JSON 键 `"allowFrom"` 和 `"allow_from"` 都能被接受。

2. 在 `__init__` 中将 `dict` 转换为模型：

```python
from typing import Any
from nanobot.bus.queue import MessageBus

class WebhookChannel(BaseChannel):
    def __init__(self, config: Any, bus: MessageBus):
        if isinstance(config, dict):
            config = WebhookConfig(**config)
        super().__init__(config, bus)
```

3. 以属性方式访问配置（而不是 `.get()`）：

```python
async def start(self) -> None:
    port = self.config.port
    token = self.config.token
```

`allowFrom` 由 `_handle_message()` 自动处理——你无需自己检查。

重写 `default_config()`，让 `nanobot onboard` 自动填充 `config.json`：

```python
@classmethod
def default_config(cls) -> dict[str, Any]:
    return WebhookConfig().model_dump(by_alias=True)
```

> **注意：** `default_config()` 返回一个普通的 `dict`（不是 Pydantic 模型），因为它会被序列化到 `config.json`。推荐做法是实例化你的配置模型并调用 `model_dump(by_alias=True)`——这会自动使用 camelCase 键（`allowFrom`），并让默认值保持在单一真源。

若不重写，基类返回 `{"enabled": false}`。

## 命名约定

| 名目 | 格式 | 示例 |
|------|--------|---------|
| PyPI 包 | `nanobot-channel-{name}` | `nanobot-channel-webhook` |
| 入口点键 | `{name}` | `webhook` |
| 配置节 | `channels.{name}` | `channels.webhook` |
| Python 包 | `nanobot_channel_{name}` | `nanobot_channel_webhook` |

## 本地开发

```bash
git clone https://github.com/you/nanobot-channel-webhook
cd nanobot-channel-webhook
python -m pip install -e .
nanobot plugins list    # 应显示已安装的示例插件为 "webhook"
nanobot gateway         # 端到端测试
```

## 验证

```bash
$ nanobot plugins list

  Name       Type      Enabled
  discord    channel   no
  telegram   channel   yes
  webhook    channel   yes
```
