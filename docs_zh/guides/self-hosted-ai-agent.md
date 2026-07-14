# 如何使用 nanobot 运行自托管 AI 智能体

本指南将把 nanobot 作为自托管的 AI 智能体运行时部署在你自己的机器或服务器上。
最终结果是一个网关进程，能够为 WebUI、聊天应用、自动化和 API 集成提供服务。

## 你将构建什么

- 一个由你掌控的 nanobot 配置和工作区
- 通过 `config.json` 连接的模型提供商
- 一个长时间运行的 `nanobot gateway`
- 可选的浏览器、聊天应用和 API 访问

## 何时使用

当你希望在本地或服务器端拥有智能体进程、工作区文件、记忆文件和提供商密钥的
所有权时使用这条路径。当智能体需要在一次终端命令结束后仍继续运行时，
这也是正确的路径。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

在部署网关之前完成 CLI 检查。当提供商和模型已知可用后，部署问题会更容易调试。

## 最小可用示例

对于聊天应用、自动化和 WebSocket 消息投递，启动网关：

```bash
nanobot gateway
```

对于浏览器界面，请改用 WebUI 启动器。它可以为你启动并管理本地网关：

```bash
nanobot webui
```

或者在 `~/.nanobot/config.json` 中连接一个频道，然后保持同一个网关进程
运行以处理消息。

## 生产建议

- 当进程需要在终端退出后继续运行时，使用 Docker、systemd 或 macOS LaunchAgent。
- 为每个部署实例提供不同的配置路径、工作区路径和端口集合。
- 将密钥保存在环境变量中，并从相同的环境启动服务。
- 使用针对网关或 API 进程的健康检查，而不是仅以聊天应用消息投递作为唯一信号。

## 安全建议

- 除非你有意向外暴露，否则将本地服务绑定到 `127.0.0.1`。
- 在将兼容 OpenAI 的 API 绑定到公网接口之前，先设置 API 密钥。
- 对支持私信的聊天应用优先使用配对，并保持任何静态 `allowFrom` 白名单严格。
- 启用 `tools.restrictToWorkspace`；在 Linux 上，为 shell 执行使用 bubblewrap 沙箱。

## 故障排查

- 使用与该服务相同的 `--config` 和 `--workspace` 参数运行 `nanobot status`。
- 在调试频道启动时运行 `nanobot gateway --verbose`。
- 如果 WebUI、WebSocket 频道或 API 端点无法绑定，检查端口冲突。

## 相关的 nanobot 文档

- [部署](../deployment.md)
- [多实例](../multiple-instances.md)
- [配置](../configuration.md)
- [聊天应用](../chat-apps.md)
- [兼容 OpenAI 的 API](../openai-api.md)
