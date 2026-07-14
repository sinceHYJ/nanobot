# 如何在 nanobot 中配置 MCP 工具

本指南向 nanobot 添加一个 MCP 服务器，使智能体可以通过 Model Context
Protocol 使用外部工具。

## 你将构建什么

- 一个可用的 nanobot 智能体
- 在 `~/.nanobot/config.json` 中的一个 MCP 服务器条目
- 一组受限的、暴露给模型的 MCP 工具

## 何时使用

当你需要的能力已经以 MCP 服务器形式存在，或者你希望外部工具在 nanobot 核心
之外进行管理时，请使用 MCP。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

需单独安装 MCP 服务器运行时。许多示例使用 `npx`、`uvx` 或远程 HTTP 端点。

## 最小可用示例

将以下内容添加到 `~/.nanobot/config.json`：

```json
{
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
        "enabledTools": ["read_file"]
      }
    }
  }
}
```

重启 nanobot，并提出一个需要该 MCP 工具的问题。

## 生产环境注意事项

- 优先使用 `enabledTools`，而不是默认暴露所有工具。
- 对于较慢的 MCP 操作，使用 `toolTimeout`。
- 仅对你信任的端点使用 HTTP MCP。
- 在部署文档或脚本中保持 MCP 服务器命令稳定并进行版本管理。

## 安全注意事项

- Stdio MCP 会启动一个本地进程；启用前请审查该命令。
- HTTP/SSE MCP 使用 nanobot 的 SSRF 防护。
- 仅在配置了狭窄的 `tools.ssrfWhitelist` CIDR 时，才允许私有 HTTP MCP 主机。
- 当可以使用环境变量或请求头时，不要将密钥放在命令参数中。

## 故障排查

- 先在 nanobot 之外运行 MCP 命令。
- 启动 `nanobot gateway --verbose` 并检查工具注册日志。
- 如果 HTTP MCP URL 被拦截，请检查它是否指向回环地址或需要显式加入白名单的
  私有地址。

## 相关 nanobot 文档

- [面向 AI 智能体的 MCP 工具](./mcp-tools-for-ai-agents.md)
- [配置：MCP](../configuration.md#mcp-model-context-protocol)
- [安全](../configuration.md#security)
