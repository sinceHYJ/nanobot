# 使用 nanobot 构建 Feishu (飞书) AI 智能体

本指南通过 `feishu` 频道将 nanobot 连接到 Feishu (飞书) 或 Lark。该频道使用 WebSocket 长连接，因此首次配置无需公网 webhook URL。

## 本指南将构建

- 一个连接到 nanobot 的 Feishu/Lark 机器人应用
- 在 `config.json` 中启用 `feishu` 频道
- 一个已通过配对审批的 Feishu 或 Lark 用户
- 首次部署时采用仅提及的群聊行为

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个可以创建或审批机器人应用的 Feishu 或 Lark 账户。
- 具备持续运行 `nanobot gateway` 的权限。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Feishu 频道

安装可选频道依赖：

```bash
nanobot plugins enable feishu
```

最简便的方式是扫码登录：

```bash
nanobot channels login feishu
```

打开打印出来的 URL 或扫描二维码。nanobot 会将生成的 `appId`、`appSecret`、`domain` 和 `enabled` 字段写入当前生效的配置文件。

如果扫码登录不可用，请手动创建一个 Feishu/Lark 应用，并把下面这段结构合并到 `~/.nanobot/config.json` 中：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "groupPolicy": "mention",
      "streaming": true,
      "domain": "feishu"
    }
  }
}
```

省略 `allowFrom` 会启用仅配对模式。新用户应先与机器人私信，获取配对码，经批准后才能正常使用机器人。

对于手动创建的应用，请启用机器人能力、接收消息事件和长连接模式。如果你的应用无法获得 `cardkit:card:write` 权限，请设置 `"streaming": false`。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试消息

先向机器人发送私信。它应返回一个配对码。从受信任的本地入口批准该配对码：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

批准后，再次向机器人发送私信，或在群聊中提及它：

```text
@nanobot Hello from Feishu
```

## 安全注意事项

- 首次配置时优先使用仅配对模式。仅在需要静态白名单时才添加 `allowFrom`。
- 在把机器人拉进繁忙群聊之前，将 `groupPolicy` 保持为 `"mention"`。
- 对于部署的服务，通过环境变量存储应用密钥。
- 在添加更多用户之前，先审查文件、Shell 及网络工具的访问权限。

## 故障排查

- 如果扫码登录不可用，请参照完整的 chat-apps 参考手动配置应用。
- 如果流式卡片失败，请确认 `cardkit:card:write` 权限，或设置 `"streaming": false`。
- 如果没有收到任何消息，请检查 Feishu/Lark 事件权限、长连接模式，以及 `nanobot gateway --verbose` 的输出。
- 如果首次私信返回配对码，在测试正常回复前先批准该配对码。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配对](../configuration.md#pairing)
- [AI 智能体记忆](./ai-agent-memory.md)
- [配置 MCP 工具](./configure-mcp-tools.md)
