# 阶段 6：构建企业平台（持续）

## 前提

到这里你已具备全部前置（Cordis / 核心脊柱 / seam / 组合 / 集成）。本阶段把 DSH 的能力映射到企业诉求，并持续构建。

## 企业诉求 → DSH 落点

| 企业诉求 | DSH 落点 | 你的工作 |
|---|---|---|
| 多租户 / 隔离 | `identity/`、`credentials/`、profile 隔离 | 每租户独立 Harness home + `dsh plugin` 安装 |
| 审计 / 合规 | 会话日志（**内建**，见「模型可见 ⟺ 已记录」） + `session-query` 导出 | 采集 `SessionEvent` 流到合规存储 |
| 模型网关 / 路由 | `llm/llm` adapter seam | 实现企业级 provider（审计、限流、计费） |
| 工具治理 | `tools/*` 事件 + `tools.restrict` 作用域限制 | 按 scope 限制工具集 |
| 权限 / 审批 | `interaction/` + approval seam | 定制审批策略 |
| 可观测性 | `telemetry/` + OTel | 接你的监控栈 |
| 部署形态 | `web` / `headless` / `sdk` | 按场景选 profile |
| 后台 / 定时 | `jobs/`、`schedule/` | — |
| 子 agent / 多 agent | `subagent/`、`workflow/`、`docs/subsystems/agent-team.md` | 编排策略 |

## 关键设计原则（回看，避免走偏）

1. **能用插件 / 事件 seam 解决的，绝不改内核。** 改变 `agent-loop` 需要同步更新 `docs/architecture.md`——代价高。
2. **注册即副作用**：你的每个贡献都经 `ctx.effect()` / `ctx.on()`，有 disposer。
3. **模型可见 ⟺ 已记录**：任何新的模型可见输入都要配一个 session event——这是审计内建而非后加的原因。
4. **无硬编码可调项**：随部署变化的取值走 cordis.yml 的 `Config`。
5. **不 fork 新可执行文件**：企业平台 = 自研 profile + bundle + patch。
6. **审计采集点**：`SessionEvent` 持久流是你的合规数据源，`session-query` 是导出工具。

## 建议的平台骨架

```
你的控制面 (Go/Python)
  └─ 经 SDK (JSON-RPC stdio) 驱动 dsh --profile <你的企业 profile>
       ├─ Harness home：每租户一个（隔离）
       ├─ bundle 堆叠：
       │    ├─ dsh-base（共享首层）
       │    ├─ 你的模型网关 provider（llm seam）
       │    ├─ 你的工具治理消费者（tools/* 事件）
       │    └─ 你的审计采集消费者（session 事件 → 你的存储）
       └─ patch 覆盖：按环境/租户注入 config
```

## 贯穿全程的避坑清单

- **waterfall 必须 `next()`**：只观察的监听器不调 `next()` 会短路整条链。
- **注册必须可逆**：漏了 disposer，插件卸载/HMR 会残留。
- **别顺序读 358 个包**：以 `capability-seams.md` 为地图按需下钻。
- **`.agents/notes/` 是历史决策，非现状**：归档的已冻结，别当权威。
- **source plane / artifact plane 别混**：静态门禁跑在 `src`，消费 `lib/` 的门禁要显式声明。
- **改外部可感知的破坏性变更要记 upgrade guide**：见 `.agents/skills/dsh-create-upgrade-guide/`。

## 检查点

- [ ] 企业 profile 成型，含自研 bundle
- [ ] 模型网关 provider 上线（审计 / 限流 / 计费至少其一）
- [ ] 审计事件流进入你的合规存储
- [ ] 多租户隔离（独立 Harness home）验证通过
- [ ] 遥测接入你的监控栈
