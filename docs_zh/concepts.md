# 概念

当你希望在修改高级设置之前理解 nanobot 时，请使用此页面。它在不需要你先阅读源码的前提下，解释了各组成部分。

如果你想了解源码文件的归属和扩展点，请在本页之后阅读 [`architecture.md`](./architecture.md)。

## 运行时结构

nanobot 有一个小的核心循环，以及几种进入它的方式：

| 组成部分 | 作用 |
|---|---|
| 智能体循环 | 构建上下文、选择会话、调用提供商、运行工具并发布回复 |
| 提供商 | LLM 后端，例如 OpenRouter、Anthropic、OpenAI、Bedrock、Ollama、vLLM 以及其他 OpenAI 兼容 API |
| 频道 | 面向用户的传输层，例如 CLI、WebUI/WebSocket、Telegram、Discord、Slack、飞书、微信、邮件、Mattermost 等 |
| 工具 | 模型可以调用的能力，包括文件、shell、网页搜索/抓取、MCP、cron、图片生成和子智能体 |
| 记忆 | 用来在轮次之间保留有用上下文的工作区文件和会话历史 |
| 网关 | 长期运行的进程，连接已启用的频道并提供健康检查端点 |

最简单的路径是 `nanobot agent -m "Hello!"`：一条入站消息经过智能体循环，并在终端打印回复。长期运行的路径是 `nanobot gateway`：频道从聊天应用或 WebUI 接收消息，将它们发布到相同的智能体循环，再把回复送回原始频道。

## 配置与工作区

默认实例位于 `~/.nanobot/` 下：

| 路径 | 含义 |
|---|---|
| `~/.nanobot/config.json` | 实例配置：提供商、模型默认设置、频道、工具、网关、API 和运行时选项 |
| `~/.nanobot/workspace/` | 智能体工作区：记忆、会话、心跳任务、cron 任务、技能和生成的产物 |

你可以用命令参数覆盖两者：

```bash
nanobot onboard --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot agent --config ./bot-a/config.json --workspace ./bot-a/workspace -m "Hello"
nanobot gateway --config ./bot-a/config.json --workspace ./bot-a/workspace
```

配置文件控制 nanobot 可以使用什么。工作区是 nanobot 为该实例保存状态的地方。

## 配置格式

`config.json` 同时接受 camelCase 与 snake_case 键。文档使用 camelCase，因为 nanobot 会以 camelCase 别名写回磁盘，例如 `apiKey`、`modelPresets`、`intervalS` 和 `maxToolResultChars`。

大多数示例是片段。请把它们合并到 `nanobot onboard` 创建的已有文件中；除非你想重置实例，否则不要替换整个文件。

## 一次智能体轮次

一次正常轮次遵循以下流程：

1. 频道收到用户消息并发布到消息总线。
2. 智能体循环选择一个会话键，并从工作区、技能、记忆、最近消息、频道元数据和运行时设置中构建上下文。
3. 提供商接收模型请求。
4. 如果模型请求调用工具，运行器执行它们并把结果反馈给模型。
5. 最终回复保存到会话，并通过频道回送。

无论消息从 CLI、WebUI、Telegram、Discord 还是其他频道开始，流程都是一样的。

## CLI、网关、API 和 WebUI

| 入口 | 命令 | 用途 |
|---|---|---|
| CLI 一次性 | `nanobot agent -m "..."` | 首次运行检查、脚本以及本地快速提问 |
| CLI 交互式 | `nanobot agent` | 具有持久会话历史的终端聊天 |
| 网关 | `nanobot gateway` | 聊天应用、WebUI、心跳、Dream 和长期服务模式 |
| OpenAI 兼容 API | `nanobot serve` | 通过 `/v1/chat/completions` 进行编程访问 |
| WebUI | `nanobot gateway` 加 WebSocket 频道 | 由 WebSocket 频道在端口 `8765` 上提供的浏览器工作台 |

网关健康检查端点位于 `gateway.port`（默认 `18790`）。浏览器 WebUI 由 WebSocket 频道提供服务（默认 `8765`），而不是由健康检查端点提供。

## 提供商和模型选择

当前模型通常应来自 `agents.defaults.modelPreset` 所选中的命名 `modelPresets` 条目。直接使用 `agents.defaults.provider` 和 `agents.defaults.model` 仍然会为较早或最简配置形成隐式的 `default` 预设。当前提供商按以下顺序解析：

1. 如果当前预设或隐式默认的 provider 不是 `"auto"`，nanobot 使用该 provider。
2. 如果 provider 是 `"auto"`，nanobot 会尝试根据模型名称、已配置的 API 密钥、本地提供商基础 URL 或网关提供商推断 provider。
3. OpenAI Codex 和 GitHub Copilot 等 OAuth 提供商需要显式登录，且必须在当前预设内显式选择 provider/model。

初次配置时，请在预设内固定 provider。这样更容易调试：

```json
{
  "modelPresets": {
    "primary": {
      "provider": "openrouter",
      "model": "anthropic/claude-opus-4.5"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

参见 [`providers.md`](./providers.md) 获取实际示例，参见 [`configuration.md#providers`](./configuration.md#providers) 获取完整的 provider 参考。

## 频道和会话

每个频道会把入站消息映射到一个会话键。这让相互独立的对话可以保留各自的历史。WebUI 还支持多个聊天以及针对项目工作区的工作区级元数据。

`agents.defaults.unifiedSession` 可以有意让多个频道共享一个会话，用于单用户多设备场景。如果你希望不同的人、群、频道或项目保留不同的上下文，就不要开启它。

## 记忆、会话和 Dream

nanobot 使用两个相关的存储：

| 存储 | 位置 | 用途 |
|---|---|---|
| 会话 | `<workspace>/sessions/*.jsonl` | 回放进入上下文的近期对话轮次 |
| 记忆 | `<workspace>/memory/MEMORY.md` 和 `<workspace>/memory/history.jsonl` | 长期事实和已整合的历史 |

Dream 是一个周期性的整合作业。它读取累积的历史并更新工作区记忆，让有用的上下文能在短期会话回放之外继续存在。

详细设计参见 [`memory.md`](./memory.md)。

## 工具与安全

工具会从内置模块和插件 entry points 自动发现。常见的工具组包括：

- 文件读/写/编辑和补丁；
- 具备可配置沙箱的 shell 执行；
- 具备 SSRF 检查的网页搜索和网页抓取；
- MCP 服务器；
- cron 提醒、本地触发器和心跳任务；
- 图片生成；
- 子智能体和运行时自省。

安全相关的控制项位于 [`configuration.md#security`](./configuration.md#security)。对于生产环境或共享聊天应用，还应配置频道访问控制，例如 `allowFrom`、配对或 WebSocket 令牌。

## 后台作业

当 `nanobot gateway` 启动时，它会运行工作区范围的自动化，并
注册系统作业：

- `dream`，当 `agents.defaults.dream.enabled` 为 true 时；
- `heartbeat`，当 `gateway.heartbeat.enabled` 为 true 时。

心跳读取 `<workspace>/HEARTBEAT.md`。如果文件在 `## Active Tasks` 下有任务，nanobot 会执行它们，并只把有用/可操作的结果发送到最近活跃的聊天目标。常规的“没有变化”结果会被抑制。

用户创建的提醒使用同一个 cron 服务，但与
受保护的心跳系统作业不同。它们作为其来源
聊天/会话中的定时轮次运行，通常把结果送回该频道。

本地触发器同样绑定到会话，但没有自己的
调度。请在目标聊天中使用 `/trigger <name>` 创建，然后当本地脚本或外部服务希望 nanobot 在该会话中响应时调用
`nanobot trigger <id> "<message>"`。Webhook 服务器、第三方认证和
事件到消息的格式化保留在 nanobot 之外。触发器交付会存储
在工作区中，直到关联的智能体轮次成功完成。如果
目标会话正忙，触发器会等待该会话空闲，而不是
被注入到当前正在进行的轮次中。该消息会作为自动化
轮次记录在会话中。交付语义是至少一次，因此外部系统应
容忍重复的触发消息；已到达智能体但失败的
交付会标记为失败，而不是无限重试。

## 下一步去哪

| 需求 | 阅读 |
|---|---|
| 首次可用安装 | [`quick-start.md`](./quick-start.md) |
| 提供商/模型配置 | [`providers.md`](./providers.md) |
| 聊天应用配置 | [`chat-apps.md`](./chat-apps.md) |
| 完整的配置参考 | [`configuration.md`](./configuration.md) |
| 运行时调试 | [`troubleshooting.md`](./troubleshooting.md) |
