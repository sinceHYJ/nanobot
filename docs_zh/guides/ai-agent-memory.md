# nanobot 中的 AI 智能体记忆如何工作

本指南解释如何使用 nanobot 的长期 AI 智能体记忆：会话历史、压缩归档、
持久记忆文件、Dream 整合，以及基于 Git 的记忆变更。

## 你将构建什么

- 一个带有持久会话历史的工作区
- 用于较早轮次的压缩历史归档
- 诸如 `USER.md` 和 `MEMORY.md` 之类的持久记忆文件
- 一个用于整理长期记忆的 Dream 工作流

## 何时使用

当智能体应该在会话之间记住稳定的偏好、项目事实、决策和反复出现的
上下文时使用记忆。不要把记忆当作每一份原始对话记录的倾倒场；
nanobot 会将短期消息与经过整理的持久知识分开管理。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

## 最小可用示例

在一次普通会话中让智能体记住一个稳定的事实，然后运行 Dream：

```text
/dream
```

查看最近的记忆变更：

```text
/dream-log
```

具体的文件位于当前工作区内，通常位于 `~/.nanobot/workspace/` 下。

## 生产建议

- 每个项目或个人场景使用一个工作区。
- 让持久事实保持简洁；旧的会话细节归属于 `history.jsonl`。
- 当工作区需要自定义的记忆指引时，使用 `/dream-prompt init`。
- 当记忆影响到重要工作流时，审查基于 Git 的记忆变更。

## 安全建议

- 记忆文件可能包含敏感的用户或项目信息。
- 在共享工作区之前，先审查 `SOUL.md`、`USER.md` 和 `memory/MEMORY.md`。
- 为个人和团队场景使用独立的工作区。

## 故障排查

- 如果记忆看起来陈旧，运行 `/dream` 并检查 `/dream-log`。
- 如果记忆被错误更新，使用 `/dream-restore` 检查并恢复到之前的版本。
- 如果新会话缺少上下文，确认它使用的是同一个工作区。

## 相关的 nanobot 文档

- [nanobot 中的 AI 智能体记忆](../memory.md)
- [核心概念](../concepts.md)
- [配置](../configuration.md#auto-compact)
- [聊天命令](../chat-commands.md)
