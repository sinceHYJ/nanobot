# 如何使用 nanobot 为 AI 智能体添加 MCP 工具

nanobot 可以连接 MCP 服务器，将它们的工具与内置的文件、shell、网络、
cron、图像生成和子智能体工具一同暴露给智能体。

## 你将构建什么

- 一个可用的 nanobot 智能体
- 在 `config.json` 中配置的一个 MCP 服务器
- 一组仅限的、暴露给模型的工具

## 何时使用

当某个工具已经以 MCP 服务器形式存在，或者另一个应用发布了 MCP 适配器，
或者你希望在 nanobot 与外部工具逻辑之间保持清晰边界时，使用 MCP。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

单独安装 MCP 服务器自身的运行时。例如，许多本地 MCP 服务器使用
`npx` 或 `uvx`。

## 最小可用示例

在 `~/.nanobot/config.json` 中添加一个 stdio MCP 服务器：

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

重启 nanobot，然后提一个需要用到该 MCP 工具的问题。

## 生产建议

- 使用 `enabledTools` 仅暴露智能体真正需要的工具。
- 为较慢的 MCP 服务器设置 `toolTimeout`。
- 对本地工具优先使用 stdio MCP，对可信的远程服务使用 HTTP MCP。
- 尽量将 MCP 服务器的安装/更新步骤放在 nanobot 配置之外。

## 安全建议

- HTTP/SSE MCP URL 使用与 web fetch 相同的 SSRF 防护。
- 本地/私有 HTTP 端点需要显式的 `tools.ssrfWhitelist` 条目。
- stdio MCP 服务器会启动本地进程；请审查其命令和参数。
- 当环境变量或请求头可用时，不要在命令行参数中传递密钥。

## 故障排查

- 使用 `nanobot gateway --verbose` 启动网关并检查 MCP 启动日志。
- 在调试 nanobot 之前，先确认 MCP 命令单独运行是可用的。
- 如果某个 HTTP MCP 服务器被阻止，审查 SSRF 白名单并使用一个精简的
  主机 CIDR。

## 相关的 nanobot 文档

- [配置 MCP 工具](./configure-mcp-tools.md)
- [配置：MCP](../configuration.md#mcp-model-context-protocol)
- [安全](../configuration.md#security)
