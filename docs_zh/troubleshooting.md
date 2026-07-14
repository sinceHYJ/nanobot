# 故障排查

使用本页来定位故障所在。先从证明最多、影响面最小的表面开始：本地 CLI 优先，然后是网关，然后是 WebUI 或聊天应用。

## 快速诊断顺序

按顺序运行：

```bash
nanobot --version
nanobot status
nanobot agent -m "Hello!"
```

只有当 CLI 可用后，再运行：

```bash
nanobot gateway
```

这样可以把故障按层分离：

| 层 | 验证内容 |
|---|---|
| `nanobot --version` | 安装和 shell 命令可发现性 |
| `nanobot status` | 配置路径、工作区路径、当前模型和提供商摘要 |
| `nanobot agent -m "Hello!"` | 配置加载、提供商/模型访问、工作区写入以及智能体循环 |
| `nanobot gateway` | 频道启动、cron 系统作业、心跳、WebUI/WebSocket 和健康检查端点 |

如果 `nanobot agent -m "Hello!"` 失败，先修复它，再去调试 WebUI、Telegram、Discord、Docker、systemd 或任何聊天应用。

## 如何阅读 `nanobot status`

`nanobot status` 不会调用模型。它只检查 nanobot 是否能找到所选的配置、所选的工作区、当前模型或预设，以及提供商配置摘要。

输出形如：

```text
nanobot Status

Config: /path/to/config.json ✓
Workspace: /path/to/workspace ✓
Model: provider/model-name (preset: primary)
Provider A: not set
Provider B: ✓
Local Provider: ✓ http://localhost:11434/v1
OAuth Provider: ✓ (OAuth)
```

按以下方式阅读：

| 行 | 好的迹象 | 若看起来不对该做什么 |
|---|---|---|
| `Config` | 指向你想使用的配置文件并显示 `✓`。 | 运行 `nanobot onboard`，或在测试非默认实例时向 `nanobot agent`、`gateway` 或 `serve` 传入 `--config`。 |
| `Workspace` | 指向你想使用的工作区并显示 `✓`。 | 运行 `nanobot onboard`、创建目录、修复权限，或在支持该参数的命令上传入 `--workspace`。 |
| `Model` | 显示你预期的当前模型或预设名称。 | 将 `agents.defaults.modelPreset` 设为目标预设，或在聊天会话中切换过模型时检查 `/model`。 |
| 提供商行 | 当前预设使用的提供商显示 `✓`、OAuth 标记或本地 URL。 | 先只配置当前提供商。未使用的提供商显示 `not set` 是正常的。 |

如果 `nanobot status` 看起来没问题但 `nanobot agent -m "Hello!"` 失败，说明安装和配置路径大概率没问题。继续查看 [提供商和模型问题](#provider-and-model-problems)。

## 安装问题

在做安装检查和模块回退时使用同一个 Python 命令。在 macOS/Linux 上可能是 `python3`；在 Windows 上可能是 `python` 或 `py`。

| 现象 | 检查 |
|---|---|
| `python: command not found` | 在 macOS/Linux 上试 `python3 --version`，在 Windows 上试 `py --version`。然后将文档命令中的 `python` 替换为生效的那个命令。 |
| `curl: command not found` | macOS/Linux 一条命令安装器无法下载脚本。请安装 curl，或使用手动隔离安装，例如 `uv tool install nanobot-ai` 或 `pipx install nanobot-ai`。 |
| 无法识别 `irm` | PowerShell 无法运行下载助手。请手动安装：`uv tool install nanobot-ai`、`pipx install nanobot-ai`，或在你可控的环境中使用 `py -m pip install nanobot-ai`。 |
| 无法下载 `raw.githubusercontent.com` | 你的网络、代理或防火墙阻断了安装脚本下载。请从 PyPI 手动安装，或配置代理后重新运行。 |
| `nanobot: command not found` | 使用模块形式，例如 `python -m nanobot ...`、`python3 -m nanobot ...` 或 `py -m nanobot ...`。用相同的 Python 命令重装，或把该 Python 的脚本目录加入 `PATH`。 |
| `No module named nanobot` | 你运行的 Python 与安装时的不同。使用与安装命令一致的方式运行 `python -m pip show nanobot-ai`、`python3 -m pip show nanobot-ai` 或 `py -m pip show nanobot-ai`。 |
| `pip is not available` | 当安装器使用虚拟环境时，它会尝试 `python -m ensurepip --upgrade`。如果失败，请为该 Python 安装 pip，或使用带有 pip 的 Python 发行版。 |
| `externally-managed-environment` | 你的系统 Python 阻止全局 pip 安装。请使用一条命令的安装器、`uv tool install nanobot-ai`、`pipx install nanobot-ai`，或创建虚拟环境；不要为 nanobot 添加 `--break-system-packages`。 |
| 安装器选择了错误的 Python | 在运行安装器之前设置 `PYTHON`，例如 `curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | PYTHON=python3 sh`，或在 PowerShell 命令前设置 `$env:PYTHON="py"`。 |
| 可编辑源码安装不更新 | 从仓库根目录使用开发用的 Python 命令重新运行 `python -m pip install -e .`，然后检查 `python -m nanobot --version` 或 `nanobot --version`。 |
| 缺少 WebUI 构建工具 | 只有 WebUI 开发才需要它们。打包安装已经包含 WebUI 构建产物。 |

## 配置问题

默认配置路径：

```text
~/.nanobot/config.json
```

默认工作区路径：

```text
~/.nanobot/workspace/
```

除非你传入显式路径，否则 `nanobot status` 会读取默认配置。在调试多个实例时，请在状态检查和运行时命令之间使用相同的 `--config` 和 `--workspace`：

```bash
nanobot status --config ./bot-a/config.json --workspace ./bot-a/workspace
nanobot agent --config ./bot-a/config.json --workspace ./bot-a/workspace -m "Hello"
nanobot gateway --config ./bot-a/config.json --workspace ./bot-a/workspace
```

常见的配置错误：

| 现象 | 检查 |
|---|---|
| JSON 解析错误 | 校验逗号、大括号和引号。大多数文档示例是待合并的片段。 |
| 未知或缺失的提供商 | 使用提供商注册表名称，例如 `openrouter`、`anthropic`、`openai`、`ollama`、`vllm`、`lm_studio`，或在 `providers` 下定义一个自定义 OpenAI 兼容的提供商键，然后在当前预设中引用完全相同的键。 |
| snake_case 与 camelCase 混淆 | 两者都被接受，但文档使用 camelCase，因为 nanobot 会以别名如 `apiKey`、`modelPresets`、`intervalS` 写回配置。 |
| 环境变量错误 | `${VAR_NAME}` 引用会在启动时解析。请在运行 nanobot 之前设置变量。 |
| 修改了配置但行为未变化 | 请重启 `nanobot gateway`；长期运行的进程只在启动时读取配置。 |

若想在不覆盖已有设置的前提下补齐缺失默认值，请运行：

```bash
nanobot onboard --refresh
```

如需在重置和补齐之间进行交互式选择，请运行 `nanobot onboard`，然后选择保留当前值并合并缺失默认值的选项。

## 提供商和模型问题

首先在 CLI 中证明提供商可用：

```bash
nanobot agent -m "Hello!"
```

然后将你的配置对照 [`providers.md`](./providers.md)。

如果你需要一个已知可用的片段而不是诊断，请使用 [`provider-cookbook.md`](./provider-cookbook.md)。

| 现象 | 可能原因 |
|---|---|
| 401、unauthorized、invalid API key | 密钥缺失、过期、带有空白，或位于错误的提供商键下。 |
| 模型找不到 | 该模型 ID 属于其他提供商或网关。 |
| 无法推断提供商 | 在当前预设中固定 `modelPresets.<name>.provider`，而不是使用 `"auto"`。对于遗留的直连配置，请固定 `agents.defaults.provider`。 |
| 本地模型连接被拒绝 | Ollama、vLLM、LM Studio 或其他本地服务器未运行，或 `apiBase` 指向错误的端口。 |
| Bedrock 校验错误 | 检查 AWS 区域、凭证、模型访问、模型 ID，以及该模型是否支持 Converse。 |
| OAuth 提供商失败 | 运行 `nanobot provider login openai-codex --set-main` 或 `nanobot provider login github-copilot --set-main`。 |
| Codex OAuth 需要代理 | 在运行登录命令前设置 `providers.openaiCodex.proxy`。该代理会用于登录、令牌刷新和 Codex API 请求。 |
| Codex 登录在远程/无头机器上运行 | 在本地浏览器中打开打印的 URL，然后把最终的 `http://localhost:1455/auth/callback?...` URL 粘贴回终端。 |
| Codex 登录在 Docker 中运行 | 使用 `docker run -it` 启动容器，让 OAuth 流程拥有交互式终端。 |
| Codex 提示某个模型不被 ChatGPT 账号支持 | 使用 provider `openai_codex` 搭配 Codex 模型，例如 `openai-codex/gpt-5.6-sol`。不要在 Codex OAuth 上使用直连 API 的 `openai/...` 前缀。 |
| 配置提示 `providers.openai_codex` 与内置提供商冲突 | 在 `providers` 下只保留规范键 `openaiCodex`，删除重复的 `openai_codex` 键。模型预设的 `provider` 值仍为 `openai_codex`。 |

## Langfuse 问题

Langfuse 追踪是可选的，由环境变量控制。

| 现象 | 检查 |
|---|---|
| `LANGFUSE_SECRET_KEY is set but langfuse is not installed` | 在运行 nanobot 的同一个 Python 环境中安装 `langfuse`，然后重启进程。 |
| 无追踪出现 | 在启动 nanobot 之前设置 `LANGFUSE_SECRET_KEY`、`LANGFUSE_PUBLIC_KEY` 和 `LANGFUSE_BASE_URL`。 |
| Langfuse 项目或区域错误 | 检查密钥对和 `LANGFUSE_BASE_URL` 是否来自同一个 Langfuse 项目/区域。 |
| 只有部分提供商产生追踪 | Langfuse 追踪应用于 OpenAI 兼容的提供商调用；原生提供商可能不走该客户端路径。 |

配置命令参见 [`configuration.md#langfuse-observability`](./configuration.md#langfuse-observability)。

## 网关问题

`nanobot gateway` 是 WebUI、聊天应用、心跳、Dream 和长期频道连接的前提。

默认端口：

| 表面 | 默认 |
|---|---|
| 网关健康检查端点 | `http://127.0.0.1:18790/health` |
| WebUI/WebSocket 频道 | `http://127.0.0.1:8765` |
| OpenAI 兼容 API（`nanobot serve`） | `http://127.0.0.1:8900` |

网关常见检查：

```bash
nanobot gateway --verbose
```

| 现象 | 检查 |
|---|---|
| 端口已被占用 | 更改 `gateway.port`、`channels.websocket.port`，或相关命令的 `--port` CLI 参数。 |
| WebUI 在 `18790` 打开却看不到有用内容 | 打开 `8765`；`18790` 是健康检查端点。 |
| 配置更改被忽略 | 重启网关。 |
| 心跳从不运行 | 保持网关运行，在 `<workspace>/HEARTBEAT.md` 的 `## Active Tasks` 下添加任务，并确认 `gateway.heartbeat.enabled` 为 true。 |
| 切换工作区后 cron 任务消失了 | cron 任务是工作区范围的，位于 `<workspace>/cron/jobs.json`；检查你使用的是目标工作区。 |

## WebUI 问题

打包的 WebUI 由 WebSocket 频道提供服务。

最小配置：

```json
{
  "channels": {
    "websocket": {
      "enabled": true
    }
  }
}
```

然后运行：

```bash
nanobot gateway
```

打开：

```text
http://127.0.0.1:8765
```

若要从其他设备访问，请把 WebSocket 频道绑定到 `0.0.0.0` 并设置 `token` 或 `tokenIssueSecret`。WebSocket 频道拒绝在没有 token 或 token issue secret 的情况下绑定到公共地址。

关于局域网配置参见 [`webui.md#lan-access`](./webui.md#lan-access)，前端开发参见 [`../webui/README.md`](../webui/README.md)。

## 聊天应用问题

在调试聊天应用之前：

```bash
nanobot agent -m "Hello!"
nanobot channels status
nanobot gateway
```

然后检查：

| 现象 | 检查 |
|---|---|
| 机器人从不回复 | 网关未运行、频道未启用，或 bot/应用 token 错误。 |
| 未知发送者被忽略 | 配置 `allowFrom`、配对，或频道特定的允许列表。 |
| Telegram 失败 | 确认 BotFather token 和 `allowFrom` 用户 ID。 |
| Discord 回复缺失 | 启用 Message Content intent，并以所需权限邀请机器人。 |
| WhatsApp 或微信登录过期 | 重新运行 `nanobot channels login whatsapp` 或 `nanobot channels login weixin`。 |
| 聊天应用可用但 WebUI 不行 | 提供商和网关很可能没问题；单独调试 WebSocket 频道。 |

频道特定配置参见 [`chat-apps.md`](./chat-apps.md)。

## 工具和工作区问题

| 现象 | 检查 |
|---|---|
| 文件访问被拒绝 | 检查 `tools.restrictToWorkspace`，以及目标路径是否位于当前工作区内。 |
| Docker 中 shell 命令失败 | 沙箱设置可能需要 Linux capabilities；参见 [`deployment.md`](./deployment.md)。 |
| 网页抓取被阻止 | SSRF 保护会阻止不安全的目标；仅在你信任的私有网络上使用 `tools.ssrfWhitelist`。 |
| MCP 工具缺失 | 检查 `tools.mcpServers`、服务器启动命令、环境变量和工具允许列表。 |
| 生成的产物缺失 | 检查当前工作区和频道的媒体目录。 |

## 记忆和会话问题

| 现象 | 检查 |
|---|---|
| 对话上下文看起来不对 | 确认当前的工作区和会话。WebUI 聊天和聊天应用会话可能使用不同的会话。 |
| 记忆没有立即更新 | Dream 整合是周期性的；近期轮次仍位于会话历史中。 |
| 迁移配置后出现了旧会话 | 会话文件存储在 `<workspace>/sessions/` 下；确认工作区路径。 |
| 你希望多个设备共享一个会话 | 有意设置 `agents.defaults.unifiedSession`；否则保持独立会话。 |

## 收集有用的证据

在提交 issue 或寻求帮助时，请附上：

- 安装方式和 `nanobot --version`；
- 操作系统和 Python 版本；
- 你运行的命令；
- 相关的 `nanobot status` 输出；
- 已脱敏的配置片段，尤其是提供商、模型、频道和工具设置；
- `nanobot gateway --verbose` 的网关日志；
- `nanobot agent -m "Hello!"` 是否可用。

永远不要把真实的 API 密钥、机器人令牌、OAuth 令牌或私密聊天 ID 粘贴到公开 issue 中。

如果你发现文档错误、过时的命令或令人困惑的步骤，请提交 issue：<https://github.com/HKUDS/nanobot/issues>。
