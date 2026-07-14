# 使用 nanobot 构建 QQ AI 智能体

本指南通过官方的 `qq` 频道将 nanobot 接入 QQ。官方频道使用 botpy SDK，目前主要聚焦于私聊消息。如需 QQ 群聊和 OneBot v11 工作流，请参阅完整聊天应用参考中的 Napcat 章节。

## 本指南将构建的内容

- 一个 QQ 机器人应用
- 在 nanobot 中启用的 `qq` 频道
- 一个通过配对审批的 QQ 私聊发送者
- 一个运行中的 nanobot 网关

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 可访问 QQ 开放平台。
- 一个已添加到机器人沙箱用于测试的 QQ 账号。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 QQ 频道

安装可选的频道依赖：

```bash
nanobot plugins enable qq
```

在 QQ 开放平台创建一个机器人应用，并复制 AppID 和 AppSecret。将你的 QQ 账号添加到沙箱测试成员，然后将以下片段合并到 `~/.nanobot/config.json`：

```json
{
  "channels": {
    "qq": {
      "enabled": true,
      "appId": "YOUR_APP_ID",
      "secret": "YOUR_APP_SECRET",
      "msgFormat": "plain"
    }
  }
}
```

省略 `allowFrom` 会启用仅配对模式。新的私聊发送者在获得正常的智能体访问权限之前应先收到一个配对码。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试一条消息

从一个沙箱账号向 QQ 机器人发送一条私聊消息。它应当返回一个配对码。从受信任的本地界面进行审批：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

审批完成后再次发送该消息。

## 安全说明

- 首次设置优先使用仅配对模式。仅在需要静态白名单时才添加 `allowFrom`。
- 将沙箱测试与生产发布分开。
- 已部署的服务应通过环境变量存储 QQ AppSecret。
- 仅在你确实需要 QQ 账号桥接和群聊功能时使用 Napcat。

## 故障排查

- 如果私聊消息未到达，请确认发送者在 QQ 机器人沙箱中，并且网关正在运行。
- 如果输出格式不稳定，请将 `msgFormat` 保持为 `"plain"`。
- 如果首次私聊消息返回了一个配对码，请在测试正常回复之前先审批该码。
- 如果需要 QQ 群，请参阅完整聊天应用参考中的 Napcat 章节。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配对](../configuration.md#pairing)
- [AI 智能体记忆](./ai-agent-memory.md)
- [配置 MCP 工具](./configure-mcp-tools.md)
