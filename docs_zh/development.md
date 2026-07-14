# 开发

本页汇总了扩展 nanobot 时面向贡献者的注意事项。面向用户的安装和运行时选项位于 [`configuration.md`](./configuration.md)。

## 添加 LLM 提供商

nanobot 使用 `nanobot/providers/registry.py` 中的提供商注册表作为 LLM 提供商元数据的唯一真实来源。大多数兼容 OpenAI 的提供商只需两处修改。

1. 向 `PROVIDERS` 中添加一个 `ProviderSpec` 条目：

```python
ProviderSpec(
    name="myprovider",
    keywords=("myprovider", "mymodel"),
    env_key="MYPROVIDER_API_KEY",
    display_name="My Provider",
    default_api_base="https://api.myprovider.com/v1",
)
```

2. 在 `nanobot/config/schema.py` 的 `ProvidersConfig` 中添加一个字段：

```python
class ProvidersConfig(BaseModel):
    ...
    myprovider: ProviderConfig = Field(default_factory=ProviderConfig)
```

环境变量、配置匹配、提供商状态以及 WebUI 凭据展示都来源于以上两个条目。

`ProviderSpec` 常用选项：

| 字段 | 描述 |
|---|---|
| `default_api_base` | 默认兼容 OpenAI 的 base URL。 |
| `env_extras` | 从提供商配置派生出的额外环境变量。 |
| `model_overrides` | 按模型级别的请求参数覆盖。 |
| `is_gateway` | 该提供商可以路由多种模型系列，例如 OpenRouter。 |
| `detect_by_key_prefix` | 通过 API 密钥前缀匹配已配置的网关。 |
| `detect_by_base_keyword` | 通过 API base URL 匹配已配置的网关。 |
| `strip_model_prefix` | 在向上游 API 发送前去掉 `provider/` 前缀。 |
| `supports_max_completion_tokens` | 使用 `max_completion_tokens` 代替 `max_tokens`。 |
| `is_transcription_only` | 该提供商有凭据但无法提供聊天补全。 |

## 添加转写提供商

转写有意分为两层：

- `nanobot/audio/transcription_registry.py` 拥有提供商名称、别名、默认模型以及适配器加载。
- `nanobot/providers/transcription.py` 拥有提供商特定的 HTTP 行为。

凭据仍存在于 `providers.<provider>` 下，以便聊天频道和 WebUI 以相同方式解析 API 密钥和 API base。

1. 在 `ProvidersConfig` 中添加提供商凭据。

```python
class ProvidersConfig(BaseModel):
    ...
    my_stt: ProviderConfig = Field(default_factory=ProviderConfig)
```

2. 在 `nanobot/providers/registry.py` 中添加一个 `ProviderSpec`。

对于仅用于转写的提供商，设置 `is_transcription_only=True`，这样它们会出现在凭据/设置界面中，但不会出现在聊天模型选择中。

```python
ProviderSpec(
    name="my_stt",
    keywords=("my_stt",),
    env_key="MY_STT_API_KEY",
    display_name="My STT",
    default_api_base="https://api.example.com/v1",
    is_transcription_only=True,
)
```

3. 在 `nanobot/providers/transcription.py` 中添加一个适配器类。

适配器会接收已解析的凭据和设置。它们在遇到提供商错误时返回空字符串，让频道语音消息安静地失败，而不是让智能体循环崩溃。

```python
class MySTTTranscriptionProvider:
    def __init__(
        self,
        api_key: str | None = None,
        api_base: str | None = None,
        language: str | None = None,
        model: str | None = None,
    ):
        self.api_key = api_key or os.environ.get("MY_STT_API_KEY")
        self.api_base = api_base or "https://api.example.com/v1"
        self.language = language or None
        self.model = model or "my-default-stt-model"

    async def transcribe(self, file_path: str | Path) -> str:
        ...
```

4. 在 `nanobot/audio/transcription_registry.py` 中注册适配器。

```python
TranscriptionProviderSpec(
    name="my_stt",
    default_model="my-default-stt-model",
    adapter="nanobot.providers.transcription:MySTTTranscriptionProvider",
    aliases=("mystt",),
)
```

5. 添加测试。

至少需要覆盖：

- `tests/providers/test_transcription.py` 中的配置解析
- 适配器的请求/响应行为以及重试/错误处理
- `tests/webui/test_settings_api.py` 中的 WebUI 设置负载/更新行为
- 若提供商出现在 Settings 中，还要覆盖提供商品牌映射

6. 更新面向用户的文档。

在 [`configuration.md`](./configuration.md) 中用户选择 `transcription.provider` 的位置添加该提供商，但把实现细节留在本开发指南中。
