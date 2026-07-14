# 提供商实用手册

本页面适用于你已经知道要连接什么、只需要一份可粘贴的配置的场景。每个方案会展示需要设置什么、需要运行什么，以及失败通常意味着什么。

如果这是你首次安装且不熟悉终端命令，请先阅读 [`start-without-technical-background.md`](./start-without-technical-background.md)。如果你想要按字段的详细说明，请阅读 [`providers.md`](./providers.md)，然后阅读 [`configuration.md#providers`](./configuration.md#providers)。

下面大多数示例都是可合并到 `~/.nanobot/config.json` 中的片段。保留你仍然需要的现有部分，并且仅在你自己的机器上将 `${OPENROUTER_API_KEY}` 这类占位密钥替换为环境变量引用或真实值。

方案只是示例，并非排名。选择与你已经打算使用的凭证、端点和模型 ID 相匹配的方案即可。

## 选择一个方案

将方案与你已经拥有的凭证或端点相匹配：

| 你拥有的东西 | 方案 | 必须匹配 |
|---|---|---|
| 一个网关密钥和形如 `provider/model-name` 这样带有模型家族路径的模型 ID | [OpenRouter 网关](#recipe-openrouter-gateway) | API 密钥、提供商配置键、预设的提供商、以及网关模型 ID |
| 一个 OpenCode Zen 或 Go 密钥 | [OpenCode Zen 或 Go](#recipe-opencode-zen-or-go) | `OPENCODE_API_KEY`、Zen/Go 提供商键、以及匹配的 OpenCode 端点上的模型 ID |
| 一个 OpenAI 平台 API 密钥和 OpenAI 模型 ID | [OpenAI 直连](#recipe-openai-direct) | `OPENAI_API_KEY`、`provider: "openai"`、以及该账户可用的 OpenAI 模型 |
| 一个 Anthropic API 密钥和 Anthropic 模型 ID | [Anthropic 直连](#recipe-anthropic-direct) | `ANTHROPIC_API_KEY`、`provider: "anthropic"`、以及非网关模型 ID |
| 一个 Kimi Coding Plan 密钥 | [Kimi Coding Plan](#recipe-kimi-coding-plan) | `KIMI_CODING_API_KEY`、`provider: "kimi_coding"`、以及 `model: "kimi-for-coding"` |
| 一个不是 nanobot 命名提供商的 OpenAI 兼容 `/v1` 端点 | [自定义 OpenAI 兼容提供商](#recipe-custom-openai-compatible-provider) | `apiBase`、可选的 API 密钥、以及该端点服务的模型 ID |
| 已经在本地运行的 Ollama | [Ollama 本地模型](#recipe-ollama-local-model) | Ollama 的 `apiBase`、已拉取的模型名称、以及本地服务器的可用性 |
| vLLM、LM Studio 或其他本地 OpenAI 兼容服务器 | [vLLM 或 LM Studio](#recipe-vllm-or-lm-studio) | 本地 `/v1` 基础 URL、所需的密钥、以及所服务的模型名称 |
| 一个主模型加上一个或多个备份 | [回退预设](#recipe-fallback-presets) | 在 `modelPresets` 中命名的预设，从 `agents.defaults.fallbackModels` 中引用 |
| 一个可工作的智能体和一个 Langfuse 项目 | [Langfuse 追踪](#recipe-langfuse-tracing) | 在启动 nanobot 的同一进程环境中设置的 Langfuse 环境变量 |

## 如何使用一个方案

1. 安装 nanobot 并运行一次 `nanobot onboard`，这样 `~/.nanobot/config.json` 就会存在。如果你更喜欢引导提示而不是手动编辑 JSON，可以使用 `nanobot onboard --wizard`。
2. 尽可能把密钥放入环境变量中。
3. 将方案片段合并到 `~/.nanobot/config.json`。
4. 运行 `nanobot status`。
5. 运行 `nanobot agent -m "Hello!"`。
6. 如果 CLI 可以工作，再去连接 WebUI、网关或聊天应用。

激活的模型通常应该来自 `agents.defaults.modelPreset`，且该名称应指向 `modelPresets` 中的一个条目。直接使用 `agents.defaults.provider` 和 `agents.defaults.model` 对旧配置仍然有效，但预设更容易切换，也更容易作为回退复用。

## 密钥设置

环境变量可以让 API 密钥远离配置文件。

使用你所选方案里给出的变量名。下面的命令仅以 `OPENROUTER_API_KEY` 为例；OpenAI 直连方案使用 `OPENAI_API_KEY`，Anthropic 直连方案使用 `ANTHROPIC_API_KEY`，而自定义端点可以使用你在 `config.json` 中引用的任意变量名。

**macOS / Linux**

```bash
export OPENROUTER_API_KEY="sk-or-v1-..."
nanobot agent -m "Hello!"
```

**Windows PowerShell**

```powershell
$env:OPENROUTER_API_KEY = "sk-or-v1-..."
nanobot agent -m "Hello!"
```

以这种方式设置的环境变量只对当前终端生效。对于 systemd、Docker、LaunchAgent 或远程 shell 这类长时间运行的服务，需要在启动 nanobot 之前，在该服务环境中设置这些变量。

## 方案：OpenRouter 网关

当一个 API 密钥可以路由到许多托管模型家族时，适用此方案。

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "${OPENROUTER_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "Primary",
      "provider": "openrouter",
      "model": "anthropic/claude-sonnet-4.5",
      "maxTokens": 4096,
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

验证：

```bash
nanobot status
nanobot agent -m "Hello!"
```

如果返回 `401` 或 `unauthorized`，检查启动 nanobot 的终端或服务中是否能看到 `OPENROUTER_API_KEY`。如果返回 `model not found`，请选择 OpenRouter 为你的账户列出的模型 ID。

## 方案：OpenCode Zen 或 Go

当你的凭证来自 OpenCode Zen 或 OpenCode Go 时，适用此方案。
两个提供商都使用 `OPENCODE_API_KEY`；根据你想要使用的订阅或余额，选择匹配的提供商配置块。

OpenCode Zen：

```json
{
  "providers": {
    "opencodeZen": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "OpenCode Zen",
      "provider": "opencode_zen",
      "model": "opencode/deepseek-v4-pro",
      "maxTokens": 4096,
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

OpenCode Go：

```json
{
  "providers": {
    "opencodeGo": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "OpenCode Go",
      "provider": "opencode_go",
      "model": "opencode-go/deepseek-v4-flash",
      "maxTokens": 4096,
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

验证：

```bash
nanobot status
nanobot agent -m "Hello!"
```

OpenCode 的文档列出了跨多种端点类型的模型。nanobot 中的 `opencode_zen`
和 `opencode_go` 提供商使用 OpenAI 兼容的
`chat/completions` 路径。如果某个模型返回 `model not found` 或端点
结构错误，请选择 OpenCode 在与之匹配的 Zen 或 Go 端点下
`chat/completions` 中列出的模型。

## 方案：OpenAI 直连

当你有一个 OpenAI API 密钥并且希望直接调用 OpenAI 而不是通过网关时，适用此方案。

```json
{
  "providers": {
    "openai": {
      "apiKey": "${OPENAI_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "OpenAI",
      "provider": "openai",
      "model": "gpt-5",
      "maxTokens": 4096,
      "contextWindowTokens": 128000,
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

验证：

```bash
OPENAI_API_KEY="sk-..." nanobot agent -m "Hello!"
```

如果你的 shell 不能使用内联环境变量，先设置 `OPENAI_API_KEY`，然后再运行 `nanobot agent -m "Hello!"`。如果提供商拒绝 `apiType`，除非你使用的是已记录在案的 OpenAI 特定模式，否则请删除 `apiType`。

## 方案：Anthropic 直连

当你的密钥来自 Anthropic，并且你的模型名称是 Anthropic 模型 ID 而不是 OpenRouter 模型路径时，适用此方案。

```json
{
  "providers": {
    "anthropic": {
      "apiKey": "${ANTHROPIC_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "Anthropic",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5",
      "maxTokens": 4096,
      "contextWindowTokens": 200000,
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

验证：

```bash
ANTHROPIC_API_KEY="sk-ant-..." nanobot agent -m "Hello!"
```

如果你复制的模型名称类似 `anthropic/claude-sonnet-4.5`，那是网关风格的模型路径，应该放在 `provider: "openrouter"` 下，而不是 `provider: "anthropic"` 下。

如果你使用一个 Anthropic 兼容的代理，请保持预设提供商为 `anthropic`，并设置 `providers.anthropic.apiBase`：

```json
{
  "providers": {
    "anthropic": {
      "apiKey": "${ANTHROPIC_API_KEY}",
      "apiBase": "https://anthropic-proxy.example.com"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "Anthropic proxy",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5",
      "maxTokens": 4096,
      "contextWindowTokens": 200000,
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

不要将 Anthropic 兼容端点配置为任意的自定义提供商名称；命名的自定义提供商使用的是 OpenAI 兼容的请求格式。

## 方案：Kimi Coding Plan

当你的密钥来自 Kimi 的 Coding Plan 端点时，适用此方案。Nanobot 为这个 Anthropic Messages API 端点使用专门的 `kimi_coding` 提供商；不要将其配置为通用的 `custom` 提供商。

```json
{
  "providers": {
    "kimiCoding": {
      "apiKey": "${KIMI_CODING_API_KEY}"
    }
  },
  "modelPresets": {
    "kimiCoding": {
      "label": "Kimi Coding",
      "provider": "kimi_coding",
      "model": "kimi-for-coding",
      "maxTokens": 4096,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "kimiCoding"
    }
  }
}
```

验证：

```bash
nanobot status
nanobot agent -m "Hello!"
```

默认的基础 URL 是 `https://api.kimi.com/coding/v1`。此端点要求 Claude 兼容的 `User-Agent`；nanobot 默认发送 `claude-code/0.1.0`。如果你的账户需要不同的值，可以通过 `providers.kimiCoding.extraHeaders.User-Agent` 覆盖它。

## 方案：自定义 OpenAI 兼容提供商

此方案适用于一个不是 nanobot 命名提供商的 OpenAI 兼容服务。

```json
{
  "providers": {
    "custom": {
      "apiKey": "${CUSTOM_API_KEY}",
      "apiBase": "https://api.example.com/v1"
    }
  },
  "modelPresets": {
    "primary": {
      "label": "Custom",
      "provider": "custom",
      "model": "provider-model-name",
      "maxTokens": 4096,
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

在归咎 nanobot 之前先验证端点：

```bash
curl -sS https://api.example.com/v1/models
nanobot agent -m "Hello!"
```

`apiBase` 是 HTTP 基础 URL，不是模型名称。当服务需要时（例如 `/v1`），请包含版本路径。如果服务需要一个非空密钥但不校验它，请使用一个占位符，比如 `"apiKey": "EMPTY"`。

对于多个自定义端点，不要把它们都塞进单个 `custom` 配置块。在 `providers` 下为每个端点命名，并在预设中引用该名称：

```json
{
  "providers": {
    "workProxy": {
      "apiKey": "${WORK_PROXY_API_KEY}",
      "apiBase": "https://proxy.example.com/v1"
    },
    "lab-local": {
      "apiBase": "http://127.0.0.1:8000/v1"
    }
  },
  "modelPresets": {
    "work": {
      "label": "Work proxy",
      "provider": "workProxy",
      "model": "gpt-4o-mini",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    },
    "lab": {
      "label": "Lab local",
      "provider": "lab-local",
      "model": "served-model-name",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "work"
    }
  }
}
```

这些自定义名称的行为与直连的 OpenAI 兼容提供商相同：`apiBase` 是必需的，`apiKey` 在端点允许匿名或占位凭证时是可选的，而 `apiType` 应保持不设置。它们不支持 Anthropic 兼容端点；对于这种情况，请使用带 `apiBase` 的 `anthropic` 提供商。

## 方案：Ollama 本地模型

当 Ollama 已经安装并且模型已在本地拉取时，适用此方案。

```bash
ollama serve
ollama pull llama3.2
```

```json
{
  "providers": {
    "ollama": {
      "apiBase": "http://localhost:11434/v1"
    }
  },
  "modelPresets": {
    "local": {
      "label": "Local",
      "provider": "ollama",
      "model": "llama3.2",
      "maxTokens": 2048,
      "contextWindowTokens": 32768,
      "temperature": 0.2
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "local"
    }
  }
}
```

验证：

```bash
curl -sS http://localhost:11434/v1/models
nanobot agent -m "Hello!"
```

如果看到 `connection refused`，说明 Ollama 没有运行或 `apiBase` 指向了错误的端口。如果响应非常慢，请尝试更小的本地模型或降低 `contextWindowTokens`。

## 方案：vLLM 或 LM Studio

当本地服务器暴露一个 OpenAI 兼容的 `/v1` API 时，适用此方案。

```json
{
  "providers": {
    "vllm": {
      "apiBase": "http://127.0.0.1:8000/v1",
      "apiKey": "EMPTY"
    }
  },
  "modelPresets": {
    "local": {
      "label": "Local",
      "provider": "vllm",
      "model": "served-model-name",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.2
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "local"
    }
  }
}
```

对于 LM Studio，使用其本地基础 URL 和提供商名称：

```json
{
  "providers": {
    "lmStudio": {
      "apiBase": "http://localhost:1234/v1"
    }
  },
  "modelPresets": {
    "local": {
      "label": "LM Studio",
      "provider": "lm_studio",
      "model": "local-model",
      "maxTokens": 2048,
      "contextWindowTokens": 32768
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "local"
    }
  }
}
```

配置键可以是 `lmStudio` 或 `lm_studio`，但预设的提供商应使用注册表名称 `lm_studio`。

## 方案：回退预设

当某个提供商偶尔会被限流、某个模型比较昂贵，或者你想要一个本地备份时，适用此方案。

```json
{
  "modelPresets": {
    "fast": {
      "label": "Fast",
      "provider": "openrouter",
      "model": "anthropic/claude-sonnet-4.5",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    },
    "deep": {
      "label": "Deep",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5",
      "maxTokens": 4096,
      "contextWindowTokens": 200000,
      "temperature": 0.1
    },
    "local": {
      "label": "Local",
      "provider": "ollama",
      "model": "llama3.2",
      "maxTokens": 2048,
      "contextWindowTokens": 32768,
      "temperature": 0.2
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast",
      "fallbackModels": ["deep", "local"]
    }
  }
}
```

`fallbackModels` 属于 `agents.defaults` 之下。字符串条目是预设名称，而不是原始模型名称。nanobot 会先尝试当前激活的预设，然后按顺序尝试回退预设。

保持回退候选者的现实性。如果本地回退的上下文窗口较小，nanobot 必须构建能够适配活动链中最小窗口的上下文。

## 方案：Langfuse 追踪

当智能体已经能工作，并且你希望对 OpenAI 兼容提供商的调用进行可观测性追踪时，适用此方案。

在运行 nanobot 的同一 Python 环境中安装可选包：

```bash
python -m pip install langfuse
```

在启动 nanobot 之前设置环境变量：

```bash
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_BASE_URL="https://cloud.langfuse.com"
nanobot agent -m "Hello!"
```

PowerShell：

```powershell
$env:LANGFUSE_SECRET_KEY = "sk-lf-..."
$env:LANGFUSE_PUBLIC_KEY = "pk-lf-..."
$env:LANGFUSE_BASE_URL = "https://cloud.langfuse.com"
nanobot agent -m "Hello!"
```

Langfuse 在 `config.json` 中不是一个模型提供商。它通过环境变量进行配置，并追踪受支持的 OpenAI 兼容提供商调用。不使用该客户端路径的原生提供商可能不会产生 Langfuse OpenAI 包装器的追踪。

## 方案：在运行时切换模型

当你有多个预设并且正在通过一个受支持的频道进行对话时使用此方案。

```json
{
  "modelPresets": {
    "fast": {
      "label": "Fast",
      "provider": "openrouter",
      "model": "anthropic/claude-sonnet-4.5",
      "maxTokens": 4096,
      "contextWindowTokens": 65536
    },
    "local": {
      "label": "Local",
      "provider": "ollama",
      "model": "llama3.2",
      "maxTokens": 2048,
      "contextWindowTokens": 32768
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast"
    }
  }
}
```

在聊天中：

```text
/model
/model local
/model fast
```

`/model` 切换仅在运行时生效。它不会重写 `config.json`，并且正在进行中的一个回合会继续使用它开始时的模型。

## 快速故障图

| 现象 | 通常意味着 | 首先检查 |
|---|---|---|
| `401`、`unauthorized` 或 `invalid API key` | 密钥缺失、错误、过期，或放在了错误的提供商下 | 在启动 nanobot 的同一终端或服务中打印或重新设置环境变量 |
| `model not found` | 模型 ID 不属于所选提供商或网关 | 比较 `modelPresets.<name>.provider` 和 `modelPresets.<name>.model` |
| `connection refused` | 本地服务器没有运行，或 `apiBase` 的端口/路径错误 | 运行 `curl <apiBase>/models` |
| `provider not found` | 提供商名称拼错，或使用了配置键而不是注册表名称 | 使用像 `openrouter`、`openai`、`anthropic`、`ollama`、`vllm`、`lm_studio` 这样的名称 |
| Langfuse 没有显示追踪 | 环境变量缺失、`langfuse` 未安装在当前活动的 Python 环境中，或提供商路径是原生的 | 运行 `python -m pip show langfuse`，并从相同的环境重启 nanobot |

## 后续参考

| 需要 | 阅读 |
|---|---|
| 字段含义与提供商解析 | [`providers.md`](./providers.md) |
| 完整 schema 与提供商表 | [`configuration.md#providers`](./configuration.md#providers) |
| Langfuse 细节 | [`configuration.md#langfuse-observability`](./configuration.md#langfuse-observability) |
| 首次运行的诊断 | [`troubleshooting.md`](./troubleshooting.md) |
