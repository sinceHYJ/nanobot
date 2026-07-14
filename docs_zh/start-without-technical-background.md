# 无技术背景起步

本页面适合从未使用过终端、编辑过 JSON 文件或配置过 AI 模型的你。

目标很小：在你的浏览器中让 nanobot 完成一次本地回复。暂不连接 Telegram、Discord、Docker、本地模型或部署。这些都会在首次回复成功之后更容易完成。

## 你正在配置的内容

Quick Start 只需要理解这些术语：

| 术语 | 通俗含义 |
|---|---|
| 终端 | 一个用来粘贴命令并按 Enter 的文本窗口。 |
| 命令 | 你在终端里运行的一行文本。 |
| API 密钥 | 来自 AI 提供商的类似密码的令牌。不要公开分享。 |
| 配置文件 | nanobot 启动时读取的设置文件。 |
| 向导 | 一个交互式终端菜单，会替你编辑配置文件。 |
| 浏览器 UI | 你与 nanobot 聊天的本地网页。 |

## 1. 打开终端

你会把命令粘贴到终端中。只复制每个代码块中的命令文本，不要复制 ``` 标记。

| 系统 | 打开方式 |
|---|---|
| Windows | 按 `Win`，输入 `PowerShell`，然后打开 **Windows PowerShell**。 |
| macOS | 按 `Command` + `Space`，输入 `Terminal`，然后按 `Enter`。 |
| Linux | 打开你的应用启动器，搜索 `Terminal`，然后打开它。 |

终端打开后，点击进入其中，粘贴命令，然后按 `Enter`。如果一个命令打印一些文本后返回到提示符，通常是正常的。

## 2. 安装 Python

从 [python.org](https://www.python.org/downloads/) 安装 Python 3.11 或更新版本。

在 Windows 上，如果安装器显示 **Add python.exe to PATH** 选项，请启用它。

在那个终端中，检查 Python：

```bash
python --version
```

如果 Windows 说找不到 `python`，请关闭并重新打开 PowerShell。如果仍不生效，试试：

```bash
py --version
```

如果 `py` 可用而 `python` 不可用，请将下方命令中的 `python` 替换为 `py`。

如果 macOS 或 Linux 说找不到 `python`，请试：

```bash
python3 --version
```

如果 `python3` 可用而 `python` 不可用，请将下方手动命令中的 `python` 替换为 `python3`。一条命令的安装器会自动检查 `python3` 和 `python`。

## 3. 获取提供商 API 密钥

nanobot 不会替你创建 AI 账号或 API 密钥。使用一个你已经掌控的 AI 提供商账号、公司端点、订阅端点或本地模型服务器。如果提供商文档中给出了 OpenAI 兼容的基础 URL，也请一并保留。

配置路径：

1. 打开提供商的 API 密钥页面。
2. 创建或复制一个 API 密钥。
3. 妥善保管密钥。
4. 如果提供商文档中列出了基础 URL，也请保留在手边。

## 4. 安装 nanobot

最简单的方式是使用一条命令的安装器。它会安装或升级 nanobot，然后启动配置向导。在 macOS 和 Linux 上，它会通过已激活的虚拟环境、`uv`、`pipx` 或 `~/.nanobot/venv` 下的托管 venv 来避免系统级 pip 安装。

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh
```

**Windows PowerShell**

```powershell
irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1 | iex
```

这些命令会安装 PyPI 上的稳定版本。若想在不更改环境的情况下预览安装动作，传入 `--dry-run`：

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh -s -- --dry-run
```

```powershell
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1))) --dry-run
```

仅在维护者要求你测试当前 `main` 分支时才使用开发版安装器：

```bash
curl -fsSL https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.sh | sh -s -- --dev
```

```powershell
& ([scriptblock]::Create((irm https://raw.githubusercontent.com/HKUDS/nanobot/main/scripts/install.ps1))) --dev
```

如果命令提示找不到 `curl` 或 `irm`，或无法从 GitHub 下载，请使用下方的手动安装命令。

如果已安装 `uv`，使用：

```bash
uv tool install nanobot-ai
```

如果你更喜欢 pip，请仅在你可控的环境中使用：

```bash
python -m pip install nanobot-ai
```

如果 pip 在 macOS 或 Linux 上报告 `externally-managed-environment`，请回到一条命令的安装器，或使用 `uv tool install nanobot-ai`、`pipx install nanobot-ai`，或先创建一个虚拟环境。

然后确认 nanobot 已安装：

```bash
nanobot --version
```

如果终端找不到 `nanobot`，请使用模块形式：

```bash
python -m nanobot --version
```

如果第 2 步中生效的 Python 命令是 `python3` 或 `py`，则使用 `python3 -m nanobot --version` 或 `py -m nanobot --version`。

## 5. 运行配置向导

一条命令的安装器会在安装完成后为你启动向导。如果你是手动安装的，请运行：

```bash
nanobot onboard --wizard
```

如果找不到 `nanobot`，请运行：

```bash
python -m nanobot onboard --wizard
```

如果第 2 步中生效的 Python 命令是 `python3` 或 `py`，请使用 `python3 -m nanobot onboard --wizard` 或 `py -m nanobot onboard --wizard`。

向导是一个终端菜单。它不是图形应用，但可以让你选择选项，而不必手动编辑每一个 JSON 字段。

你会看到类似这样的菜单：

```text
> What would you like to do?
  [Q] Quick Start
  [A] Advanced Settings
  [X] Exit
```

按以下方式操作向导：

| 当你看到 | 操作 |
|---|---|
| 一个菜单 | 用方向键高亮选项，然后按 `Enter`。 |
| 提供商菜单 | 选择你想使用的公司或服务。 |
| 端点菜单 | 选择与你密钥匹配的标准 API 或订阅计划端点。 |
| API 密钥字段 | 粘贴密钥，然后按 `Enter`。 |
| 提供商基础 URL 字段 | 从提供商文档中粘贴基础 URL，然后按 `Enter`。 |
| Model ID 字段 | 从你的提供商粘贴一个模型名称，然后按 `Enter`。 |
| Advanced Settings 中的返回选项 | 选择它以回到上一级菜单。 |

首次配置请选择 `[Q] Quick Start`。它会为你配置推荐的本地浏览器 UI 和默认 AI 设置。仅在你需要聊天应用、工具配置或提供商特定字段时，稍后再使用 `Advanced Settings`。

1. 选择 `[Q] Quick Start`。
2. 选择你想使用的提供商。
3. 如果向导询问端点，例如 Standard API、Coding Plan、Token Plan 或 Step Plan，请选择相应端点。
4. 如果向导要求，粘贴你的 API 密钥。
5. 如果向导要求，粘贴提供商基础 URL。
6. 粘贴该提供商可以运行的模型 ID。
7. 确认 Quick Start 应当配置本地 WebUI。
8. 出现提示时设置 WebUI 密码。
9. 查看 Quick Start 摘要。Quick Start 完成时向导会自动保存并退出。

推荐路径会配置本地 WebUI、要求 WebUI 密码，并写入默认 AI 设置。首次运行你不需要另外选择聊天应用。

如果你已经知道自己需要自定义请求头、提供商特定的请求字段、聊天应用或工具，请改选 `Advanced Settings`。[`provider-cookbook.md`](./provider-cookbook.md) 提供了几个常见提供商配置的可复制示例。当你修改高级设置后，主菜单中会出现保存选项。选择 `[S] Save and Exit`。

向导会创建或更新：

| 路径 | 含义 |
|---|---|
| `~/.nanobot/config.json` | 设置文件。 |
| `~/.nanobot/workspace/` | 用于记忆、会话和生成文件的工作目录。 |

如果 Quick Start 成功完成，请跳到 [打开 WebUI](#7-open-the-webui)。接下来的两节仅用于手动配置。

## 手动配置：如何合并 JSON 片段

大多数文档示例是片段，而不是完整文件。你的 `config.json` 只有一个最外层的 `{ ... }`。将 `providers`、`modelPresets`、`agents` 或 `channels` 等新的顶层小节添加到同一个最外层对象内部。

不要把两个独立的 JSON 对象粘贴到同一个文件中：

```text
{
  "providers": { "...": "..." }
}
{
  "channels": { "...": "..." }
}
```

将它们合并为一个对象：

```json
{
  "providers": {
    "custom": {
      "apiKey": "your-api-key",
      "apiBase": "https://api.example.com/v1"
    }
  },
  "channels": {
    "websocket": {
      "tokenIssueSecret": "your-webui-password",
      "websocketRequiresToken": true
    }
  }
}
```

注意 `providers` 块后的逗号。JSON 在同级小节之间需要逗号，但最后一个小节之后不需要。如果这让你觉得困难，请尽量使用 `nanobot onboard --wizard`。

## 6. 手动配置：配置文件回退

仅在向导不可用或你更喜欢自己打开文件时使用此方法。

如果 `~/.nanobot/config.json` 尚不存在，请先运行 `nanobot onboard`。

使用以下命令之一：

**Windows PowerShell**

```powershell
notepad "$env:USERPROFILE\.nanobot\config.json"
```

**macOS**

```bash
open -e ~/.nanobot/config.json
```

**Linux**

```bash
xdg-open ~/.nanobot/config.json
```

如果这是全新安装且你尚未配置其他内容，请将文件内容替换为以下最小配置：

```json
{
  "providers": {
    "custom": {
      "apiKey": "your-api-key",
      "apiBase": "https://api.example.com/v1"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "Primary",
      "provider": "custom",
      "model": "model-id-from-your-provider",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  },
  "channels": {
    "websocket": {
      "tokenIssueSecret": "your-webui-password",
      "websocketRequiresToken": true
    }
  }
}
```

将 `your-api-key`、`https://api.example.com/v1`、`model-id-from-your-provider` 和 `your-webui-password` 替换为你自己的值。

如需可复制的提供商特定示例，请使用 [`provider-cookbook.md`](./provider-cookbook.md)。

保存文件。

## 7. 打开 WebUI

首先检查 nanobot 是否能读取保存的配置：

```bash
nanobot status
```

这应显示配置文件路径、工作区路径以及当前的模型或预设。如果找不到 `nanobot`，请使用 `python -m nanobot status`、`python3 -m nanobot status` 或 `py -m nanobot status`，取决于第 2 步中生效的 Python 命令。

大多数提供商显示 `not set` 是正常的。只有你为当前预设选定的提供商需要看起来已配置。

启动本地浏览器 UI：

```bash
nanobot webui
```

这会启动 nanobot 并在浏览器中打开 `http://127.0.0.1:8765`。使用 WebUI 时请保持终端打开。如果浏览器询问，请输入你在向导中设置的 WebUI 密码。

在浏览器中发送第一条消息：

```text
Hello!
```

如果成功，说明 nanobot 已安装并可以调用模型。你应该在浏览器中看到一条正常的助手回复。具体措辞会有所不同，但应类似以下形式：

```text
Hello! How can I help you today?
```

如果找不到 `nanobot`，请运行：

```bash
python -m nanobot webui
```

如果第 2 步中生效的 Python 命令是 `python3` 或 `py`，请使用 `python3 -m nanobot webui` 或 `py -m nanobot webui`。

一旦成功，nanobot 就可以帮你完成下一步配置。在浏览器 UI 中，请它阅读这些文档并针对一个具体目标更新你当前的配置，然后当 nanobot 告诉你配置已就绪时运行 `/restart`。例如，让它添加一个提供商预设或配置一个聊天应用。

## 8. 如果出错

不要一次性修改很多东西。先看清具体错误：

| 错误或现象 | 通常含义 |
|---|---|
| `JSON parse error` | 配置文件中有缺失逗号、多余逗号或不匹配的大括号。请再次复制示例。 |
| `401`、`unauthorized` 或 `invalid API key` | API 密钥错误、过期、含多余空格，或被粘贴到了错误的提供商下。 |
| `model not found` | 你的账号无法使用默认模型。返回到 `nanobot onboard --wizard`，选择 `Advanced Settings`，然后编辑 `Model Presets`。 |
| `nanobot: command not found` | Python 安装可用，但你的 shell 找不到脚本。请使用 `python -m nanobot ...`、`python3 -m nanobot ...` 或 `py -m nanobot ...`，取决于此前生效的 Python 命令。 |
| 编辑配置后没有反应 | 请重启命令。长期运行的进程会在启动时读取配置。 |

如需更完整的诊断路径，参见 [`troubleshooting.md`](./troubleshooting.md)。

## 暂不需要配置的内容

在第一条本地消息成功之前，先跳过这些：

- `apiBase`：许多内置托管提供商已经有默认端点。你只在本地模型、代理、自定义 OpenAI 兼容提供商，或特殊的区域/订阅端点时才需要 `apiBase`。
- 聊天应用：先证明本地浏览器 UI 能够回答。
- 回退模型：稍后会有用，但首次回复不需要。
- Langfuse：对可观测性有用，但首次配置不需要。

## 下一步

首次回复成功后，只选择一个下一步目标。使用 WebUI 期间请一直保留运行 `nanobot webui` 的终端。聊天应用底层使用的是相同的网关服务。

### 再次打开浏览器 UI

运行：

```bash
nanobot webui
```

保持终端打开；浏览器应会自动打开。

之后要停止 WebUI，请回到网关终端并按 `Ctrl+C`。

如果找不到 `nanobot`，请运行 `python -m nanobot webui`、`python3 -m nanobot webui` 或 `py -m nanobot webui`，取决于此前生效的 Python 命令。更多细节请见 [`webui.md`](./webui.md)。

### 连接聊天应用

1. 阅读 [`chat-apps.md`](./chat-apps.md) 中某个应用的章节。
2. 只添加该应用的配置片段。将其合并到现有文件中，而不是替换整个文件。
3. 运行：

```bash
nanobot channels status
nanobot gateway
```

4. 保持网关终端打开，然后从允许的账号发送一条消息。

先从私聊或测试服务器开始。除非你确实希望所有能够访问该频道的人都可与机器人对话，否则不要将 `allowFrom` 设置为 `["*"]`。

### 更换模型或添加备用

在提供商/模型组合失败时使用 [`providers.md`](./providers.md)，需要可复制片段时使用 [`provider-cookbook.md`](./provider-cookbook.md)。把模型选择放在 `modelPresets` 中，然后用 `agents.defaults.modelPreset` 选择当前使用的。

### 寻求帮助

请求帮助时，请附上：

- 你的操作系统；
- 你运行的命令；
- `nanobot --version`；
- `nanobot status`；
- 浏览器 UI 是否能回答 `Hello!`；
- 完整的错误文本；
- 一个已移除 API 密钥和令牌的配置片段。

永远不要把真实的 API 密钥、机器人令牌、OAuth 令牌或私密聊天 ID 粘贴到公开的 issue 或聊天中。

如果你发现文档错误、过时的命令或令人困惑的步骤，请提交 issue：<https://github.com/HKUDS/nanobot/issues>。
