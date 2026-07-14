# My 工具

让智能体感知并调整自己的运行时状态——就像问同事"你忙吗？能换个更大的显示器吗？"

## 你为什么需要它

普通的工具让智能体操作外部世界（读/写文件、搜索代码）。但智能体对自己一无所知——它不知道自己运行在哪个模型上、还剩多少次迭代，也不知道消耗了多少 token。

My 工具填补了这个缺口。有了它，智能体可以：

- **知道自己是谁**：我在用哪个模型？我的工作区在哪里？还剩多少次迭代？
- **动态调整**：任务复杂？扩大上下文窗口。简单闲聊？切换到更快的模型。
- **跨轮次记忆**：把笔记存进你的草稿本，在下一轮对话中依然可用。

## 配置

默认启用（只读模式）。智能体可以检查自身状态但不能修改。

```yaml
tools:
  my:
    enable: true       # 默认：true
    allow_set: false   # 默认：false（只读）
```

若允许智能体修改配置（例如切换模型、调整参数），请设置 `tools.my.allow_set: true`。

旧的 `tools.myEnabled` / `tools.mySet` 键会在加载时自动迁移，并在下次 `nanobot onboard` 刷新配置时就地重写。

所有修改仅保存在内存中——重启会恢复默认值。

---

## check —— 检查 "my" 当前状态

不带参数时，返回关键配置概览：

```text
my(action="check")
# → max_iterations: 40
#   context_window_tokens: 200000
#   model: 'anthropic/claude-sonnet-4-6'
#   workspace: PosixPath('/tmp/workspace')
#   provider_retry_mode: 'standard'
#   max_tool_result_chars: 16000
#   _current_iteration: 3
#   _last_usage: {'prompt_tokens': 45000, 'completion_tokens': 8000}
#   Note: prompt_tokens is cumulative across all turns, not current context window occupancy.
```

传入 key 参数可深入查看具体配置：

```text
my(action="check", key="_last_usage.prompt_tokens")
# → 到目前为止我用了多少 prompt token

my(action="check", key="model")
# → 我当前运行在哪个模型上

my(action="check", key="web_config.enable")
# → 网页搜索是否已启用
```

### 你可以用它做什么

| 场景 | 方法 |
|----------|-----|
| "你在用什么模型？" | `check("model")` |
| "当前激活的是哪个模型预设？" | `check("model_preset")` |
| "你还能再调用几次工具？" | `check("max_iterations")` 减去 `check("_current_iteration")` |
| "这次对话用了多少 token？" | `check("_last_usage")` —— 所有轮次累计 |
| "你的工作目录在哪里？" | `check("workspace")` |
| "把你的完整配置给我看看" | `check()` |
| "有没有子智能体在运行？" | `check("subagents")` —— 显示阶段、迭代、耗时、工具事件 |

---

## set —— 运行时调优

修改会立即生效，无需重启。

```text
my(action="set", key="max_iterations", value=80)
# → 将迭代上限从 40 提升到 80

my(action="set", key="model_preset", value="fast")
# → 切换到已配置的模型预设

my(action="set", key="model", value="fast-model")
# → 切换到原始模型并清空当前预设

my(action="set", key="context_window_tokens", value=262144)
# → 为长文档扩大上下文窗口
```

你也可以在自己的草稿本中存储自定义状态：

```text
my(action="set", key="current_project", value="nanobot")
my(action="set", key="user_style_preference", value="concise")
my(action="set", key="task_complexity", value="high")
# → 这些值会保留到下一轮对话
```

### 受保护的参数

以下参数具有类型和范围校验——无效值会被拒绝：

| 参数 | 类型 | 范围 | 用途 |
|-----------|------|-------|---------|
| `max_iterations` | int | 1–100 | 每轮对话的最大工具调用次数 |
| `context_window_tokens` | int | 4,096–1,000,000 | 上下文窗口大小 |
| `model` | str | 非空 | 使用的 LLM 模型 |
| `model_preset` | str | 已配置的预设名称 | 使用的命名预设 |

其他参数（例如 `workspace`、`provider_retry_mode`、`max_tool_result_chars`）可以自由设置，只要值是 JSON 安全的即可。

---

## 实用场景

### "这个任务复杂，我需要更大的空间"

```text
Agent: 这个代码库很大，让我扩大上下文窗口来处理它。
→ my(action="set", key="context_window_tokens", value=262144)
```

### "简单问题，别浪费算力"

```text
Agent: 这是个简单直接的问题，我切换到 fast 预设吧。
→ my(action="set", key="model_preset", value="fast")
```

### "跨轮记住用户偏好"

```text
第一轮：my(action="set", key="user_prefers_concise", value=True)
第二轮：my(action="check", key="user_prefers_concise")
# → True（仍记得用户喜欢简洁的回复）
```

### "自我诊断"

```text
User: "你为什么不搜网页？"
Agent: 让我检查一下 web 配置。
→ my(action="check", key="web_config.enable")
# → False
Agent: 网页搜索被禁用了——请在配置中设置 web.enable: true。
```

### "Token 预算管理"

```text
Agent: 让我看看还剩多少预算。
→ my(action="check", key="_last_usage")
# → {"prompt_tokens": 45000, "completion_tokens": 8000}
Agent: 目前一共用了大约 53k token。我剩下的回答会保持简洁。
```

### "监控子智能体"

```text
Agent: 让我看看后台任务的情况。
→ my(action="check", key="subagents")
# → 2 subagent(s):
#   [task-1] 'Code review'
#     phase: running, iteration: 5, elapsed: 12.3s
#     tools: read(✓), grep(✓)
#     usage: {'prompt_tokens': 8000, 'completion_tokens': 1200}
#   [task-2] 'Write tests'
#     phase: pending, iteration: 0, elapsed: 0.2s
#     tools: none
Agent: 代码评审进展顺利。测试任务还没开始。
```

---

## 安全机制

核心设计原则：**所有修改仅存于内存中。重启后恢复默认值。** 智能体无法造成持久性的破坏。

### 完全禁止（BLOCKED）

无法被检查或修改——完全隐藏：

| 类别 | 属性 | 原因 |
|----------|-----------|--------|
| 核心基础设施 | `bus`、`provider`、`_running` | 修改会导致系统崩溃 |
| 工具注册表 | `tools` | 不允许移除自身的工具 |
| 子系统 | `runner`、`sessions`、`consolidator` 等 | 影响其他用户/会话 |
| 敏感数据 | `_mcp_servers`、`_pending_queues` 等 | 包含凭据和消息路由 |
| 安全边界 | `restrict_to_workspace`、`channels_config` | 绕过它会破坏隔离 |
| Python 内部 | `__class__`、`__dict__` 等 | 防止沙箱逃逸 |

### 只读（仅可 check）

可以检查但不能设置：

| 类别 | 属性 | 原因 |
|----------|-----------|--------|
| 子智能体管理器 | `subagents` | 可观察，但替换会破坏系统 |
| 执行配置 | `exec_config` | 可以检查沙箱/启用状态，但不能修改 |
| Web 配置 | `web_config` | 可以检查启用状态，但不能修改 |
| 迭代计数器 | `_current_iteration` | 仅由 runner 更新 |

### 敏感字段保护

匹配敏感名称的子字段（`api_key`、`password`、`secret`、`token` 等）无论父路径为何都会被禁止检查和设置。这可以防止通过点路径遍历（例如 `web_config.search.api_key`）泄露凭据。
