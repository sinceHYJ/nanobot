# 使用 nanobot 构建 WhatsApp AI 智能体

本指南通过 `whatsapp` 频道将 nanobot 连接到 WhatsApp。该频道以 WhatsApp 设备的形式登录，并使用与 CLI 和 WebUI 相同的 nanobot 智能体运行时、工具、记忆和工作区。

## 本指南将构建

- 已安装的 WhatsApp 可选依赖
- 一个已链接的 WhatsApp 设备会话
- 在 `config.json` 中启用 `whatsapp` 频道
- 一个已通过配对审批的 WhatsApp 发送者

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个可以链接新设备的 WhatsApp 账户。
- 一台可以持续运行 `nanobot gateway` 的机器。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 WhatsApp 频道

安装可选频道依赖：

```bash
nanobot plugins enable whatsapp
```

将 WhatsApp 作为设备登录：

```bash
nanobot channels login whatsapp
```

在 WhatsApp -> 设置 -> 已链接的设备 中扫描二维码。

将下面这段片段合并到 `~/.nanobot/config.json` 中：

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "groupPolicy": "mention"
    }
  }
}
```

省略 `allowFrom` 会为私聊启用仅配对模式。该频道中 `groupPolicy` 默认值为 `"open"`，但对于首次部署，`"mention"` 更为安全。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试消息

向机器人发送一条私人 WhatsApp 消息。它应返回一个配对码。从受信任的本地入口批准该配对码：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

批准后再次发送消息。回复应使用与本地 CLI 检查相同的模型和工作区。

## 安全注意事项

- 将 WhatsApp 会话数据库视为等同于账户访问权限。
- 首次配置时优先使用仅配对模式。仅在需要静态白名单时才添加 `allowFrom`。
- 在把机器人加入群组之前，将 `groupPolicy` 保持为 `"mention"`。
- 除非机器人是有意公开或隔离部署，否则避免使用 `allowFrom: ["*"]`。

## 故障排查

- 如果扫码链接失败，请重新运行 `nanobot channels login whatsapp`。
- 如果你在从旧版桥接迁移，请移除 `bridgeUrl` 和 `bridgeToken`，然后重新登录。
- 如果发送者显示为 LID 而不是手机号，可以让 nanobot 在运行时学习这一映射关系，或在完整参考中使用 `lidMappings`。
- 如果首次私聊返回配对码，请先批准，再测试正常回复。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配对](../configuration.md#pairing)
- [安全的本地 AI 智能体](./secure-local-ai-agent.md)
- [部署](../deployment.md)
