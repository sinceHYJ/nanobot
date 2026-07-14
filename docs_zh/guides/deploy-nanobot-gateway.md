# 如何部署长期运行的 nanobot AI 智能体网关

nanobot 网关是长期运行的自托管 AI 智能体进程，用于保持 WebUI 会话、聊天
应用、自动化、本地触发器、心跳任务、Dream 以及 WebSocket 投递持续在线。

## 你将构建什么

- 一份经过验证的 nanobot 配置
- 一个网关进程
- 一条使用 Docker、systemd 或 macOS LaunchAgent 的服务或容器部署路径

## 何时使用

当 nanobot 需要在单次 CLI 交互之后继续运行时，请使用本指南。聊天应用、
浏览器会话、后台自动化、本地触发器和服务端集成都依赖一个在线的网关。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot status
nanobot agent -m "Hello!"
```

## 最小可用示例

在前台运行网关：

```bash
nanobot gateway
```

用于 WebUI 后台使用：

```bash
nanobot webui --background
nanobot gateway status
nanobot gateway logs
```

## 生产环境注意事项

- Docker Compose 是最具可复现性的 Linux 容器部署方式。
- systemd 用户级服务适合 Linux 用户级网关部署。
- macOS LaunchAgent 可在登录后保持网关持续存活。
- 持久化配置、工作区、会话、记忆文件、频道登录状态以及生成的产物。
- 在编辑 `config.json` 之后重启网关。

## 安全注意事项

- 在暴露服务前先规划端口。网关健康检查默认使用 `18790`，WebUI/WebSocket
  默认使用 `8765`，`nanobot serve` 默认使用 `8900`。
- 仅在已配置 token 或 API 密钥后再对外绑定。
- 在部署前有意识地设定聊天访问控制。
- 当为无人值守工作启用 shell 工具时，使用 Docker 或 Linux 沙箱。

## 故障排查

- 状态检查与服务启动使用相同的 `--config` 和 `--workspace` 参数。
- 通过 `docker compose logs`、`journalctl`、LaunchAgent 日志或
  `nanobot gateway --verbose` 查看日志。
- 如果 Docker 端口发布不生效，请确认服务未仅绑定到容器回环地址。

## 相关 nanobot 文档

- [部署](../deployment.md)
- [多实例](../multiple-instances.md)
- [配置](../configuration.md)
