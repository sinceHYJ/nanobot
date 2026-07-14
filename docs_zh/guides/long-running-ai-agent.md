# 如何使用 nanobot 运行长时间运行的 AI 智能体

nanobot 可以通过持续目标、持久会话、定时自动化、本地触发器以及持续运行的
网关进程，让智能体的工作在多轮之间保持存活。

## 你将构建什么

- 一个可用的本地智能体
- 一次持久的聊天会话
- 一个长时间运行的目标或自动化
- 一个用于后台消息投递的网关进程

## 何时使用

当任务不是一次性回答时使用：项目工作、周期性检查、定时摘要、文件维护、
多步研究，或来自脚本和构建任务的本地触发。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

## 最小可用示例

启动一个网关：

```bash
nanobot gateway
```

在 WebUI 或聊天会话中，启动一个持续目标：

```text
/goal Review this workspace, identify missing tests, and propose the smallest next fix.
```

对于定时或基于触发器的运行，请从目标聊天创建自动化，以便 nanobot 能将
它关联到正确的会话和工作区。

## 生产建议

- 为聊天应用、WebUI 会话、自动化和本地触发器保持网关运行。
- 对需要保留上下文的工作，使用稳定的会话键或聊天会话。
- 保持目标有边界，并对完成条件表达明确。
- 在依赖某个定时任务之前，先在 WebUI 中审查自动化。

## 安全建议

- 将长时间运行的目标视为拥有真实工具访问权限的委派工作。
- 在安排无人值守任务之前，限制工作区和 shell 执行权限。
- 保持聊天访问范围精简，避免未知用户创建目标或自动化。

## 故障排查

- 如果某个目标看起来卡住了，检查当前会话和网关日志。
- 如果某个自动化未运行，检查它是否关联到一个聊天/会话，
  并且网关是否仍在运行。
- 如果本地触发器失败，检查从 WebUI 自动化视图复制的命令。

## 相关的 nanobot 文档

- [自动化](../automations.md)
- [WebUI 自动化](../webui.md#automations)
- [聊天命令](../chat-commands.md)
- [记忆](../memory.md)
- [部署](../deployment.md)
