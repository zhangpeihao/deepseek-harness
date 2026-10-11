# 附录：速查表

## A. 常用命令

```sh
# 构建与运行
pnpm install                         # pnpm workspaces, node ^22.19 || >=24
pnpm run build                       # tsc 出 lib/types, tsdown 打包运行时
pnpm dsh --profile headless "task"   # 从源码跑一个任务（需 DEEPSEEK_API_KEY）
pnpm dsh --profile web --dump-config # 打印实际启动的插件树 ← 关键

# 测试与门禁
pnpm run test                        # 单元测试
pnpm run test:coverage               # CI 覆盖率门禁：每文件 100%
pnpm run test:e2e                    # 真实 API，缺 key 自动跳过
pnpm run test:snapshot               # 无 key 录制会话回放
pnpm run typecheck
pnpm run lint
pnpm run doc-sync                    # 文档门禁
pnpm run test:docs                   # 快速文档检查
pnpm run hygiene                     # publint + workspace/package 检查

# 开发
pnpm run dev:web | dev:desktop       # 构建后启动
pnpm run start:web | start:desktop   # 跳过构建
```

## B. 五大出厂 profile

| profile | 形态 | 说明 |
|---|---|---|
| `web` | 有 UI | 浏览器应用 |
| `headless` | 无 server | 一次性 runner |
| `sdk` | 被驱动 | JSON-RPC server |
| `sdk-minimal` | 被驱动 | 独立 bundle，不套 base |
| `acp` | 自动化 | ACP server |

## C. 核心包与 ctx key

| 包 | 拥有 | ctx key |
|---|---|---|
| `core/session` | append-only `SessionEvent` 日志 + 内存存储 | `ctx.sessions` |
| `core/system-prompt` | 提示段与工具 schema 组装 | `ctx.systemPrompt` |
| `core/tools` | 作用域工具注册表 + 执行管线 | `ctx.tools` |
| `core/agent` | `Agent` 接口、活体注册表、`agent/*` 事件 | `ctx.agents` |
| `core/agent-loop` | 默认驱动器 | `ctx.agentLoop` |
| `core/scope` | 每 agent 作用域原语 | 库，无 key |
| `llm/llm` | 消息与流词汇 + 适配器 seam | `ctx.llm` |
| `web/web` | 联网能力（search + fetch） | `ctx.web` |

## D. 事件速查

| 类别 | 事件 | 说明 |
|---|---|---|
| Session（持久） | `turn/*` `step/*` `system/message` `user/message` `assistant/message` `assistant/attempt` `tool/*` | 进日志、可重放 |
| Agent（活体） | `agent/pre-step` `agent/request` `agent/created` `agent/turn-stopping` | 观察/拦截飞行中工作 |
| Capability | `fs/*` `tools/*` `telemetry/*` | 挂策略/适配器，不 import loop |

**waterfall（必须 `next()`）**：`agent/pre-step`、`agent/request`、`llm/stream`、`tools/pre-execute`、`tools/execute`、`tools/post-execute`。
**serial（无 `next()`）**：`agent/turn-stopping`。

## E. 术语（详见 `docs/glossary.md`）

- **seam（能力缝）**：可换能力，三角色 = Service Definition / Provider / Consumer
- **scope（作用域）**：每 agent 注册单元；贡献要么全局要么 scoped，两层扁平，不向下继承
- **turn / step / round**：turn = 一次输入排空；step = 一次模型请求 + 其工具；round = 外层策略迭代
- **scope carrier / scoped dispatch**：关于某 agent 活动的事件用该 agent 的 carrier 派发
- **shadowing**：最具体者胜的名字解析（scoped 覆盖同名 global）
- **lineage**：父子事实作为数据携带，从不影响可见性

## F. 关键文档索引

| 主题 | 文件 |
|---|---|
| 总架构 | `docs/architecture.md` |
| Cordis 入门 | `docs/cordis-primer.md` + `docs/cordis-tutorial/` |
| 服务图 | `docs/capability-seams.md`（生成） |
| 术语 | `docs/glossary.md` |
| Agent 生命周期 | `docs/agent-lifecycle.md` |
| 工具管线 | `docs/tool-execution-pipeline.md` |
| 事件生产/消费 | `docs/event-producer-consumer.md` |
| 测试策略 | `docs/testing.md` |
| 开发布局 | `docs/development.md` |
| 防御模式 | `docs/defensive-patterns.md` |
| 配置目录 | `docs/config-catalog.md`（生成） |
| 子系统详解 | `docs/subsystems/*.md`（中英双语） |
| Cookbook | `docs/cookbook/*.md` |

## G. 避坑速记

1. waterfall 不调 `next()` → 短路整条链
2. 注册漏 disposer → 卸载/HMR 残留
3. 顺序读 358 个包 → 崩溃；改用 `capability-seams.md` 按需下钻
4. 把 `.agents/notes/` 当现状 → 归档已冻结
5. 混用 source/artifact plane → 门禁失败
6. fork 新可执行文件 → `verify-application-entrypoints` 拒绝
7. 硬编码可调项 → 违反「无硬编码 tunables」，应走 Config
8. 新模型可见输入没配 session event → 违反「模型可见 ⟺ 已记录」
