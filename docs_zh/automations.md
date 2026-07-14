# 自动化

<!-- Meta description: Create, run, and manage nanobot scheduled automations, local triggers, and heartbeat-backed background checks. -->

自动化是在关联的聊天/会话中稍后运行的智能体轮次。当 nanobot
需要在无人主动输入时完成工作时使用它们：提醒、
周期性检查、每晚汇总、CI 后续处理、本地脚本报告，或
由 webhook 驱动的事件。

从希望看到结果的聊天、频道或 WebUI 会话中创建自动化。这样 nanobot
就能保持正确的会话历史、工作区和
回复目标。

## 选择自动化类型

| 类型 | 起始来源 | 最适用场景 | 创建方式 |
|---|---|---|---|
| 计划自动化 | 时间、间隔或 cron 表达式 | 周期性提醒、计划汇总、一次性未来任务 | 在目标会话中让 nanobot 使用 `cron` 工具进行调度 |
| 本地触发器 | 本地 `nanobot trigger ...` 命令 | CI 作业、webhook、shell 脚本、生成的报告 | 在目标会话中执行 `/trigger <name>` |
| 心跳 | 受保护的系统调度 | 只在有有用结果时才报告的静默周期性检查 | 编辑 `<workspace>/HEARTBEAT.md` |

用户创建的两种自动化类型是计划自动化和本地
触发器。心跳使用相同的后台服务，但由系统管理并
受到保护，不受常规自动化编辑影响。

## 创建前准备

保持 `nanobot gateway` 运行。网关负责聊天
应用、WebUI 会话、计划自动化、本地触发器、心跳和
Dream 作业的后台投递。

网关与任何发送本地触发器消息的进程应使用相同的工作区和配置。如果你运行多个 nanobot 实例，请为 `nanobot trigger` 传入匹配的
`--config` 或 `--workspace` 选项。

从目标会话中创建每个自动化。没有关联
聊天/会话的自动化无法从 WebUI 启用或运行，因为 nanobot 不知道
该把轮次投递到哪里。

## 计划自动化

计划自动化由智能体的 `cron` 工具创建。在实际操作中，从目标聊天或 WebUI 会话中让
nanobot 执行：

```text
Every weekday at 9am, check open pull requests and summarize blockers here.
```

或：

```text
Tomorrow at 4pm, remind me to send the release notes.
```

cron 工具支持间隔调度、cron 表达式和一次性
计划任务。cron 表达式可以包含 IANA 时区，例如
`America/Vancouver`；否则 nanobot 使用运行时默认时区。

计划自动化通常会将结果回传到创建它们的会话。使用它们来处理那些需要按可预测的调度运行并
汇报每次运行结果的工作。

对于只在有值得报告的内容时才应说话的后台检查，
请使用心跳，而不是用户创建的计划自动化。

## 本地触发器

本地触发器允许本地脚本或外部服务稍后向
特定的 nanobot 会话发送消息。

从希望后续消息到达的聊天或 WebUI 会话中创建触发器：

```text
/trigger PR review
```

nanobot 会回复一个触发器 ID 以及形如以下的命令：

```bash
nanobot trigger trg_8K4P2Q9X "Review PR #4502"
```

将引号中的文本替换为 nanobot 应接收的消息。对于生成的
或较长的内容，通过 stdin 传入：

```bash
generate-report | nanobot trigger trg_8K4P2Q9X
```

对于多实例场景，使用与网关相同的配置或工作区选择器：

```bash
nanobot trigger --config ./bot-a/config.json trg_8K4P2Q9X "Nightly report"
nanobot trigger --workspace ./bot-a/workspace trg_8K4P2Q9X "Nightly report"
```

nanobot 不为本地触发器提供内置的公共 webhook 接收器。
如果 GitHub、CI 或其他外部系统需要唤醒 nanobot，请运行你自己的
小型 webhook 服务，并让它在构建好最终消息后再调用 `nanobot trigger`。

## 心跳

心跳适用于通常应保持静默的周期性工作区检查。它
读取 `<workspace>/HEARTBEAT.md`，执行活跃任务，并只将有用或
可操作的结果发送到最近活跃的聊天目标。

将心跳用于诸如"关注此仓库是否出现重大失败"或
"周期性检查此工作区，只在需要采取行动时通知我"的检查。当每次运行都应产生可见提醒
或报告时，请改用计划自动化。

当 `nanobot gateway` 启动时，心跳默认启用。请在
[`configuration.md#gateway-heartbeat`](./configuration.md#gateway-heartbeat) 中配置它。

## 管理自动化

使用 WebUI 的 Automations 视图可以：

- 按全部、活跃、暂停、需关注或系统作业进行筛选；
- 按任务名称、消息、触发命令、关联聊天、调度或
  状态进行搜索；
- 按下次运行时间、上次运行时间、更新时间或名称排序；
- 立即运行计划自动化；
- 暂停或恢复、重命名或删除用户创建的自动化；
- 复制本地触发器的 CLI 命令；
- 检查受保护的系统自动化而不做修改。

本地触发器在 WebUI 中没有"立即运行"操作，因为每次运行都需要一条
消息。请从 WebUI 复制 `nanobot trigger ...` 命令，并将
`"message"` 替换为应投递的内容。

## 投递与可靠性

自动化投递是工作区本地的。计划作业和本地触发器
投递使用与网关相同的工作区。

本地触发器消息会写入持久化队列。如果网关尚未
运行，消息会在该工作区中等待。如果关联会话
正在运行某个轮次，触发器会等待会话变为空闲，而不是
被注入到活跃轮次中。

本地触发器队列是至少一次（at-least-once）而非恰好一次（exactly-once）。如果网关在
认领投递后、关联轮次完成前退出，下次网关
启动时会重新入队该投递。外部脚本应确保重复的触发器
消息是安全的。如果投递到达智能体但轮次失败，
该投递会被标记为失败，而不是永远重试。

每次本地触发器投递都会在
`<workspace>/triggers/runs` 下写入一条审计记录。每个工作区应仅运行一个网关消费者；本地
队列不是分布式的多消费者队列。

## 常见模式

对于每晚报告，从目标会话中执行：

```text
Every night at 9pm, review today's workspace changes and summarize anything I should handle tomorrow.
```

对于 CI 后续处理，先创建一次触发器：

```text
/trigger CI follow-up
```

然后让你的 CI 或 webhook 适配器调用：

```bash
nanobot trigger <trigger-id> "Build failed on main. Inspect the logs and suggest the next fix."
```

对于本地报告脚本：

```bash
generate-report | nanobot trigger <trigger-id>
```

## 故障排查

如果自动化没有运行，请确认 `nanobot gateway` 正在运行、
该自动化已启用，并且它是从关联的聊天/会话中创建的。

如果本地触发器一直等待，请确认命令使用了与网关相同的工作区或
配置。

如果重启后触发器消息出现两次，请视为预期的
至少一次投递，并让外部消息具备幂等性。

如果你需要编辑、暂停、恢复、重命名、删除或检查自动化，请使用
WebUI 的 Automations 视图。

## 相关文档

- [`webui.md#automations`](./webui.md#automations) 查看浏览器管理视图
- [`chat-commands.md#local-triggers`](./chat-commands.md#local-triggers) 了解 `/trigger`
- [`cli-reference.md#local-triggers`](./cli-reference.md#local-triggers) 了解 `nanobot trigger`
- [`configuration.md#gateway-heartbeat`](./configuration.md#gateway-heartbeat) 了解心跳设置
- [`guides/long-running-ai-agent.md`](./guides/long-running-ai-agent.md) 了解长时间运行的智能体工作
