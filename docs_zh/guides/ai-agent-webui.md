# 如何在 nanobot 中使用 AI 智能体 WebUI

nanobot 内置了一个浏览器 WebUI，用于持久聊天会话、可见的智能体活动、
工作区控制、Apps、MCP 预设、技能、设置和自动化。

## 你将构建什么

- 一个本地浏览器工作台
- 一次持久的聊天会话
- 一个可见的智能体消息、工具调用和文件编辑差异的时间线
- 一个由网关支持的 WebSocket 连接

## 何时使用

当你想要一个比终端更易操作的本地 AI 智能体界面时使用 WebUI，
尤其适合项目工作、文件附件、模型切换、工作区选择、Apps、技能和定时自动化。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

已发布的 wheel 包已经包含 WebUI 打包产物。仅当你需要修改前端时，
才需要 `webui/` 源码目录。

## 最小可用示例

```bash
nanobot webui
```

启动器会检查设置、在确认后启用本地 WebSocket 频道、启动网关并打开浏览器。

当 nanobot 编辑文件时，WebUI 活动时间线可以显示改动的行数、统一格式的差异，
以及一个 **Open file** 操作以供只读预览。文件预览使用当前聊天的工作区
访问模式：受限访问会保持在选定的工作区内，而完全访问（Full Access）
在网关允许时可以预览工作区之外的文件。

## 生产建议

- 当你不想保持终端打开时，使用 `nanobot webui --background`。
- 使用 `nanobot gateway status`、`logs`、`restart` 和 `stop` 管理后台网关。
- 如果你将 WebUI 暴露到 localhost 之外，请设置令牌签发密钥并审查工作区/工具访问。

## 安全建议

- 首次运行的 WebUI 路径默认绑定到 `127.0.0.1`。
- 未经有意的访问模型规划，不要将 WebUI 暴露到局域网或公共主机。
- 在邀请其他用户之前，将文件和 shell 工具限定在工作区内。

## 故障排查

- WebUI 默认由 WebSocket 频道在端口 `8765` 上提供服务。
- 网关健康检查端点与浏览器 UI 是分开的。
- 如果页面打开但消息发送失败，用 `nanobot agent -m "Hello!"` 检查提供商设置。

## 相关的 nanobot 文档

- [Nanobot WebUI](../webui.md)
- [快速开始](../quick-start.md)
- [WebSocket 协议](../websocket.md)
- [配置](../configuration.md)
