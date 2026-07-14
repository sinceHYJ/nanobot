# 如何使用 nanobot 将 AI 智能体连接到聊天应用

nanobot 可以作为自托管的聊天机器人或 AI 智能体运行在 Telegram、Discord、
Slack、微信、邮件、Mattermost 以及其他聊天应用中。网关接收聊天消息、
运行智能体，并将回复发回同一频道。

## 你将构建什么

- 一个可用的本地智能体
- 一个已启用的聊天频道
- 一个正在运行的网关
- 一个基于配对的审批流程，或一个精简的静态白名单

## 何时使用

当智能体应该出现在用户已经进行沟通的地方时使用聊天应用：
私聊、团队频道、群聊、邮件线程或机器人工作区。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

然后选择一份平台指南：

- [Telegram AI 智能体](./telegram-ai-agent.md)
- [Discord AI 智能体](./discord-ai-agent.md)
- [Slack AI 智能体](./slack-ai-agent.md)
- [飞书 AI 智能体](./feishu-ai-agent.md)
- [WhatsApp AI 智能体](./whatsapp-ai-agent.md)
- [微信 AI 智能体](./wechat-ai-agent.md)
- [QQ AI 智能体](./qq-ai-agent.md)
- [邮件 AI 智能体](./email-ai-agent.md)
- [Mattermost AI 智能体](./mattermost-ai-agent.md)

## 最小可用示例

每个频道都遵循同样的模式：

1. 获取平台令牌、登录状态、webhook 或邮箱凭据。
2. 将频道片段合并到 `~/.nanobot/config.json` 中。
3. 对支持私聊的频道优先使用配对：省略 `allowFrom`，然后审批首条私聊
   的配对码。
4. 对不支持配对的频道（例如邮件），使用 `allowFrom` 或平台特定的
   允许列表，将访问范围保持精简。
5. 检查状态：

```bash
nanobot channels status
```

6. 启动网关：

```bash
nanobot gateway
```

7. 发送一条测试私聊消息，在收到提示时审批配对码，然后再次发送该测试
   消息。

## 生产建议

- 对于始终在线的聊天应用，将网关作为服务保持运行。
- 在向繁忙的频道开放机器人之前，先使用仅提及（mention-only）的群组策略。
- 调试时一次只使用一个频道。
- 首次测试优先使用私聊；配对仅在私聊中工作，而群聊会引入额外的权限
  和路由行为。

## 安全建议

- 优先使用配对或显式的白名单；不要在有意的沙箱之外使用 `allowFrom: ["*"]`。
- 如果机器人令牌被粘贴到日志或共享文件中，请轮换令牌。
- 在邀请其他用户之前，审查文件、shell 和 Web 工具的访问权限。

## 故障排查

- 如果 `nanobot channels status` 没有显示该频道，很可能缺少配置键或
  可选依赖。
- 如果首条私聊返回一个配对码，请先用 `/pairing approve <code>` 审批它，
  再期望收到正常回复。
- 如果消息未到达，运行 `nanobot gateway --verbose` 并对比平台凭据、
  事件权限和允许列表。
- 如果群组回复不符合预期，审查该频道的群组策略。

## 相关的 nanobot 文档

- [聊天应用](../chat-apps.md)
- [配置](../configuration.md#channel-settings)
- [配对](../configuration.md#pairing)
- [部署](../deployment.md)
