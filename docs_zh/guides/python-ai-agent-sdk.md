# Nanobot Python SDK：从 Python 运行 AI 智能体

本指南展示了何时应使用 Nanobot Python SDK，而不是直接调用模型。该 SDK 运行的是与 CLI 相同的智能体运行时：模型路由、工具、工作区访问、会话历史、记忆、流式事件以及运行时辅助能力。

## 你将构建的内容

- 一个创建 `Nanobot` 的 Python 脚本
- 一次通过代码进行的智能体运行
- 一次可选的、带有工具可见性的流式运行

## 何时使用

在 notebooks、评测、产品后端、本地脚本、工作流运行器，以及需要直接访问智能体会话、记忆、钩子、运行时状态或结构化运行结果的集成中，请使用 Python SDK。

当另一种语言或进程需要通过 HTTP 调用 nanobot 时，请改用 OpenAI 兼容 API。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

## 最小可运行示例

```python
import asyncio

from nanobot import Nanobot


async def main() -> None:
    async with Nanobot.from_config() as bot:
        result = await bot.run("List the top-level files in this workspace.")
    print(result.content)


asyncio.run(main())
```

## 生产环境说明

- 对相关工作复用同一个 `Nanobot` 实例。
- 当某个用户、任务或评测用例需要持久化历史时，请传入 `session_key`。
- 当调用方需要实时的文本、工具或失败事件时，请使用 `bot.stream(...)`。
- 使用钩子来实现审计日志或自定义可观测性。

## 安全说明

- SDK 使用与 CLI 相同的配置、工作区、工具和密钥。
- 不要在拥有宽泛文件或 shell 权限的情况下运行不受信任的提示词。
- 为不同的产品或租户保留独立的配置/工作区路径。

## 故障排查

- 如果 SDK 代码失败，请先在同一环境中运行 `nanobot agent -m "Hello!"`。
- 打印 `bot.runtime.workspace` 和 `bot.runtime.model` 以确认加载了预期的配置。
- 当脚本从服务中运行时，请使用显式的 `config_path` 和 `workspace`。

## 相关 nanobot 文档

- [Nanobot Python SDK](../python-sdk.md)
- [OpenAI 兼容 API](../openai-api.md)
- [配置](../configuration.md)
- [概念](../concepts.md)
