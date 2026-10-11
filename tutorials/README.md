# deepseek-harness 系统学习方案

面向有 Python / Go / C 后端与系统架构经验、目标为「基于 DSH 构建企业级 AI Agent 平台」的学习者。

> 本目录是**个人学习集合**，不属于 `docs/` 文档树。此处的文件不参与仓库的 doc-sync / 双语对齐等门禁，可按需自由修改。权威内容永远以 `docs/`、`packages/*/README.md` 和源码为准。

## 一句话结论

DSH 是 TypeScript / ESM / Cordis 插件式框架，共 **358 个包**。对后端工程师而言，真正的学习曲线**不是 TypeScript 语法，而是 Cordis 的插件/服务/事件模型**。因此本方案的核心是：**先建立 Cordis 心智模型，再沿能力缝（capability seam）切入，最后才碰内核**。

## 学习曲线诊断

| 维度 | 现状 | 结论 |
|---|---|---|
| 语言 | Python / Go / C | TypeScript 是**工具**而非障碍；ESM + 类型体操约 3–5 天适应 |
| 架构 | 有系统架构经验 | 直接命中 DSH 的「服务 / 依赖 / 生命周期」设计，可快速迁移 |
| **框架** | **不熟悉 Cordis** | **唯一真正的陡坡，约占学习时间 40%** |
| 领域 | Agent 运行时细节待补 | 中等陡坡，但有大量现成文档 |

把 Cordis 当作「一个你从没见过的 依赖注入 + 事件总线 + 插件生命周期 框架」来学——它之于 DSH，相当于 Spring 之于 Java 生态、`net/http` 之于 Go 生态。

## 战略定位：扩展，而不是重写

DSH 的设计哲学是 **「没有特权内核可打补丁」**：模型适配器、工具注册表、会话日志、甚至 agent loop 本身都是可替换的插件。

- 要做编排 / 审计 / 多租户 / 网关 → **以插件形式挂在旁边**（不 fork）
- 要替换模型路由 / 工具集 / 存储 → **沿既有扩展点替换**
- 只在 agent loop 层面必须改的行为 → 才动核心（代价高，需同步改 `docs/architecture.md`）

**第一原则**：能用插件 / 事件 seam 解决的，绝不改内核。

## 阶段总览

建议 8–12 周，可按需压缩。每阶段都有可验证的产出物。

| 阶段 | 主题 | 时长 | 产出物 |
|---|---|---|---|
| [0](phase-0-get-running.md) | 跑起来 + 建立仪表 | 2–3 天 | `--dump-config` 运行时地图 |
| [1](phase-1-cordis.md) | Cordis 心智模型 | 1–2 周 | 一个注册工具 + 提示段的自写插件 |
| [2](phase-2-core-spine.md) | 核心脊柱 Core Spine | 2–3 周 | 一次会话日志的逐事件手工回放 |
| [3](phase-3-capability-seams.md) | 能力缝 Capability Seam | 2 周 | 自实现一个 seam provider |
| [4](phase-4-composition.md) | 组合层 Profiles & Bundles | 1–2 周 | 一个自研 profile，堆叠自研 bundle |
| [5](phase-5-integration.md) | 集成边界 SDK & API Gateway | 2 周 | Python SDK 驱动的端到端 session 采集 |
| [6](phase-6-enterprise-platform.md) | 构建企业平台 | 持续 | 企业平台骨架 |
| [附录](appendix.md) | 术语 / 命令 / 包地图 / 避坑 | — | 速查表 |

## 依赖关系

```
0 ──▶ 1 ──▶ 2 ──▶ 3 ──▶ 4 ──▶ 5 ──▶ 6
      (必)  (必)  (主战场) (打包) (集成) (构建)
```

阶段 1 不可跳过。阶段 3 是企业平台的主战场。阶段 6 与前序并行推进。

## 如何使用本目录

1. 按阶段顺序推进；每阶段先读「目标」和「检查点」，再动。
2. 每完成一个检查点，在下方清单打勾。
3. 每个阶段的「产出物」必须真实落地——读代码不算学会，能跑通才算。
4. 遇到「为什么这样设计」，查 `.agents/notes/`（决策档案，非教程）。
5. 遇到术语不明，查 [附录](appendix.md) 或 `docs/glossary.md`。

## 抗知识过载策略（358 个包，别硬啃）

1. **以 `docs/capability-seams.md` 为地图，不以目录树为顺序**——它给了服务图，按需下钻。
2. **每个包先读 `package.json` + `README.md`**，只读当前 seam 相关的。`docs/subsystems/` 每篇都有中英双语。
3. **`.agents/notes/` 是决策档案而非教程**——只在问「为什么」时查，别顺序读。归档的 notes 已冻结，不当现状。
4. **双语文档是同级权威**——中文版可加速阅读，但实现细节以 `packages/*/README.md` 为准。
5. **建立你自己的索引**：把 `seam → 包 → ctx key → 事件名` 做成一页 cheat sheet，学一个补一行。

## 进度清单

```
[ ] 阶段 0  ── 跑通 web profile，保存 --dump-config
[ ] 阶段 1  ── Cordis 七章教程读完，写出第一个插件
[ ] 阶段 2  ── 吃透 turn/step 模型，手工回放一次会话日志
[ ] 阶段 3  ── 沿 shell 模板实现一个 provider
[ ] 阶段 4  ── 组合出自研 profile
[ ] 阶段 5  ── Python SDK 端到端跑通并采集事件
[ ] 阶段 6  ── 企业平台骨架成型
```
