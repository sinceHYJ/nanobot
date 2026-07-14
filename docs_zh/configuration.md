# 配置

配置文件：`~/.nanobot/config.json`

这是完整的参考文档。如果这是你的首次安装，请从 [`quick-start.md`](./quick-start.md) 开始。如果你正在尝试选择模型或修复提供商/模型匹配，请先阅读 [`providers.md`](./providers.md)，然后再回到这里查看确切的字段和高级选项。

下面的 JSON 示例通常是要合并到你现有配置中的部分片段，而不是完整的替换文件。有关配置、工作区、网关、频道、会话、工具和记忆背后的心智模型，请参见 [`concepts.md`](./concepts.md)。

生成的 `config.json` 使用 camelCase 键，例如 `apiKey` 和 `intervalS`。为了兼容性也接受 snake_case 键，但文档更倾向于 camelCase，因为 nanobot 会以这种形式写回磁盘。

对于安装和运行时故障，在同时更改多个配置区域之前，请先按 [`troubleshooting.md`](./troubleshooting.md) 中的诊断顺序进行排查。

> [!NOTE]
> 如果你的配置文件比当前 schema 更旧，你可以在不覆盖现有值的情况下刷新它：运行 `nanobot onboard`，然后在询问是否覆盖配置时回答 `N`。nanobot 会合并缺失的默认字段并保留你当前的设置。

## 配置指南

本页是完整的配置参考。对于任务导向的设置，请先使用
聚焦的指南，然后再回到这里查看确切的字段和默认值。

| 任务 | 指南 |
|---|---|
| 添加 MCP 工具 | [`guides/configure-mcp-tools.md`](./guides/configure-mcp-tools.md) |
| 启用网络搜索和网络抓取 | [`guides/configure-web-search.md`](./guides/configure-web-search.md) |
| 配置模型回退 | [`guides/configure-model-fallback.md`](./guides/configure-model-fallback.md) |
| 添加 OpenAI 兼容的提供商 | [`guides/configure-openai-compatible-provider.md`](./guides/configure-openai-compatible-provider.md) |
| 添加 Langfuse 可观测性 | [`guides/configure-langfuse-observability.md`](./guides/configure-langfuse-observability.md) |
| 保护本地 AI 智能体 | [`guides/secure-local-ai-agent.md`](./guides/secure-local-ai-agent.md) |
| 部署网关 | [`guides/deploy-nanobot-gateway.md`](./guides/deploy-nanobot-gateway.md) |

## 快速跳转

| 需求 | 章节 |
|---|---|
| 将密钥从 `config.json` 中移除 | [用于密钥的环境变量](#environment-variables-for-secrets) |
| 使用环境变量调整进程级行为 | [运行时环境变量](#runtime-environment-variables) |
| 追踪模型调用 | [Langfuse 可观测性](#langfuse-observability) |
| 配置凭证和端点 | [提供商](#providers) |
| 命名并切换模型选择 | [模型预设](#model-presets) |
| 添加回退链 | [模型回退](#model-fallbacks) |
| 配置语音转录 | [转录设置](#transcription-settings) |
| 调整频道默认值 | [频道设置](#channel-settings) |
| 配置网络搜索和抓取 | [网络工具](#web-tools) |
| 启用图像生成 | [图像生成](#image-generation) |
| 添加 MCP 服务器 | [MCP](#mcp-model-context-protocol) |
| 检查 shell、工作区和 SSRF 控制 | [安全](#security) |
| 控制访问和配对 | [配对](#pairing) |
| 调整网关作业、会话和工具 | [网关心跳](#gateway-heartbeat)、[自动压缩](#auto-compact)、[统一会话](#unified-session)、[工具提示最大长度](#tool-hint-max-length) |

## 从哪里开始编辑

如果你不确定某个设置属于哪里，请从你正在尝试完成的任务开始。大多数更改涉及一个配置节和一个验证命令。

| 任务 | 首先要检查的键 | 验证方式 | 深入了解 |
|---|---|---|---|
| 让第一次模型响应正常工作 | `providers.<name>.apiKey`、可选的 `providers.<name>.apiBase`、`modelPresets.<preset>`、`agents.defaults.modelPreset` | `nanobot status`，然后 `nanobot agent -m "Hello!"` | [提供商](#providers)、[模型预设](#model-presets) |
| 添加回退模型 | `modelPresets.<fallback>`、`agents.defaults.fallbackModels` | `nanobot status`，然后一次正常的智能体运行 | [模型回退](#model-fallbacks) |
| 将密钥从配置文件中移除 | 任何字符串值内的 `${ENV_VAR}` 占位符 | 从设置了该变量的相同环境启动 nanobot | [用于密钥的环境变量](#environment-variables-for-secrets) |
| 打开内置的 WebUI | `channels.websocket.enabled`、可选的 `channels.websocket.port`、`channels.websocket.tokenIssueSecret` | `nanobot webui` | [频道设置](#channel-settings)、[WebSocket 文档](./websocket.md) |
| 连接一个聊天应用 | `channels.<channel>.enabled`、频道凭证、可选的配对或 `channels.<channel>.allowFrom` | `nanobot channels status`，然后 `nanobot gateway --verbose` | [频道设置](#channel-settings)、[聊天应用](./chat-apps.md) |
| 启用语音转录 | `transcription.enabled`、`transcription.provider`、匹配的 `providers.<name>.apiKey` | 通过已配置的界面发送或上传一条短语音消息 | [转录设置](#transcription-settings) |
| 启用网络搜索或抓取 | `tools.web.search.*`、`tools.web.fetch.*`、可选的 `tools.ssrfWhitelist` | 提出需要当前网络信息的问题，如需要则检查日志 | [网络工具](#web-tools)、[安全](#security) |
| 启用图像生成 | `tools.imageGeneration.enabled`、`tools.imageGeneration.provider`、`tools.imageGeneration.model`、匹配的提供商凭证 | 在 WebUI 中启用图像生成并发送一个图像请求 | [图像生成](#image-generation) |
| 通过 MCP 添加外部工具 | `tools.mcpServers.<name>` | 启动 `nanobot gateway --verbose` 并检查启动/工具日志 | [MCP](#mcp-model-context-protocol) |
| 加强工具和网络安全 | `tools.restrictToWorkspace`、`tools.exec.sandbox`、`tools.ssrfWhitelist`、`channels.*.allowFrom` | 通过你计划暴露的频道或 CLI 运行相同的工作流 | [安全](#security)、[配对](#pairing) |
| 调整请求超时或进程并发 | `NANOBOT_LLM_TIMEOUT_S`、`NANOBOT_STREAM_IDLE_TIMEOUT_S`、`NANOBOT_MAX_CONCURRENT_REQUESTS` | 从相同环境启动 nanobot 并检查启动/运行时日志 | [运行时环境变量](#runtime-environment-variables) |
| 运行多个隔离的机器人 | 单独的 `--config` 和 `--workspace` 路径，以及当进程一起运行时不同的 `gateway.port` 或频道端口 | 对 `nanobot status`、`agent`、`webui`、`gateway` 和 `serve` 使用相同的显式路径 | [多实例](./multiple-instances.md)、[CLI 参考](./cli-reference.md) |
| 观察模型调用 | `LANGFUSE_SECRET_KEY`、`LANGFUSE_PUBLIC_KEY`、`LANGFUSE_BASE_URL` 环境变量 | 运行一次模型调用，然后检查匹配的 Langfuse 项目 | [Langfuse 可观测性](#langfuse-observability) |

## 用于密钥的环境变量

与其将密钥直接存储在 `config.json` 中，你可以使用 `${VAR_NAME}` 引用，它们会在启动时从环境变量中解析：

```json
{
  "channels": {
    "telegram": { "token": "${TELEGRAM_TOKEN}" },
    "email": {
      "imapPassword": "${IMAP_PASSWORD}",
      "smtpPassword": "${SMTP_PASSWORD}"
    }
  },
  "providers": {
    "groq": { "apiKey": "${GROQ_API_KEY}" }
  }
}
```

`config.json` 中的任何字符串值都可以使用 `${VAR_NAME}`。解析在启动时运行一次，仅在内存中——解析后的值绝不会写回磁盘，因此通过 `nanobot onboard` 或 WebUI 编辑配置会保留占位符。

如果引用的变量未设置，nanobot 会在启动时快速失败并报错 `ValueError: Environment variable 'NAME' referenced in config is not set`。

### 更多示例

**MCP 服务器** — 同时支持 stdio 的 `env` 和 HTTP 的 `headers`：

```json
{
  "tools": {
    "mcpServers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
      },
      "remote": {
        "url": "https://example.com/mcp/",
        "headers": { "Authorization": "Bearer ${REMOTE_MCP_TOKEN}" }
      }
    }
  }
}
```

**网络搜索提供商：**

```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

### 在启动时加载变量

选择适合你部署方式的任何机制——nanobot 只在启动时读取 `os.environ`，因此任何填充进程环境的机制都可以工作。

**systemd** — 在服务单元中使用 `EnvironmentFile=` 从只有部署用户可读的文件加载变量：

```ini
# /etc/systemd/system/nanobot.service (excerpt)
[Service]
EnvironmentFile=/home/youruser/nanobot_secrets.env
User=nanobot
ExecStart=...
```

```bash
# /home/youruser/nanobot_secrets.env (mode 600, owned by youruser)
TELEGRAM_TOKEN=your-token-here
IMAP_PASSWORD=your-password-here
```

**Docker** — 向本地构建的镜像传递 env 文件（每行一个 `KEY=VALUE`），或使用 `-e KEY=value`：

```bash
docker run --rm --env-file=./nanobot.env \
  -v ~/.nanobot:/home/nanobot/.nanobot \
  nanobot agent -m "Hello"
```

**direnv** — 在你的工作目录中放置一个 `.envrc` 并运行 `direnv allow`：

```bash
# .envrc (auto-loaded by direnv)
export TELEGRAM_TOKEN=your-token-here
export ANTHROPIC_API_KEY=...
```

**密钥管理器（1Password、Bitwarden、pass）** — 包裹进程使得密钥仅在运行期间作为环境变量存在，绝不落盘：

```bash
# 1Password — references in .env.tpl look like `op://Vault/Item/field`
op run --env-file=.env.tpl -- nanobot agent

# pass (passwordstore.org)
ANTHROPIC_API_KEY="$(pass show api/anthropic)" nanobot agent

# Bitwarden
ANTHROPIC_API_KEY="$(bw get password api/anthropic)" nanobot agent
```

## 运行时环境变量

这些变量是进程级开关。在启动 nanobot 的同一终端、服务单元、容器或监督器中设置它们。

### 运行时控制

| 变量 | 默认值 | 描述 |
|----------|---------|-------------|
| `NANOBOT_MAX_CONCURRENT_REQUESTS` | `3` | 同时运行的入站智能体请求的最大并发数。必须是整数；设置 `0` 或负值表示不受限制。 |
| `NANOBOT_LLM_TIMEOUT_S` | `300` | 普通 LLM 请求周围的墙钟超时（秒）。设置 `0` 以禁用。持续目标轮次会绕过此墙钟上限。 |
| `NANOBOT_STREAM_IDLE_TIMEOUT_S` | `90` | 流式提供商使用的流式空闲超时（秒）。无效或非正值会被忽略；高于 `3600` 的值会被夹紧。 |
| `NANOBOT_OPENAI_COMPAT_TIMEOUT_S` | `120` | OpenAI 兼容提供商的 HTTP 请求超时（秒）。无效或非正值会被忽略。 |
| `NANOBOT_WORKSPACE_SANDBOX_ENFORCED` | 未设置 | 标记外部工作区沙箱已被强制执行。真值（`1`、`true`、`yes`、`on`、`enabled`）使用 `NANOBOT_WORKSPACE_SANDBOX_PROVIDER` 作为标签；任何其他非假值被视为提供商名称。 |
| `NANOBOT_WORKSPACE_SANDBOX_PROVIDER` | `unknown` | 当 `NANOBOT_WORKSPACE_SANDBOX_ENFORCED` 为真时外部工作区沙箱的显示标签，例如 `macos_app_sandbox` 或 `bwrap`。 |
| `NANOBOT_SANDBOX_ENFORCED` | 未设置 | `NANOBOT_WORKSPACE_SANDBOX_ENFORCED` 的旧兼容别名。 |
| `NANOBOT_TMUX_SOCKET_DIR` | `${TMPDIR:-/tmp}/nanobot-tmux-sockets` | 内置 `tmux` 技能脚本使用的套接字目录。 |

### 安装器、构建和 WebUI 开发

| 变量 | 默认值 | 描述 |
|----------|---------|-------------|
| `NANOBOT_BIN_DIR` | `$HOME/.local/bin` | macOS/Linux 上安装器启动器目录。 |
| `NANOBOT_VENV` | `$HOME/.nanobot/venv` | 安装器回退使用的受管虚拟环境路径。 |
| `NANOBOT_SKIP_WIZARD` | 未设置 | 设置为 `1` 以在一键安装后跳过 `nanobot onboard --wizard`。 |
| `NANOBOT_SKIP_WEBUI_BUILD` | 未设置 | 设置为 `1` 以在包构建期间跳过打包 WebUI。 |
| `NANOBOT_FORCE_WEBUI_BUILD` | 未设置 | 设置为 `1` 以在 `nanobot/web/dist/index.html` 已存在时重新构建打包的 WebUI。 |
| `NANOBOT_API_URL` | `http://127.0.0.1:8765` | Vite WebUI 开发服务器代理的网关目标。 |

诸如 `NANOBOT_RESTART_*` 和 `NANOBOT_PATH_*` 之类的内部变量由 nanobot 自身设置，不属于受支持的用户配置面。

## Langfuse 可观测性

nanobot 可以通过 Langfuse 的 OpenAI SDK 包装器追踪 OpenAI 兼容的提供商调用。此功能通过环境变量配置，而不是 `config.json`。

在运行 nanobot 的同一 Python 环境中安装可选包：

```bash
nanobot plugins enable langfuse
```

在启动 `nanobot agent`、`nanobot gateway` 或 `nanobot serve` 之前设置 Langfuse 凭证：

```bash
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_BASE_URL="https://cloud.langfuse.com"
```

对于 PowerShell：

```powershell
$env:LANGFUSE_SECRET_KEY = "sk-lf-..."
$env:LANGFUSE_PUBLIC_KEY = "pk-lf-..."
$env:LANGFUSE_BASE_URL = "https://cloud.langfuse.com"
```

当设置了 `LANGFUSE_SECRET_KEY` 且已安装 `langfuse` 包时，nanobot 会为 OpenAI 兼容的提供商使用 `langfuse.openai.AsyncOpenAI`，因此模型请求会在后台发送到 Langfuse。如果设置了密钥但缺少 `langfuse`，nanobot 会记录警告并回退到常规的 OpenAI 客户端。

使用与你的项目匹配的 Langfuse 区域或自托管 URL。[Langfuse OpenAI SDK 文档](https://langfuse.com/integrations/model-providers/openai-py) 使用 `LANGFUSE_BASE_URL` 用于云区域和自托管实例。

追踪覆盖通过 nanobot 的 OpenAI 兼容客户端路径的提供商。不使用该客户端的原生提供商可能不会产生 Langfuse OpenAI 包装器追踪。

## 提供商

> [!TIP]
> - **语音转录**：语音消息和 WebUI 麦克风输入使用共享的顶级 `transcription` 设置。默认 `transcription.provider` 值为 `"groq"`；将其设置为 `"openai"` 使用 OpenAI Whisper，`"openrouter"` 使用 OpenRouter 语音转文本模型，`"xiaomi_mimo"` 使用小米 MiMo ASR，或 `"assemblyai"` 使用 AssemblyAI。API 密钥仍保留在匹配的 `providers.<provider>` 配置中。
> - **MiniMax Coding Plan**：nanobot 社区专属折扣链接：[海外](https://platform.minimax.io/subscribe/coding-plan?code=9txpdXw04g&source=link) · [中国大陆](https://platform.minimaxi.com/subscribe/token-plan?code=GILTJpMTqZ&source=link)
> - **MiniMax（中国大陆）**：如果你的 API 密钥来自 MiniMax 的中国大陆平台（minimaxi.com），请在你的 minimax 提供商配置中设置 `"apiBase": "https://api.minimaxi.com/v1"`。
> - **MiniMax 思考模式**：`providers.minimaxAnthropic` 是用于 `reasoningEffort` / 思考模式的配置块。MiniMax 通过其 Anthropic 兼容端点暴露该能力，因此 nanobot 将其保留为单独的提供商，而不是在通用的 OpenAI 兼容 `minimax` 端点上猜测 MiniMax 特定的思考参数。它使用相同的 `MINIMAX_API_KEY`。默认 Anthropic 兼容基础 URL：`https://api.minimax.io/anthropic`；对于中国大陆使用 `https://api.minimaxi.com/anthropic`。
> - **Kimi Coding Plan**：为 Kimi 专用的 Anthropic Messages API 端点使用 `providers.kimiCoding` 与 `provider: "kimi_coding"`。该端点需要 Claude 兼容的 `User-Agent`；nanobot 默认发送 `claude-code/0.1.0`，如果你的账户需要不同的值，可以通过 `extraHeaders.User-Agent` 覆盖它。
> - **VolcEngine / BytePlus Coding Plan**：订阅端点通过专用提供商 `volcengineCodingPlan` 或 `byteplusCodingPlan` 配置，与按量付费的 `volcengine` / `byteplus` 提供商分开。
> - **OpenCode Zen / Go**：`providers.opencode`（规范 Zen）、旧兼容的 `providers.opencodeZen` 和 `providers.opencodeGo` 使用相同的 `OPENCODE_API_KEY`，但路由到不同的 OpenCode 网关。这些提供商使用 OpenCode 的 OpenAI 兼容 `chat/completions` 端点；从该端点系列中选择模型 ID。
> - **Zhipu Coding Plan**：如果你使用智谱的编码计划，请在你的 zhipu 提供商配置中设置 `"apiBase": "https://open.bigmodel.cn/api/coding/paas/v4"`。
> - **阿里云百炼**：如果你使用阿里云百炼的 OpenAI 兼容端点，请在你的 dashscope 提供商配置中设置 `"apiBase": "https://dashscope.aliyuncs.com/compatible-mode/v1"`。
> - **StepFun Step Plan**：如果你使用 StepFun 的 Step Plan 订阅，请在你的 stepfun 提供商配置中设置 `"apiBase": "https://api.stepfun.ai/step_plan/v1"`。支持的模型包括 `step-3.5-flash`、`step-3.5-flash-2603` 和 `step-router-v1`。
> - **阶跃星辰（中国大陆）**：如果你的 API 密钥来自阶跃星辰的中国大陆平台（stepfun.com），请在你的 stepfun 提供商配置中设置 `"apiBase": "https://api.stepfun.com/v1"`。
> - **小米 MiMo 思考模式**：MiMo 模型（例如 `mimo-v2.5-pro`）默认启用思考。使用 `agents.defaults.reasoningEffort: "none"` 禁用它，或 `"low"` / `"medium"` / `"high"` 保持开启。省略该字段会保留提供商的每个模型默认值。
> - **小米 MiMo Token Plan**：如果你使用 MiMo 的 token 计划，请在你的 xiaomi_mimo 提供商配置中设置 `"apiBase": "https://token-plan-sgp.xiaomimimo.com/v1"`。
> - **自定义 OpenAI 兼容提供商**：除了内置的 `custom` 提供商，`providers` 下的任何额外键都可以定义自己的 OpenAI 兼容端点。例如，`providers.companyProxy.apiBase` 加上 `modelPresets.primary.provider: "companyProxy"` 会创建一个单独的自定义提供商。设置 `apiBase`；仅当端点需要时才设置 `apiKey`。这条命名的自定义路径仅使用 OpenAI 兼容的请求格式。对于 Anthropic 兼容的代理，请使用 `providers.anthropic.apiBase` 与 `provider: "anthropic"`。
> - **提供商范围的代理**：`providers.<name>.proxy` 仅将该提供商通过 HTTP 代理路由。它支持 OpenAI 兼容的提供商和 `openai_codex`。原生提供商后端（如 `anthropic`、`bedrock`、`azure_openai` 和 `github_copilot`）会拒绝 `proxy`。

| 提供商 | 用途 | 获取 API 密钥 |
|----------|---------|-------------|
| `custom` | 任何 OpenAI 兼容端点 | — |
| `openrouter` | LLM 网关，用于托管模型系列 + 语音转录（STT 模型） | [openrouter.ai](https://openrouter.ai) |
| `opencode` | LLM 网关（OpenCode Zen 编码智能体模型） | [opencode.ai/docs/zen](https://opencode.ai/docs/zen/) |
| `opencode_zen` | LLM 网关（OpenCode Zen 的旧别名） | [opencode.ai/docs/zen](https://opencode.ai/docs/zen/) |
| `opencode_go` | LLM 网关（OpenCode Go 低成本编码模型） | [opencode.ai/docs/go](https://opencode.ai/docs/go/) |
| `huggingface` | LLM（Hugging Face 推理提供商） | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) |
| `skywork` | LLM（Skywork / APIFree API 网关） | [apifree.ai](https://www.apifree.ai) |
| `volcengine` | LLM（火山引擎，按量付费） | [Coding Plan](https://www.volcengine.com/activity/codingplan?utm_campaign=nanobot&utm_content=nanobot&utm_medium=devrel&utm_source=OWO&utm_term=nanobot) · [volcengine.com](https://www.volcengine.com) |
| `volcengine_coding_plan` | LLM（火山引擎编码计划订阅端点） | [volcengine.com](https://www.volcengine.com/activity/codingplan?utm_campaign=nanobot&utm_content=nanobot&utm_medium=devrel&utm_source=OWO&utm_term=nanobot) |
| `byteplus` | LLM（火山引擎国际版，按量付费） | [Coding Plan](https://www.byteplus.com/en/activity/codingplan?utm_campaign=nanobot&utm_content=nanobot&utm_medium=devrel&utm_source=OWO&utm_term=nanobot) · [byteplus.com](https://www.byteplus.com) |
| `byteplus_coding_plan` | LLM（BytePlus 编码计划订阅端点） | [byteplus.com](https://www.byteplus.com/en/activity/codingplan?utm_campaign=nanobot&utm_content=nanobot&utm_medium=devrel&utm_source=OWO&utm_term=nanobot) |
| `anthropic` | LLM（Claude 直连） | [console.anthropic.com](https://console.anthropic.com) |
| `azure_openai` | LLM（Azure OpenAI） | [portal.azure.com](https://portal.azure.com) |
| `bedrock` | LLM（AWS Bedrock Converse、Claude/Nova/Llama 等） | [aws.amazon.com/bedrock](https://aws.amazon.com/bedrock/) |
| `openai` | LLM + 语音转录（Whisper） | [platform.openai.com](https://platform.openai.com) |
| `assemblyai` | 仅语音转录 | [assemblyai.com](https://www.assemblyai.com/) |
| `deepseek` | LLM（DeepSeek 直连） | [platform.deepseek.com](https://platform.deepseek.com) |
| `groq` | LLM + 语音转录（Whisper，默认） | [console.groq.com](https://console.groq.com) |
| `minimax` | LLM（MiniMax 直连） | [platform.minimaxi.com](https://platform.minimaxi.com) |
| `minimax_anthropic` | LLM（MiniMax Anthropic 兼容端点，思考模式） | [platform.minimaxi.com](https://platform.minimaxi.com) |
| `gemini` | LLM（Gemini 直连） | [aistudio.google.com](https://aistudio.google.com) |
| `aihubmix` | LLM（API 网关，可访问所有模型） | [aihubmix.com](https://aihubmix.com) |
| `siliconflow` | LLM（SiliconFlow/硅基流动） | [siliconflow.cn](https://siliconflow.cn) |
| `novita` | LLM（Novita AI OpenAI 兼容网关） | [novita.ai](https://novita.ai) |
| `dashscope` | LLM（Qwen） | [dashscope.console.aliyun.com](https://dashscope.console.aliyun.com) |
| `moonshot` | LLM（Moonshot/Kimi） | [platform.kimi.com](https://platform.kimi.com?aff=nanobot) |
| `kimi_coding` | LLM（Kimi 编码计划，Anthropic Messages API） | [platform.kimi.com](https://platform.kimi.com?aff=nanobot) |
| `zhipu` | LLM（智谱 GLM） | [open.bigmodel.cn](https://open.bigmodel.cn) |
| `xiaomi_mimo` | LLM（MiMo） | [platform.xiaomimimo.com](https://platform.xiaomimimo.com) |
| `longcat` | LLM（LongCat） | [longcat.chat](https://longcat.chat/platform/docs/zh/) |
| `ant_ling` | LLM（Ant Ling / 蚂蚁百灵） | [developer.ant-ling.com](https://developer.ant-ling.com/en/docs/api-reference/openai/) |
| `ollama` | LLM（本地，Ollama） | — |
| `lm_studio` | LLM（本地，LM Studio） | — |
| `atomic_chat` | LLM（本地，[Atomic Chat](https://atomic.chat/)） | — |
| `mistral` | LLM | [docs.mistral.ai](https://docs.mistral.ai/) |
| `stepfun` | LLM（阶跃星辰） + 语音转录（ASR） | [platform.stepfun.com](https://platform.stepfun.com) |
| `ovms` | LLM（本地，OpenVINO Model Server） | [docs.openvino.ai](https://docs.openvino.ai/2026/model-server/ovms_docs_llm_quickstart.html) |
| `vllm` | LLM（本地，任何 OpenAI 兼容服务器） | — |
| `nvidia` | LLM（NVIDIA NIM） | [build.nvidia.com](https://build.nvidia.com/) |
| `openai_codex` | LLM（Codex，OAuth） | `nanobot provider login openai-codex --set-main` |
| `github_copilot` | LLM（GitHub Copilot，OAuth） | `nanobot provider login github-copilot` |
| `qianfan` | LLM（百度千帆） | [cloud.baidu.com](https://cloud.baidu.com/doc/qianfan/s/Hmh4suq26) |

<details>
<summary><b>OpenAI</b></summary>

默认情况下，OpenAI 使用 `apiType: "auto"`：nanobot 正常调用 Chat Completions，并在有用时通过 Responses API 路由 GPT-5/o 系列或显式的 `reasoningEffort` 请求。你可以强制使用特定的 API 表面：

```json
{
  "providers": {
    "openai": {
      "apiKey": "${OPENAI_API_KEY}",
      "apiType": "chat_completions"
    }
  }
}
```

有效的 `apiType` 值仅为 `auto`、`chat_completions` 和 `responses`。

`extraBody` 遵循所选的 OpenAI API 表面。使用 Chat Completions 时，nanobot 会将其作为 SDK 的 `extra_body` 值传递。使用 Responses 时，请以 Responses API body 形式配置它；nanobot 将普通的顶层字段合并到 Responses 请求体中，将 `extraBody.tools` 附加在生成的函数工具之后，并合并 `extraBody.include` 而不重复：

```json
{
  "providers": {
    "openai": {
      "apiKey": "${OPENAI_API_KEY}",
      "apiType": "responses",
      "extraBody": {
        "tools": [{ "type": "web_search" }],
        "include": ["web_search_call.action.sources"]
      }
    }
  }
}
```

</details>

<details>
<summary><b>Azure OpenAI</b></summary>

`azure_openai` 提供商通过 OpenAI **Responses API**（`/openai/v1/responses`）与你的 Azure OpenAI 资源通信。模型名称映射到 **部署名称**，而不是 OpenAI 模型 ID。支持两种认证模式。

**模式 1：静态 API 密钥**（最简单）

```json
{
  "providers": {
    "azure_openai": {
      "apiKey": "${AZURE_OPENAI_API_KEY}",
      "apiBase": "https://my-resource.openai.azure.com"
    }
  },
  "modelPresets": {
    "azure": {
      "provider": "azure_openai",
      "model": "my-gpt-5-deployment"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "azure"
    }
  }
}
```

**模式 2：通过 `DefaultAzureCredential` 使用 Microsoft Entra ID（Azure AD）**

省略 `apiKey`（或将其保留为空/未设置）。提供商会回退到 [`DefaultAzureCredential`](https://learn.microsoft.com/azure/developer/python/sdk/authentication/credential-chains#defaultazurecredential-overview)，并为每个请求获取一个作用域为 `https://cognitiveservices.azure.com/.default` 的 bearer token。Azure SDK 自身的基于 MSAL 的缓存无需网络往返即可返回有效 token。

```json
{
  "providers": {
    "azure_openai": {
      "apiBase": "https://my-resource.openai.azure.com"
    }
  },
  "modelPresets": {
    "azure": {
      "provider": "azure_openai",
      "model": "my-gpt-5-deployment"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "azure"
    }
  }
}
```

安装可选依赖：

```bash
nanobot plugins enable azure
```

`DefaultAzureCredential` 按顺序遍历此链，并使用第一个成功的身份：

1. **EnvironmentCredential** — 读取 `AZURE_TENANT_ID`、`AZURE_CLIENT_ID`，以及 `AZURE_CLIENT_SECRET` / `AZURE_CLIENT_CERTIFICATE_PATH` / `AZURE_USERNAME` + `AZURE_PASSWORD` 之一。
2. **WorkloadIdentityCredential** — 用于 AKS 工作负载身份 / 联合 token（`AZURE_FEDERATED_TOKEN_FILE`）。
3. **ManagedIdentityCredential** — 用于 Azure VM、App Service、Functions、Container Apps 等。
4. **AzureCliCredential** — 使用你开发机上 `az login` 的 token。
5. **AzurePowerShellCredential** — 使用 `Connect-AzAccount` 的 token。
6. **AzureDeveloperCliCredential** — 使用 `azd auth login` 的 token。
7. **InteractiveBrowserCredential** *（默认禁用）*。

最终签署请求的身份**必须被分配 `Cognitive Services OpenAI User` RBAC 角色**（或更高级别）到 Azure OpenAI 资源上。没有该角色，你将在第一次请求时看到 `401`/`403` 错误。

> `apiBase` 在两种模式下都是必需的——它是你的 Azure 资源端点，无法推断。如果既未设置 `apiKey`，也未安装 `azure-identity`，提供商会抛出明确的错误，指引你使用 `nanobot plugins enable azure`。

</details>

<details>
<summary><b>Skywork / APIFree</b></summary>

Skywork 使用 APIFree 的 OpenAI 兼容 Agent API 端点。一次配置提供商，然后使用 Skywork 模型 ID，例如 `skywork-ai/skyclaw-v1`。

```json
{
  "providers": {
    "skywork": {
      "apiKey": "${SKYWORK_API_KEY}",
      "apiBase": "https://api.apifree.ai/agent/v1"
    }
  },
  "modelPresets": {
    "skywork": {
      "provider": "skywork",
      "model": "skywork-ai/skyclaw-v1",
      "maxTokens": 32768,
      "contextWindowTokens": 131072
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "skywork"
    }
  }
}
```

如果你的环境中凭据名为 `${APIFREE_API_KEY}`，也可以在 `apiKey` 中引用。

</details>

<details>
<summary><b>AWS Bedrock (Converse API)</b></summary>

Bedrock 使用原生的 `bedrock-runtime` Converse API，因此可以调用 Bedrock 模型 ID，例如 Claude Opus 4.7、Claude Sonnet、Amazon Nova、Meta Llama、Mistral、Qwen 以及其他支持 Converse 的模型。它支持普通对话、流式输出、工具调用、工具结果、token 用量以及 Bedrock 错误元数据。

此提供商用于 Bedrock 的原生 Converse API，而不是 Bedrock 兼容 OpenAI 的 `/openai/v1` 端点。对于兼容 OpenAI 的 Bedrock 模型，如果你特别想使用该 API 表面，仍然可以使用 `custom`。

首先安装 Bedrock 支持：

```bash
nanobot plugins enable bedrock
```

> [!NOTE]
> 如果你在 `boto3` 成为可选依赖之前就已配置 Bedrock，请在升级后运行
> `nanobot plugins enable bedrock`。否则该提供商在首次尝试创建 Bedrock 客户端时
> 将会失败。

**1. 配置凭据**

使用常规的 AWS 凭据链（`AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`、AWS profile 或 IAM 角色）。IAM 身份需要：

```json
{
  "Effect": "Allow",
  "Action": [
    "bedrock:InvokeModel",
    "bedrock:InvokeModelWithResponseStream"
  ],
  "Resource": "*"
}
```

你也可以将 `providers.bedrock.apiKey` 设置为 Bedrock API 密钥；nanobot 会将其作为 `AWS_BEARER_TOKEN_BEDROCK` 导出供 AWS SDK 使用。

凭据选项：

- **AWS CLI/默认 profile**：留空 `apiKey` 和 `profile`，然后运行 `aws configure` 或提供 `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`。
- **命名 AWS profile**：将 `profile` 设置为 `~/.aws/config` 或 `~/.aws/credentials` 中的某个 profile。
- **IAM 角色**：在 EC2/ECS/Lambda 上，留空 `apiKey` 和 `profile`，并附加带有 Bedrock 权限的角色。
- **Bedrock API 密钥**：设置 `apiKey` 或 `AWS_BEARER_TOKEN_BEDROCK`；`profile` 可以保持 `null`。

**2. 最小配置**

对于非 Anthropic 模型（例如 Amazon Nova）：

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1"
    }
  },
  "modelPresets": {
    "bedrockNova": {
      "provider": "bedrock",
      "model": "bedrock/amazon.nova-lite-v1:0",
      "reasoningEffort": null
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "bedrockNova"
    }
  }
}
```

使用 Bedrock API 密钥：

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1",
      "apiKey": "${AWS_BEARER_TOKEN_BEDROCK}"
    }
  },
  "modelPresets": {
    "bedrockNova": {
      "provider": "bedrock",
      "model": "bedrock/amazon.nova-lite-v1:0",
      "reasoningEffort": null
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "bedrockNova"
    }
  }
}
```

使用命名 AWS profile：

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1",
      "profile": "my-bedrock-profile"
    }
  },
  "modelPresets": {
    "bedrockNova": {
      "provider": "bedrock",
      "model": "bedrock/amazon.nova-lite-v1:0"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "bedrockNova"
    }
  }
}
```

**3. Claude Opus 4.7 示例**

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1"
    }
  },
  "modelPresets": {
    "bedrockClaude": {
      "provider": "bedrock",
      "model": "bedrock/global.anthropic.claude-opus-4-7",
      "reasoningEffort": "medium",
      "maxTokens": 8192
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "bedrockClaude"
    }
  }
}
```

对于区域路由，请使用 Bedrock 的某个推理 ID，例如 `bedrock/us.anthropic.claude-opus-4-7`、`bedrock/eu.anthropic.claude-opus-4-7` 或 `bedrock/jp.anthropic.claude-opus-4-7`。

Claude Opus 4.7 不接受 `temperature`、`top_p` 或 `top_k`；nanobot 会为该模型自动省略 `temperature`。如果 `reasoningEffort` 被设置为 `low`、`medium`、`high`、`max` 或 `adaptive`，nanobot 会发送 Bedrock 的自适应思考参数。

Bedrock 上的 Anthropic 模型可能还需要进行 Anthropic 用例注册，并受 Anthropic 支持的国家/区域限制的约束。如果 Claude 因不支持的国家或区域而返回 `ValidationException` 失败，请尝试使用非 Anthropic 的 Bedrock 模型（例如 Amazon Nova）来验证提供商设置。

**4. 模型 ID**

在 nanobot 配置中，使用带有 `bedrock/` 前缀的 Bedrock 模型 ID 或推理配置文件 ID。nanobot 在调用 AWS 前会去除该前缀。

示例：

- `bedrock/amazon.nova-micro-v1:0`
- `bedrock/amazon.nova-lite-v1:0`
- `bedrock/global.anthropic.claude-opus-4-7`
- `bedrock/us.anthropic.claude-opus-4-7`
- `bedrock/openai.gpt-oss-20b-1:0`
- `bedrock/meta.llama...`
- `bedrock/mistral...`

请查阅 Bedrock 控制台以确认确切的模型 ID 和区域可用性。某些模型需要跨区域推理配置文件 ID，例如 `us.*`、`eu.*` 或 `global.*`。

**5. 高级模型字段**

模型特有的字段可以通过 `extraBody` 提供；nanobot 会将其合并到 Converse 的 `additionalModelRequestFields` 中：

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1",
      "extraBody": {
        "thinking": {
          "type": "adaptive",
          "effort": "medium",
          "display": "summarized"
        }
      }
    }
  }
}
```

仅在需要自定义 Bedrock Runtime 端点 URL（例如 VPC 端点或代理）时使用 `apiBase`。对于常规 AWS 区域并不需要。

当前范围：nanobot 会传递 `messages`、`system`、`inferenceConfig`、`toolConfig` 和 `additionalModelRequestFields`。Bedrock Prompt Management、Guardrails、`serviceTier` 以及其他顶级 Converse 选项目前还不是一等配置字段。

**6. 快速检查**

```bash
# For AWS credential-chain usage:
aws sts get-caller-identity

# For API-key usage:
export AWS_BEARER_TOKEN_BEDROCK="your-bedrock-api-key"
export AWS_REGION="us-east-1"
```

然后运行：

```bash
nanobot agent -m "Reply with one short sentence."
```

</details>


<details>
<summary><b>OpenAI Codex (OAuth)</b></summary>

Codex 使用 OAuth 而不是 API 密钥，并且需要 ChatGPT Plus 或 Pro 账户。使用一条命令即可完成认证并将当前旗舰模型设为当前智能体模型：

```bash
nanobot provider login openai-codex --set-main
```

然后运行：

```bash
nanobot agent -m "Hello!"
```

有关代理、远程/无头登录、模型名或配置密钥错误，请参见 [`troubleshooting.md`](./troubleshooting.md#provider-and-model-problems)。

</details>


<details>
<summary><b>GitHub Copilot (OAuth)</b></summary>

GitHub Copilot 使用 OAuth 而不是 API 密钥。需要已配置 [带有套餐的 GitHub 账户](https://github.com/features/copilot/plans)。在 `config.json` 中不需要 `providers.github_copilot` 块；`nanobot provider login` 会将 OAuth 会话存储在配置之外。

对于 GitHub Enterprise / Copilot for Business，请在登录前设置你需要的端点覆盖：
```bash
export NANOBOT_GITHUB_COPILOT_CLIENT_ID="your-enterprise-client-id"
export NANOBOT_GITHUB_DEVICE_CODE_URL="https://ghe.example/login/device/code"
export NANOBOT_GITHUB_ACCESS_TOKEN_URL="https://ghe.example/login/oauth/access_token"
export NANOBOT_GITHUB_USER_URL="https://api.ghe.example/user"
export NANOBOT_COPILOT_TOKEN_URL="https://api.ghe.example/copilot_internal/v2/token"
export NANOBOT_COPILOT_BASE_URL="https://copilot-api.ghe.example"
```

**1. 登录：**
```bash
nanobot provider login github-copilot
```

**2. 设置模型**（合并到 `~/.nanobot/config.json`）：
```json
{
  "modelPresets": {
    "copilot": {
      "provider": "github_copilot",
      "model": "github-copilot/gpt-4.1"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "copilot"
    }
  }
}
```

**3. 对话：**
```bash
nanobot agent -m "Hello!"

# Target a specific workspace/config locally
nanobot agent -c ~/.nanobot-telegram/config.json -m "Hello!"

# One-off workspace override on top of that config
nanobot agent -c ~/.nanobot-telegram/config.json -w /tmp/nanobot-telegram-test -m "Hello!"
```

> Docker 用户：使用 `docker run -it` 进行交互式 OAuth 登录。

</details>

<details>
<summary><b>OpenCode Zen / Go</b></summary>

OpenCode Zen 和 OpenCode Go 通过 nanobot 内置的
兼容 OpenAI 的提供商流程使用。它们共享 `OPENCODE_API_KEY` 环境
变量，但使用不同的提供商键和默认 base URL：

| 提供商 | 默认 API base | nanobot 接受的模型前缀 |
|----------|------------------|-----------------------------------|
| `opencode` | `https://opencode.ai/zen/v1` | `opencode/<model-id>` |
| `opencode_zen` | `https://opencode.ai/zen/v1` | `opencode/<model-id>` |
| `opencode_go` | `https://opencode.ai/zen/go/v1` | `opencode-go/<model-id>` |

OpenCode Zen：

```json
{
  "providers": {
    "opencode": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "opencodeZen": {
      "provider": "opencode",
      "model": "opencode/deepseek-v4-pro"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "opencodeZen"
    }
  }
}
```

`providers.opencodeZen` / `provider: "opencode_zen"` 仍作为现有配置的兼容别名可用。

OpenCode Go：

```json
{
  "providers": {
    "opencodeGo": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "opencodeGo": {
      "provider": "opencode_go",
      "model": "opencode-go/deepseek-v4-flash"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "opencodeGo"
    }
  }
}
```

OpenCode 自己的文档在 `responses`、`messages`、
提供商特定的模型端点以及 `chat/completions` 中列出了模型。nanobot 的 OpenCode
提供商使用兼容 OpenAI 的 `chat/completions` 路径，因此请从该端点系列中
选择模型 ID。`opencode/...` 和 `opencode-go/...` 前缀出于配置可读性
的目的被接受，并在发送请求前被去除。

</details>

<details>
<summary><b>LongCat（兼容 OpenAI）</b></summary>

LongCat 通过 nanobot 内置的兼容 OpenAI 的提供商流程使用。默认 API base 已经指向 `https://api.longcat.chat/openai/v1`，因此你通常只需要设置 `apiKey`。

```json
{
  "providers": {
    "longcat": {
      "apiKey": "${LONGCAT_API_KEY}"
    }
  },
  "modelPresets": {
    "longcat": {
      "provider": "longcat",
      "model": "LongCat-2.0-Preview",
      "maxTokens": 8192,
      "contextWindowTokens": 1048576
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "longcat"
    }
  }
}
```

当前 LongCat API 文档将 `LongCat-2.0-Preview` 列为受支持的模型。较早的 `LongCat-Flash-*` 模型已由 LongCat 于 2026-05-29 下线。

</details>

<details>
<summary><b>Xiaomi MiMo</b></summary>

当模型名称包含 `mimo` 时，`xiaomi_mimo` 提供商会自动检测 Xiaomi MiMo 模型。默认 API base 是 `https://api.xiaomimimo.com/v1`。

> **Token 套餐**：如果你在使用 MiMo 的 token 套餐，请使用专用端点覆盖 `apiBase`：
>
> ```json
> {
>   "providers": {
>     "xiaomi_mimo": {
>       "apiKey": "${XIAOMIMIMO_API_KEY}",
>       "apiBase": "https://token-plan-sgp.xiaomimimo.com/v1"
>     }
>   },
>   "modelPresets": {
>     "mimo": {
>       "provider": "xiaomi_mimo",
>       "model": "xiaomi/mimo-v2.5-pro"
>     }
>   },
>   "agents": {
>     "defaults": {
>       "modelPreset": "mimo"
>     }
>   }
> }
> ```
>
> 使用 MiMo token 套餐控制台中的模型 ID 和 API 密钥，并在 MiMo 平台上查看最新支持的模型名称。

</details>

<details>
<summary><b>StepFun Step Plan（订阅制）</b></summary>

Step Plan 是 StepFun 面向高频 AI 开发者的订阅制服务。如果你订阅了 Step Plan，请覆盖已有 `stepfun` 提供商配置中的 `apiBase`，将其指向专用的 Step Plan 端点。

```json
{
  "providers": {
    "stepfun": {
      "apiKey": "${STEPFUN_API_KEY}",
      "apiBase": "https://api.stepfun.ai/step_plan/v1"
    }
  },
  "modelPresets": {
    "stepfun": {
      "provider": "stepfun",
      "model": "step-3.5-flash"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "stepfun"
    }
  }
}
```

支持的模型包括 `step-3.5-flash`、`step-3.5-flash-2603` 和 `step-router-v1`。

</details>

<details>
<summary><b>Ant Ling (OpenAI-compatible)</b></summary>

Ant Ling 可通过 nanobot 内置的 OpenAI 兼容提供商流程使用。默认 API 基础地址指向 `https://api.ant-ling.com/v1`，因此你通常只需要设置 `apiKey`。

```json
{
  "providers": {
    "antLing": {
      "apiKey": "${ANT_LING_API_KEY}"
    }
  },
  "modelPresets": {
    "antLing": {
      "provider": "ant_ling",
      "model": "Ling-2.6-flash"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "antLing"
    }
  }
}
```

官方 OpenAI 兼容模型名称包括 `Ling-2.6-1T`、`Ling-2.6-flash`、`Ling-2.5-1T`、`Ling-1T`、`Ring-2.5-1T` 和 `Ring-1T`。

</details>

<details>
<summary><b>Custom Provider (Any OpenAI-compatible API)</b></summary>

直接连接到任何 OpenAI 兼容端点 —— llama.cpp、Together AI、Fireworks、Azure OpenAI 或任何自托管服务器。模型名称按原样传递。

```json
{
  "providers": {
    "custom": {
      "apiKey": "your-api-key",
      "apiBase": "https://api.your-provider.com/v1"
    }
  },
  "modelPresets": {
    "custom": {
      "provider": "custom",
      "model": "your-model-name"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "custom"
    }
  }
}
```

> 对于不需要认证的本地服务器，将 `apiKey` 设置为 `null`。
>
> 对于暴露 OpenAI 兼容的 **chat completions** API 的提供商，`custom` 是正确的选择。它 **不会** 强制第三方端点使用 OpenAI/Azure 的 **Responses API**。
>
> 如果你的代理或网关明确兼容 Responses API，请配置 `azure_openai` 提供商结构，并将 `apiBase` 指向该端点：
>
> ```json
> {
>   "providers": {
>     "azure_openai": {
>       "apiKey": "your-api-key",
>       "apiBase": "https://api.your-provider.com",
>       "defaultModel": "your-model-name"
>     }
>   },
>   "modelPresets": {
>     "responsesProxy": {
>       "provider": "azure_openai",
>       "model": "your-model-name"
>     }
>   },
>   "agents": {
>     "defaults": {
>       "modelPreset": "responsesProxy"
>     }
>   }
> }
> ```
>
> Anthropic 兼容端点是独立的：使用 `providers.anthropic.apiBase` 并将预设的 provider 设置为 `anthropic`。任意自定义提供商名称不会使用 Anthropic Messages API 格式。
>
> 简言之：**chat-completions 兼容端点 → `custom` 或命名的自定义提供商**；**Responses 兼容端点 → `azure_openai`**；**Anthropic 兼容端点 → 带 `apiBase` 的 `anthropic`**。

某些 OpenAI 兼容网关暴露了请求体扩展，例如 vLLM 的引导解码或本地采样控制。将这些放在 `extraBody` 下；nanobot 会在其提供商默认值之后将它们合并到 chat-completions 请求体中：

```json
{
  "providers": {
    "custom": {
      "apiKey": "your-api-key",
      "apiBase": "https://api.your-provider.com/v1",
      "extraBody": {
        "repetition_penalty": 1.15,
        "chat_template_kwargs": {
          "enable_thinking": false
        }
      }
    }
  }
}
```

如果自定义 OpenAI 兼容端点暴露了特定于提供商的思考开关，请设置 `thinkingStyle`，以便 nanobot 能将 `reasoningEffort` 转换为正确的请求体。支持的样式包括 `thinking_type`（`{"thinking":{"type":"enabled"}}`）、`enable_thinking`（`{"enable_thinking": true}`）和 `reasoning_split`（`{"reasoning_split": true}`）：

```json
{
  "providers": {
    "companyProxy": {
      "apiKey": "${COMPANY_PROXY_API_KEY}",
      "apiBase": "https://api.your-provider.com/v1",
      "thinkingStyle": "enable_thinking"
    }
  },
  "modelPresets": {
    "company": {
      "provider": "companyProxy",
      "model": "served-model-name",
      "reasoningEffort": "high"
    }
  }
}
```

除非端点明确记录了这些线路格式之一，否则请不要设置 `thinkingStyle`。`extraBody` 仍在最后应用，因此高级用户可以覆盖生成的值。

</details>

<a id="local-providers"></a>
<a id="ollama-local"></a>
<details>
<summary><b>Ollama (local)</b></summary>

使用 Ollama 运行本地模型，然后添加到配置中：

**1. 启动 Ollama**（示例）：
```bash
ollama run llama3.2
```

**2. 添加到配置**（部分 —— 合并到 `~/.nanobot/config.json`）：
```json
{
  "providers": {
    "ollama": {
      "apiBase": "http://localhost:11434"
    }
  },
  "modelPresets": {
    "ollama": {
      "provider": "ollama",
      "model": "llama3.2"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "ollama"
    }
  }
}
```

> 当配置了 `providers.ollama.apiBase` 时，`provider: "auto"` 也可以工作，但在预设中固定 `"provider": "ollama"` 是最清晰的选择。

</details>

<details>
<summary><b>LM Studio (local)</b></summary>

[LM Studio](https://lmstudio.ai/) 提供用于运行 LLM 的本地 OpenAI 兼容服务器。通过 LM Studio UI 下载模型，然后启动本地服务器。

**1. 启动 LM Studio 服务器：**
- 启动 LM Studio
- 前往 "Local Server" 标签页
- 加载模型（例如 Llama、Mistral、Qwen）
- 点击 "Start Server"（默认端口：1234）

**2. 添加到配置**（部分 —— 合并到 `~/.nanobot/config.json`）：
```json
{
  "providers": {
    "lm_studio": {
      "apiKey": null,
      "apiBase": "http://localhost:1234/v1"
    }
  },
  "modelPresets": {
    "lmStudio": {
      "provider": "lm_studio",
      "model": "local-model"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "lmStudio"
    }
  }
}
```

> **注意：** 由于 LM Studio 在本地运行且不需要认证，请将 `apiKey` 设置为 `null`。模型名称应与 LM Studio UI 中显示的一致。当配置了 `providers.lm_studio.apiBase` 时，`provider: "auto"` 也可以工作，但在预设中固定 `"provider": "lm_studio"` 是最清晰的选择。

</details>

<a id="atomic-chat-local"></a>
<details>
<summary><b>Atomic Chat (local)</b></summary>

[Atomic Chat](https://atomic.chat/) 是一个本地优先的桌面应用，暴露了 **OpenAI 兼容** 的 HTTP API（默认 `http://localhost:1337/v1`）。当你想让 nanobot 使用自己机器上的模型而不是托管 API 提供商时，可以采用这种设置。

**1. 启动 Atomic Chat**

- 在你的机器上安装 [Atomic Chat](https://atomic.chat/)。
- 打开 Atomic Chat，下载一个模型，并保持应用运行。本地 API 默认启用。
- 复制本地 API 暴露的模型 ID。例如，`Qwen 3 32B` 的模型 ID 可能是 `qwen3-32b`。

**2. 添加到配置**（部分 —— 合并到 `~/.nanobot/config.json`）：

```json
{
  "providers": {
    "atomic_chat": {
      "apiKey": null,
      "apiBase": "http://localhost:1337/v1"
    }
  },
  "modelPresets": {
    "atomic": {
      "provider": "atomic_chat",
      "model": "qwen3-32b"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "atomic"
    }
  }
}
```

> **注意：** 将 `qwen3-32b` 替换为 Atomic Chat 提供的模型 ID。如果你的 Atomic Chat 服务器不需要密钥，请将 `apiKey` 设置为 `null`。如果需要，请将 `apiKey`（或 `ATOMIC_CHAT_API_KEY` 环境变量）设置为 Atomic Chat 期望的值。

> 当配置了 `providers.atomic_chat.apiBase` 时，`provider: "auto"` 也可以工作，但在预设中固定 `"provider": "atomic_chat"` 是最清晰的选择。

</details>

<details>
<summary><b>OpenVINO Model Server (local / OpenAI-compatible)</b></summary>

使用 [OpenVINO Model Server](https://docs.openvino.ai/2026/model-server/ovms_docs_llm_quickstart.html) 在 Intel GPU 上本地运行 LLM。OVMS 在 `/v3` 暴露 OpenAI 兼容 API。

> 需要 Docker 和具有驱动访问权限的 Intel GPU（`/dev/dri`）。

**1. 拉取模型**（示例）：

```bash
mkdir -p ov/models && cd ov

docker run -d \
  --rm \
  --user $(id -u):$(id -g) \
  -v $(pwd)/models:/models \
  openvino/model_server:latest-gpu \
  --pull \
  --model_name openai/gpt-oss-20b \
  --model_repository_path /models \
  --source_model OpenVINO/gpt-oss-20b-int4-ov \
  --task text_generation \
  --tool_parser gptoss \
  --reasoning_parser gptoss \
  --enable_prefix_caching true \
  --target_device GPU
```

> 这将下载模型权重。等待容器完成后再继续。

**2. 启动服务器**（示例）：

```bash
docker run -d \
  --rm \
  --name ovms \
  --user $(id -u):$(id -g) \
  -p 8000:8000 \
  -v $(pwd)/models:/models \
  --device /dev/dri \
  --group-add=$(stat -c "%g" /dev/dri/render* | head -n 1) \
  openvino/model_server:latest-gpu \
  --rest_port 8000 \
  --model_name openai/gpt-oss-20b \
  --model_repository_path /models \
  --source_model OpenVINO/gpt-oss-20b-int4-ov \
  --task text_generation \
  --tool_parser gptoss \
  --reasoning_parser gptoss \
  --enable_prefix_caching true \
  --target_device GPU
```

**3. 添加到配置**（部分 —— 合并到 `~/.nanobot/config.json`）：

```json
{
  "providers": {
    "ovms": {
      "apiBase": "http://localhost:8000/v3"
    }
  },
  "modelPresets": {
    "ovms": {
      "provider": "ovms",
      "model": "openai/gpt-oss-20b"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "ovms"
    }
  }
}
```

> OVMS 是本地服务器 —— 不需要 API 密钥。支持工具调用（`--tool_parser gptoss`）、推理（`--reasoning_parser gptoss`）和流式传输。更多详情请参见 [OVMS 官方文档](https://docs.openvino.ai/2026/model-server/ovms_docs_llm_quickstart.html)。
</details>

<a id="vllm-local-openai-compatible"></a>
<details>
<summary><b>vLLM (local / OpenAI-compatible)</b></summary>

使用 vLLM 或任何 OpenAI 兼容服务器运行你自己的模型，然后添加到配置中：

**1. 启动服务器**（示例）：
```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --port 8000
```

**2. 添加到配置**（部分 —— 合并到 `~/.nanobot/config.json`）：

*提供商（对于本地服务器，将 API 密钥设置为 null）：*
```json
{
  "providers": {
    "vllm": {
      "apiKey": null,
      "apiBase": "http://localhost:8000/v1"
    }
  }
}
```

*模型预设：*
```json
{
  "modelPresets": {
    "vllm": {
      "provider": "vllm",
      "model": "meta-llama/Llama-3.1-8B-Instruct"
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "vllm"
    }
  }
}
```

</details>

添加新提供商的贡献者说明位于 [`development.md`](./development.md#adding-an-llm-provider)。

## 模型预设

模型预设允许你为完整的模型配置命名，并在运行时通过 `/model <preset>` 切换。它们是推荐的模型配置方式，因为相同的名称可以用于启动选择、聊天命令切换和回退链。

现有配置不需要更改。直接的 `agents.defaults.model`、`provider`、`maxTokens`、`contextWindowTokens`、`temperature` 和 `reasoningEffort` 字段仍定义隐式的 `default` 预设。对于新配置，建议使用顶层 `modelPresets` 加上 `agents.defaults.modelPreset`。

```json
{
  "modelPresets": {
    "fast": {
      "provider": "openrouter",
      "model": "anthropic/claude-sonnet-4.5",
      "maxTokens": 4096,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast",
      "fallbackModels": ["deep", "localSmall"]
    }
  },
  "modelPresets": {
    "fast": {
      "label": "Fast",
      "model": "gpt-4.1-mini",
      "provider": "openai",
      "maxTokens": 4096,
      "contextWindowTokens": 128000,
      "temperature": 0.2,
      "reasoningEffort": "low"
    },
    "deep": {
      "label": "Deep",
      "model": "claude-opus-4-5",
      "provider": "anthropic",
      "maxTokens": 8192,
      "contextWindowTokens": 200000,
      "reasoningEffort": "high"
    },
    "localSmall": {
      "label": "Local Small",
      "model": "llama3.2",
      "provider": "ollama",
      "maxTokens": 4096,
      "contextWindowTokens": 32768,
      "temperature": 0.2
    }
  }
}
```

`modelPresets` 是一个顶层对象。其下的键（`fast`、`deep`、`coding` 等）是用户自定义的预设名称。每个预设支持：

| 字段 | 描述 |
|-------|-------------|
| `label` | 可选的显示名称，展示在模型列表中。 |
| `model` | 该预设使用的模型名称。 |
| `provider` | 提供商名称，或 `"auto"` 以使用提供商自动检测。 |
| `maxTokens` | 最大补全/输出 token 数。 |
| `contextWindowTokens` | 用于构建提示词和整合决策的上下文窗口大小。 |
| `temperature` | 采样温度。 |
| `reasoningEffort` | 可选的推理/思考设置。不同提供商的支持情况有所不同。 |

`default` 为保留名称，始终代表由 `agents.defaults.*` 直接字段构建的隐式预设；请勿定义 `modelPresets.default`。在现有配置中使用 `/model default` 可切换回这些直接字段。

设置 `agents.defaults.modelPreset` 以选择启动时使用的预设。当 `modelPreset` 为 `null` 或省略时，启动会使用由 `agents.defaults.*` 直接字段构成的隐式 `default` 预设。运行时通过 `/model <preset>` 所做的更改不会写回 `config.json`；它们仅影响后续对话轮次，直到进程重启或被其他模型/配置更改替换。

### 模型回退

`agents.defaults.fallbackModels` 为当前活跃模型配置定义一条有序的故障切换链。主模型仍由 `agents.defaults.modelPreset` 选择，在较旧的配置中则由 `agents.defaults.*` 直接字段构成的隐式 `default` 预设选择。

每个回退候选项可以是：

- `modelPresets` 中的一个预设名，例如 `"deep"`。这是推荐形式。该预设的完整模型、提供商、生成参数和上下文窗口配置都会被使用。
- 一个至少包含 `provider` 和 `model` 的内联回退对象。省略时，可选的 `maxTokens`、`contextWindowTokens` 和 `temperature` 字段会从当前活跃的主配置继承。`reasoningEffort` 不继承；省略它可为该回退关闭推理，或对支持推理的模型显式设置。

预设回退链：

```json
{
  "modelPresets": {
    "fast": {
      "model": "gpt-4.1-mini",
      "provider": "openai",
      "maxTokens": 4096,
      "contextWindowTokens": 128000,
      "temperature": 0.2
    },
    "deep": {
      "model": "claude-opus-4-5",
      "provider": "anthropic",
      "maxTokens": 8192,
      "contextWindowTokens": 200000,
      "reasoningEffort": "high"
    },
    "localSmall": {
      "model": "llama3.2",
      "provider": "ollama",
      "maxTokens": 4096,
      "contextWindowTokens": 32768
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast",
      "fallbackModels": ["deep", "localSmall"]
    }
  }
}
```

字符串条目是预设名，而不是原始模型名。在上例中，`"deep"` 表示 `modelPresets.deep`；nanobot 不会将其解释为提供商的模型 ID。更改一个预设会同时更新 `/model <preset>` 切换和任何引用它的回退链。

内联回退对象：

```json
{
  "modelPresets": {
    "fast": {
      "provider": "openrouter",
      "model": "anthropic/claude-sonnet-4.5",
      "maxTokens": 4096,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast",
      "fallbackModels": [
        {
          "provider": "deepseek",
          "model": "deepseek-v4-pro",
          "maxTokens": 4096,
          "contextWindowTokens": 262144
        }
      ]
    }
  }
}
```

仅当某个回退不值得命名为可复用预设时才使用内联对象。`fallbackModels` 应放在 `agents.defaults` 下，而不是单个 `modelPresets` 条目内。

当主提供商在流式返回任何回答文本之前返回可重试的模型/提供商错误时，故障切换通常会启动。流式停滞超时是恢复的例外情况：如果提供商已经发出部分回答文本然后停滞，nanobot 会关闭当前流片段并在新片段中重试/切换。典型的回退场景包括超时、连接错误、5xx 服务器错误、429 限速、过载以及配额/余额耗尽。它不会在以下情况下运行：请求格式错误、认证/权限错误、内容过滤/拒绝，或上下文长度/消息格式错误。

如果回退候选使用了较小的 `contextWindowTokens` 值，nanobot 会使用当前活跃链中最小的窗口来构建上下文，以便每个候选都能收到相同的提示词。

## 转录设置

音频转录是聊天频道语音消息和 WebUI 麦克风输入共享的能力。聊天频道语音消息会在进入智能体之前自动转录。WebUI 麦克风输入会先转录到编辑框中，以便你在发送前编辑文本。

在顶层 `transcription` 部分配置转录：

```json
{
  "transcription": {
    "enabled": true,
    "provider": "groq",
    "model": null,
    "language": null,
    "maxDurationSec": 120,
    "maxUploadMb": 25
  }
}
```

| 设置 | 默认值 | 描述 |
|---------|---------|-------------|
| `enabled` | `true` | 为聊天频道语音消息和 WebUI 麦克风输入启用音频转录。 |
| `provider` | `"groq"` | 转录后端：`"groq"`、`"openai"`、`"openrouter"`、`"xiaomi_mimo"`、`"stepfun"` 或 `"assemblyai"`。 |
| `model` | 提供商默认 | 可选的转录模型覆盖。默认值：Groq 使用 `whisper-large-v3`，OpenAI 使用 `whisper-1`，OpenRouter 使用 `openai/whisper-1`，小米 MiMo ASR 使用 `mimo-v2.5-asr`，StepFun ASR 使用 `stepaudio-2.5-asr`，AssemblyAI 使用 `universal-3-pro,universal-2`。OpenRouter 的转录端点仅接受语音转文字模型，例如 `nvidia/parakeet-tdt-0.6b-v3`、`openai/whisper-1` 或 `openai/gpt-4o-transcribe`；在此处聊天 LLM 会被拒绝。AssemblyAI 接受以逗号分隔的模型回退列表。 |
| `language` | `null` | 可选的 ISO-639 语言提示，例如 `"en"`、`"zh"`、`"ko"` 或 `"ja"`。 |
| `maxDurationSec` | `120` | WebUI 最大录音时长。 |
| `maxUploadMb` | `25` | WebUI 最大音频上传大小。 |

出于向后兼容考虑，提供商和语言的解析顺序如下：

1. `transcription.provider` / `transcription.language`
2. 遗留的 `channels.transcriptionProvider` / `channels.transcriptionLanguage`
3. 内置默认值（`provider: "groq"`，无语言提示）

遗留的 `channels.*` 转录字段是在转录成为聊天频道和 WebUI 麦克风输入共享能力之前存在的。它们仍会被读取以便旧的 `config.json` 文件继续可用，但已不再是首选的配置方式。如果新旧字段同时存在，顶层 `transcription` 的值将作为唯一权威来源。

转录凭据有意不存储在 `transcription` 中。请将 API 密钥和可选端点放入匹配的提供商配置：

```json
{
  "providers": {
    "groq": {
      "apiKey": "gsk-...",
      "apiBase": "https://api.groq.com/openai/v1"
    }
  },
  "transcription": {
    "provider": "groq",
    "language": "zh"
  }
}
```

选择转录提供商本身并不会配置凭据。例如，为兼容性起见有效提供商可能默认为 Groq，但只有在 `providers.groq.apiKey` 或匹配的环境变量支持的配置可用时，转录才可使用。设置界面只写入顶层 `transcription` 字段。

如果你要新增一个转录提供商，请参阅 [`development.md`](./development.md#adding-a-transcription-provider)。

## 频道设置

适用于所有频道的全局设置。在 `~/.nanobot/config.json` 的 `channels` 部分进行配置：

```json
{
  "channels": {
    "sendProgress": true,
    "sendToolHints": false,
    "extractDocumentText": true,
    "sendMaxRetries": 3,
    "telegram": {
      "enabled": false
    }
  }
}
```

| 设置 | 默认值 | 描述 |
|---------|---------|-------------|
| `sendProgress` | `true` | 将智能体的文本进度流式发送到频道 |
| `sendToolHints` | `false` | 流式发送工具调用提示（例如 `read_file("…")`） |
| `showReasoning` | `true` | 允许频道呈现模型的推理/思考内容（DeepSeek-R1 `reasoning_content`、Anthropic `thinking_blocks`、内联 `<think>` 标签）。推理作为独立流通过 `_reasoning_delta` / `_reasoning_end` 标记流转 —— 频道可覆盖 `send_reasoning_delta` / `send_reasoning_end` 以就地渲染更新。即便设为 `true`，未提供这些覆盖的频道也会静默无操作。目前 CLI 和 WebSocket/WebUI 已支持（斜体闪烁标题，流结束后自动折叠）；Telegram / Slack / Discord / Feishu / WeChat / Matrix / Mattermost 在其气泡 UI 适配前保持基础无操作。与 `sendProgress` 独立。 |
| `extractDocumentText` | `true` | 将支持的文档/文本附件抽取到模型提示词中。标准安装已包含 PDF、DOCX、XLSX 和 PPTX 读取器。设置为 `false` 可让文档内容不进入提示词，改为包含附件路径引用。 |
| `sendMaxRetries` | `3` | 每条出站消息的最大发送尝试次数，包括首次发送（配置范围 0-10，实际最少 1 次尝试） |

`channels.transcriptionProvider` 和 `channels.transcriptionLanguage` 是已废弃的兼容字段。它们作为旧配置的只读回退保留，但新配置应使用顶层 `transcription.provider` 和 `transcription.language`。

`sendProgress` 和 `sendToolHints` 也可以按频道单独覆盖。全局值作为未设置自身值的频道的默认值：

```json
{
  "channels": {
    "sendProgress": true,
    "sendToolHints": false,
    "telegram": {
      "enabled": true,
      "sendProgress": false
    },
    "websocket": {
      "enabled": true,
      "sendToolHints": true
    }
  }
}
```

Telegram 的 `richMessages` 默认为 `false`。仅在你希望启用 Bot API 10.1 `sendRichMessage` 渲染时开启；对于会显示不支持消息错误的 Telegram Web 客户端，请保持禁用。

### 重试行为

重试有意保持简单。

当频道 `send()` 抛出异常时，nanobot 会在频道管理器层进行重试。默认情况下 `channels.sendMaxRetries` 为 `3`，此计数包含首次发送。

- **第 1 次尝试**：立即发送
- **第 2 次尝试**：`1s` 后重试
- **第 3 次尝试**：`2s` 后重试
- **更高的重试预算**：退避时间以 `1s`、`2s`、`4s` 延续，之后固定上限为 `4s`
- **瞬时失败**：网络抖动和临时 API 限制通常会在下一次尝试时恢复
- **永久性失败**：无效令牌、被撤销的访问或被封禁的频道将耗尽重试预算并干净地失败

> [!NOTE]
> 这种设计是刻意的：频道实现应在发送失败时抛出异常，由频道管理器承担共享的重试策略。
>
> 某些频道内部仍可能应用少量特定于 API 的重试。例如，Telegram 会在向管理器暴露最终失败之前单独重试超时和洪水控制错误。
>
> 如果某个频道完全不可达，nanobot 无法通过同一频道通知用户。请关注日志中的 `Failed to send to {channel} after N attempts` 以发现持续的送达失败。

## 网络工具

nanobot 集成了访问网络的基础工具。这些工具包括通过 API 搜索、以 Markdown 格式获取任意网页。它们默认启用，可在 `~/.nanobot/config.json` 的 `tools.web` 下配置。

如果你想禁用它们（这会同时从发送给 LLM 的工具列表中移除 `web_search` 和 `web_fetch`），将 `tools.web.enable` 设置为 `false`：

```json
{
  "tools": {
    "web": {
      "enable": false
    }
  }
}
```

nanobot 为内置网络请求和 HTTP/SSE MCP 连接使用共享的 SSRF 保护。默认情况下，它会阻止回环地址、RFC1918/私有网段、CGNAT/Tailscale 网段、链路本地地址以及云元数据端点。如果你需要允许受信任的私有网段，请使用 `tools.ssrfWhitelist` 明确将其排除在 SSRF 阻止之外：

```json
{
  "tools": {
    "ssrfWhitelist": ["100.64.0.0/10"]
  }
}
```

请让白名单条目尽可能狭窄，例如单主机的 CIDR（`192.168.1.50/32`）。该白名单对共享 SSRF 保护是全局生效的；它不局限于某个工具或某个 MCP 服务器。

HTTP/SSE MCP 连接使用与 `web_fetch` 相同的进程级代理环境行为：被代理的目标使用配置的代理，而被 `NO_PROXY` 排除的 URL 仍保持 DNS 锁定的直连。

> [!TIP]
> 使用 `tools.web` 中的 `proxy` 将网络请求路由到代理：
> ```json
> { "tools": { "web": { "proxy": "http://127.0.0.1:7890" } } }
> ```
> `web_fetch` 对直连应用 DNS 锁定。当显式的 `tools.web.proxy` 或进程级代理环境变量适用于目标 URL 时，nanobot 仍会在本地验证请求的 URL，但出站请求的 DNS 解析发生在代理处；请只配置受信任的代理。除非配置了 `tools.web.proxy`，被 `NO_PROXY` 排除的 URL 会保持 DNS 锁定的直连路径。

### `tools.web`

| 选项 | 类型 | 默认值 | 描述 |
|--------|------|---------|-------------|
| `enable` | boolean | `true` | 启用或禁用所有内置网络工具（`web_search` + `web_fetch`） |
| `proxy` | string 或 null | `null` | 网络请求的代理，例如 `http://127.0.0.1:7890`。`web_fetch` 的 DNS 锁定仅适用于直连；被代理的请求依赖所配置的代理作为受信任的网络出口。 |
| `userAgent` | string 或 null | `null` | 所有网络请求的 User-Agent 头。若为 null，将使用一个浏览器 User-Agent |

### 网络搜索

nanobot 支持多个网络搜索提供商。在 `~/.nanobot/config.json` 的 `tools.web.search` 下配置。

默认情况下，网络搜索使用 `duckduckgo`，无需 API 密钥即可开箱即用。

| 提供商 | 配置字段 | 环境变量回退 | 免费 |
|----------|--------------|------------------|------|
| `brave` | `apiKey` | `BRAVE_API_KEY` | 否 |
| `tavily` | `apiKey` | `TAVILY_API_KEY` | 否 |
| `jina` | `apiKey` | `JINA_API_KEY` | 免费额度（1000 万 token） |
| `kagi` | `apiKey` | `KAGI_API_KEY` | 否 |
| `olostep` | `apiKey` | `OLOSTEP_API_KEY` | 否 |
| `bocha` | `apiKey` | `BOCHA_API_KEY` | 免费额度（初创企业 100 万次调用） |
| `volcengine` | `apiKey` | `VOLCENGINE_SEARCH_API_KEY` 或 `WEB_SEARCH_API_KEY` | 每月配额，超出后付费 |
| `keenable` | `apiKey`（可选） | `KEENABLE_API_KEY` | 是（无需密钥；提供密钥可提高上限） |
| `searxng` | `baseUrl` | `SEARXNG_BASE_URL` | 是（自托管） |
| `duckduckgo`（默认） | — | — | 是 |

**Brave：**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

**Tavily：**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "tavily",
        "apiKey": "${TAVILY_API_KEY}"
      }
    }
  }
}
```

**Jina**（免费额度 1000 万 token）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "jina",
        "apiKey": "${JINA_API_KEY}"
      }
    }
  }
}
```

**Kagi：**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "kagi",
        "apiKey": "${KAGI_API_KEY}"
      }
    }
  }
}
```

**Olostep：**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "olostep",
        "apiKey": "${OLOSTEP_API_KEY}"
      }
    }
  }
}
```

你也可以在环境中设置 `OLOSTEP_API_KEY`，而不用将其存储在配置文件中。

**Bocha**（针对 AI 优化的搜索，提供免费额度）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "bocha",
        "apiKey": "${BOCHA_API_KEY}"
      }
    }
  }
}
```

在 [open.bochaai.com](https://open.bochaai.com) 创建你的 API 密钥。
Bocha 返回针对 AI 消费优化的结构化结果，可选带摘要。
你可以在环境中设置 `BOCHA_API_KEY`，而不用将其存储在配置文件中。

**Volcengine Search：**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "volcengine",
        "apiKey": "${VOLCENGINE_SEARCH_API_KEY}"
      }
    }
  }
}
```

你也可以设置 `WEB_SEARCH_API_KEY` 以兼容火山引擎 web-search 技能。在[火山引擎 web 搜索控制台](https://console.volcengine.com/search-infinity/web-search)创建密钥，然后从 [API 密钥](https://console.volcengine.com/search-infinity/api-key)复制。火山引擎 Ark 密钥是独立的，不适用于该搜索提供商。

**Keenable**（免费额度下无需 API 密钥即可工作）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "keenable"
      }
    }
  }
}
```

Keenable 搜索无需账户即可通过其无令牌公共端点开箱即用（免费额度，每小时限 1,000 次请求）。从 [keenable.ai](https://keenable.ai) 设置 `apiKey`（或 `KEENABLE_API_KEY`）可解除每小时限制。

**Serper**（Google 搜索 API）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "serper",
        "apiKey": "${SERPER_API_KEY}"
      }
    }
  }
}
```

在 [serper.dev](https://serper.dev) 创建密钥。你也可以在环境中设置 `SERPER_API_KEY`，而不用将其存储在配置文件中。

**SearXNG**（自托管，无需 API 密钥）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "searxng",
        "baseUrl": "https://searx.example"
      }
    }
  }
}
```

**DuckDuckGo**（无需配置）：
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "duckduckgo"
      }
    }
  }
}
```

#### `tools.web.search`

| 选项 | 类型 | 默认值 | 说明 |
|--------|------|---------|-------------|
| `provider` | string | `"duckduckgo"` | 搜索后端：`brave`、`tavily`、`jina`、`kagi`、`olostep`、`bocha`、`volcengine`、`keenable`、`serper`、`searxng`、`duckduckgo` |
| `apiKey` | string | `""` | 基于 API 的搜索提供商的 API 密钥 |
| `baseUrl` | string | `""` | SearXNG 的基础 URL |
| `maxResults` | integer | `5` | 每次搜索返回的结果数（1–10） |

### Web Fetch

> [!TIP]
> 如果你在使用中遇到 JS proof-of-work 或 Cloudflare 验证码问题，请设置一个随机 user agent 并禁用 Jina Reader：
> ```json
> { "tools": { "web": { "userAgent": "Not-A-Browser", "fetch": { "useJinaReader": false } } } }
> ```

nanobot 默认使用第三方 API [Jina Reader](https://jina.ai/reader/) 将任意页面转换为 Markdown 格式，方便 LLM 消化理解；如果前者失败，会基于 [readability-lxml](https://github.com/buriy/python-readability) 进行本地回退。

如果你希望始终使用本地转换，可以强制启用：

```json
{
  "tools": {
    "web": {
      "fetch": {
        "useJinaReader": false
      }
    }
  }
}
```

#### `tools.web.fetch`

| 选项 | 类型 | 默认值 | 说明 |
|--------|------|---------|-------------|
| `useJinaReader` | boolean | `true` | 若为 true，将优先使用 Jina Reader 而非本地转换 |

## 图像生成

图像生成在 `tools.imageGeneration` 下配置，并使用所选提供商的 `providers.<name>` 块中的凭据。

有关 WebUI 用法、提供商示例、产物存储与故障排查，请参见 [图像生成](./image-generation.md)。

## MCP（Model Context Protocol）

> [!TIP]
> 该配置格式与 Claude Desktop / Cursor 兼容。你可以直接从任何 MCP 服务器的 README 中复制 MCP 服务器配置。

nanobot 支持 [MCP](https://modelcontextprotocol.io/) — 连接外部工具服务器并将其作为原生智能体工具使用。

在你的 `config.json` 中添加 MCP 服务器：

```json
{
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
      },
      "my-remote-mcp": {
        "url": "https://example.com/mcp/",
        "headers": {
          "Authorization": "Bearer xxxxx"
        }
      }
    }
  }
}
```

支持两种传输模式：

| 模式 | 配置 | 示例 |
|------|--------|---------|
| **Stdio** | `command` + `args` | 通过 `npx` / `uvx` 运行的本地进程 |
| **HTTP** | `url` + `headers`（可选） | 远程端点（`https://mcp.example.com/sse`） |

> [!IMPORTANT]
> HTTP/SSE MCP URL 会在探测或连接前被校验，每个发出的 MCP HTTP 请求都会在跟随重定向前再次校验。默认情况下，`localhost`、`127.0.0.1`、RFC1918/私网 IP、CGNAT/Tailscale 网段、链路本地地址以及云元数据端点都会被阻止。这可能会导致之前可用的本地或私网 HTTP MCP 配置失效，直到通过 `tools.ssrfWhitelist` 显式允许该端点为止，最好使用单主机 CIDR，例如 `127.0.0.1/32`、`::1/128` 或 `192.168.1.50/32`。Stdio MCP 服务器不受影响。

使用 `toolTimeout` 可为较慢的服务器覆盖每次调用默认的 30 秒超时：

```json
{
  "tools": {
    "mcpServers": {
      "my-slow-server": {
        "url": "https://example.com/mcp/",
        "toolTimeout": 120
      }
    }
  }
}
```

使用 `enabledTools` 可只注册 MCP 服务器工具的一个子集：

```json
{
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
        "enabledTools": ["read_file", "mcp_filesystem_write_file"]
      }
    }
  }
}
```

`enabledTools` 支持原始 MCP 工具名称（例如 `read_file`）或包装后的 nanobot 工具名称（例如 `mcp_filesystem_write_file`）。

- 省略 `enabledTools`，或将其设置为 `["*"]`，将注册所有能力（工具、资源和提示词）。
- 将 `enabledTools` 设置为 `[]` 则不注册该服务器的任何工具。资源和提示词也会被跳过，因为它们没有按名称过滤的机制。
- 将 `enabledTools` 设置为非空的名称列表将只注册这些工具 — 资源和提示词不会被注册。

MCP 工具会在启动时被自动发现和注册。LLM 可将它们与内置工具一同使用 — 无需额外配置。




## 安全

> [!TIP]
> 对于生产部署，请在配置中同时设置 `"restrictToWorkspace": true` 和 `"tools.exec.sandbox": "bwrap"`。`restrictToWorkspace` 启用 nanobot 应用层的工作区防护；`tools.exec.sandbox` 为 shell 命令提供进程级隔离。

对于 API 密钥、令牌和其他敏感信息，请参见 [用于机密的环境变量](#environment-variables-for-secrets) — 避免将它们直接存储在 `config.json` 中。

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `tools.restrictToWorkspace` | `false` | 当为 `true` 时，为工作区感知工具启用 nanobot 的应用层工作区防护。文件工具会将路径解析到当前工作区下；选定的内部根目录可以作为只读或显式启用写入的根目录添加，媒体上传默认为只读。Shell 执行会拒绝工作区外的 `working_dir` 值，并对命令路径进行尽力而为的检查，但这不是操作系统级沙箱。 |
| `tools.exec.sandbox` | `""` | Shell 命令的沙箱后端。设置为 `"bwrap"` 会将 exec 调用包裹在 [bubblewrap](https://github.com/containers/bubblewrap) 沙箱中 — 进程只能看到工作区（读写）和媒体目录（只读）；配置文件和 API 密钥被隐藏。会自动为文件工具启用工作区限制。**仅限 Linux** — 需要安装 `bwrap`（`apt install bubblewrap`；Docker 镜像中已预装）。在 macOS 或 Windows 上不可用（bwrap 依赖 Linux 内核命名空间）。 |
| `tools.exec.enable` | `true` | 当为 `false` 时，根本不注册 shell `exec` 工具。用于完全禁用 shell 命令执行。 |
| `tools.exec.timeout` | `60` | Shell 命令的默认硬超时时间（秒）。配置值可能超过单次调用工具上限；设置为 `0` 可为可信的长运行命令禁用硬超时。 |
| `tools.exec.pathPrepend` | `""` | 在运行 shell 命令时，向 `PATH` 前置附加的额外目录。当已配置的工具应在可执行文件查找中优先胜出时使用此项，例如 Python 虚拟环境的 `bin` 或 `Scripts` 目录。 |
| `tools.exec.pathAppend` | `""` | 在运行 shell 命令时，向 `PATH` 追加的额外目录（例如为 `ufw` 追加 `/usr/sbin`）。 |
| `tools.webuiAllowRemotePackageInstall` | `false` | 当为 `false` 时，WebUI 仅在与 nanobot 位于同一台机器上打开的浏览器中，可以安装缺失的可选包。仅在允许受信任的远程管理员向此 nanobot 环境安装 Python 包时，设置为 `true`。 |
| `tools.ssrfWhitelist` | `[]` | 免于共享 SSRF 防护的 CIDR 范围，该防护用于 web 抓取和 HTTP/SSE MCP 连接。优先使用精确的主机 CIDR，如 `192.168.1.50/32`；宽泛的范围会增加 SSRF 暴露风险。 |
| `channels.*.allowFrom` | 省略 | 每个频道的访问控制。省略以使用仅配对模式；设置 `["*"]` 允许所有人；或列出具体的用户 ID。详见 [配对](#pairing)。 |

**Docker 安全**：官方 Docker 镜像以非 root 用户（`nanobot`，UID 1000）运行，并预装 bubblewrap。使用 `docker-compose.yml` 时，容器会丢弃除 `SYS_ADMIN`（bwrap 的命名空间隔离所需）以外的所有 Linux capabilities。


## 配对（Pairing）

配对允许用户通过简单的代码交换获得机器人的访问权限 — 无需编辑配置。这适用于新用户以及从新频道连接的现有用户（例如已在 Telegram 上被批准的人正在设置 Discord）。

### 工作原理

1. 用户在支持配对的频道上向机器人发送 DM，而其尚未被批准。这包括 Telegram、Discord、微信，以及 Slack 或 Mattermost 等在 DM 策略设置为 `allowlist` 时的频道。
2. 机器人回复一个配对码（如 `ABCD-EFGH`）并告诉用户将其转发给你。
3. 你批准该代码：

```text
/pairing approve ABCD-EFGH
```

4. 该用户现在可以正常与机器人聊天。

配对仅在 **DM** 中生效 — 群聊中未被批准的用户会被静默忽略。

### 仅配对模式

默认情况下，如果你不设置 `allowFrom`，支持配对的频道在未被批准的用户向机器人发送 DM 时会颁发一个配对码。这意味着你可以完全跳过 `allowFrom`，通过配对来管理访问权限：

```json
{
  "channels": {
    "telegram": {
      "enabled": true
    }
  }
}
```

Slack 和 Mattermost 的 DM 默认开放。要在那里使用配对，请将该频道的
`dm.policy` 设置为 `"allowlist"`，并在你批准用户之前保持 `dm.allowFrom` 为空：

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "dm": { "policy": "allowlist" }
    }
  }
}
```

如果你希望允许所有人无需审批：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "allowFrom": ["*"]
    }
  }
}
```

### 管理访问

| 命令 | 作用 |
|---------|-------------|
| `/pairing` | 显示所有待处理的配对请求 |
| `/pairing approve <code>` | 批准请求 — 发送者即可开始聊天 |
| `/pairing deny <code>` | 拒绝一个待处理的请求 |
| `/pairing revoke <user_id>` | 从当前频道移除一个先前已批准的用户 |
| `/pairing revoke <channel> <user_id>` | 从指定频道移除一个用户 |

你可以在 `/pairing list` 的输出中找到用户 ID。

从终端：

```bash
nanobot agent -m "/pairing list"
nanobot agent -m "/pairing approve ABCD-EFGH"
```


## 网关心跳（Gateway Heartbeat）

网关可运行一个受保护的心跳 cron 任务，定期检查当前工作区中的 `HEARTBEAT.md`。运行 `nanobot gateway` 时默认启用。

```json
{
  "gateway": {
    "heartbeat": {
      "enabled": true,
      "intervalS": 1800,
      "keepRecentMessages": 8
    }
  }
}
```

如果 `HEARTBEAT.md` 在 `## Active Tasks` 下有任务，智能体会执行它们，并只将有用/可操作的结果发送到最近活跃的聊天目标。如果文件没有活跃任务，或结果是例行且没有值得报告的内容，心跳会被静默跳过。

这与用户创建的 cron 任务是有意区分的。用 `cron` 工具创建的 cron 任务会作为其源聊天/会话中的一次预定回合运行，通常会将结果传回该频道。对于不应在每次运行时都通知用户的循环后台检查，请使用 `HEARTBEAT.md`。

心跳任务由与用户创建的提醒相同的 cron 服务支持。它存储在当前工作区下（`<workspace>/cron/jobs.json`），并在 `cron(action="list")` 中显示为 `heartbeat`，但它是由系统管理的，不能通过 `cron` 工具删除。如果不希望周期性心跳检查，请在配置中禁用并重启网关。

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `gateway.heartbeat.enabled` | `true` | 在网关启动时注册内置的心跳 cron 任务。 |
| `gateway.heartbeat.intervalS` | `1800` | 心跳检查之间的秒数。 |
| `gateway.heartbeat.keepRecentMessages` | `8` | 每次运行后保留的近期心跳会话消息数量。 |
| `gateway.restartMode` | `auto` | `/restart` 的重启策略：`auto` 在 Windows 前台运行时使用 `spawn`，其他情况使用 `exec`。对于 WinSW 或 nssm 等 Windows 服务包装器，使用 `exit` 以让服务管理器负责重启。 |


## 子智能体并发

默认情况下，nanobot 一次只允许一个已派生的子智能体。达到上限时，`spawn` 工具会返回错误，以便智能体决定等待还是重新安排工作。这可以避免本地 LLM 服务器同时加载多个 KV 缓存。如果你的提供商可以处理更多并行工作，可提高上限：

```json
{
  "agents": {
    "defaults": {
      "maxConcurrentSubagents": 2
    }
  }
}
```

当子智能体的某个工具返回执行错误时，它们也会立即停止。该默认行为可让失败对父智能体保持可见。如果你的子智能体工作流使用的工具可能瞬时失败并应由模型重试或绕过，请禁用硬停行为：

```json
{
  "agents": {
    "defaults": {
      "failOnToolError": false
    }
  }
}
```

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `agents.defaults.maxConcurrentSubagents` | `1` | 同一时间可以运行的已派生子智能体的最大数量。超过此限制的派生尝试将返回错误。 |
| `agents.defaults.failOnToolError` | `true` | 当工具执行失败时停止已派生的子智能体。设置为 `false` 会将工具错误返回给子智能体模型，使其能够在同一次运行中恢复。 |


## 自动压缩（Auto Compact）

当用户空闲时间超过配置阈值时，nanobot 会**主动**将会话上下文的较早部分压缩为摘要，同时保留最近的合法后缀作为存活消息。这可以在用户回来时降低 token 成本和首个 token 延迟 — 模型收到的不是使用已过期 KV 缓存重新处理的漫长陈旧上下文，而是紧凑的摘要、最近的存活上下文，以及新的输入。

```json
{
  "agents": {
    "defaults": {
      "idleCompactAfterMinutes": 15
    }
  }
}
```

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `agents.defaults.idleCompactAfterMinutes` | `15` | 自动压缩启动前的空闲分钟数。设置为 `0` 可禁用。默认值接近典型 LLM KV 缓存的过期窗口，因此陈旧会话会在用户回来前被压缩。 |

为了向后兼容，`sessionTtlMinutes` 仍作为遗留别名被接受，但今后 `idleCompactAfterMinutes` 是首选的配置键。

工作原理：
1. **空闲检测**：在每个空闲 tick（约 1 秒）时，检查所有会话是否过期。
2. **后台压缩**：空闲会话通过 LLM 对较早的存活前缀进行摘要，并保留最近的合法后缀（当前为 8 条消息）。
3. **摘要注入**：当用户返回时，摘要作为运行时上下文注入（一次性，不持久化），与保留的最近后缀一起提供。
4. **重启后可恢复**：摘要也会被镜像到会话元数据中，即使进程重启后仍可恢复。

> [!NOTE]
> 心智模型：“对较旧上下文进行摘要，保留最新的存活轮次，**并用紧凑形式覆盖会话文件。**”这并非完整的 `session.clear()`，但它是一次写入 — 而非软性游标移动。
>
> 具体来说，自动压缩会就地重写 `sessions/<key>.jsonl`：较早的消息（包括其结构化的 `tool_calls` / `tool_call_id` / `reasoning_content`）会被仅保留的最近后缀（当前为 8 条消息）取代，而归档的前缀仅作为附加到 `memory/history.jsonl` 的纯文本摘要（或在 LLM 摘要失败时以 `[RAW] ...` 扁平化转储的形式）被保留。那些回合的原始结构化 JSON 将无法再从会话文件中恢复。
>
> 这与提示词超出上下文预算时触发的**由 token 驱动的软合并**不同：那条路径仅推进一个内部的 `last_consolidated` 游标，会话文件保持不变，因此原始工具调用轨迹仍留在磁盘上，仍可回放或审计。如果你依赖该轨迹进行调试或审计，请将 `idleCompactAfterMinutes` 设置为 `0`，只让由 token 驱动的路径运行。

## 时区

时间即上下文。上下文应当精确。

默认情况下，nanobot 使用 `UTC` 作为运行时时间上下文。如果你希望智能体以本地时间思考，请将 `agents.defaults.timezone` 设置为一个有效的 [IANA 时区名称](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)：

```json
{
  "agents": {
    "defaults": {
      "timezone": "Asia/Shanghai"
    }
  }
}
```

这会影响展示给模型的运行时时间字符串，例如运行时上下文。它还会成为 cron 计划的默认时区（当 cron 表达式省略 `tz` 时），以及一次性 `at` 时间的默认时区（当 ISO datetime 没有显式偏移时）。

常用示例：`UTC`、`America/New_York`、`America/Los_Angeles`、`Europe/London`、`Europe/Berlin`、`Asia/Tokyo`、`Asia/Shanghai`、`Asia/Singapore`、`Australia/Sydney`。

> 需要其他时区？请浏览完整的 [IANA 时区数据库](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

## 统一会话（Unified Session）

默认情况下，每个 频道 × chat ID 组合都有各自独立的会话。如果你跨多个频道使用 nanobot（如 Telegram + Discord + CLI），并希望它们共享同一段对话，请启用 `unifiedSession`：

```json
{
  "agents": {
    "defaults": {
      "unifiedSession": true
    }
  }
}
```

启用后，所有进入的消息 — 无论从哪个频道到达 — 都会被路由到一个共享会话。从 Telegram 切换到 Discord（或任何其他频道）时，同一段对话会无缝延续。

| 行为 | `false`（默认） | `true` |
|----------|-------------------|--------|
| 会话键 | `channel:chat_id` | `unified:default` |
| 跨频道连贯性 | 否 | 是 |
| `/new` 清除 | 当前频道会话 | 共享会话 |
| `/stop` 查找任务 | 按频道会话 | 按共享会话 |
| 已存在的 `session_key_override`（如 Telegram 线程） | 遵循 | 仍然遵循 — 不会被覆盖 |

> 该设计面向单用户、多设备场景。它**默认关闭** — 现有用户不会看到任何行为变化。

## 已禁用的技能

nanobot 内置了一些技能，你的工作区也可以在 `skills/` 下定义自定义技能。如果你想对智能体隐藏特定技能，将 `agents.defaults.disabledSkills` 设置为技能目录名的列表：

```json
{
  "agents": {
    "defaults": {
      "disabledSkills": ["github", "weather"]
    }
  }
}
```

被禁用的技能会从主智能体的技能摘要、始终开启的技能注入以及子智能体的技能摘要中排除。这在你的部署不需要某些内置技能，或不应向最终用户暴露它们时很有用。

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `agents.defaults.disabledSkills` | `[]` | 要从加载中排除的技能目录名列表。同时适用于内置技能和工作区技能。 |

## 工具提示最大长度

工具提示是智能体调用工具时显示的简短进度消息（例如 `$ cd …/project && npm test`）。默认情况下，它们会被截断为 40 个字符，这可能使长命令难以阅读。

设置 `agents.defaults.toolHintMaxLength` 以控制截断阈值：

```json
{
  "agents": {
    "defaults": {
      "toolHintMaxLength": 120
    }
  }
}
```

| 选项 | 默认值 | 说明 |
|--------|---------|-------------|
| `agents.defaults.toolHintMaxLength` | `40` | 工具提示显示的最大字符数。范围：20–500。较大的值可显示更多命令或路径；较小的值使提示更紧凑。 |
