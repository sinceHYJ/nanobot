# 使用 nanobot 构建 WeChat (微信) AI 智能体

本指南通过 `weixin` 频道将 nanobot 接入 WeChat。该频道通过所支持的上游 API 使用 HTTP 长轮询与二维码登录。

## 本指南将构建的内容

- 在 nanobot 中启用的 `weixin` 频道
- 一次二维码登录会话
- 一个通过配对审批的 WeChat 发送者
- 一个用于消息传递的运行中网关

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个能够完成二维码登录的 WeChat 账号。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 WeChat 频道

安装可选的频道依赖：

```bash
nanobot plugins enable weixin
```

将以下片段合并到 `~/.nanobot/config.json`：

```json
{
  "channels": {
    "weixin": {
      "enabled": true
    }
  }
}
```

省略 `allowFrom` 会启用仅配对模式。来自新发送者的第一条 WeChat 私聊消息会收到一个配对码，而不是获得智能体访问权限。

登录：

```bash
nanobot channels login weixin
```

如果需要放弃已保存的登录状态并重新认证，请使用 `--force`。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试一条消息

向机器人发送一条 WeChat 私聊消息。它应当回复一个配对码。从受信任的本地界面进行审批：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

审批完成后再次发送该消息，并观察网关日志中的发送者 ID 和回复。

## 安全说明

- 首次设置优先使用仅配对模式。仅在需要静态白名单时才添加 `allowFrom`。
- 将已保存的登录状态视为敏感的账号访问凭据。
- 避免将个人账号连接到不受信任的工作区或过于宽泛的工具权限。

## 故障排查

- 如果登录失败，重新运行 `nanobot channels login weixin --force`。
- 如果首次私聊消息返回了一个配对码，这是预期行为。请在测试正常智能体回复之前先审批该配对码。
- 如果消息被拒绝但未返回配对码，请检查网关日志，查看 WeChat 是否提供了 nanobot 回复所需的上下文令牌。
- 如果轮询断开，请重启网关并检查到上游服务的网络可达性。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [AI 智能体记忆](./ai-agent-memory.md)
- [安全的本地 AI 智能体](./secure-local-ai-agent.md)
- [部署](../deployment.md)
