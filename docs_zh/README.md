# nanobot 文档

如需查看已发布的正式版文档，请访问 [nanobot.wiki](https://nanobot.wiki/docs/latest/getting-started/nanobot-overview)。此目录下的页面跟踪当前代码仓库，可能会描述尚未发布到线上站点的功能。

如果你从未使用过终端或编辑过配置文件，请先阅读 [`start-without-technical-background.md`](./start-without-technical-background.md)。否则，请从 [`quick-start.md`](./quick-start.md) 开始，使用 `nanobot webui` 打开浏览器工作台，并在需要进行底层诊断时使用终端检查。

本文档中的大多数 JSON 示例都是需要合并到 `~/.nanobot/config.json` 中的片段，而非完整的替换文件。

提供商示例是具体的操作演示，不是排名或推荐。请使用你实际掌握其密钥、端点和模型 ID 的提供商。

如果你发现文档错误、过时的命令或令人困惑的步骤，请提交 issue：<https://github.com/HKUDS/nanobot/issues>。

## 选择路径

| 你的情况 | 从这里开始 | 然后使用 |
|---|---|---|
| 不熟悉终端和配置文件 | [`start-without-technical-background.md`](./start-without-technical-background.md) | 如果首次回复失败，请查看 [`troubleshooting.md`](./troubleshooting.md) |
| 熟悉粘贴命令和 JSON | [`quick-start.md`](./quick-start.md) | [`provider-cookbook.md`](./provider-cookbook.md) 提供可粘贴的提供商配置 |
| 运营一个长期运行的机器人 | [`concepts.md`](./concepts.md) | [`chat-apps.md`](./chat-apps.md)、[`webui.md`](./webui.md) 和 [`deployment.md`](./deployment.md) |
| 集成或扩展 nanobot | [`architecture.md`](./architecture.md) | [`configuration.md`](./configuration.md)、[`openai-api.md`](./openai-api.md)、[`python-sdk.md`](./python-sdk.md)、[`development.md`](./development.md) 和 [`channel-plugin-guide.md`](./channel-plugin-guide.md) |

## 从这里开始

| 目标 | 阅读 | 结果 |
|---|---|---|
| 无技术背景起步 | [`start-without-technical-background.md`](./start-without-technical-background.md) | 一条命令完成安装、终端基础、配置、API 密钥和首次回复 |
| 安装并获得首次回复 | [`quick-start.md`](./quick-start.md) | 一个可用的 CLI 智能体和已知可用的配置路径 |
| 理解各部分如何协同 | [`concepts.md`](./concepts.md) | 关于配置、工作区、网关、频道、工具、记忆和会话的心智模型 |
| 选择或更换模型提供商 | [`providers.md`](./providers.md) | 无需通读完整的配置参考即可正确配对提供商/模型 |
| 复制一个提供商配置示例 | [`provider-cookbook.md`](./provider-cookbook.md) | 可粘贴的 OpenRouter、OpenAI、Anthropic、本地模型、回退和 Langfuse 配置 |
| 修复首次运行或运行时问题 | [`troubleshooting.md`](./troubleshooting.md) | 诊断顺序和针对常见故障的定向检查 |

## 任务指南

当你已经知道想要的工作流，并且不想先浏览完整参考时使用这些页面。

| 目标 | 指南 |
|---|---|
| 构建个人 AI 智能体 | [`guides/build-a-personal-ai-agent.md`](./guides/build-a-personal-ai-agent.md) |
| 运行自托管 AI 智能体 | [`guides/self-hosted-ai-agent.md`](./guides/self-hosted-ai-agent.md) |
| 使用浏览器 AI 智能体 WebUI | [`guides/ai-agent-webui.md`](./guides/ai-agent-webui.md) |
| 将 AI 智能体连接到聊天应用 | [`guides/chat-app-ai-agent.md`](./guides/chat-app-ai-agent.md) |
| 运行长期任务的智能体 | [`guides/long-running-ai-agent.md`](./guides/long-running-ai-agent.md) |
| 调度或触发智能体轮次 | [`automations.md`](./automations.md) |
| 为智能体添加长期记忆 | [`guides/ai-agent-memory.md`](./guides/ai-agent-memory.md) |
| 为智能体添加 MCP 工具 | [`guides/mcp-tools-for-ai-agents.md`](./guides/mcp-tools-for-ai-agents.md) |
| 从 Python 中运行智能体 | [`guides/python-ai-agent-sdk.md`](./guides/python-ai-agent-sdk.md) |
| 暴露 OpenAI 兼容的智能体 API | [`guides/openai-compatible-agent-api.md`](./guides/openai-compatible-agent-api.md) |
| 部署长期运行的智能体网关 | [`guides/deploy-nanobot-gateway.md`](./guides/deploy-nanobot-gateway.md) |

平台特定的聊天指南：
[`Telegram`](./guides/telegram-ai-agent.md)、
[`Discord`](./guides/discord-ai-agent.md)、
[`Slack`](./guides/slack-ai-agent.md)、
[`飞书`](./guides/feishu-ai-agent.md)、
[`WhatsApp`](./guides/whatsapp-ai-agent.md)、
[`微信`](./guides/wechat-ai-agent.md)、
[`QQ`](./guides/qq-ai-agent.md)、
[`邮件`](./guides/email-ai-agent.md) 和
[`Mattermost`](./guides/mattermost-ai-agent.md)。

配置指南：
[`MCP 工具`](./guides/configure-mcp-tools.md)、
[`网页搜索`](./guides/configure-web-search.md)、
[`模型回退`](./guides/configure-model-fallback.md)、
[`OpenAI 兼容提供商`](./guides/configure-openai-compatible-provider.md)、
[`Langfuse`](./guides/configure-langfuse-observability.md)、
[`本地安全`](./guides/secure-local-ai-agent.md) 和
[`网关部署`](./guides/deploy-nanobot-gateway.md)。

## 首次回复成功之后

不要一次性配置所有内容。选择一个下一步的方向：

如果本地 `nanobot agent` 会话已经能正常回复，你也可以让 nanobot 帮助自己进行配置：让它阅读相关文档，检查你当前的配置，进行一个具体的下一步更改，并告诉你何时运行 `/restart`。

| 下一目标 | 阅读 | 首要检查 |
|---|---|---|
| 在浏览器中使用 nanobot | [`webui.md`](./webui.md) | 运行 `nanobot webui` 并打开本地浏览器工作台 |
| 通过聊天应用对话 | [`chat-apps.md`](./chat-apps.md) | 合并一个频道片段，运行 `nanobot channels status`，保持 `nanobot gateway` 运行 |
| 更换提供商或添加回退 | [`provider-cookbook.md`](./provider-cookbook.md) | 保留命名的 `modelPresets` 并设置 `agents.defaults.modelPreset` |
| 从 Python 调用 nanobot | [`python-sdk.md`](./python-sdk.md) | 从代码中复用相同的配置/工作区，然后运行或流式返回一次智能体轮次 |
| 在长期运营前先理解 | [`concepts.md`](./concepts.md) | 了解配置、工作区、网关、会话、记忆和工具的含义 |
| 诊断新故障 | [`troubleshooting.md`](./troubleshooting.md) | 从 `nanobot status` 开始，然后 `nanobot agent -m "Hello!"` |

## 使用 nanobot

| 目标 | 阅读 | 结果 |
|---|---|---|
| 打开内置浏览器 UI | [`webui.md`](./webui.md) | `nanobot webui`、聊天工作区、Apps、Skills、Automations 和设置 |
| 连接 Telegram、Discord、微信、Slack、邮件、Mattermost 或其他聊天应用 | [`chat-apps.md`](./chat-apps.md) | 一个由网关支持、带访问控制的聊天频道 |
| 使用自动化 | [`automations.md`](./automations.md) | 定时自动化、本地触发器、心跳、WebUI 管理和交付行为 |
| 使用斜杠命令 | [`chat-commands.md`](./chat-commands.md) | 配对、模型预设、本地触发器、心跳任务和聊天侧控制 |
| 生成图片 | [`image-generation.md`](./image-generation.md) | 图片提供商配置、WebUI 图片模式和产物行为 |
| 运行多个隔离的机器人 | [`multiple-instances.md`](./multiple-instances.md) | 独立的配置、工作区、端口和会话 |
| 在终端之外部署 | [`deployment.md`](./deployment.md) | Docker、systemd 用户服务和 macOS LaunchAgent 配置 |
| 加入智能体社区 | [`agent-social-network.md`](./agent-social-network.md) | 外部智能体社区配置 |

## 参考

| 领域 | 阅读 | 最适合 |
|---|---|---|
| 完整的配置模式 | [`configuration.md`](./configuration.md) | 精确的字段、默认值、提供商表、网页工具、MCP、安全性和运行时选项 |
| CLI 命令 | [`cli-reference.md`](./cli-reference.md) | 命令名称、常用参数和入口 |
| 架构 | [`architecture.md`](./architecture.md) | 关于核心流程、提供商、频道、工具、WebUI、记忆、安全性和扩展点的源码级运行时图 |
| 版本存档 | [`release-archive.md`](./release-archive.md) | 从 README 中迁出的旧版本和每日更新亮点 |
| 开发 | [`development.md`](./development.md) | 为贡献者提供的添加提供商和转写适配器的说明 |
| 记忆 | [`memory.md`](./memory.md) | 会话历史、Dream 整合、记忆文件和版本管理 |
| 可观测性 | [`configuration.md#langfuse-observability`](./configuration.md#langfuse-observability) | Langfuse 追踪配置和所需的环境变量 |
| WebSocket 协议 | [`websocket.md`](./websocket.md) | 自定义客户端、令牌签发、多路复用聊天、媒体和协议事件 |
| OpenAI 兼容 API | [`openai-api.md`](./openai-api.md) | `/v1/chat/completions`、`/v1/models`、文件上传和 SDK 兼容使用 |
| Python SDK | [`python-sdk.md`](./python-sdk.md) | SDK 入门、会话、流式、模型覆盖、运行时辅助函数和钩子 |
| 运行时自省 | [`my-tool.md`](./my-tool.md) | 检查和调整当前智能体运行 |

## 快速查找

| 需求 | 跳转至 |
|---|---|
| 提供商/模型解析顺序 | [`providers.md#provider-resolution`](./providers.md#provider-resolution) |
| 模型预设和回退链 | [`providers.md#model-presets`](./providers.md#model-presets) 和 [`providers.md#fallback-models`](./providers.md#fallback-models) |
| Langfuse 环境变量 | [`configuration.md#langfuse-observability`](./configuration.md#langfuse-observability) |
| WebSocket/WebUI 协议详情 | [`websocket.md`](./websocket.md) |
| OpenAI 兼容 API 用法 | [`openai-api.md`](./openai-api.md) |
| Python SDK 用法 | [`python-sdk.md`](./python-sdk.md) |
| 定时自动化和本地触发器 | [`automations.md`](./automations.md) |
| 多份配置、工作区和端口 | [`multiple-instances.md`](./multiple-instances.md) |
| 安全、沙箱和 SSRF 控制 | [`configuration.md#security`](./configuration.md#security) |
| 频道插件开发 | [`channel-plugin-guide.md`](./channel-plugin-guide.md) |

## 扩展 nanobot

| 目标 | 阅读 | 结果 |
|---|---|---|
| 添加提供商或转写适配器 | [`development.md`](./development.md) | 一条与注册表/模式对齐的实现路径 |
| 添加聊天频道插件 | [`channel-plugin-guide.md`](./channel-plugin-guide.md) | 通过 entry points 发现的打包频道 |
| 添加自定义 MCP 服务器 | [`configuration.md#mcp-model-context-protocol`](./configuration.md#mcp-model-context-protocol) | 通过 MCP 向智能体暴露的外部工具 |
| 调整工具安全 | [`configuration.md#security`](./configuration.md#security) | Shell 沙箱、工作区限制和 SSRF 策略 |

## 阅读策略

当你不确定该看哪里时，按此顺序使用文档：

1. 如果你不熟悉终端命令或配置文件，[`start-without-technical-background.md`](./start-without-technical-background.md) 会解释配置用语，并使用一个具体的提供商示例，让你每次只需做一个决定。
2. [`quick-start.md`](./quick-start.md) 验证安装、配置加载和提供商访问。
3. [`concepts.md`](./concepts.md) 解释运行时模型，让后续页面更易于阅读。
4. [`provider-cookbook.md`](./provider-cookbook.md) 提供可粘贴的提供商、回退、本地模型和 Langfuse 配置示例。
5. 一个任务指南，例如 [`chat-apps.md`](./chat-apps.md)、[`image-generation.md`](./image-generation.md) 或 [`deployment.md`](./deployment.md)，让某个工作流跑起来。
6. [`configuration.md`](./configuration.md) 是需要特定字段、默认值或高级选项时的权威来源。
7. [`troubleshooting.md`](./troubleshooting.md) 帮助判断故障属于安装、配置、提供商、网关、频道还是工具层面。
