# AgentLoop `_state_*` 状态机设计

> 源码位置：`nanobot/agent/loop.py`

## 整体设计：状态机模式

`AgentLoop` 把一次完整的消息处理回合（Turn）拆成 7 个有序阶段，每个阶段对应一个状态和一个处理方法，由统一的转移表驱动流转。

### 三要素

**1. 状态枚举 `TurnState`**（`loop.py:100`）

```python
class TurnState(Enum):
    RESTORE = auto()
    COMPACT = auto()
    COMMAND = auto()
    BUILD = auto()
    RUN = auto()
    SAVE = auto()
    RESPOND = auto()
    DONE = auto()
```

正常流转顺序：

```
RESTORE → COMPACT → COMMAND → BUILD → RUN → SAVE → RESPOND → DONE
```

**2. 转移表 `_TRANSITIONS`**（`loop.py:238`）

声明式定义 `(当前状态, 事件) → 下一状态` 的映射，集中管理所有流转规则：

```python
_TRANSITIONS: dict[tuple[TurnState, str], TurnState] = {
    (TurnState.RESTORE, "ok"): TurnState.COMPACT,
    (TurnState.COMPACT, "ok"): TurnState.COMMAND,
    (TurnState.COMMAND, "dispatch"): TurnState.BUILD,
    (TurnState.COMMAND, "shortcut"): TurnState.DONE,
    (TurnState.BUILD, "ok"): TurnState.RUN,
    (TurnState.RUN, "ok"): TurnState.SAVE,
    (TurnState.SAVE, "ok"): TurnState.RESPOND,
    (TurnState.RESPOND, "ok"): TurnState.DONE,
}
```

**3. 分发循环**（`loop.py:1418`）

通过**命名约定**（`_state_` + 状态名小写）反射查找 handler，每个 handler 返回一个事件字符串，由转移表决定下一跳：

```python
while ctx.state is not TurnState.DONE:
    handler_name = f"_state_{ctx.state.name.lower()}"  # 约定式命名
    handler = getattr(self, handler_name)
    event = await handler(ctx)
    ctx.state = self._TRANSITIONS.get((ctx.state, event))
```

## 各阶段职责

| 方法 | 状态 | 职责 | 返回事件 |
|---|---|---|---|
| `_state_restore` | RESTORE | 恢复会话检查点、提取媒体附件、持久化消息作用域 | `ok` |
| `_state_compact` | COMPACT | 触发上下文自动压缩，准备摘要 | `ok` |
| `_state_command` | COMMAND | 斜杠命令分发；命中则走 `shortcut` 跳过后续阶段 | `dispatch` / `shortcut` |
| `_state_build` | BUILD | 构建历史、运行时上下文、初始消息、持久化用户消息 | `ok` |
| `_state_run` | RUN | 执行 LLM 多轮对话循环（工具调用、流式响应） | `ok` |
| `_state_save` | SAVE | 保存回合、清理检查点、触发后台记忆整理 | `ok` |
| `_state_respond` | RESPOND | 组装最终出站消息（或抑制响应） | `ok` |

## 设计亮点

- **约定优于配置**：handler 与状态名通过 `_state_xxx` 命名约定绑定，无需注册，新增状态只需加枚举值 + 方法 + 转移规则。
- **声明式转移**：`_TRANSITIONS` 把控制流数据化，可一眼看出全部合法路径，包括 COMMAND 的两分支（`dispatch` 正常走、`shortcut` 命令直达 DONE）。
- **可观测性**：每个状态执行前后都记录 `StateTraceEntry`（含耗时、事件、异常），`ctx.trace` 形成完整执行轨迹，便于调试和性能分析。
- **单一职责**：每个 `_state_*` 只做一件事，输入 `TurnContext`、修改它、返回事件字符串，边界清晰。
- **短路优化**：命令命中时通过 `shortcut` 事件直接跳到 `DONE`，跳过 BUILD/RUN/SAVE 这套重量级流程。

## 总结

这套设计本质上是把一个原本可能写成巨型 `async def process_message` 的过程式流程，拆解成了**可测试、可观测、可扩展的状态机管线**，属于典型的"把隐式控制流显式化"重构。在 agent 框架里处理消息回合这个场景下，是个相当合理的选择。
