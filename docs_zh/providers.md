# 提供商与模型

当首次回复因提供商/模型不匹配而失败时，或者你希望将具体的配置示例调整为其他提供商时，请查阅本页。如果你已经知道要使用哪个提供商，只需要一个可粘贴的配置示例，请参阅 [`provider-cookbook.md`](./provider-cookbook.md)。

对于每一种配置，都需要回答三个问题：

1. 凭证或端点属于哪个提供商？
2. 该提供商期望的模型名称是什么？
3. 该提供商需要 `apiKey`、`apiBase`、OAuth 登录、云凭证，还是仅需要一个本地服务器 URL？

推荐为模型/提供商配对创建一个具名的 `modelPresets` 条目，然后通过 `agents.defaults.modelPreset` 选中它。直接使用 `agents.defaults.provider` 和 `agents.defaults.model` 对现有配置仍然有效，但预设可以让运行时的 `/model` 切换以及回退链更加清晰。在配置阶段，请在预设内固定 `provider`；之后你可以再切换回 `"auto"`。

## 无需猜测地选择提供商

文档中出现具体的提供商名称是为了让 JSON 可以直接复制粘贴，而不是因为 nanobot 对提供商做了排名。请从你实际掌控的服务或端点出发：

| 如果你有…… | 请配置…… |
|---|---|
| 来自托管提供商或网关的 API key | 该提供商的 `providers.<name>.apiKey`，然后一个使用该提供商名称与该服务模型 ID 的预设。 |
| 一个 OpenCode Zen 或 Go 的 key | `providers.opencodeZen.apiKey` 或 `providers.opencodeGo.apiKey`，然后一个 `provider: "opencode_zen"` 或 `provider: "opencode_go"` 的预设。 |
| 公司代理或区域端点 | 相应的提供商配置块，如果代理提供了 URL，则再加上 `apiBase`。 |
| 一个本地 OpenAI 兼容服务器 | 一个本地提供商配置块，例如 `ollama`、`vllm`、`lmStudio` 或 `custom`，通常搭配 `apiBase`。 |
| 一个基于 OAuth 的账户 | 运行相应的 `nanobot provider login ...` 命令，然后在预设中显式选择该提供商。 |
| 还没有任何提供商 | 根据账户可用性、价格、区域可用性、隐私要求以及你所需的模型 ID，在 nanobot 之外挑选一个。然后带着它的 key 和模型 ID 回来。 |

## 最小结构

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "openrouter",
      "model": "anthropic/claude-opus-4.5",
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

提供商配置为 nanobot 提供凭证和端点详情。模型预设命名了提供商/模型配对。智能体默认设置选择在常规对话轮次中使用哪个具名预设。请同时替换示例中的提供商和模型；将一个提供商的 API key 与另一个提供商的模型 ID 混用，是首次运行时最常见的失败原因。

## Provider、Model、API Key 与 Base URL

这些字段回答的是不同的问题：

| 字段 | 所在位置 | 含义 |
|---|---|---|
| `provider` | `modelPresets.<name>.provider` | 应由哪个 nanobot 提供商适配器发送请求。 |
| `model` | `modelPresets.<name>.model` | 该提供商或网关期望的模型 ID。 |
| `apiKey` | `providers.<provider>.apiKey` | 该提供商的凭证。密钥请使用 `${ENV_VAR}`。 |
| `apiBase` | `providers.<provider>.apiBase` | 提供商端点的 HTTP 基础 URL。 |
| `proxy` | `providers.<provider>.proxy` | 仅针对该提供商可选的 HTTP 代理。支持 OpenAI 兼容提供商以及 OpenAI Codex。 |

对于像 OpenRouter、Anthropic 直连、OpenAI 直连、Groq 或 Bedrock 这类内置的托管提供商，通常可以省略 `apiBase`，因为 nanobot 已经知道它们的默认端点。对于 `custom`、本地 OpenAI 兼容服务器、提供商代理、区域端点或订阅端点，请设置 `apiBase`。当端点要求带上 API 版本路径时，需要包含在内，例如 `https://api.example.com/v1` 或 `http://localhost:11434/v1`。

当某个提供商必须通过代理发送 HTTP 流量、又不希望修改整个进程范围的 `HTTP_PROXY` / `HTTPS_PROXY` 时，请使用 `proxy`。它适用于使用 nanobot OpenAI 兼容客户端的提供商，包括 `openai`、`custom`、具名的自定义提供商、OpenRouter 风格的网关、本地 OpenAI 兼容服务器以及类似的注册表条目。同时也支持 `openai_codex`，包括 Codex OAuth 令牌交换/刷新以及 Codex Responses API 请求。像 `anthropic`、`bedrock`、`azure_openai` 和 `github_copilot` 这样的原生提供商后端会拒绝 `proxy`；请改用它们各自端点特有的配置。

## 常见提供商模式

### OpenRouter 网关

针对通过 OpenRouter 提供的模型 ID 的网关式配置。

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "${OPENROUTER_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "openrouter",
      "model": "anthropic/claude-opus-4.5",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

模型 ID 请严格按照 OpenRouter 列出的名称使用。

### OpenCode Zen 与 Go

OpenCode Zen 和 OpenCode Go 是由 OpenCode 管理的、面向编码智能体模型的网关。
它们共用 `OPENCODE_API_KEY`，但在 nanobot 中使用不同的提供商配置键和默认基础 URL。

```json
{
  "providers": {
    "opencodeZen": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "opencode_zen",
      "model": "opencode/deepseek-v4-pro",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

对于 OpenCode Go，请切换提供商配置块和预设：

```json
{
  "providers": {
    "opencodeGo": {
      "apiKey": "${OPENCODE_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "opencode_go",
      "model": "opencode-go/deepseek-v4-flash",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  }
}
```

OpenCode 将模型 ID 记为：Zen 使用 `opencode/<model-id>`，Go 使用
`opencode-go/<model-id>`。nanobot 接受这些前缀，并在向 OpenCode 发送请求之前将其剥离。请使用 OpenCode 在
`chat/completions` 端点下列出的模型 ID；仅在 `responses`、`messages` 或提供商专属端点下列出的模型，不会被这个
OpenAI 兼容提供商路径处理。

### Anthropic 直连

```json
{
  "providers": {
    "anthropic": {
      "apiKey": "${ANTHROPIC_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "anthropic",
      "model": "claude-opus-4-5",
      "maxTokens": 8192,
      "contextWindowTokens": 200000
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

Anthropic 直连使用原生 Anthropic 提供商。除非提供商是 OpenRouter，否则不要使用 OpenRouter 的模型 ID。

如果你使用的是 Anthropic 兼容代理，请保持提供商为 `anthropic` 并覆盖 `apiBase`：

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
      "provider": "anthropic",
      "model": "claude-sonnet-4-5"
    }
  }
}
```

任意的自定义提供商名称仅支持 OpenAI 兼容格式，不会使用 Anthropic Messages API 的请求格式。

### OpenAI 直连

```json
{
  "providers": {
    "openai": {
      "apiKey": "${OPENAI_API_KEY}"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "openai",
      "model": "gpt-5",
      "maxTokens": 8192,
      "contextWindowTokens": 128000
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

当你需要强制指定某个 OpenAI API 接口形态时，可以设置 `providers.openai.apiType`。其他提供商会拒绝 `apiType`；在 `providers.openai` 之外请不要设置它。请将 model 替换为你的 OpenAI 账户可用的模型 ID。

### 自定义 OpenAI 兼容端点

`custom` 提供商适用于一个未被具名提供商代表的 OpenAI 兼容端点。

```json
{
  "providers": {
    "custom": {
      "apiKey": "${CUSTOM_API_KEY}",
      "apiBase": "https://example.com/v1"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "custom",
      "model": "provider-model-name",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

`custom` 不会推断出默认的基础 URL。请设置 `apiBase`。

如果你有多个自定义 OpenAI 兼容端点，请在 `providers` 下为每个端点使用各自的提供商键，并在模型预设中使用同样的键。这个键可以是在你环境中有意义的名称，例如 `companyProxy`、`tenant-a` 或 `dev-local`。

```json
{
  "providers": {
    "companyProxy": {
      "apiKey": "${COMPANY_PROXY_API_KEY}",
      "apiBase": "https://llm-proxy.example.com/v1"
    },
    "tenant-a": {
      "apiBase": "https://tenant-a.example.com/v1"
    }
  },
  "modelPresets": {
    "company": {
      "provider": "companyProxy",
      "model": "gpt-4o-mini",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    },
    "tenantA": {
      "provider": "tenant-a",
      "model": "served-model-name",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "company"
    }
  }
}
```

自定义提供商键会被视为直接的 OpenAI 兼容提供商。由于 nanobot 无法知道端点 URL，因此 `apiBase` 是必填项。对于不需要 API key 的本地服务器或私有代理，`apiKey` 是可选的。请选择一个不会与内置提供商名称或别名冲突的名字，例如避免 `openai`、`openai-codex`、`github-copilot`、`lm-studio`。不要在自定义提供商键上设置 `apiType`；`apiType` 仅用于 `providers.openai`。

如果你的自定义端点文档中说明了一个非标准的思考开关，请将 `providers.<name>.thinkingStyle` 设置为 `thinking_type`、`enable_thinking` 或 `reasoning_split`；nanobot 会随后将 `reasoningEffort` 映射到该提供商特有的请求体上。对于常规的 OpenAI 兼容端点，请保持其未设置。

这个具名自定义提供商路径不适用于 Anthropic 兼容端点。对于 Anthropic 兼容代理，请使用 `providers.anthropic.apiBase`，并将预设的 provider 设置为 `anthropic`。

### Ollama

请先单独启动 Ollama，然后将 nanobot 指向其 OpenAI 兼容端点。

```json
{
  "providers": {
    "ollama": {
      "apiBase": "http://localhost:11434/v1"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "ollama",
      "model": "llama3.2",
      "maxTokens": 4096,
      "contextWindowTokens": 32768
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

大多数 Ollama 部署不需要 API key。

### vLLM 或其他本地 OpenAI 兼容服务器

```json
{
  "providers": {
    "vllm": {
      "apiBase": "http://127.0.0.1:8000/v1",
      "apiKey": "EMPTY"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "vllm",
      "model": "served-model-name",
      "maxTokens": 8192,
      "contextWindowTokens": 65536
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

有些本地 OpenAI 兼容服务器即使不校验 API key，也要求提供一个非空的值。

### LM Studio

```json
{
  "providers": {
    "lmStudio": {
      "apiBase": "http://localhost:1234/v1"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "lm_studio",
      "model": "local-model",
      "maxTokens": 4096,
      "contextWindowTokens": 32768
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

配置键可以使用 camelCase 或 snake_case。模型预设中的提供商名称应使用注册表名称，例如 `lm_studio`。

### AWS Bedrock

Bedrock 可以根据你的 AWS 配置使用 AWS 凭证链、profile、区域或 Bedrock bearer token。

```json
{
  "providers": {
    "bedrock": {
      "region": "us-east-1",
      "profile": "default"
    }
  },
  "modelPresets": {
    "primary": {
      "provider": "bedrock",
      "model": "bedrock/anthropic.claude-sonnet-4-5-20250929-v1:0",
      "maxTokens": 8192,
      "contextWindowTokens": 200000
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "primary"
    }
  }
}
```

Bedrock 特有的说明请参阅 [`configuration.md#providers`](./configuration.md#providers)。

### OAuth 提供商

有些提供商不在 `config.json` 中使用 API key。

对于 OpenAI Codex：

```bash
nanobot provider login openai-codex --set-main
```

对于 GitHub Copilot：

```bash
nanobot provider login github-copilot --set-main
```

每条命令都会对所选提供商进行认证，并将其当前的默认模型激活。OAuth 提供商不能作为有效的自动回退。关于代理、无头登录、模型名称以及配置键相关的错误，请参阅 [`troubleshooting.md`](./troubleshooting.md#provider-and-model-problems)。

## 提供商解析

推荐的方式是通过 `agents.defaults.modelPreset` 选择一个具名预设。有效的模型参数来源于：

1. 由 `agents.defaults.modelPreset` 引用的具名 `modelPresets` 条目；
2. 否则来自根据 `agents.defaults.model`、`provider`、`maxTokens`、`contextWindowTokens`、`temperature` 及相关字段构建的隐式 `default` 预设。

提供商选择遵循以下实用规则：

- 活跃预设或隐式默认配置中显式的 `provider` 优先。
- `provider: "auto"` 会依次尝试模型名关键字、已配置的键、本地基础 URL 以及网关提供商。
- 像 OpenRouter 和 AiHubMix 这样的网关提供商可以路由许多模型系列，因此模型名必须在该网关上有效。
- 本地提供商通常应显式指定，因为通用的本地模型名（例如 `llama3.2`）不一定包含提供商关键字。

### 模型名前缀

`family/model-name` 并不总是选择提供商 `family`。基于前缀的提供商推断仅在活跃 provider 为 `"auto"` 时才会执行。

- 显式提供商优先：`provider: "openrouter"` 搭配 `model: "anthropic/claude-sonnet-4.5"` 会调用 OpenRouter，而不是 Anthropic。
- 在 `provider: "auto"` 下，与已配置的内置或具名自定义提供商匹配的前缀可以选中该提供商。具名自定义前缀会在请求前被剥离，因此 `companyProxy/gpt-4o-mini` 上游发送时会变成 `gpt-4o-mini`。
- 使用显式的具名自定义提供商时，模型按原样发送；`provider: "companyProxy"` 搭配 `model: "openai/gpt-4o-mini"` 会向 `companyProxy` 发送 `openai/gpt-4o-mini`。

当使用像 `anthropic/claude-sonnet-4.5` 这种网关目录 ID 时，请在预设中固定 `provider`。

## 模型预设

模型预设是推荐的模型配置入口。当你希望使用具名的模型选择、运行时的 `/model` 切换或可复用的回退目标时，请使用它。

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
      "model": "claude-opus-4-5",
      "maxTokens": 8192,
      "contextWindowTokens": 200000,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast"
    }
  }
}
```

预设名 `default` 被保留给隐式的 `agents.defaults` 设置。请不要定义 `modelPresets.default`；在旧配置中，可以使用 `/model default` 回到直接的 `agents.defaults.*` 字段。

## 回退模型

回退对于临时性的提供商故障、限流或模型可用性问题很有用。回退模型应与任务规模和工具使用兼容。优先使用回退预设，这样每个候选项都拥有名称，以及完整的提供商、模型、生成和上下文窗口配置。

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
      "model": "claude-opus-4-5",
      "maxTokens": 8192,
      "contextWindowTokens": 200000,
      "temperature": 0.1
    },
    "localSmall": {
      "label": "Local Small",
      "provider": "ollama",
      "model": "llama3.2",
      "maxTokens": 4096,
      "contextWindowTokens": 32768,
      "temperature": 0.2
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

`fallbackModels` 中的字符串条目是预设名，而不是原始的模型名。nanobot 会在活跃预设之后按顺序尝试它们。每个回退预设使用自己的 `provider`、`model`、`maxTokens`、`contextWindowTokens`、`temperature` 以及可选的 `reasoningEffort`。

只有当一个模型不值得作为预设命名时，才使用内联的回退对象：

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

`fallbackModels` 应放在 `agents.defaults` 下，而不是每个预设内部。如果回退候选使用了更小的上下文窗口，nanobot 会使用活跃链中最小的窗口来构建上下文，以便每个候选都能接收到相同的提示。失败条件请参阅 [`configuration.md#model-fallbacks`](./configuration.md#model-fallbacks)。

## 快速自检

在调试聊天应用之前，请先运行这些命令：

```bash
nanobot status
nanobot agent -m "Hello!"
```

如果 `nanobot agent -m "Hello!"` 失败：

| 症状 | 可能原因 |
|---|---|
| 401、unauthorized、invalid API key | key 缺失、过期、复制时带了空白字符，或者存放在错误的提供商下 |
| model not found | 所选提供商或网关不存在该模型 ID |
| connection refused | 本地提供商服务未运行，或者 `apiBase` 指向了错误的端口 |
| provider not found | 活跃预设使用了拼写错误的提供商；请使用注册表名称，例如 `openrouter`、`anthropic`、`ollama`、`vllm`、`lm_studio` |
| 在 CLI 中可用但在聊天应用中不可用 | 提供商没问题；请在 [`chat-apps.md`](./chat-apps.md) 或 [`troubleshooting.md`](./troubleshooting.md) 中调试网关/频道配置 |

完整的提供商表以及高级的提供商专属说明，请参阅 [`configuration.md#providers`](./configuration.md#providers)。
