# 如何使用 nanobot 构建个人 AI 智能体

本指南将构建一个可以在本地运行、通过终端或浏览器与之交谈的个人 AI 智能体，
之后可以连接到聊天应用、记忆、工具和自动化。

## 你将构建什么

- 一个已配置的 nanobot 安装
- 一个可用的模型提供商
- 一次本地智能体回复
- 一个用于持续工作的浏览器 WebUI 会话

## 何时使用

当你想要一个由自己掌控的个人 AI 智能体，而不是一个托管的仅聊天界面时使用。
当智能体需要本地工作区访问、工具调用、会话历史、记忆、定时任务或聊天应用
消息投递时，nanobot 非常有用。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

向导会创建 `~/.nanobot/config.json`，并帮助你选择提供商和模型。
如果你对终端和配置文件不熟悉，请改用
[无技术背景入门](../start-without-technical-background.md)。

## 最小可用示例

首先验证运行时能够响应：

```bash
nanobot agent -m "Hello!"
```

然后打开浏览器工作台：

```bash
nanobot webui
```

WebUI 会启动本地网关、打开浏览器，并为较长的工作保持持久的聊天会话。

## 生产建议

- 每个项目或个人场景使用一个工作区。
- 当你希望为快速、深度、本地或回退模型使用稳定的名称时，使用 `modelPresets`。
- 保持 `nanobot gateway` 运行，用于 WebUI、聊天应用、自动化和 WebSocket 频道。
- 当另一个程序需要调用智能体时，使用 Python SDK 或兼容 OpenAI 的 API。

## 安全建议

- 不要将 API 密钥直接存储在共享文件中；请使用环境变量。
- 首次设置时优先使用聊天应用配对。仅在静态白名单场景下使用 `allowFrom`，
  并保持这些列表尽可能精简。
- 在向其他用户暴露文件或 shell 工具之前，先启用工作区限制。
- 对可能修改文件的实验，使用独立的工作区。

## 故障排查

- `nanobot status` 会显示配置路径、工作区路径和当前使用的模型。
- 如果 `nanobot agent -m "Hello!"` 失败，先修复提供商设置，
  然后再打开 WebUI 或聊天应用。
- 如果 WebUI 已打开但没有响应，检查网关日志和提供商凭据。

## 相关的 nanobot 文档

- [快速开始](../quick-start.md)
- [核心概念](../concepts.md)
- [WebUI](../webui.md)
- [配置](../configuration.md)
- [故障排查](../troubleshooting.md)
