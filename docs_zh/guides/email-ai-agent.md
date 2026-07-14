# 使用 nanobot 构建 Email AI 智能体

本指南将 nanobot 变为一个 Email AI 智能体，它通过 IMAP 轮询已接受的消息，并通过 SMTP 回复。

## 本指南将构建的内容

- 一个专用于 nanobot 的邮箱
- 在 `config.json` 中的 IMAP 和 SMTP 凭据
- 一个允许的发送者列表
- 一个进行轮询和回复的网关进程

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个供机器人使用的邮箱。
- IMAP 和 SMTP 访问权限。对于 Gmail，请使用应用专用密码，而非账号密码。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Email 频道

将以下片段合并到 `~/.nanobot/config.json`，并替换其中的地址和密码：

```json
{
  "channels": {
    "email": {
      "enabled": true,
      "consentGranted": true,
      "imapHost": "imap.gmail.com",
      "imapPort": 993,
      "imapUsername": "my-nanobot@gmail.com",
      "imapPassword": "your-app-password",
      "smtpHost": "smtp.gmail.com",
      "smtpPort": 587,
      "smtpUsername": "my-nanobot@gmail.com",
      "smtpPassword": "your-app-password",
      "fromAddress": "my-nanobot@gmail.com",
      "allowFrom": ["your-real-email@gmail.com"],
      "autoReplyEnabled": true
    }
  }
}
```

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试一条消息

从 `allowFrom` 中的地址向机器人邮箱发送一封邮件。请让网关运行足够长的时间，以便轮询间隔能够接收到它。

## 安全说明

- 使用专用邮箱，而不是你的主要个人收件箱。
- 将 `consentGranted` 设为 `false` 可完全禁用邮箱访问。
- Email 不使用私聊配对。请将 `allowFrom` 保持较窄；`["*"]` 会接受来自任何人的邮件。
- 使用环境变量来存储邮箱密码。
- 仅在智能体需要时才启用附件类型。

## 故障排查

- 如果登录失败，请确认 IMAP/SMTP 访问权限和应用专用密码的配置。
- 如果机器人能读取但不回复，请检查 `autoReplyEnabled`、SMTP 设置以及允许的发送者地址。
- 如果附件缺失，请检查 `allowedAttachmentTypes`、大小限制和网关日志。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [安全的本地 AI 智能体](./secure-local-ai-agent.md)
- [AI 智能体记忆](./ai-agent-memory.md)
- [OpenAI 兼容的智能体 API](./openai-compatible-agent-api.md)
