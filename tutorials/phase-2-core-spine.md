# 阶段 2：核心脊柱 Core Spine（2–3 周）

## 目标

吃透 DSH 的「内脏」：会话日志、工具管线、系统提示组装、模型流、agent loop。

## 阅读

`docs/subsystems/` 下的：`core.md`、`session.md`、`tools.md`、`system-prompt.md`、`llm-streaming.md`；以及 `docs/agent-lifecycle.md`、`docs/tool-execution-pipeline.md`。

## 四大核心 + 一个原语

| 包 | 拥有 | ctx key |
|---|---|---|
| `core/session` | append-only 的 `SessionEvent` 日志 + 内存存储 | `ctx.sessions` |
| `core/system-prompt` | 提示词分段与工具 schema 组装 | `ctx.systemPrompt` |
| `core/tools` | 作用域工具注册表 + 受保护的执行管线 | `ctx.tools` |
| `core/agent` | `Agent` 接口、活体注册表、`agent/*` 事件 | `ctx.agents` |
| `core/agent-loop` | 实现该接口的默认驱动器 | `ctx.agentLoop` |
| `core/scope` | 每 agent 作用域注册原语 | 库，无 key |
| `llm/llm` | 消息与流词汇 + 适配器 seam | `ctx.llm` |

## 必须吃透的机制

### 1. Turn / Step 模型

- **step** = 一次模型请求 + 它触发的工具调用
- **turn** = 零或多个 step；在首个输入被认领前开启，在无所亏欠时关闭

背下 `docs/architecture.md` 里的 turn flow 图——它是整个运行时的心跳：

```text
turn/start
  claim next-step input plus one queued message
  assemble prompt sections + tool schemas
  -> agent/pre-step; project runtime context at fallback
     step/start
     agent/request -> prepareCall
     reconcile system/message
     append entered messages as user/message
     log request/header and request/context
     derive and freeze model history from the log
     stream the bound prepared call -> llm/stream
     assistant/message | assistant/attempt
     tool/call* -> tools/pre-execute -> tools/execute -> tools/post-execute -> tool/result*
     step/end
     tools owe another request, or next-step input arrived -> claim -> next step
  -> agent/turn-stopping
turn/end
```

### 2. 事件三域（选择正确的事件域是多数改动的第一步）

- **Session events**（`turn/*`、`step/*`、`system/message`、`user/message`、`assistant/message`、`assistant/attempt`、`tool/*`）——**持久事实**，进日志、可重放。需要「重启后仍存在」就用它。
- **Agent events**（`agent/*`）——携带活体 `Agent`（inbox、step、status、request、validation、continuation）。用于观察或拦截飞行中的工作。
- **Capability events**（`fs/*`、`tools/*`、`telemetry/*`）——给 seam 挂策略/适配器，**不 import loop**。

**waterfall 事件**：`agent/pre-step`、`agent/request`、`llm/stream`、三个 `tools/*`（监听器必须调 `next()`）。`agent/turn-stopping` 是 serial，没有 `next()`。

### 3. 铁律：模型可见 ⟺ 已记录（Model-visible ⟺ logged）

任何进入模型请求的东西，都必须能从会话日志重建。**新的模型可见输入 = 一个新的 session event。**

**这是你企业级审计的基石**——审计不是后期加的日志，而是框架内建的：`deriveMessages()` 从日志投影出模型历史；fork、resume、transcript、telemetry、persistence 都从这些持久沉淀派生。

## 关键实验（本阶段核心产出）

1. 跑一次任务
2. 用 `session-query`（包 `packages/session-query/`，文档 `docs/subsystems/session-query.md`）导出会话日志
3. **逐条事件对照 turn flow 图**，确认每一步都有对应事件
4. 特别验证：`assistant/message` 是否内嵌了产生其内容的完整紧凑时序流？一次失败重试是否记成了 `assistant/attempt`（不增加模型历史）？

## 检查点

- [ ] 能画出 turn flow，并说出每步产出哪个事件
- [ ] 能区分「持久 session event」与「活体 agent event」
- [ ] 手工回放一次会话日志，验证模型可见 ⟺ 已记录
- [ ] 能说出 `agent/pre-step` 如何决定被接受的输入（重写/拒绝/`startsRequestSeries`）

## 产出物

一次会话日志的逐事件手工回放分析。
