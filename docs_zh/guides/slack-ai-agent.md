# 使用 nanobot 构建 Slack AI 智能体

本指南通过 Socket Mode 将 nanobot 连接到 Slack。首次可运行的配置无需公网 webhook URL。

## 本指南将构建

- 一个启用了 Socket Mode 的 Slack 应用
- 一个 bot token 和一个 app-level token
- 在 nanobot 中启用 `slack` 频道
- 从已批准的 Slack 用户完成一次私信配对流程和提及测试

## 前置条件

- 一个可正常工作的 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 拥有在某个工作区中创建 Slack 应用的权限。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Slack 频道

安装可选频道依赖：

```bash
nanobot plugins enable slack
```

在 Slack 中，创建一个应用，启用 Socket Mode，创建具有 `connections:write` 权限的 app-level token，添加机器人 scope，订阅机器人事件，并将应用安装到你的工作区。

将下面这段片段合并到 `~/.nanobot/config.json` 中：

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "xoxb-...",
      "appToken": "xapp-...",
      "groupPolicy": "mention",
      "dm": {
        "policy": "allowlist"
      }
    }
  }
}
```

Slack 私信默认是开放的。将 `dm.policy` 设置为 `"allowlist"`，且 `dm.allowFrom` 保持为空时，新的私信发送者会收到配对码。批准该配对码后才能正常使用机器人。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试消息

直接向 Slack 机器人发送私信。它应返回一个配对码。从受信任的本地入口批准该配对码：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

然后再次向机器人发送私信，或在频道中提及它：

```text
@nanobot Hello from Slack
```

## 安全注意事项

- 除非机器人有意监听每个频道的消息，否则将 `groupPolicy` 保持为 `mention`。
- 当希望使用基于配对的审批时，将 `dm.policy` 保持为 `"allowlist"`。
- 对已批准的频道使用配合白名单模式的 `groupAllowFrom`。
- 修改 scope 后需要重新安装 Slack 应用。
- 不要将 bot 和 app token 提交到配置文件中。

## 故障排查

- 如果 Socket Mode 失败，请确认 app-level token 以 `xapp-` 开头。
- 如果机器人无法发送文件，请添加 `files:write`、重新安装应用，并重启 nanobot。
- 如果私信正常回复但没有出现配对流程，请检查 `dm.policy` 是否为 `"allowlist"`。
- 如果频道消息被忽略，请检查事件订阅和 group policy。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配置网络搜索](./configure-web-search.md)
- [长时间运行的 AI 智能体](./long-running-ai-agent.md)
- [部署](../deployment.md)
