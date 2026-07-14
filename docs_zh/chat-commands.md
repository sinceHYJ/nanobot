# 聊天内命令

这些命令可在聊天频道和交互式智能体会话中使用：

| 命令 | 说明 |
|---------|-------------|
| `/new` | 停止当前任务并开始新的对话 |
| `/stop` | 停止当前任务 |
| `/restart` | 重启机器人 |
| `/status` | 显示机器人状态 |
| `/model` | 显示当前模型及可用的模型预设 |
| `/model <preset>` | 为后续轮次切换运行时模型预设 |
| `/dream` | 立即运行 Dream 记忆整合 |
| `/dream-log` | 显示最近一次 Dream 记忆变更 |
| `/dream-log <sha>` | 显示指定的 Dream 记忆变更 |
| `/dream-restore` | 列出最近的 Dream 记忆版本 |
| `/dream-restore <sha>` | 将记忆恢复到指定变更之前的状态 |
| `/dream-prompt` | 显示 Dream 如何被引导来处理记忆 |
| `/dream-prompt init` | 在 `prompts/dream.md` 创建可编辑的 Dream 记忆引导文件 |
| `/skill` | 列出已启用的技能及其说明 |
| `/trigger` | 显示本地触发器用法 |
| `/trigger <name>` | 为当前聊天/会话创建一个命名的本地触发器 |
| `/pairing` | 列出待处理的配对请求 |
| `/pairing approve <code>` | 批准一个配对码 |
| `/pairing deny <code>` | 拒绝一个待处理的配对请求 |
| `/pairing revoke <user_id>` | 在当前频道撤销先前已批准的用户 |
| `/pairing revoke <channel> <user_id>` | 在指定频道撤销先前已批准的用户 |
| `/help` | 显示可用的聊天内命令 |

## 配对

当有人向机器人发送私信但不在白名单上时——无论是新用户，还是已存在的用户从新频道发送——nanobot 会自动回复一个 **配对码**（如 `ABCD-EFGH`），该配对码 10 分钟后失效。要授予访问权限：

```text
/pairing approve ABCD-EFGH
```

要查看谁在等待，使用 `/pairing`。要在之后移除某人，使用 `/pairing revoke <user_id>`——可以在 `/pairing list` 的输出中找到用户 ID。

完整设置指南请见 [配置：配对](./configuration.md#pairing)。

## 模型预设

使用 `/model` 查看当前的运行时模型：

```text
/model
```

响应会显示当前模型、当前预设，以及可用的预设名称。命名预设来自顶层的 `modelPresets` 配置，是配置模型选择的推荐方式。`default` 始终可用，代表来自 `agents.defaults.*` 字段直接设置的模型设置。

要为后续轮次切换预设：

```text
/model fast
/model deep
/model default
```

预设名称来自顶层的 `modelPresets` 配置。切换仅在运行时生效：它不会重写 `config.json`，正在进行中的轮次会继续使用其开始时的模型。设置详情请见 [配置：模型预设](./configuration.md#model-presets)。

## 本地触发器

当本地脚本或其他服务需要在稍后向当前聊天/会话发送消息时，使用 `/trigger <name>`。名称是必需的；不带参数的 `/trigger` 只会显示用法提示。

在希望后续消息送达的聊天中创建触发器：

```text
/trigger PR review
```

nanobot 会回复一个触发器 ID 以及形如下面的命令：

```bash
nanobot trigger trg_8K4P2Q9X "Review PR #4502"
```

将 `"Review PR #4502"` 替换为你希望 nanobot 收到的消息。触发器绑定到创建它的会话，因此消息会返回到相同的聊天。请保持 `nanobot gateway` 运行，以便触发器消息能被投递。触发器消息会启动一个记录在该会话中的自动化轮次，携带你通过 CLI 传入的消息；它不会被当作普通的用户消息处理。如果该会话已经正在运行一个轮次，触发器会等待会话空闲，而不是被注入到正在进行的轮次中。

触发器投递会存储在工作区中，直到其关联的智能体轮次成功完成。如果网关在认领投递之后、轮次完成之前退出，下一次网关启动会将该投递重新入队。这是一个至少一次的本地队列：如果进程在错误的时机退出，一次投递可能运行多次，因此外部脚本应保证重复触发消息是安全的。如果投递到达了智能体但智能体轮次失败，该投递会在自动化中被标记为失败，而不是无限重试。

对于较长或生成式的内容，省略消息参数并通过 stdin 传入：

```bash
printf '%s\n' "Review the latest failed CI job" | nanobot trigger trg_8K4P2Q9X
```

如果外部 webhook 需要唤醒 nanobot，请运行你自己的小型 webhook 服务，让它在构建好最终消息后调用触发器命令：

```bash
nanobot trigger <trigger-id> "<message>"
```

如果你运行多个 nanobot 实例，请传入网关所使用的相同 config 或 workspace 选择器：

```bash
nanobot trigger --config ./bot-a/config.json trg_8K4P2Q9X "Nightly report"
nanobot trigger --workspace ./bot-a/workspace trg_8K4P2Q9X "Nightly report"
```

在 WebUI 的自动化视图中管理触发器。你可以在那里搜索、暂停/恢复、重命名、删除以及复制触发器命令。一个会话可以有多个触发器，就像它可以有多个计划自动化一样。

关于本地触发器如何与计划自动化、心跳和网关投递协同工作，请见 [自动化](./automations.md)。

## 周期性任务

周期性后台检查由工作区中的 `HEARTBEAT.md` 驱动（`~/.nanobot/workspace/HEARTBEAT.md`）。当 `nanobot gateway` 启动时，它会默认注册一个受保护的心跳 cron 任务。每 30 分钟，该任务会检查该文件；如果发现 `## Active Tasks` 下有任务，智能体会执行它们，并且只将通过通知门的结果送达你最近活跃的聊天频道。如果没有活动任务，或者结果只是例行的、没有值得报告的内容，心跳会被静默跳过。

将心跳用于通常应保持安静的重复检查。用户创建的 cron 任务则不同：它们作为计划轮次运行在创建它们的聊天/会话中，通常会将结果送回该频道。

**设置：** 编辑 `~/.nanobot/workspace/HEARTBEAT.md`（由 `nanobot onboard` 自动创建）：

```markdown
## Active Tasks

- Check weather forecast and notify me only if storms are expected
- Scan inbox for urgent emails and notify me if any are found
```

智能体本身也可以管理这个文件——请它"添加一个周期性后台检查"或"周期性检查这个但只在发生变化时通知我"，它就会为你更新 `HEARTBEAT.md`。已完成的任务应该从文件中删除，而不是移动到其他章节。

你可以在 `~/.nanobot/config.json` 中修改间隔或禁用内置心跳：

```json
{
  "gateway": {
    "heartbeat": {
      "enabled": true,
      "intervalS": 1800
    }
  }
}
```

心跳任务在 `cron(action="list")` 中可见为 `heartbeat`，但它由系统管理，无法通过 `cron` 工具移除。要停止它，请将 `gateway.heartbeat.enabled` 设置为 `false` 并重启网关。

> **注意：** 网关必须处于运行状态（`nanobot gateway`），并且你必须至少与机器人聊过一次，这样它才知道要投递到哪个频道。
