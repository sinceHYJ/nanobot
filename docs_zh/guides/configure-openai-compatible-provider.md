# 如何在 nanobot 中配置 OpenAI 兼容的提供商

nanobot 可以通过配置 `apiBase`、可选的 `apiKey` 以及一个指向该提供商名称的
模型预设，来调用 OpenAI 兼容的模型提供商。

## 你将构建什么

- 一个自定义提供商条目
- 一个指向该提供商的模型预设
- 一次成功的 `nanobot agent` 运行

## 何时使用

用于暴露 OpenAI 兼容端点的本地或托管服务，包括内部网关、本地模型服务器以及
nanobot 中尚未命名的提供商代理。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
```

在调试 nanobot 之前，先确认端点可响应：

```bash
curl -sS https://api.example.com/v1/models
```

## 最小可用示例

将以下内容合并到 `~/.nanobot/config.json`：

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

然后运行：

```bash
nanobot agent -m "Hello!"
```

## 生产环境注意事项

- 当服务期望 `/v1` 时，将版本路径包含在 `apiBase` 中。
- 为不同的端点使用不同的提供商名称。
- 只有当端点要求非空密钥但不校验密钥时，才使用像 `EMPTY` 这样的占位密钥。
- 对于 OpenAI 兼容的自定义端点，保持 `apiType` 不设置。

## 安全注意事项

- 将提供商密钥保存在环境变量中。
- 将内部模型网关视为敏感的网络服务。
- 不要将私有工作区的 nanobot 指向不受信任的代理端点。

## 故障排查

- 如果 `curl /models` 失败，请先修复提供商端点，再修改 nanobot。
- 如果 nanobot 提示模型未知，请检查提供商所期望的 model ID。
- 如果鉴权失败，请确认提供商是否需要 Bearer 鉴权，以及启动 nanobot 的环境中
  是否存在该密钥。

## 相关 nanobot 文档

- [Provider Cookbook：自定义 OpenAI 兼容提供商](../provider-cookbook.md#recipe-custom-openai-compatible-provider)
- [提供商：自定义 OpenAI 兼容端点](../providers.md#custom-openai-compatible-endpoint)
- [OpenAI 兼容的 Agent API](./openai-compatible-agent-api.md)
