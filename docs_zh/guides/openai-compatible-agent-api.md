# 如何使用 nanobot 运行 OpenAI 兼容的智能体 API

nanobot 可以在 `/v1/chat/completions` 后暴露一个本地的 OpenAI 兼容端点。这让现有的 OpenAI 风格的客户端可以与一个使用工具的 nanobot 智能体通信，而不是直接与原始模型通信。

## 你将构建的内容

- 一个可工作的 nanobot 智能体
- 一个位于 `127.0.0.1:8900` 的本地 API 服务器
- 一次 `/v1/chat/completions` 请求
- 可选的基于 `session_id` 的会话隔离

## 何时使用

当已有的客户端、另一种语言或另一个进程已经知道如何调用 OpenAI 兼容 API 时，请使用它。当你希望以进程内方式访问会话、记忆、运行时辅助能力和钩子时，请使用 Python SDK。

## 安装

```bash
python -m pip install nanobot-ai
nanobot plugins enable api
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

## 最小可运行示例

启动 API 服务器：

```bash
nanobot serve
```

调用聊天端点：

```bash
curl http://127.0.0.1:8900/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "hi"}],
    "session_id": "demo"
  }'
```

## 生产环境说明

- 传入 `session_id` 来隔离用户、任务或工作流。
- 当 `stream` 为 `true` 时，流式使用 Server-Sent Events。
- `/v1/models` 报告兼容客户端所期望的固定模型接口。
- 通过 JSON base64 或 multipart form data 支持文件上传。

## 安全说明

- 本地 `127.0.0.1` 使用无需 API key。
- 如果 `api.host` 是 `0.0.0.0` 或 `::`，请在启动前配置 `api.apiKey`。
- 将该 API 视为智能体访问，而不仅仅是模型访问：工具和工作区权限仍然重要。

## 故障排查

- 如果 `/v1/chat/completions` 失败，请先测试 `nanobot agent -m "Hello!"`。
- 如果远程客户端无法连接，请检查 `api.host`、`api.port`、防火墙和 API key 配置。
- 如果会话混在了一起，请传入唯一的 `session_id` 值。

## 相关 nanobot 文档

- [Nanobot OpenAI 兼容 API](../openai-api.md)
- [Python SDK](../python-sdk.md)
- [配置](../configuration.md)
- [部署](../deployment.md)
