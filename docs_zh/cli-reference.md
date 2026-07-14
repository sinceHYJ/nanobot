# CLI 参考

当你已经知道要运行什么并需要命令形式时，使用此页面。若需要一次入门指引，请先阅读 [`quick-start.md`](./quick-start.md)。

## 选择一条命令

| 目标 | 命令 | 备注 |
|---|---|---|
| 检查安装 | `nanobot --version` | 如果失败，请尝试 `python -m nanobot --version` |
| 创建或刷新配置 | `nanobot onboard` | 创建 `~/.nanobot/config.json` 和 `~/.nanobot/workspace/` |
| 非交互式刷新配置 | `nanobot onboard --refresh` | 保留已有值并补齐缺失默认字段，不产生交互提示 |
| 使用引导式配置 | `nanobot onboard --wizard` | 当你更喜欢提示而非手写 JSON 时使用 |
| 打开浏览器工作台 | `nanobot webui` | 准备本地 WebUI 设置、启动网关并打开浏览器 |
| 不调用模型即可检查配置 | `nanobot status` | 汇总所选配置、工作区、当前模型和提供商 |
| 发送一条测试消息 | `nanobot agent -m "Hello!"` | 首次证明安装、配置、提供商、模型和工作区都可用 |
| 在终端中聊天 | `nanobot agent` | 交互式本地聊天；使用 `exit`、`/exit`、`:q` 或 `Ctrl+D` 退出 |
| 直接运行网关 | `nanobot gateway` | 用于 WebUI、聊天应用、cron 和心跳的服务/运维命令 |
| 发送本地触发消息 | `nanobot trigger <id> "message"` | 需先在目标聊天/会话中通过 `/trigger <name>` 创建 |
| 提供 OpenAI 兼容 API | `nanobot serve` | 启动 `/v1/chat/completions`、`/v1/models` 和 `/health` |
| 检查聊天频道配置 | `nanobot channels status` | 启动 `nanobot gateway` 之前很有用 |
| 管理可选功能 | `nanobot plugins list` | 列出你可以开启的频道和可选能力 |
| 登录 QR/OAuth 风格的频道 | `nanobot channels login <channel>` | 用于 WhatsApp、微信等频道 |
| 登录 OAuth 模型提供商 | `nanobot provider login <provider>` | 用于 OpenAI Codex、GitHub Copilot 等 OAuth 提供商 |

## 全局

```bash
nanobot --help
nanobot --version
python -m nanobot --help
python -m nanobot --version
```

当包已安装但 `nanobot` 脚本不在 `PATH` 上时，`python -m nanobot ...` 很有用。

## 常见用法

大多数日常命令使用默认的配置和工作区。高级或多实例运行通常会显式传入两个路径：

```bash
nanobot agent --config ./bot-a/config.json --workspace ./bot-a/workspace -m "Hello"
nanobot gateway --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot serve --config ./bot-a/config.json --workspace ./bot-a/workspace
```

当你需要查看启动或运行时日志时，在长期运行的进程上使用 `--verbose`：

```bash
nanobot gateway --verbose
nanobot serve --verbose
```

长期运行的命令会一直运行，直到你停止它们。在该终端中按 `Ctrl+C`
以停止前台的 `nanobot gateway` 或 `nanobot serve`。如果你使用 `--background`
启动了网关，请使用 `nanobot gateway stop`。

## 初始化

| 命令 | 描述 |
|---|---|
| `nanobot onboard` | 初始化或刷新默认配置和工作区 |
| `nanobot onboard --refresh` | 无交互刷新现有配置，保留已有值 |
| `nanobot onboard --wizard` | 使用交互式配置向导 |
| `nanobot onboard --config <path> --workspace <path>` | 初始化或刷新特定实例 |

默认路径：

| 路径 | 默认 |
|---|---|
| 配置 | `~/.nanobot/config.json` |
| 工作区 | `~/.nanobot/workspace/` |

## Agent CLI

| 命令 | 描述 |
|---|---|
| `nanobot agent -m "Hello!"` | 发送一条消息后退出 |
| `nanobot agent` | 启动交互式终端聊天 |
| `nanobot agent --session <id>` | 使用指定的会话键 |
| `nanobot agent --workspace <path>` | 覆盖工作区 |
| `nanobot agent --config <path>` | 使用指定的配置文件 |
| `nanobot agent --no-markdown` | 打印纯文本，而不是使用 Rich 渲染的 Markdown |
| `nanobot agent --logs` | 聊天期间显示运行时日志 |

在交互模式下，`Enter` 发送当前消息。按 `Alt+Enter` 在发送前添加换行。

使用 `exit`、`quit`、`/exit`、`/quit`、`:q` 或 `Ctrl+D` 退出交互模式。

## WebUI

| 命令 | 描述 |
|---|---|
| `nanobot webui` | 在需要时创建配置/工作区，经过确认后启用本地 WebUI 频道，启动网关，并打开 `http://127.0.0.1:8765` |
| `nanobot webui --background` | 启动或复用后台网关，然后打开 WebUI |
| `nanobot webui --no-open` | 准备并启动 WebUI，但不打开浏览器 |
| `nanobot webui --port <port>` | 设置 WebUI/WebSocket 端口 |
| `nanobot webui --gateway-port <port>` | 覆盖网关健康检查端口 |
| `nanobot webui --yes` | 不经确认应用安全的本地 WebUI 默认值；提供商凭证仍需交互式设置 |

首次运行 WebUI 默认绑定到 `127.0.0.1`。在把 WebSocket 频道暴露到 localhost 之外之前，请手动配置并设置 WebUI 密码。

## 网关

`nanobot gateway` 启动已启用的聊天频道，配置时启用 WebUI/WebSocket，启动 cron 支撑的系统作业、Dream、心跳以及健康检查端点。大多数本地浏览器用户应从 `nanobot webui` 开始；在服务管理、聊天应用运行以及高级部署时才直接使用 `gateway`。默认情况下它在前台运行，这样保留原有脚本和终端工作流不变。在你希望获得一个可通过 CLI 管理的本地 macOS、Linux 或 Windows 进程时，使用 `--background`。

| 命令 | 描述 |
|---|---|
| `nanobot gateway` | 使用配置默认值在前台启动网关 |
| `nanobot gateway --verbose` | 显示详细运行时输出 |
| `nanobot gateway --port <port>` | 覆盖健康检查端点的 `gateway.port` |
| `nanobot gateway --workspace <path>` | 覆盖工作区 |
| `nanobot gateway --config <path>` | 使用指定的配置文件 |
| `nanobot gateway --background` | 以后台进程启动网关 |
| `nanobot gateway status` | 显示记录的后台网关 PID、状态文件和日志文件 |
| `nanobot gateway logs --no-follow` | 打印近期的后台网关日志并退出 |
| `nanobot gateway logs` | 跟随打印后台网关日志 |
| `nanobot gateway restart` | 使用当前配置重启记录的后台网关 |
| `nanobot gateway stop` | 停止记录的后台网关 |
| `nanobot gateway install-service` | 安装 systemd 用户服务或 macOS LaunchAgent |
| `nanobot gateway install-service --dry-run` | 预览生成的服务文件和系统命令 |
| `nanobot gateway uninstall-service` | 移除已安装的系统服务 |

对于自定义实例，向管理命令传入相同的选择器参数：

```bash
nanobot gateway --background --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot gateway status --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot gateway stop --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot gateway install-service --config ./bot-a/config.json --workspace ./bot-a/workspace --name bot-a
```

`--background` 是一个轻量的分离进程。`install-service` 用于
登录/开机集成：Linux 使用 systemd 用户服务；macOS 使用
LaunchAgent plist。系统服务由 OS 监督程序在前台运行网关，而不是嵌套另一个后台进程。

默认健康检查端点：

```text
http://127.0.0.1:18790/health
```

内置 WebUI 由 WebSocket 频道提供服务，通常在端口 `8765` 上，而不是网关健康检查端点。

## 本地触发器

`nanobot trigger` 通过一个由聊天/会话中 `/trigger <name>` 创建的
触发器交付一条本地消息。

```bash
nanobot trigger trg_8K4P2Q9X "Review PR #4502"
```

保持 `nanobot gateway` 运行，以便消息能够送达关联的
聊天/会话。该消息会作为自动化轮次记录在会话中，
而不是作为用户输入的普通聊天消息。

命令写入一个工作区本地的持久队列。如果 `nanobot gateway` 尚未
运行，消息会在该工作区中等待。如果目标会话
已经在运行一次轮次，触发器会等待该会话空闲。如果
网关在认领了一次交付但在关联轮次完成之前退出，
下一次网关启动会重新入队该交付。该队列是至少一次，而不是
恰好一次，因此在进程被中断后，同一条消息可能会再次
被交付。如果智能体接收到交付但轮次失败，
该交付会被标记为失败，而不是无限重试。每次交付还会写入
`<workspace>/triggers/runs` 下的审计记录。每个工作区只运行
一个网关消费者；该本地队列不是分布式多消费者队列。

当其他本地进程生成消息时，使用 stdin：

```bash
generate-report | nanobot trigger trg_8K4P2Q9X
```

选项：

| 命令 | 描述 |
|---|---|
| `nanobot trigger <id> "message"` | 通过触发器交付一条消息 |
| `nanobot trigger <id>` | 从 stdin 读取消息 |
| `nanobot trigger --config <path> <id> "message"` | 使用指定配置对应的工作区 |
| `nanobot trigger --workspace <path> <id> "message"` | 使用指定的工作区 |

触发器在 WebUI Automations 视图中管理，而不是通过独立的
`list`、`revoke` 或 `delete` CLI 子命令。在那里你可以暂停/恢复、
重命名、删除、搜索，并复制每个触发器的命令。

对于 webhook 或其他外部系统，请自行运行小型服务，让它
在决定 nanobot 应接收哪条消息后调用此 CLI。

关于更广泛的自动化模型、WebUI 管理和交付行为，参见
[Automations](./automations.md)。

## OpenAI 兼容 API

| 命令 | 描述 |
|---|---|
| `nanobot serve` | 启动 `/v1/chat/completions`、`/v1/models` 和 `/health` |
| `nanobot serve --host <host>` | 覆盖 API 绑定主机 |
| `nanobot serve --port <port>` | 覆盖 API 端口 |
| `nanobot serve --timeout <seconds>` | 覆盖单请求超时 |
| `nanobot serve --verbose` | 显示运行时日志 |
| `nanobot serve --workspace <path>` | 覆盖工作区 |
| `nanobot serve --config <path>` | 使用指定的配置文件 |

默认 API 端点：

```text
http://127.0.0.1:8900
```

公共绑定（`0.0.0.0` 或 `::`）需要 `api.apiKey`；请在 API 路由上以 Bearer token 发送。

请求示例参见 [`openai-api.md`](./openai-api.md)。

## 状态

```bash
nanobot status
```

在不调用模型的前提下显示配置路径、工作区路径、当前模型和提供商摘要。

| 命令 | 描述 |
|---|---|
| `nanobot status` | 检查默认实例 |
| `nanobot status --config <path>` | 检查指定配置 |
| `nanobot status --config <path> --workspace <path>` | 检查指定配置并覆盖工作区 |

## 频道

| 命令 | 描述 |
|---|---|
| `nanobot channels status` | 显示已配置的频道状态 |
| `nanobot channels status --config <path>` | 显示指定配置的频道状态 |
| `nanobot channels login <channel>` | 对支持的频道运行交互式登录 |
| `nanobot channels login <channel> --force` | 即便已有凭证也重新认证 |
| `nanobot channels login <channel> --config <path>` | 使用指定的配置文件 |
| `nanobot plugins list --config <path>` | 显示指定配置的插件/频道启用状态 |

示例：

```bash
nanobot channels login whatsapp
nanobot channels login weixin
nanobot channels status
```

频道特定配置参见 [`chat-apps.md`](./chat-apps.md)。

## 可选功能

在你希望 nanobot 添加或移除某项内置能力而
不想手动编辑 JSON 时使用这些命令。启用可能会先安装支持包。
禁用适用于 Telegram、Matrix 或 Slack 等频道；它会保留你
已保存的设置并关闭该频道。

出于兼容性，命令名 `plugins` 保留，但这些条目是
nanobot 运行时支持包，而不是 WebUI Apps 中显示的用户可调用工具。
它们不能通过 `@` 附加到某次聊天轮次。

| 功能名称 | 启用内容 |
|---|---|
| `api` | OpenAI 兼容的 `nanobot serve` 进程所需依赖 |
| `azure` | 面向 Azure 托管模型的 Azure identity 支持 |
| `bedrock` | AWS Bedrock 模型提供商支持 |
| `langfuse` | 面向 OpenAI 兼容提供商的 Langfuse 追踪支持 |
| `olostep` | Olostep 网页搜索提供商支持 |
| 频道名称，例如 `telegram` 或 `slack` | 连接器包和已保存的频道启用状态 |

| 命令 | 描述 |
|---|---|
| `nanobot plugins list` | 显示可用频道和可选能力 |
| `nanobot plugins enable <name>` | 安装缺失的支持并启用该功能或频道 |
| `nanobot plugins enable <name> --logs` | 启用时显示包安装日志 |
| `nanobot plugins disable <channel>` | 关闭频道而不删除其已保存的设置 |
| `nanobot plugins list --config <path>` | 读取指定的配置文件 |
| `nanobot plugins enable <name> --config <path>` | 更新指定的配置文件 |
| `nanobot plugins disable <channel> --config <path>` | 在指定配置文件中关闭频道 |

文档和 PDF 阅读已包含在标准安装中。旧的
`nanobot plugins enable documents` 和 `nanobot plugins enable pdf` 命令
仍作为空操作的兼容别名被接受。

## 提供商 OAuth

| 命令 | 描述 |
|---|---|
| `nanobot provider login openai-codex --set-main` | 认证 Codex 并选择其当前默认模型 |
| `nanobot provider login github-copilot --set-main` | 认证 GitHub Copilot 并选择其当前默认模型 |
| `nanobot provider logout openai-codex` | 移除 OpenAI Codex OAuth 状态 |
| `nanobot provider logout github-copilot` | 移除 GitHub Copilot OAuth 状态 |

关于 OAuth 提供商何时需要显式选择 provider/model，参见 [`providers.md`](./providers.md#oauth-providers)。

## 有用的首要检查

```bash
nanobot --version
nanobot status
nanobot agent -m "Hello!"
```

如果这些失败，请先使用 [`troubleshooting.md`](./troubleshooting.md)，再去调试 WebUI、聊天应用、Docker、systemd 或 SDK 集成。
