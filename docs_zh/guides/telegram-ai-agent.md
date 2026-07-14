# 使用 nanobot 构建 Telegram AI 智能体

本指南将 nanobot 连接到 Telegram，让已完成配对的 Telegram 用户可以通过消息与由你常规的 nanobot 配置、工具、记忆和工作区支持的自托管 AI 智能体对话。

## 本指南将构建

- 通过 BotFather 创建的 Telegram 机器人
- 在 nanobot 中启用 `telegram` 频道
- 运行中的 nanobot 网关
- 一个已通过配对审批的 Telegram 账户

## 前置条件

- 一个可正常工作的 nanobot CLI 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个 Telegram 账户。
- 一个来自 `@BotFather` 的机器人令牌。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Telegram 频道

安装可选频道依赖：

```bash
nanobot plugins enable telegram
```

将下面这段片段合并到 `~/.nanobot/config.json` 中：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN"
    }
  }
}
```

省略 `allowFrom` 会启用仅配对模式。新用户发来的第一条私信会收到一个配对码，而不是直接获得智能体访问权限。

Telegram 默认使用长轮询。Webhook 模式适用于公网 HTTPS 部署；首次测试请先使用长轮询。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

测试消息期间保持网关持续运行。

## 测试消息

打开 Telegram，向机器人发送私信：

```text
Hello from Telegram
```

机器人应回复一个配对码。从已受信任的入口（例如本地 CLI）批准该配对码：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

批准后再次发送消息。回复应使用与本地 CLI 检查相同的模型和工作区。

## 安全注意事项

- 首次配置时优先使用仅配对模式。仅在需要静态白名单而非配对码审批时才添加 `allowFrom`。
- 除非机器人是隔离部署或有意公开的，否则不要使用 `allowFrom: ["*"]`。
- 如果 BotFather 令牌被粘贴到日志或共享文件中，请及时轮换。
- 在添加群聊或更多用户之前，先审查工具访问权限。

## 故障排查

- 如果没有列出该频道，请在同一个 Python 环境中重新执行 `nanobot plugins enable telegram`。
- 如果消息未送达，请运行 `nanobot gateway --verbose` 并检查机器人令牌。
- 如果首次私信返回配对码，这是预期行为。在测试正常智能体回复前先批准配对码。
- 如果 Telegram Web 显示不支持的富消息，请保持 `richMessages` 处于禁用状态。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [AI 智能体记忆](./ai-agent-memory.md)
- [长时间运行的 AI 智能体](./long-running-ai-agent.md)
- [配置 MCP 工具](./configure-mcp-tools.md)
