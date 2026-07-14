# 安装与快速开始

本页帮助你完成一次本地 nanobot 回复。之后，你可以再添加 WebUI、聊天应用、本地模型、网页搜索、MCP、部署或自定义插件。

如果你从未使用过终端或编辑过配置文件，请先阅读 [`start-without-technical-background.md`](./start-without-technical-background.md)。本页假定你能够熟练粘贴命令和编辑 JSON 片段。

## 开始之前

你需要：

- Python 3.11 或更新版本。
- 一个可调用的 LLM 提供商、公司端点、订阅端点或本地模型服务器。下方示例使用通用的 OpenAI 兼容 `custom` 提供商，这样简洁路径不会推荐某个特定托管服务；只要密钥、提供商名称和模型 ID 匹配，任何受支持的提供商都能使用。
- Git（仅在从源码安装时需要）。
- Node.js 或 Bun（仅在开发 WebUI 本身时需要）。

> [!IMPORTANT]
> 仓库文档可能描述先在源码中出现的功能。若你希望使用日常稳定版本，请从 PyPI 或 `uv` 安装；若你想要最新的仓库行为或计划贡献代码，请从源码安装。

## 1. 安装

选择一种安装方式。

**一条命令完成安装：**

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh
```

Windows PowerShell：

```powershell
irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1 | iex
```

默认命令会从 PyPI 安装或升级 `nanobot-ai`，然后启动 `nanobot onboard --wizard`。它会通过已激活的虚拟环境、`uv`、`pipx` 或 `~/.nanobot/venv` 下的托管 venv 来避免系统级 pip 安装。如果 Quick Start 结束了，直接进入 [打开 WebUI](#5-open-the-webui)。

若想在不更改环境的情况下预览计划，传入 `--dry-run`；再叠加 `--dev`，即可预览基于 main 分支的安装。

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh -s -- --dry-run
```

```powershell
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1))) --dry-run
```

若想改为安装当前 `main` 分支，传入 `--dev`：

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh -s -- --dev
```

```powershell
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1))) --dev
```

如果 `curl` 或 `irm` 不可用，或者网络中拦截了 GitHub 原始下载，请使用下方的手动安装方法之一。

如果你希望先查看脚本内容，请打开 [`../scripts/install.sh`](../scripts/install.sh) 或 [`../scripts/install.ps1`](../scripts/install.ps1)。

**使用 `uv` 安装稳定版：**

```bash
uv tool install nanobot-ai
nanobot --version
```

**使用 pip 安装稳定版：**

```bash
python -m pip install nanobot-ai
nanobot --version
```

仅在你可控的环境中使用 pip。如果 pip 在 macOS 或 Linux 上报告 `externally-managed-environment`，请改用一条命令的安装器、`uv tool install nanobot-ai`、`pipx install nanobot-ai`，或先创建一个虚拟环境。

**最新源码检出：**

```bash
git clone https://github.com/HKUDS/nanobot.git
cd nanobot
python -m pip install -e .
nanobot --version
```

如果通过 pip 安装后你的 shell 找不到 `nanobot`，请使用模块形式运行：

```bash
python -m nanobot --version
python -m nanobot onboard
```

在 Windows 上，文档中的 `~` 表示你的用户目录，例如 `C:\Users\you`。

文档中命令使用 `python`。如果你的系统将 Python 3.11+ 暴露为 `python3` 或 `py`，请在相同位置使用该命令，例如 `python3 -m pip install nanobot-ai` 或 `py -m nanobot --version`。

## 2. 初始化

如果一条命令的安装已经启动了向导且 Quick Start 已在其中完成，请跳过本节。

```bash
nanobot onboard
```

如果你更喜欢通过提示而不是手动编辑 JSON，请使用向导：

```bash
nanobot onboard --wizard
```

初始化会创建：

| 路径 | 含义 |
|------|------------|
| `~/.nanobot/config.json` | 主设置文件，配置提供商、模型、频道、工具、网关和 API |
| `~/.nanobot/workspace/` | 智能体工作区，存放记忆、会话、心跳任务、技能和产物 |

如果你已经有配置，`nanobot onboard` 可以在不覆盖已有值的情况下补齐缺失的默认字段。使用 `nanobot onboard --refresh` 可以在无交互提示下完成同样的刷新。

## 3. 配置提供商

如果你已经在向导中配置了提供商和模型，请跳过本节。

打开 `~/.nanobot/config.json`。将下面的块添加或合并到 `nanobot onboard` 所创建的文件中；除非你想重置配置，否则不要替换整个文件。

**API 密钥：**

```json
{
  "providers": {
    "custom": {
      "apiKey": "your-api-key",
      "apiBase": "https://api.example.com/v1"
    }
  }
}
```

**模型预设：**

```json
{
  "modelPresets": {
    "primary": {
      "label": "Primary",
      "provider": "custom",
      "model": "model-id-from-your-provider",
      "maxTokens": 8192,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

预设内的提供商和模型必须匹配。上面的片段只是示例。若使用其他提供商，请一并替换以下这些值：

| 需替换 | 位置 |
|---|---|
| 提供商配置键，例如 `custom` | `providers.<provider>` |
| API 密钥或环境变量 | `providers.<provider>.apiKey` |
| 预设中的提供商名称 | `modelPresets.primary.provider` |
| 模型 ID | `modelPresets.primary.model` |
| 端点 URL（仅在需要时） | `providers.<provider>.apiBase` |

直接使用 `agents.defaults.provider` 和 `agents.defaults.model` 仍适用于现有配置，但推荐使用命名预设，因为它们同时支持 `/model` 切换和回退链。有关直连、网关、OAuth、云端和本地场景的提供商特定示例，参见 [`providers.md`](./providers.md)。

**关于 `apiBase` / 基础 URL？**

`apiBase` 是提供商端点的 HTTP 基础 URL，不是模型名称。nanobot 中大多数托管提供商已经知道其默认端点，因此你通常只需要设置 `apiKey` 和一个模型预设。在以下情况需要设置 `apiBase`：

- 使用 `custom` 连接第三方或自托管的 OpenAI 兼容 API；
- 使用 Ollama、vLLM 或 LM Studio 等本地 OpenAI 兼容服务器；
- 使用提供商特定的备用端点、区域端点、代理或订阅端点。

示例：

```json
{
  "providers": {
    "custom": {
      "apiKey": "${CUSTOM_API_KEY}",
      "apiBase": "https://api.example.com/v1"
    }
  }
}
```

```json
{
  "providers": {
    "ollama": {
      "apiBase": "http://localhost:11434/v1"
    }
  }
}
```

如果提供商文档说明端点是 `/v1`，则 `apiBase` 中要包含 `/v1`。模型 ID 仍属于当前激活的 `modelPresets` 条目。

如果你不想在 `config.json` 中存储密钥，可以引用环境变量，并在启动 nanobot 之前设置它：

```json
{
  "providers": {
    "custom": {
      "apiKey": "${PROVIDER_API_KEY}",
      "apiBase": "https://api.example.com/v1"
    }
  }
}
```

## 4. 检查配置

```bash
nanobot status
```

此命令应显示配置路径、工作区路径、当前的模型或预设，以及提供商摘要。它不会向模型发送消息，因此可作为首次真实请求前的快速配置检查。

按以下方式阅读：

| 状态行 | 期望情况 |
|---|---|
| `Config` | 显示对勾。 |
| `Workspace` | 显示对勾。 |
| `Model` | 显示你预期的模型或预设。 |
| 提供商列表 | 大多数提供商可以显示 `not set`；当前预设所用的提供商应显示对勾、OAuth 状态或本地 URL。 |

## 5. 打开 WebUI

启动浏览器工作台：

```bash
nanobot webui
```

`nanobot webui` 会在需要时准备本地 WebSocket 频道和 WebUI 引导密钥，启动网关，并打开 `http://127.0.0.1:8765`。首次运行的 WebUI 默认绑定到 `127.0.0.1`，因此不会暴露到你的局域网。当你希望网关持续运行而无需保留打开的终端时，可使用 `nanobot webui --background`。

## 6. 测试一条 CLI 消息

如果你跳过了 Quick Start、拒绝了 WebSocket 频道，或想只做终端检查，请使用此路径。

运行一次性的 CLI 消息：

```bash
nanobot agent -m "Hello!"
```

首次成功运行可证明：

- `nanobot` 命令已安装；
- `~/.nanobot/config.json` 可以被加载；
- 所选提供商和模型能够回答；
- 默认工作区可以被创建和使用。

回复文本本身会有所不同。任何正常的助手回答都意味着安装、配置、提供商、模型和工作区路径都可用。

如果成功，就可以开启一个交互式 CLI 聊天：

```bash
nanobot agent
```

一旦交互会话能够正常回答，nanobot 可以帮忙进行下一步配置。让它阅读相关文档、检查你当前的 `~/.nanobot/config.json`，并进行一次具体的更改，例如启用 WebUI、添加提供商预设或配置一个聊天频道。当 nanobot 表示配置已更新后，在聊天中运行 `/restart` 或手动重启 nanobot 进程，以便长期运行的进程重新加载 `config.json`。

示例提示：

```text
Read docs/quick-start.md, docs/providers.md, and docs/configuration.md in this checkout.
Then update ~/.nanobot/config.json to add a model preset named "primary" for my provider.
Tell me exactly what changed and whether I need to run /restart.
```

在交互模式下，`Enter` 会发送当前消息。按 `Alt+Enter` 可在发送前添加换行。

使用 `exit`、`quit`、`/exit`、`/quit`、`:q` 或 `Ctrl+D` 退出交互模式。

## 7. 选择下一步

| 想要... | 前往 |
|---|---|
| 理解配置、工作区、网关、频道、记忆和工具 | [`concepts.md`](./concepts.md) |
| 复制另一个提供商或本地模型配置 | [`provider-cookbook.md`](./provider-cookbook.md) |
| 理解提供商/模型的匹配 | [`providers.md`](./providers.md) |
| 打开内置浏览器 UI | [`webui.md`](./webui.md) |
| 连接 Telegram、Discord、微信、Slack、邮件、Mattermost 或其他聊天应用 | [`chat-apps.md`](./chat-apps.md) |
| 配置网页搜索、MCP、安全、记忆、网关或运行时设置 | [`configuration.md`](./configuration.md) |
| 通过 Docker、systemd 或 LaunchAgent 运行 | [`deployment.md`](./deployment.md) |
| 调试故障 | [`troubleshooting.md`](./troubleshooting.md) |

## 更新

**pip：**

```bash
python -m pip install -U nanobot-ai
nanobot --version
```

如果 pip 报告 `externally-managed-environment`，请使用与安装 nanobot 时相同的隔离方法进行升级，例如 `uv tool upgrade nanobot-ai`、`pipx upgrade nanobot-ai`，或使用一条命令安装器创建的托管 venv。

**uv：**

```bash
uv tool upgrade nanobot-ai
nanobot --version
```

**pipx：**

```bash
pipx upgrade nanobot-ai
nanobot --version
```

**源码检出：**

```bash
git pull
python -m pip install -e .
nanobot --version
```

如果你从源码检出使用 WhatsApp，请保持可选依赖已安装：

```bash
nanobot plugins enable whatsapp
```

## 首次运行的故障排查

| 现象 | 检查项 |
|---------|---------------|
| `nanobot: command not found` | 使用 `python -m nanobot ...`，或将 Python 脚本目录加入 `PATH`。 |
| `ModuleNotFoundError: nanobot` | 确认你安装到了运行命令的同一个 Python 环境中。 |
| JSON 解析错误 | 检查 `~/.nanobot/config.json` 中的逗号和大括号；以上示例都是待合并的片段。 |
| 认证或 401 错误 | 检查 API 密钥有效、复制时没有空格，并放置在你所选的提供商下。 |
| 提供商/模型错误 | 确保当前预设使用的是拥有你 API 密钥的提供商，且模型在该处存在。 |
| CLI 可用但聊天应用不回复 | 首先保持 `nanobot gateway` 运行，然后按照 [`chat-apps.md`](./chat-apps.md) 操作。 |
| WebUI 无法打开 | 运行 `nanobot webui`；浏览器 UI 使用端口 `8765`，而不是网关健康检查端口 `18790`。 |

如需更完整的诊断流程，参见 [`troubleshooting.md`](./troubleshooting.md)。
