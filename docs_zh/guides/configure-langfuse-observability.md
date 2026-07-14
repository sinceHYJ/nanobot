# 如何为 nanobot 配置 Langfuse 可观测性

nanobot 可以通过 Langfuse 的 OpenAI SDK 封装器，对受支持的 OpenAI 兼容
提供商调用进行追踪。

## 你将构建什么

- 在与 nanobot 相同的 Python 环境中安装 Langfuse
- 在启动前设置 Langfuse 环境变量
- 一次被追踪的 nanobot 模型调用

## 何时使用

当你在开发或生产运行期间，需要对模型请求、时延、错误、成本或提示行为进行
可观测性时，请使用 Langfuse。

## 安装

安装 nanobot 并验证智能体可用：

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

安装 Langfuse：

```bash
python -m pip install langfuse
```

## 最小可用示例

在启动 nanobot 之前设置凭据：

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

## 生产环境注意事项

- Langfuse 通过环境变量配置，而不是 `config.json`。
- 从会导出相同变量的环境中启动服务。
- 在提供商可用之后再添加追踪；它不应作为初始配置的第一步。
- 不使用 OpenAI 兼容客户端路径的原生提供商，可能不会产生 Langfuse
  OpenAI 封装器追踪。

## 安全注意事项

- 将 Langfuse 项目视为存有敏感提示与输出的可观测性数据存储。
- 为个人、预发和生产流量使用不同的项目。
- 不要将 Langfuse 密钥提交到服务文件中。

## 故障排查

- 如果没有追踪出现，请确认服务进程能看到相应的环境变量。
- 确认提供商路径是 OpenAI 兼容的。
- 在调试服务日志之前，先在本地执行一次 `nanobot agent -m "Hello!"`。

## 相关 nanobot 文档

- [配置：Langfuse 可观测性](../configuration.md#langfuse-observability)
- [Provider Cookbook：Langfuse 追踪](../provider-cookbook.md#recipe-langfuse-tracing)
- [部署](../deployment.md)
