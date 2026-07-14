# 使用 nanobot 构建 Discord AI 智能体

本指南将 nanobot 连接到 Discord，让 Discord 用户或服务器频道可以通过 nanobot 网关与你的自托管 AI 智能体对话。

## 本指南将构建

- 一个 Discord 机器人应用
- 已启用 Message Content intent
- 在 nanobot 中启用 `discord` 频道
- 一次私信或提及的消息测试

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 可访问 Discord 开发者门户。
- 一个你可以邀请机器人加入的 Discord 服务器。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Discord 频道

安装可选频道依赖：

```bash
nanobot plugins enable discord
```

创建一个 Discord 应用，添加机器人，复制令牌，并在机器人设置中启用 `MESSAGE CONTENT INTENT`。

将下面这段片段合并到 `~/.nanobot/config.json` 中：

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowChannels": [],
      "groupPolicy": "mention",
      "streaming": true
    }
  }
}
```

省略 `allowFrom` 会启用仅配对模式。新用户应先与机器人私信，获取配对码，经批准后才能在服务器中使用机器人。

以具备读取历史记录和发送消息权限的方式邀请机器人。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试消息

先向机器人发送私信。它应返回一个配对码。从受信任的本地入口批准配对码：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

批准后，在允许的服务器频道中提及机器人：

```text
@your-bot Hello from Discord
```

## 安全注意事项

- 首次部署时将 `groupPolicy` 保持为 `mention`。
- 使用 `allowChannels` 指定机器人应工作的服务器频道。
- 用户访问建议优先使用仅配对模式；仅在需要静态白名单时才添加 `allowFrom`。
- 在会话路由清晰之前，避免在繁忙频道中启用开放的群组行为。
- 在把机器人邀请到共享服务器之前，先审查工具访问权限。

## 故障排查

- 如果没有收到任何消息，请确认已启用 Message Content intent。
- 如果私信返回配对码，请先批准，再测试正常回复。
- 如果服务器消息被忽略，请检查配对审批、`allowChannels` 以及是否@了机器人。
- 如果机器人无法回复，请确认邀请权限及频道覆盖设置。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配对](../configuration.md#pairing)
- [AI 智能体记忆](./ai-agent-memory.md)
- [配置 MCP 工具](./configure-mcp-tools.md)
