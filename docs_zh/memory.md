# nanobot 中的 AI 智能体记忆

本页解释 nanobot 如何实现长期 AI 智能体记忆：会话历史、压缩归档、持久化的知识文件、Dream 整合，以及由 Git 支持的记忆变更。

nanobot 的记忆基于一个简单的信念：记忆应该有生命感，但不应该混乱无序。

好的记忆不是一堆笔记的堆砌，而是一种安静的注意力系统。它会注意到值得保留的内容，让不再需要聚焦的东西悄然退场，将经历过的体验转化为平和、持久且有用的东西。

这就是 nanobot 中记忆的形态。

## 设计

nanobot 不把记忆当作一个大文件来处理。

它将记忆分层，因为不同类型的记忆值得使用不同的工具：

- `session.messages` 承载正在进行中的短期对话。
- `memory/history.jsonl` 是不断累积、由过去对话压缩而成的归档。
- `SOUL.md`、`USER.md` 和 `memory/MEMORY.md` 是持久化的知识文件。
- `GitStore` 记录这些持久化文件随时间的变化。

这让系统在当下保持轻盈，在长期具备反思能力。

## 流程

记忆在 nanobot 中分两个阶段流动。

### 阶段 1：整合器（Consolidator）

当对话增长到足以对上下文窗口造成压力时，nanobot 并不会试图永远地承载每一条旧消息。

相反，`Consolidator` 会对最早、可以安全处理的一部分对话进行总结，并把总结追加到 `memory/history.jsonl`。

这个文件是：

- 只追加（append-only）
- 基于游标（cursor-based）
- 优先适合机器消费，其次便于人工检查

每一行都是一个 JSON 对象：

```json
{"cursor": 42, "timestamp": "2026-04-03 00:02", "content": "- User prefers dark mode\n- Decided to use PostgreSQL"}
```

它不是最终的记忆，而是最终记忆得以成形的素材。

### 阶段 2：Dream

`Dream` 是更慢、更深思熟虑的一层。默认情况下它按 cron 调度运行，也可以手动触发。

Dream 会读取：

- `memory/history.jsonl` 中的新条目
- 当前的 `SOUL.md`
- 当前的 `USER.md`
- 当前的 `memory/MEMORY.md`

然后在一次运行中对长期文件进行外科手术式的修改——不是把所有东西重写一遍，而是做出最小、诚实的改动，让记忆保持连贯。

正因如此，nanobot 的记忆不仅是归档式的，也是解释性的。

## 文件

```text
workspace/
├── SOUL.md              # 机器人长期的声音和交流风格
├── USER.md              # 关于用户的稳定知识
├── prompts/
│   ├── README.md        # 关于记忆引导文件的说明
│   └── dream.md         # 关于 Dream 如何组织记忆的可选说明
└── memory/
    ├── MEMORY.md        # 项目事实、决策以及持久化上下文
    ├── history.jsonl    # 只追加的历史摘要
    ├── .cursor          # Consolidator 的写入游标
    ├── .dream_cursor    # Dream 的消费游标
    └── .git/            # 长期记忆文件的版本历史
```

这些文件扮演不同角色：

- `SOUL.md` 记住 nanobot 应该有的声音。
- `USER.md` 记住用户是谁以及他们的偏好。
- `MEMORY.md` 记住关于工作本身仍然为真的事情。
- `history.jsonl` 记住走到这里的过程。

## 为什么使用 `history.jsonl`

旧的 `HISTORY.md` 格式便于随意阅读，但作为运行时底座过于脆弱。

`history.jsonl` 给 nanobot 带来了：

- 稳定的增量游标
- 更安全的机器解析
- 更容易批处理
- 更清晰的迁移和压实
- 原始历史和整理后知识之间更清晰的边界

你仍然可以用熟悉的工具搜索它：

```bash
# grep
grep -i "keyword" memory/history.jsonl

# jq
cat memory/history.jsonl | jq -r 'select(.content | test("keyword"; "i")) | .content' | tail -20

# Python
python -c "import json; [print(json.loads(l).get('content','')) for l in open('memory/history.jsonl','r',encoding='utf-8') if l.strip() and 'keyword' in l.lower()][-20:]"
```

其中的区别既是哲学上的，也是技术上的：

- `history.jsonl` 是关于结构的
- `SOUL.md`、`USER.md` 和 `MEMORY.md` 是关于意义的

## 命令

记忆并没有隐藏在幕后，用户可以检查并引导它。

| 命令 | 作用 |
|---------|--------------|
| `/dream` | 立即运行 Dream |
| `/dream-log` | 显示最近一次 Dream 记忆变更 |
| `/dream-log <sha>` | 显示某次特定的 Dream 变更 |
| `/dream-restore` | 列出最近的 Dream 记忆版本 |
| `/dream-restore <sha>` | 将记忆恢复到某次特定变更之前的状态 |
| `/dream-prompt` | 显示当前用于引导 Dream 记忆的提示 |
| `/dream-prompt init` | 在 `prompts/dream.md` 创建一个可编辑的 Dream 记忆引导 |

之所以存在这些命令，是因为：自动化记忆很强大，但用户始终应该保留检查、理解和恢复它的权利。

## 版本化的记忆

当 Dream 修改了长期记忆文件后，nanobot 可以用 `GitStore` 记录这次变更。

这让记忆本身也有了历史：

- 你可以检查发生了什么变化
- 你可以对比不同版本
- 你可以恢复到之前的状态

它把记忆从一次静默的变更转变为一个可审计的过程。

## 引导 Dream

Dream 使用 nanobot 内置的记忆指令来决定保留、更新还是遗忘。大多数用户可以不用管这个。

如果某个工作区需要不同的记忆风格，可以创建一个可编辑的引导文件：

```text
/dream-prompt init
```

这会创建：

```text
workspace/prompts/dream.md
```

用普通 Markdown 编辑该文件。当它有内容时，Dream 会在阅读最新对话历史之前，为该工作区遵循这份引导。你不需要把历史粘贴到文件里；Dream 会自动加入当前的 `## Conversation History` 块。

要恢复到 nanobot 的默认行为，删除 `prompts/dream.md` 或将其留空。

每个工作区都有自己的引导。修改该文件不会影响其他 nanobot 工作区。

## 配置

Dream 在 `agents.defaults.dream` 下配置：

```json
{
  "agents": {
    "defaults": {
      "dream": {
        "intervalH": 2,
        "modelOverride": null,
        "maxBatchSize": 20,
        "maxIterations": 10
      }
    }
  }
}
```

| 字段 | 含义 |
|-------|---------|
| `intervalH` | Dream 的运行频率，单位为小时 |
| `cron` | Cron 表达式覆盖（优先级高于 `intervalH`） |
| `modelOverride` | 可选的 Dream 专用模型覆盖 *（待实现）* |
| `maxBatchSize` | *（已弃用，不再使用）* |
| `maxIterations` | *（已弃用，不再使用）* |

实践中：

- `intervalH` 是配置 Dream 频率的常规方式。内部按 `every` 调度运行。
- 设置了 `cron` 会覆盖 `intervalH`，允许使用精确的 cron 表达式（例如 `0 */4 * * *`）。
- `modelOverride` 保留给未来的版本。目前 Dream 使用与主智能体相同的模型。
- `maxBatchSize` 和 `maxIterations` 为了配置兼容而保留，但不再影响行为。

## 实际效果

在日常使用中，这意味着：

- 对话可以保持迅捷，而不必背负无限的上下文
- 持久化的事实会随着时间变得更清晰，而不是更嘈杂
- 用户在需要时可以检查和恢复记忆

记忆不应该像一个垃圾堆，而应该像一种连续性。

这是本设计想要守护的东西。
