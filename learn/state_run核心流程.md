# `_state_run` 核心流程详解

> 源码位置：
> - `nanobot/agent/loop.py`（`_state_run` / `_run_agent_loop`）
> - `nanobot/agent/runner.py`（`AgentRunner.run` / `_run_core`）

## 分层调用结构

```
_state_run (loop.py:1635)                 状态机入口，标记运行开始、回填结果
  └─ _run_agent_loop (loop.py:760)        构造运行环境、绑定上下文、组装 spec
      └─ AgentRunner.run (runner.py:274)  hook 包装 + 异常兜底
          └─ _run_core (runner.py:338)    真正的多轮迭代循环
```

---

## 一、`_state_run`（状态机入口）

职责最薄，只做三件事：

1. 标记 `visible_run_started_at`，通知 runtime events 状态变为 `"running"`
2. 调 `_run_agent_loop(...)`，传入 BUILD 阶段产出的 `initial_messages`、runtime、tools、各类回调
3. 把返回的 `(final_content, tools_used, all_messages, stop_reason, had_injections)` 回填到 `TurnContext`，再调 `turn_continuation.maybe_continue_turn(ctx)` 判断是否续跑

---

## 二、`_run_agent_loop`（环境装配）

不碰对话内容，只负责"把一个干净的 spec 喂给 runner"。

**定义两个关键内部回调：**

| 回调 | 作用 |
|---|---|
| `_checkpoint(payload)` | 把当前对话快照写入 session，支持崩溃恢复 |
| `_drain_pending()` | 从 `pending_queue` 拉取后续消息（子 agent 结果、用户追加输入），作为注入消息塞回对话；无消息但有运行中子 agent 时阻塞等待（最多 300s） |

**绑定三个上下文 token**（`finally` 中 reset）：

- `bind_file_states` — 文件状态追踪
- `bind_request_context` — 请求级上下文（channel、chat_id、workspace、runtime）
- `bind_workspace_scope` — 工作区作用域

**持续目标（sustained goal）：** `_goal_continue()` 在 session metadata 存在目标时生成"请继续工作"提示消息，驱动长任务。

**组装 `AgentRunSpec` 关键参数：**

- `max_iterations`、`concurrent_tools=True`（工具并发）
- `checkpoint_callback`、`injection_callback`
- `goal_active_predicate`、`goal_continue_message`
- `llm_timeout_s`（由 session metadata 动态计算，持续目标允许更长）
- `finalize_on_max_iterations`（迭代耗尽时是否做无工具收尾请求）

**后处理：** `max_iterations` 时把兜底内容推给流式通道；`error` 时记日志。

---

## 三、`AgentRunner.run`（hook 包装）

用 try/except/finally 包住 `_run_core`，保证 hook 完整执行：

```
before_run → _run_core → after_run / on_error → on_finally
```

- `CancelledError` → `stop_reason="cancelled"` 后 re-raise
- 其他异常 → `on_error` 后 re-raise
- `finally` 里调 `on_finally`（异常时容错）

---

## 四、`_run_core`（真正的迭代循环）

核心结构：`for iteration in range(max_iterations)`，每轮做 **准备上下文 → 调用 LLM → 分支处理**。

### 每轮迭代详细步骤

**1. 上下文治理**（`context_governor.prepare_for_model`）

对 messages 副本做修复/压缩（剥离占位 assistant、修复畸形 tool_calls、丢弃孤儿 tool 结果、补全缺失结果），**不动原始 messages**，保证后续保存的 append 边界不变。治理失败时降级为最小修复。

**2. 调用 LLM**（`_request_model`）

发请求，拿到 `response`（含 content、tool_calls、reasoning、usage、finish_reason）。

**3. 分支 A — 需要执行工具**（`response.should_execute_tools`）

```
checkpoint(awaiting_tools)
  → before_execute_tools
  → _execute_tools（并发执行，_partition_tool_batches 分批）
  → 每个 tool 结果 normalize 后 append 到 messages
  → checkpoint(tools_completed)
  → drain 注入（工具完成后可能有子 agent 结果到达）
  → continue 下一轮
```

致命工具错误 → `stop_reason="tool_error"`，除非有注入否则 break。

**4. 分支 B — 无需工具（最终响应路径）**

按顺序检查：

- **空内容重试**：连续空响应最多 `_MAX_EMPTY_RETRIES` 次，超过则做一次 `_request_finalization_retry`
- **length 截断恢复**：`finish_reason="length"` 时追加恢复消息继续生成（最多 `_MAX_LENGTH_RECOVERIES` 次）
- **注入检查**（最终响应前）：drain pending 队列，有注入则保持流式不结束，继续循环
- **错误处理**：`finish_reason="error"` → 区分欠费（`_ARREARAGE_ERROR_MESSAGE`）/ 普通错误
- **正常完成**：append assistant message、checkpoint(final_response)、`break`

**5. 循环耗尽**（`for...else`）

`max_iterations` 用完未拿到最终响应：

- 最后再 drain 一次注入
- `finalize_on_max_iterations=True` → 无工具收尾请求（`_try_finalize_after_max_iterations`）
- 否则用兜底消息（`_max_iterations_fallback`）

### 返回值 `AgentRunResult`

| 字段 | 说明 |
|---|---|
| `final_content` | 最终文本（可能为 None） |
| `messages` | 完整对话历史（含本轮所有 append） |
| `tools_used` | 成功执行的工具名列表 |
| `usage` | 累计 token 用量 |
| `stop_reason` | completed / tool_error / error / empty_final_response / max_iterations / cancelled |
| `error` | 错误文本（无则 None） |
| `tool_events` | 每个工具的执行事件 |
| `had_injections` | 本轮是否消费了注入消息 |

---

## 五、关键设计点

- **工具并发**：`concurrent_tools=True`，一轮里的多个 tool_call 经 `_partition_tool_batches` 分批后用 `asyncio.gather` 并发
- **三类注入检查点**：工具执行后、最终响应前、错误/空响应后——保证 pending 消息不丢失、按序消费
- **checkpoint 四阶段**：`awaiting_tools` / `tools_completed` / `final_response` / 错误，支持崩溃恢复
- **上下文治理与持久化解耦**：给模型的 messages 可被压缩/修复，但写回 session 的原始 messages 保持 append-only
- **持续目标驱动**：`goal_active_predicate` + `goal_continue_message` 在循环中持续注入"请继续"提示

---

## 六、完整流程图

### 6.1 顶层分层调用

```mermaid
flowchart TD
    START(["TurnContext 进入 _state_run"]) --> MARK["标记 visible_run_started_at<br/>通知 runtime: running"]
    MARK --> ENV["_run_agent_loop<br/>构造运行环境 + 组装 AgentRunSpec"]
    ENV --> HOOK["AgentRunner.run<br/>before_run hook"]
    HOOK --> CORE["_run_core<br/>多轮迭代循环"]
    CORE --> POST["_run_agent_loop 后处理<br/>max_iter 兜底 / error 日志"]
    POST --> FILL["回填 TurnContext<br/>final_content / tools_used /<br/>all_messages / stop_reason / had_injections"]
    FILL --> CONT["turn_continuation.maybe_continue_turn"]
    CONT --> DONE(["返回 ok → 进入 SAVE 状态"])

    style START fill:#4a90d9,color:#fff
    style DONE fill:#7ed957,color:#fff
    style CORE fill:#f5a623,color:#fff
```

### 6.2 `_run_agent_loop` 环境装配

```mermaid
flowchart TD
    A(["进入 _run_agent_loop"]) --> B["定义 _checkpoint 回调<br/>写入 session 快照"]
    A --> C["定义 _drain_pending 回调<br/>从 pending_queue 拉取注入消息<br/>无消息+子agent运行中则阻塞≤300s"]
    B --> D["workspace_scopes.for_turn<br/>解析工作区路径"]
    C --> D
    D --> E["构造 RequestContext<br/>绑定到 ContextVar"]
    E --> F["绑定三个上下文 token<br/>file_states / request_context / workspace_scope"]
    F --> G["_goal_continue 闭包<br/>持续目标提示消息生成"]
    G --> H["组装 AgentRunSpec"]
    H --> I["runner.run(spec)"]
    I --> J{"stop_reason?"}
    J -->|"max_iterations"| K["推兜底内容到流式通道"]
    J -->|"error"| L["logger.error 记录"]
    J -->|"其他"| M(["返回 5 元组"])
    K --> M
    L --> M

    style A fill:#4a90d9,color:#fff
    style M fill:#7ed957,color:#fff
    style I fill:#f5a623,color:#fff
```

### 6.3 `_run_core` 迭代主循环（核心）

```mermaid
flowchart TD
    LOOP(["for iteration in range max_iterations"])

    LOOP --> GOV["context_governor.prepare_for_model<br/>修复/压缩 messages 副本<br/>不改动原始 messages"]
    GOV --> GOVOK{"治理成功?"}
    GOVOK -->|"否"| MINIMAL["降级最小修复<br/>strip 占位/畸形/孤儿 + 补全"]
    GOVOK -->|"是"| BI["hook.before_iteration"]
    MINIMAL --> BI
    BI --> REQ["_request_model<br/>调用 LLM"]
    REQ --> PARSE["解析响应<br/>extract_reasoning<br/>accumulate usage"]
    PARSE --> EMIT{"有 reasoning<br/>且未流式过?"}
    EMIT -->|"是"| ER["emit_reasoning + emit_reasoning_end"]
    EMIT -->|"否"| CHKTOOLS{"should_execute_tools?"}
    ER --> CHKTOOLS

    %% ===== 分支 A: 工具执行 =====
    CHKTOOLS -->|"是"| CPA["checkpoint(awaiting_tools)"]
    CPA --> SETOOL["hook.before_execute_tools"]
    SETOOL --> EXE["_execute_tools<br/>_partition_tool_batches 分批<br/>concurrent → asyncio.gather<br/>否则串行"]
    EXE --> EXERES["收集 results / events / fatal_error"]
    EXERES --> NORM["normalize_tool_result<br/>append tool messages 到 messages"]
    NORM --> CPB["checkpoint(tools_completed)"]
    CPB --> DRAIN1["_try_drain_injections<br/>工具完成后检查注入"]
    DRAIN1 --> DRAIN1OK{"有注入?"}
    DRAIN1OK -->|"是"| INJ1["had_injections=True<br/>continue 下一轮"]
    DRAIN1OK -->|"否"| CONT1["hook.after_iteration<br/>continue"]
    INJ1 --> LOOP
    CONT1 --> LOOP

    %% ===== 工具致命错误 =====
    EXERES --> FATAL{"fatal_error?"}
    FATAL -->|"是"| TE["stop_reason=tool_error<br/>append final_message"]
    TE --> DRAIN2["_try_drain_injections"]
    DRAIN2 --> DRAIN2OK{"有注入?"}
    DRAIN2OK -->|"是"| INJ2["had_injections=True<br/>continue"]
    DRAIN2OK -->|"否"| BREAK1(["break"])
    INJ2 --> LOOP

    %% ===== 分支 B: 最终响应路径 =====
    CHKTOOLS -->|"否"| FINREASON{"finish_reason?"}

    FINREASON -->|"有 tool_calls 但不应执行"| WARN["警告并忽略 tool_calls"]
    WARN --> CLEAN["finalize_content"]
    FINREASON -->|"正常"| CLEAN

    CLEAN --> BLANK{"content 为空?"}
    BLANK -->|"是"| EMPTYRT{"空响应重试<br/>< _MAX_EMPTY_RETRIES?"}
    EMPTYRT -->|"是"| RTY["stream_end(resuming=False)<br/>after_iteration<br/>continue"]
    RTY --> LOOP
    EMPTYRT -->|"否"| FINRETRY["_request_finalization_retry<br/>无工具收尾请求"]
    FINRETRY --> CLEAN2["重新 finalize_content"]
    CLEAN2 --> LENCHK

    BLANK -->|"否"| LENCHK{"finish_reason=length<br/>且恢复次数不超限?"}
    LENCHK -->|"是"| LENREC["append 截断内容<br/>append length_recovery_message<br/>stream_end(resuming=True)<br/>continue"]
    LENREC --> LOOP
    LENCHK -->|"否"| INJCHK

    %% ===== 最终响应前注入检查 =====
    INJCHK["_try_drain_injections<br/>allow_goal_continue=True<br/>检查 pending / 持续目标"]
    INJCHK --> INJOK{"should_continue?"}
    INJOK -->|"是"| INJ3["had_injections=True<br/>stream_end(resuming=True)<br/>continue"]
    INJ3 --> LOOP
    INJOK -->|"否"| STREND["stream_end(resuming=False)"]

    STREND --> ERRCHK{"finish_reason<br/>== error?"}
    ERRCHK -->|"是"| ARREAR{"欠费?"}
    ARREAR -->|"是"| ARRMSG["final=欠费提示"]
    ARREAR -->|"否"| ERRMSG["final=clean 或 error_message"]
    ARRMSG --> ERRINJ["_try_drain_injections"]
    ERRMSG --> ERRINJ
    ERRINJ --> ERRINJOK{"有注入?"}
    ERRINJOK -->|"是"| INJ4["had_injections=True<br/>continue"]
    ERRINJOK -->|"否"| BREAK2(["break"])
    INJ4 --> LOOP

    ERRCHK -->|"否"| FINALBLANK{"content 仍为空?"}
    FINALBLANK -->|"是"| EMPTYFINAL["final=空响应提示<br/>stop_reason=empty_final_response"]
    EMPTYFINAL --> EMPTYINJ["_try_drain_injections"]
    EMPTYINJ --> EMPTYINJOK{"有注入?"}
    EMPTYINJOK -->|"是"| INJ5["had_injections=True<br/>continue"]
    EMPTYINJOK -->|"否"| BREAK3(["break"])
    INJ5 --> LOOP

    FINALBLANK -->|"否"| NORMAL["append assistant_message<br/>checkpoint(final_response)"]
    NORMAL --> SETFINAL["final_content=clean<br/>stop_reason=completed"]
    SETFINAL --> AIT["hook.after_iteration"]
    AIT --> BREAK4(["break"])

    %% ===== for...else 循环耗尽 =====
    LOOP -.->|"max_iterations 用完<br/>未 break"| ELSE["for...else 分支"]
    ELSE --> DRAINMAX["_try_drain_injections<br/>最后一次注入检查"]
    DRAINMAX --> FINALIZE{"finalize_on<br/>_max_iterations?"}
    FINALIZE -->|"是"| TRYFIN["_try_finalize_after_max_iterations<br/>无工具收尾请求"]
    FINALIZE -->|"否"| FALLBACK["_max_iterations_fallback<br/>兜底消息"]
    TRYFIN --> RET

    BREAK1 --> RET(["返回 AgentRunResult"])
    BREAK2 --> RET
    BREAK3 --> RET
    BREAK4 --> RET
    FALLBACK --> RET

    style LOOP fill:#4a90d9,color:#fff
    style RET fill:#7ed957,color:#fff
    style REQ fill:#f5a623,color:#fff
    style EXE fill:#bd10e0,color:#fff
    style BREAK1 fill:#e57373,color:#fff
    style BREAK2 fill:#e57373,color:#fff
    style BREAK3 fill:#e57373,color:#fff
    style BREAK4 fill:#e57373,color:#fff
```

### 6.4 `_execute_tools` 工具执行细节

```mermaid
flowchart TD
    A(["_execute_tools 入参<br/>tool_calls 列表"]) --> B["_partition_tool_batches<br/>按依赖/冲突分批"]
    B --> C{"遍历每个 batch"}
    C --> D{"concurrent_tools<br/>且 batch > 1?"}
    D -->|"是"| E["asyncio.gather 并发执行<br/>每个 tool_call → _run_tool"]
    D -->|"否"| F["串行执行<br/>每个 tool_call → _run_tool"]
    E --> G["合并 results / events / fatal_error"]
    F --> G
    G --> H{"还有 batch?"}
    H -->|"是"| C
    H -->|"否"| I(["返回<br/>results, events, fatal_error"])

    style A fill:#4a90d9,color:#fff
    style I fill:#7ed957,color:#fff
    style E fill:#bd10e0,color:#fff
```

### 6.5 `_run_tool` 单个工具执行

```mermaid
flowchart TD
    A(["_run_tool 入参<br/>tool_call"]) --> B{"repeated_external<br/>_lookup_error?"}
    B -->|"是"| C["返回错误提示 + hint<br/>fail_on_tool_error → fatal"]
    B -->|"否"| D["prepare_call 校验参数"]
    D --> E{"prep_error?"}
    E -->|"是"| F["_classify_violation<br/>工作区违规分类"]
    F --> G{"handled?"}
    G -->|"是"| H["返回 handled 结果"]
    G -->|"否"| I["返回 prep_error + hint<br/>fail_on_tool_error → fatal"]
    E -->|"否"| J["hook.before_execute_tool"]
    J --> K{"tool 对象存在?"}
    K -->|"是"| L["tool.execute(**params)"]
    K -->|"否"| M["tools.execute(name, params)"]
    L --> N{"异常?"}
    M --> N
    N -->|"CancelledError"| RAISE(["re-raise"])
    N -->|"其他异常"| O["hook.on_execute_tool_error<br/>记录 error event"]
    O --> P["返回错误 payload<br/>fail_on_tool_error → fatal"]
    N -->|"无异常"| Q["hook.on_execute_tool_success<br/>记录 ok event"]
    Q --> R(["返回 result, event, None"])

    style A fill:#4a90d9,color:#fff
    style R fill:#7ed957,color:#fff
    style RAISE fill:#e57373,color:#fff
    style P fill:#e57373,color:#fff
```

### 6.6 `_try_drain_injections` 注入检查机制

```mermaid
flowchart TD
    A(["_try_drain_injections<br/>phase / allow_goal_continue"]) --> B{"injection_cycles<br/>< _MAX_INJECTION_CYCLES?"}
    B -->|"是"| C["_drain_injections<br/>调 injection_callback<br/>拉取 pending 消息<br/>上限 _MAX_INJECTIONS_PER_TURN"]
    B -->|"否"| D["injections = []"]
    C --> E{"拿到真实注入?"}
    E -->|"否<br/>且 allow_goal_continue<br/>且 assistant_message 非空"| F{"goal_active<br/>_predicate()?"}
    E -->|"是"| G["real_injection=True"]
    F -->|"是"| H["注入 goal_continue_message<br/>(非真实注入，不增 cycles)"]
    F -->|"否"| I(["返回 False, cycles<br/>无注入，不 continue"])
    D --> I
    H --> J
    G --> J["append assistant_message<br/>+ checkpoint(final_response)"]
    J --> K["_append_injected_messages<br/>注入消息追加到 messages"]
    K --> L{"real_injection?"}
    L -->|"是"| M["cycles += 1<br/>log:Injected N messages"]
    L -->|"否"| N["log:持续目标续跑"]
    M --> O(["返回 True, cycles<br/>通知 caller continue"])
    N --> O

    style A fill:#4a90d9,color:#fff
    style I fill:#e57373,color:#fff
    style O fill:#7ed957,color:#fff
```

### 6.7 运行时崩溃恢复（checkpoint 四阶段）

```mermaid
flowchart LR
    subgraph 正常流程
        A1["awaiting_tools<br/>已调LLM待执行工具"] --> A2["tools_completed<br/>工具执行完毕"]
        A2 --> A3["final_response<br/>最终响应已生成"]
    end

    subgraph 异常阶段
        B1["tool_error<br/>工具致命错误"]
        B2["error<br/>LLM 返回错误"]
    end

    A1 -.->|"崩溃后 _restore_runtime_checkpoint<br/>重放 assistant_message<br/>+ 已完成 tool_results"| A2
    A2 -.->|"崩溃恢复<br/>从已完成结果继续"| A3
    B1 -.->|"崩溃恢复"| A3
    B2 -.->|"崩溃恢复"| A3

    style A1 fill:#f5a623,color:#fff
    style A2 fill:#f5a623,color:#fff
    style A3 fill:#7ed957,color:#fff
    style B1 fill:#e57373,color:#fff
    style B2 fill:#e57373,color:#fff
```
