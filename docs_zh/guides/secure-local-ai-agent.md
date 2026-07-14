# 如何使用 nanobot 保护本地 AI 智能体的安全

本指南涵盖在允许 nanobot 智能体访问文件、shell 命令、网页抓取、聊天应用或
远程用户之前，需要审查的一些实用控制项。

## 你将构建什么

- 一个限定在工作区范围内的智能体配置
- 收窄的频道访问权限
- 更安全的密钥处理方式
- 可选的 Linux shell 沙箱

## 何时使用

在将 nanobot 暴露给团队成员、聊天应用、公网、大范围网页访问或无人值守自动化
之前，请使用本指南。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

## 最小可用示例

从工作区限制开始：

```json
{
  "tools": {
    "restrictToWorkspace": true,
    "exec": {
      "enable": true,
      "sandbox": "bwrap"
    }
  }
}
```

`bwrap` 仅适用于 Linux，且需要 bubblewrap。在 macOS 或 Windows 上，请保持
`restrictToWorkspace` 启用，并仔细审查 shell 访问。

## 生产环境注意事项

- 使用环境变量存储提供商密钥、机器人 token 及邮箱密码。
- 每个信任边界使用一个工作区。
- 对于支持 DM 的聊天应用优先使用配对（pairing），仅在有意采用静态白名单时
  才使用狭窄的 `allowFrom` 列表；群组策略在最初保持仅响应 @提及。
- 除非确有远程访问需求，否则将 WebUI、WebSocket 和 API 服务绑定到 localhost。

## 安全注意事项

- `restrictToWorkspace` 是一种应用层防护，并非操作系统级沙箱。
- `tools.exec.enable: false` 会完全移除 shell 执行能力。
- HTTP 网页抓取和 HTTP MCP 默认使用 SSRF 防护。
- 添加宽泛的 `tools.ssrfWhitelist` 范围会增加暴露面。
- `allowFrom: ["*"]` 会绕过配对，意味着任何能访问该频道的人都可以与机器人
  对话。

## 故障排查

- 如果所需的文件无法读取，请确认当前工作区路径。
- 如果 shell 命令在 `bwrap` 下失败，请检查该命令是否需要访问沙箱之外的
  文件。
- 如果本地 HTTP 工具被拦截，请检查 SSRF 白名单并使用狭窄的 CIDR。

## 相关 nanobot 文档

- [配置：安全](../configuration.md#security)
- [配对](../configuration.md#pairing)
- [部署](../deployment.md)
- [聊天应用](../chat-apps.md)
