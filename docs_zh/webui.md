# Nanobot WebUI：面向自托管 AI 智能体的浏览器工作台

<!-- Meta description: Run nanobot from a browser WebUI with persistent chat sessions, visible tool activity, workspace controls, Apps, MCP presets, Skills, settings, and Automations. -->

WebUI 是 nanobot 的浏览器工作台，将持久化的聊天会话、可见的
智能体活动、工作区控制、Apps、技能、设置和自动化集中于
一处。

已发布的 `nanobot-ai` wheel 已经包含 WebUI 打包产物。只有当
你需要修改前端本身时，才需要 `webui/` 源码目录。

## 打开 WebUI

使用启动器：

```bash
nanobot webui
```

`nanobot webui` 会在需要时创建配置/工作区，检查提供商设置，
在模型提供商未准备好时提供快速开始，在确认后启用本地
WebSocket 频道，在缺失时生成 WebUI 引导密钥，启动网关，
并打开浏览器。首次运行路径默认将 WebUI 绑定到 `127.0.0.1`，
因此局域网中的其他设备无法访问它。

如果你不想一直保留终端，可以在后台运行它：

```bash
nanobot webui --background
```

使用 `nanobot gateway status`、`nanobot gateway logs`、
`nanobot gateway restart` 和 `nanobot gateway stop` 来管理后台网关。

手动配置仍然可用。同机 localhost 的 WebUI 访问可以不使用
浏览器密码。当你有意将 WebUI 暴露到 localhost 之外或者希望
使用浏览器密码时，请设置 `tokenIssueSecret`：

```json
{
  "channels": {
    "websocket": {
      "enabled": true,
      "host": "127.0.0.1",
      "tokenIssueSecret": "your-webui-password",
      "websocketRequiresToken": true
    }
  }
}
```

WebUI 默认由 WebSocket 频道通过 `8765` 端口提供服务。网关的
健康检查端点默认使用 `18790`，它不是浏览器 UI。

## 用途

| 领域 | 用于 |
|---|---|
| 聊天 | 开启、切换、搜索、派生和删除浏览器会话 |
| 智能体活动 | 在上下文中查看思考过程、工具调用、带 diff 的文件编辑、命令输出和生成的产物 |
| 工作区 | 在请求文件或 shell 操作之前选择项目工作区 |
| 访问 | 从网关配置允许的本地能力中选择访问模式 |
| 输入框 | 发送文本、图片、语音输入、斜杠命令，以及对 Apps 或 MCP 预设的 `@` 提及 |
| Apps | 安装、测试、更新和使用本地 CLI App 适配器与 MCP 预设 |
| 技能 | 在依赖内置技能和工作区技能之前查看它们 |
| 自动化 | 查看、搜索、运行、暂停、编辑和删除定时或本地触发的智能体轮次 |
| 设置 | 调整模型、提供商、图像生成、语音、web 工具、运行时和安全选项 |

## 聊天工作区

侧边栏是会话切换器。每个会话保留自己的历史、标题、
工作区元数据和关联的自动化任务。当你希望使用独立的上下文时
新建一个会话；当你希望从已有的某个点继续而不影响原线程时
使用派生。

消息时间线同时展示面向用户的回复和智能体活动。当你需要
细节时，可以展开较长的工具或推理片段。

当智能体写入或编辑文件时，活动条目会显示目标路径、
状态、修改的行数，以及（如可用）统一 diff。使用 **View
diff** 展开变更；较大的 diff 可能隐藏未修改行或截断
内联预览。在文件编辑项上使用 **Open file** 打开只读文件
预览面板。

文件预览遵循当前会话的访问模式。受限工作区访问只能预览
所选工作区下的文件。当网关允许 Full Access 时，它可以预览
工作区之外的文件。

## 工作区与访问

在开始项目相关工作之前，请使用工作区选择器。这会为智能体
提供正确的项目上下文，用于文件路径、shell 命令和会话
元数据。

输入框中的访问控制用于控制当前聊天的本地能力级别。它
不会绕过你的网关、提供商、shell 沙箱或操作系统配置；它
只是在该 WebUI 会话已有的能力之间进行选择。

远程 WebUI 会话可能会降低当前工作区的访问级别。选择
不同的工作区或启用 Full Access 仍然仅限于本地和原生
客户端。

## 输入框

输入框支持纯文本消息、图片附件、在配置了转写时的语音
输入、斜杠命令，以及针对已安装 Apps 或 MCP 预设的 `@` 提及。
模型徽章显示当前模型或预设，并在设置未完成时链接回模型
设置。

要使用图像生成，请先配置一个图像提供商，然后在输入框中
使用 WebUI 图像模式。有关提供商配置和输出行为，请参见
[`image-generation.md`](./image-generation.md)。

## Apps

从侧边栏打开 Apps 来管理 nanobot 可以附加到聊天轮次的
工具。默认的 **Ready** 视图仅显示可以立即使用的工具：

- **Apps** 是 nanobot 在你机器上运行的本地命令行适配器。
  安装适配器并不会修改它所连接的原生桌面或 Web 应用。
- **Integrations** 是 MCP 服务器。预设提供已知的配置，
  自定义集成面板接受 stdio、HTTP 和 SSE 服务器。

Apps 有意不列出诸如 `api` 或 `bedrock` 之类的 nanobot 运行时
支持包。这些包用于启用提供商、服务器或频道；它们
不是可以通过 `@` 附加到某一轮次的工具。请从 **System**、
**Models** 或 **Web** 管理它们。PDF 和常见 Office 文档阅读器
已包含在 nanobot 中，并在附加文件时自动激活。
可选集成的等效 CLI 仍然是 `nanobot plugins`。参见
[`cli-reference.md`](./cli-reference.md#optional-features)。

一些 MCP 预设连接到托管的无密钥端点。例如，Firecrawl
预设使用 Firecrawl 托管的 MCP 端点，无需 API key 即可提供
搜索、抓取、爬取和抽取工具。这并不替代 nanobot 内置的
web 搜索提供商；当某一轮次需要 Firecrawl 更丰富的 web 数据
工具时，请通过 `@` 提及 Firecrawl MCP 预设。

在某个 App 或集成可用后，从输入框使用 `@` 提及它，
即可把该工具附加到下一条消息。

## 技能

Skills 视图展示智能体可用的技能说明，包括内置技能和
工作区提供的技能。在请求 nanobot 执行某项任务之前，
查看此视图可以了解它是否已经具备针对该任务的专门工作流。

## 自动化

自动化是稍后在关联的聊天/会话中运行的智能体轮次。
它们应当从其预期运行的聊天、频道或会话中创建，以便
nanobot 保留正确的目标上下文。当自动化运行时，它通常
将结果送回该关联聊天。

关于完整的自动化模型、创建流程、触发器 CLI 用法和交付
语义，请参见 [`automations.md`](./automations.md)。

面向用户的自动化有两种类型：

- 定时自动化，由智能体的 cron 工具创建，按照某个时间、
  间隔或 cron 表达式运行。
- 本地触发器，通过 `/trigger <name>` 创建，当你调用本地
  命令（如 `nanobot trigger trg_8K4P2Q9X "Review PR #4502"`）
  时运行。

对于希望在没有可报告内容时保持安静的周期性后台检查，
请通过编辑 `HEARTBEAT.md` 使用受保护的心跳任务，而不要
创建聊天自动化。

使用 Automations 视图可以：

- 按全部、活动、暂停、需要处理或系统作业过滤。
- 按任务名称、消息、触发命令、关联聊天、调度或状态搜索。
- 按下次运行、上次运行、更新时间或名称排序。
- 立即运行定时自动化。
- 暂停或恢复、重命名或删除用户创建的自动化。
- 复制本地触发器的 CLI 命令。
- 查看受保护的系统自动化而不做修改。

搜索接受纯文本以及字段过滤器，例如 `name:backup`、
`chat:WeChat`、`schedule:09:30`、`cron:"0 23 * * *"`、`trigger`
和 `status:paused`。

没有关联聊天的自动化无法从 WebUI 启用或运行，因为
nanobot 不知道该把定时轮次送到哪里。请从目标聊天或频道
重新创建它，让该自动化具备完整的上下文。

本地触发器没有 WebUI 的 "Run now" 操作，因为每次运行都
需要一条消息。请使用复制得到的 `nanobot trigger ...` 命令，
并将 `"message"` 替换为需要发送的内容。

## 设置

Settings 是浏览器会话以及由网关支持的运行时配置的控制界面。
使用它可以查看或调整模型预设、提供商可见性、
图像生成、语音转写、web 工具、Apps、Automations、
Skills、运行时身份，以及高级安全控制。

某些设置会立即生效。影响网关或智能体进程的运行时设置
可能需要重启；WebUI 会在相关控件旁边显示该要求。

仅限浏览器的显示偏好（例如文件编辑显示模式）会立即
在当前浏览器生效，且不会更改网关配置。

## 局域网访问

要从同一网络中的另一台设备打开 WebUI，请将 WebSocket
频道绑定到所有网卡，并设置一个 token 或 token 签发密钥：

```json
{
  "channels": {
    "websocket": {
      "host": "0.0.0.0",
      "port": 8765,
      "tokenIssueSecret": "your-secret-here"
    }
  }
}
```

如果没有配置 `token` 或 `tokenIssueSecret`，当 `host` 设置为
`"0.0.0.0"` 时网关会拒绝启动。网关启动后，从另一台设备
打开 `http://<your-ip>:8765` 并在登录表单中输入该密钥。

持有有效 token 的远程 WebUI 客户端可以查看并使用 Apps。
默认情况下，安装缺失的 nanobot 支持包（例如添加频道
依赖）等操作会被阻止。若要允许受信任的远程管理员通过
WebUI 修改 Python 环境，请显式启用：

```json
{
  "tools": {
    "webuiAllowRemotePackageInstall": true
  }
}
```

仅在每个已认证 WebUI 用户都可信、允许其修改 nanobot 所在
Python 环境的私有部署中才使用此选项。如果你通过 Nginx、
Caddy、Cloudflare Tunnel 或类似服务发布 WebUI，请把它视为
远程访问；除非你有意为之，否则请保持包安装为禁用状态。

可选功能的安装会使用 pip 已配置的软件包索引，包括
`PIP_INDEX_URL`。

当 WebUI 暴露到私有、受信任网络之外时，请保持远程包
安装为禁用状态。

## 故障排查

如果页面无法打开，请按以下顺序检查：

1. 在同一 Python 环境中 `nanobot agent -m "Hello!"` 可正常工作。
2. `~/.nanobot/config.json` 中没有显式将 `channels.websocket.enabled` 设置为 `false`。
3. `nanobot gateway` 仍在运行。
4. 你打开的是端口 `8765`，而不是网关健康检查端口。
5. 局域网访问使用 `host: "0.0.0.0"` 并配置了 token 或 token 签发密钥。

有关详细诊断，请参见
[`troubleshooting.md#webui-problems`](./troubleshooting.md#webui-problems)。
关于前端开发，请参见 [`../webui/README.md`](../webui/README.md)。
