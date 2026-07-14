# Project Memory — nanobot

## Architecture (verified 2026-07-14)
- Two independent process entry points: **CLI** (`nanobot agent`) and **Gateway** (`nanobot gateway` / `nanobot webui`). Both reuse the same `AgentLoop` + `MessageBus` core.
- **Channel is NOT an independent entry point** — it is an in-process message adapter loaded by `ChannelManager` inside the Gateway. Cannot run standalone.
- WebUI = Gateway's WebSocket channel + browser. `nanobot serve` is a separate OpenAI-compatible API server (no ChannelManager). Python SDK also available.
- Message flow: external platform → channel.publish_inbound → MessageBus → AgentLoop → MessageBus → ChannelManager._dispatch_outbound → channel → platform.
- Key files: `nanobot/cli/commands.py`, `nanobot/channels/manager.py`, `nanobot/bus/queue.py`.
