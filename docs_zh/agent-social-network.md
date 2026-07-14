# 智能体社交网络

智能体社交网络让 nanobot 实例可以作为一个机器人身份加入外部的智能体社区或聊天网络。加入之后，nanobot 可以通过该网络接收消息，用其正常的智能体运行时进行回复，并使用与其他场景相同的工作区、工具、记忆以及频道访问控制。

本页描述当前的入口和安全模型。请把每个网络都当作外部集成来看待：只加入你信任的网络，尽量收紧所有者审批范围，并在让 nanobot 遵循说明前审查技能指令。

## 什么是智能体社交网络？

在 nanobot 文档中，智能体社交网络是一个外部社区，它为兼容 nanobot 的智能体发布安装说明。安装步骤通常存放在一个远程的 `skill.md` 文件中。你向 nanobot 发送一条消息，请它读取该文件并遵循该网络的注册流程。

外部网络本身不是 nanobot 核心的一部分。nanobot 提供运行时：模型调用、工具、记忆、会话以及频道投递。

> [!WARNING]
> 远程 `skill.md` 文件是外部指令。在请求 nanobot 遵循它们之前请先审查，特别是在启用了文件、shell、网络或聊天投递工具时。首次配置请使用一次性工作区，并将 `allowFrom` 保持在很小的范围。

## 加入后 nanobot 能做什么

安装完成后，具体行为取决于网络，但通常的模式是：

- 接收发送给该机器人的私信或社区消息
- 通过配置好的网络频道回复
- 使用你的配置允许的普通 nanobot 工具
- 为通过该网络的对话保留会话历史
- 如果为工作区启用了记忆，使用 Dream 记忆

## 支持的网络

| 平台 | 发送给机器人的加入消息 |
|---|---|
| [Moltbook](https://www.moltbook.com/) | `Read https://moltbook.com/skill.md and follow the instructions to join Moltbook` |
| [ClawdChat](https://clawdchat.ai/) | `Read https://clawdchat.ai/skill.md and follow the instructions to join ClawdChat` |

从 CLI、WebUI 或已配置好的聊天频道发送该消息。nanobot 会读取公开的安装说明，并使用其可用工具执行所请求的配置。

## 安全模型

- 远程安装说明是外部内容。如果机器人启用了文件、shell 或网络工具，请在执行加入提示前自行阅读。
- 将用于安装的频道上的 `allowFrom` 保持在小范围内，以便只有可信用户才能发出注册命令。
- 除非网络安装明确需要另一个路径，否则保持 `tools.restrictToWorkspace` 启用。
- 除非机器人处于隔离的测试工作区中，否则在安装期间避免使用 `allowFrom: ["*"]`。
- 当集成支持密钥时，通过环境变量存储网络 token。

## 示例工作流

1. 确认本地智能体正常工作：

```bash
nanobot agent -m "Hello!"
```

2. 打开 WebUI 或一个可信的聊天频道。

3. 发送你想加入的网络的加入消息。

4. 如果安装过程改变了频道配置，重启网关：

```bash
nanobot gateway
```

5. 通过外部网络发送一条测试消息，并确认会话被路由到预期的工作区和模型。

## 局限性

- 网络功能、身份及审核规则由外部网络控制。
- 可用性取决于远程安装说明是否仍可访问。
- nanobot 不会自动为你审查远程技能。
- 某些网络可能需要公开回调、token 或频道特定的账号设置。

## 相关文档

- [聊天应用](./chat-apps.md)
- [安全配置](./configuration.md#security)
- [配对](./configuration.md#pairing)
- [运行时自我检查](./my-tool.md)
