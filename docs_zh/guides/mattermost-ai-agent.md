# 使用 nanobot 构建 Mattermost AI 智能体

本指南通过内置的 Mattermost 频道将 nanobot 接入 Mattermost，使用 WebSocket 事件与 Mattermost REST API。

## 本指南将构建的内容

- 一个 Mattermost 机器人账号或令牌
- 在 nanobot 中启用的 `mattermost` 频道
- 首次部署时仅在 @提及 (mention) 时才响应群聊的行为
- 一个通过配对审批的私聊或 @提及 测试

## 前置条件

- 一个可正常工作的本地 nanobot 回复：

```bash
nanobot agent -m "Hello!"
```

- 一个 Mattermost 服务器 URL。
- 一个机器人账号的机器人令牌或个人访问令牌。

## 安装 nanobot

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

## 启用 Mattermost 频道

将以下片段合并到 `~/.nanobot/config.json`：

```json
{
  "channels": {
    "mattermost": {
      "enabled": true,
      "serverUrl": "https://mattermost.example.com",
      "token": "YOUR_MATTERMOST_TOKEN",
      "teamId": "YOUR_TEAM_ID",
      "groupPolicy": "mention",
      "replyInThread": true,
      "dm": {
        "policy": "allowlist"
      }
    }
  }
}
```

`teamId` 将该频道限定在某个 Mattermost 团队。首次测试时请将 `groupPolicy` 保持为 `mention`。

Mattermost 私聊默认是开放的。将 `dm.policy` 设为 `"allowlist"` 且不填写任何 `dm.allowFrom` 条目，可以让新的私聊发送者收到一个配对码。审批该配对码后再正常使用该机器人。

## 运行 nanobot 网关

```bash
nanobot channels status
nanobot gateway
```

## 测试一条消息

向机器人账号发送私聊。它应当返回一个配对码。从受信任的本地界面进行审批：

```bash
nanobot agent -m "/pairing approve ABCD-EFGH"
```

然后再次向机器人发送私聊，或在机器人有访问权限的频道中 @提及 它：

```text
@nanobot Hello from Mattermost
```

## 安全说明

- 已部署的服务应将 Mattermost 令牌存储在环境变量中。
- 当你希望使用基于配对的审批时，请将 `dm.policy` 保持为 `"allowlist"`。
- 在向繁忙频道开放机器人之前，先使用仅 @提及 的群聊行为。
- 在邀请到宽泛的频道访问权限之前，先审查文件和 shell 工具。

## 故障排查

- 如果启动日志显示 `serverUrl and token must be configured`，请检查 camelCase 配置键。
- 如果私聊被忽略，请检查 `dm` 策略和配对审批状态。
- 如果频道消息被忽略，请确认机器人被 @提及 并且属于该团队/频道。
- 如果线程回复行为出乎意料，请检查 `replyInThread` 和 `includeThreadContext`。

## 下一步：记忆、自动化、MCP 工具

- [聊天应用参考](../chat-apps.md)
- [配对](../configuration.md#pairing)
- [长时运行 AI 智能体](./long-running-ai-agent.md)
- [部署](../deployment.md)
