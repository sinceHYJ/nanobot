# 图像生成

nanobot 可以通过 `generate_image` 工具生成和编辑图像。在 WebUI Settings 中启用该工具，然后就可以在聊天中正常请求图像；智能体会决定何时调用它，并且可以在同一次对话中持续迭代已生成的图像。

该特性默认关闭。在 `~/.nanobot/config.json` 中启用它，配置一个受支持的图像提供商，然后重启网关。

## 快速配置

以下片段使用当前内置的图像生成默认值，让 JSON 有具体的名称。这不是对提供商的推荐；请把 `provider` 和 `model` 替换为你打算使用的任何受支持的图像提供商和模型。

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "${OPENROUTER_API_KEY}"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "openrouter",
      "model": "openai/gpt-5.4-image-2"
    }
  }
}
```

关于 Custom、AIHubMix、MiniMax、Gemini、Ollama、StepFun 和 Zhipu 的配置示例，请参见 [提供商说明](#provider-notes)。

> [!TIP]
> API 密钥优先使用环境变量。nanobot 在启动时会解析 `${VAR_NAME}` 值。

## WebUI 用法

1. 打开 Settings，为一个已配置的提供商和模型启用 **Image Generation**。
2. 在聊天中描述你想要的图像或编辑。
3. 当配置的默认值不合适时，在请求中包含宽高比或尺寸。
4. 编辑已有图像时可附上参考图。

生成的图像会作为助手媒体渲染在聊天中。后续的提示，例如 "make it warmer"、"change the background" 或 "try a 16:9 version"，可以复用最近生成的产物。

WebUI 对用户隐藏了提供商存储细节。智能体内部可以看到已保存产物的路径，并可以将其作为 `reference_images` 传回给 `generate_image` 进行迭代编辑。

## 配置参考

| 选项 | 类型 | 默认值 | 描述 |
|--------|------|---------|-------------|
| `tools.imageGeneration.enabled` | boolean | `false` | 注册 `generate_image` 工具 |
| `tools.imageGeneration.provider` | string | `"openrouter"` | 当前内置的图像提供商默认值。支持的值：`openrouter`、`openai`、`openai_codex`、`custom`、`aihubmix`、`minimax`、`gemini`、`ollama`、`stepfun`、`zhipu` |
| `tools.imageGeneration.model` | string | `"openai/gpt-5.4-image-2"` | 提供商模型名 |
| `tools.imageGeneration.defaultAspectRatio` | string | `"1:1"` | 当 prompt/工具调用未指定时使用的默认比例 |
| `tools.imageGeneration.defaultImageSize` | string | `"1K"` | 默认尺寸提示，例如 `1K`、`2K`、`4K` 或 `1024x1024` |
| `tools.imageGeneration.maxImagesPerTurn` | number | `4` | 单次工具调用接受的最大 `count`。有效范围：`1` 到 `8` |
| `tools.imageGeneration.saveDir` | string | `"generated"` | 位于 nanobot 媒体目录下、用于存放生成产物的相对目录 |

提供商设置复用普通的提供商配置字段：

| 选项 | 描述 |
|--------|-------------|
| `providers.<name>.apiKey` | 提供商 API 密钥。建议使用 `${ENV_VAR}` |
| `providers.<name>.apiBase` | 可选的自定义 base URL |
| `providers.<name>.extraHeaders` | 会被合并进提供商请求的头部 |
| `providers.<name>.extraBody` | 会被合并进提供商请求体的额外 JSON 字段 |

配置键的 camelCase 和 snake_case 都被接受，但文档使用 camelCase 以与 `config.json` 保持一致。

## 提供商说明

### OpenRouter

OpenRouter 使用类似 chat-completions 风格的图像响应。配置：

```json
{
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "openrouter",
      "model": "openai/gpt-5.4-image-2"
    }
  }
}
```

如果你想使用参考图编辑，请使用支持图像生成和图像编辑的模型。

### Custom（兼容 OpenAI）

`custom` 图像提供商适合实现了同步 OpenAI Images API 的服务：

```text
POST /v1/images/generations
```

响应必须在 `data[].b64_json` 或 `data[].url` 中包含生成的图像。原生预测 API，例如 Replicate 的 `/v1/models/{owner}/{model}/predictions`，除非在其前面放一个 OpenAI 兼容网关，否则并不直接兼容。

配置：

```json
{
  "providers": {
    "custom": {
      "apiKey": "${CUSTOM_IMAGE_API_KEY}",
      "apiBase": "https://api.example.com/v1"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "custom",
      "model": "your-model-name"
    }
  }
}
```

`apiBase` 是必填的。该提供商使用 OpenAI Images API 格式向 `{apiBase}/images/generations` 发送请求，并带上 `response_format: "b64_json"`。对于本地或未鉴权的端点，`apiKey` 可选。通用 `custom` 提供商不支持参考图编辑。

`extraBody` 可以适配提供商特有的差异，因为它最后被合并进请求体。示例：

- Agnes AI 文档表明使用 URL 响应，因此使用 `"extraBody": {"response_format": "url"}`。
- Together AI 文档要求 `"response_format": "base64"`，需要覆盖默认值。
- Volcengine Ark Seedream 模型可能需要如 `"2K"`、`"3K"`、`"4K"` 之类的尺寸提示，或明确的尺寸。将 `tools.imageGeneration.defaultImageSize` 或 `providers.custom.extraBody.size` 设置为所选模型支持的值。

为了与 nanobot 默认设置兼容，custom 会把 `defaultImageSize: "1K"` 映射为 `1024x1024`。其他显式的尺寸提示会原样透传。

### AIHubMix

AIHubMix `gpt-image-2-free` 通过 AIHubMix 的统一预测 API 得到支持。nanobot 内部会调用：

```text
/v1/models/openai/gpt-image-2-free/predictions
```

配置：

```json
{
  "providers": {
    "aihubmix": {
      "apiKey": "${AIHUBMIX_API_KEY}",
      "extraBody": {
        "quality": "low"
      }
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "aihubmix",
      "model": "gpt-image-2-free"
    }
  }
}
```

`quality: low` 是可选的。它可以让免费的图像模型更快、更不易超时，但对正确性并不必要。

### MiniMax

MiniMax `image-01` 支持文本生成图像和参考图（subject reference）编辑。支持的宽高比包括 `1:1`、`16:9`、`4:3`、`3:2`、`2:3`、`3:4`、`9:16` 和 `21:9`。

```json
{
  "providers": {
    "minimax": {
      "apiKey": "${MINIMAX_API_KEY}"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "minimax",
      "model": "image-01",
      "defaultAspectRatio": "1:1"
    }
  }
}
```

### Gemini

nanobot 通过 Google 的 Generative Language API 支持两类 Gemini 图像生成模型系列：

| 模型 | 端点 | 参考图 |
|-------|----------|-----------------|
| `imagen-4.0-generate-001` | `:predict` | 该集成不支持 |
| `gemini-2.5-flash-image` | `:generateContent` | 支持 |

对于参考图编辑，请使用 Gemini Flash 图像模型：

```json
{
  "providers": {
    "gemini": {
      "apiKey": "${GEMINI_API_KEY}"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "gemini",
      "model": "gemini-2.5-flash-image"
    }
  }
}
```

Imagen 4 支持的宽高比为 `1:1`、`9:16`、`16:9`、`3:4` 和 `4:3`。不支持的比例会被忽略，模型将使用其默认值。`defaultImageSize` 对 Gemini 模型无效；尺寸仅通过 `defaultAspectRatio` 控制。传给 Imagen 模型的参考图会被忽略（并记录警告）。

### Ollama

Ollama 实验性的原生图像生成 API 可与本地服务器和托管的 ollama.com 模型配合使用。访问本地 `http://localhost:11434/api` 不需要 API 密钥；仅当目标为 `https://ollama.com/api` 时才需要设置 `providers.ollama.apiKey`。

```json
{
  "providers": {
    "ollama": {
      "apiBase": "http://localhost:11434/api"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "ollama",
      "model": "x/z-image-turbo",
      "defaultAspectRatio": "16:9",
      "defaultImageSize": "2K"
    }
  }
}
```

Ollama 会把 `defaultAspectRatio` 和 `defaultImageSize` 映射为原生的 `width` 与 `height` 值。该集成不支持参考图。

### StepFun

StepFun（阶跃星辰）`step-image-edit-2` 支持文本生成图像。`step-1x-medium` 变体还额外支持 **style-reference** 图像编辑，即用参考图指导输出的视觉风格。

支持的宽高比：`1:1`、`16:9`、`9:16`、`3:4`、`4:3`。尺寸以 `WIDTHxHEIGHT` 指定（例如 `1024x1024`、`1280x800`、`800x1280`）。

```json
{
  "providers": {
    "stepfun": {
      "apiKey": "${STEPFUN_API_KEY}"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "stepfun",
      "model": "step-image-edit-2"
    }
  }
}
```

> [!NOTE]
> StepFun 提供商复用现有的 `providers.stepfun` 配置块（与 StepFun 的 LLM API 使用的是同一个）。设置一次 `providers.stepfun.apiKey`，就会在文本和图像生成之间共享。
>
> 使用 `step-image-edit-2` 时，`reference_images` 会被忽略（该模型不支持风格参考）。切换到 `step-1x-medium` 才能使用参考图引导的生成。

#### StepPlan（订阅版）

StepPlan 是 StepFun 的订阅层级，使用不同的 API base URL。图像生成端点路径相同——只需覆盖 `apiBase`：

```json
{
  "providers": {
    "stepfun": {
      "apiKey": "${STEPFUN_API_KEY}",
      "apiBase": "https://api.stepfun.ai/step_plan/v1"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "stepfun",
      "model": "step-image-edit-2"
    }
  }
}
```

`apiBase` 优先级高于注册表默认值，因此在配置 StepPlan base URL 后，图像请求会被发送到 `https://api.stepfun.ai/step_plan/v1/images/generations` —— 与 LLM 调用使用相同的路径前缀。API 密钥与标准 StepFun 提供商共享。

### Zhipu

Zhipu（智谱）`glm-image` 模型支持文本生成图像。API 返回临时图像 URL（有效期 30 天）；nanobot 会下载并将其重新编码为 base64 数据 URL。

支持的宽高比：`1:1`、`16:9`、`9:16`、`3:4`、`4:3`。尺寸可以以 `WIDTHxHEIGHT` 指定（例如 `1280x1280`、`1728x960`）或使用宽高比预设。

```json
{
  "providers": {
    "zhipu": {
      "apiKey": "${ZAI_API_KEY}"
    }
  },
  "tools": {
    "imageGeneration": {
      "enabled": true,
      "provider": "zhipu",
      "model": "glm-image"
    }
  }
}
```

其他支持的模型：`cogview-4`、`cogview-4-250304`、`cogview-3-flash`。该集成不支持参考图。

## 产物

生成的图像存储在当前活跃 nanobot 实例的媒体目录下：

```text
~/.nanobot/media/generated/YYYY-MM-DD/img_<id>.<ext>
~/.nanobot/media/generated/YYYY-MM-DD/img_<id>.json
```

对于非默认的配置位置，媒体目录相对于当前配置文件所在的目录。

JSON sidecar 存储：

| 字段 | 含义 |
|-------|---------|
| `id` | 生成图像的短 id，例如 `img_ab12cd34ef56` |
| `path` | 内部使用的本地图像路径，用于后续编辑 |
| `mime` | 检测到的图像 MIME 类型 |
| `prompt` | 用于生成的提示 |
| `model` | 提供商模型 |
| `provider` | 提供商名称 |
| `source_images` | 用于编辑的参考图路径 |
| `created_at` | 创建时间戳 |

不要在聊天中粘贴 base64 图像负载。除非用户明确要求调试细节，否则智能体应把本地产物路径保留在内部。

## 提示词编写

好的图像提示包括：

- 主体和场景。
- 构图、镜头或布局。
- 风格、氛围、光线和配色。
- 必须出现在图像中的确切文字，加引号引出。
- 约束条件，例如 "keep the same character" 或 "preserve the logo"。

示例：

```text
A minimal app icon for nanobot: friendly robot head, rounded square, soft blue and white palette, clean vector style, no text
```

对于编辑，描述应该改变什么以及应保持不变的部分：

```text
Use the reference image. Keep the same robot and composition, change the palette to warm orange, and add a subtle sunrise background.
```

## 故障排查

| 症状 | 检查点 |
|---------|-------|
| `generate_image` 不可用 | 将 `tools.imageGeneration.enabled` 设为 `true` 并重启网关 |
| 缺少 API 密钥错误 | 配置 `providers.<provider>.apiKey`；若使用 `${VAR_NAME}`，确认环境变量对网关进程可见 |
| `unsupported image generation provider` | 使用 `openrouter`、`openai`、`openai_codex`、`custom`、`aihubmix`、`minimax`、`gemini`、`ollama`、`stepfun` 或 `zhipu` |
| AIHubMix 报 `Incorrect model ID` | 使用 `model: "gpt-image-2-free"`；nanobot 会在内部把它展开为所需的 `openai/gpt-image-2-free` 模型路径 |
| 生成超时 | 尝试更小/默认尺寸、将 AIHubMix `extraBody.quality` 设为 `"low"`，或稍后重试 |
| 参考图被拒 | 参考图路径必须在工作区或 nanobot 媒体目录内，并且必须是有效的图像文件 |
