# 如何在 nanobot 中配置模型回退

模型回退让 nanobot 先尝试主模型，然后在主提供商失败或触发限流时回退到一个
或多个具名预设。

## 你将构建什么

- 两个或更多 `modelPresets`
- 一个主 `agents.defaults.modelPreset`
- 一条有序的 `agents.defaults.fallbackModels` 链

## 何时使用

当你希望在限流、提供商故障、本地模型宕机或成本敏感路由方面获得更好的可靠性
时，请使用回退。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

在把某个提供商添加为回退之前，先确认它可以正常工作。

## 最小可用示例

将下面的结构合并到 `~/.nanobot/config.json`，并将提供商/模型名替换为你可控
的实际值：

```json
{
  "modelPresets": {
    "fast": {
      "label": "Fast",
      "provider": "primary-provider",
      "model": "primary-model-id",
      "maxTokens": 4096,
      "contextWindowTokens": 65536,
      "temperature": 0.1
    },
    "deep": {
      "label": "Deep",
      "provider": "fallback-provider",
      "model": "fallback-model-id",
      "maxTokens": 4096,
      "contextWindowTokens": 200000,
      "temperature": 0.1
    }
  },
  "agents": {
    "defaults": {
      "modelPreset": "fast",
      "fallbackModels": ["deep"]
    }
  }
}
```

`fallbackModels` 中的字符串条目是预设名，而不是原始的 model ID。请将占位的
模型 ID 替换为你所用提供商当前支持的 model ID。[Provider Cookbook](../provider-cookbook.md)
提供了针对常见提供商的具体配方。

## 生产环境注意事项

- 保持回退的上下文窗口切合实际；较小的回退窗口会限制可容纳的上下文量。
- 在可接受的情况下，将更便宜或更快的回退放在昂贵的回退之前。
- 使用 `/model <preset>` 在运行时切换而无需修改配置。
- 使标签保持易读，以便在 WebUI 模型列表中显示。

## 安全注意事项

- 不同提供商可能有不同的数据处理策略。
- 不要将提供商密钥直接放入共享的配置文件中。
- 确认回退模型可以安全接收相同的提示与文件。

## 故障排查

- 如果回退从未触发，请确认主错误被视为可重试/可回退的错误。
- 如果启动失败，请检查每个回退字符串是否与 `modelPresets` 下的键匹配。
- 如果回退后输出被截断，请检查 `maxTokens` 和 `contextWindowTokens`。

## 相关 nanobot 文档

- [提供商与模型](../providers.md)
- [Provider Cookbook：回退预设](../provider-cookbook.md#recipe-fallback-presets)
- [配置：模型回退](../configuration.md#model-fallbacks)
