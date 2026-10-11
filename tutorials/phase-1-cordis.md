# 阶段 1：Cordis 心智模型（1–2 周，不可跳过）

## 为什么这是投入产出比最高的一周

Cordis 是 DSH 之下 vendored（内嵌并改过的）插件框架。跳过它直接读 `agent-loop`，你会处处受挫。读完这一周，你能读懂 80% 的包。

## 阅读顺序

1. `docs/cordis-primer.md`（45 行）——五个核心概念
2. `docs/cordis-tutorial/` 七章实战：
   - `01-first-plugin.md`
   - `02-lifecycle-and-effects.md`
   - `03-services.md`
   - `04-events.md`
   - `05-config.md`
   - `06-composition-and-hmr.md`
   - `07-into-the-harness.md`
3. `vendor/README.md` + `vendor/` 源码——**它是改过的内嵌副本，必须读真实源码，别信记忆里的 Cordis**

## 五个核心概念（Cordis In Five Ideas）

- **插件 = 实现 Service 的对象**。可以是带可选 `inject` 和 `apply(ctx)` 字段的函数，也可以是 `Service` 子类。
- **context = 服务的仓库**。服务从 context 认领一个稳定的 `ctx.<key>`（如 `ctx.tools`、`ctx.llm`、`ctx.sessions`）；其他插件靠 key 找到服务，而不是 import 具体实现。
- **靠 `inject` 声明服务依赖**。声明所需服务的插件会等待这些服务就绪，**加载顺序由服务需求表达，而非手写 boot 序列**。
- **类型化事件用于通信**。服务用 TypeScript 声明合并暴露事件名，再按语义用 `emit` / `waterfall` / `parallel` / `serial` / `bail` 派发。
- **注册即可逆副作用**。提示段、工具 schema、适配器、provider、监听器都经 `ctx.effect()` / `ctx.on()` 安装，使 reload 与 teardown 可预测地回卷。

## 事件派发模式（必须记住的表）

| 模式 | 是否 await | 派发顺序 | 有返回值？ |
|---|---|---|---|
| `emit` | 否 | 按注册顺序观察 | 否 |
| `waterfall` | 否 | 按注册顺序观察 | 是 |
| `parallel` | 是 | 所有监听器并行观察 | 否 |
| `serial` | 是 | 按注册顺序 | 是 |
| `bail` | 否 | 按注册顺序，直到有人 bail | 是 |

**派发模式是事件的公开契约**。新事件用 `@mode` 标注，生成目录会校验声明与派发点一致。

## 三个必须内化的反直觉点（后端工程师最易踩坑）

1. **注册必须可逆**：每个 `ctx.effect()` / `ctx.on()` 都要能被 disposer 干净卸载。这是 HMR 和插件卸载的基础。若 teardown 顺序重要，把相关工作放进**同一个 effect**，让释放按预期顺序回卷。

2. **Waterfall ≠ 普通事件监听**：`ctx.waterfall` 是**环绕中间件**。监听器收到 `(...args, next)`，**必须调 `next()` 才委托**，否则短路整条链。值经 `next()` 的返回值传播。这是 DSH 里最多 bug 的地方。
   - 协作式监听器通常改动共享对象再委托；`prepend: true` 只在必须抢在普通注册之前时用。
   - 单决策事件里，短路是设计本身：拥有决策的监听器可不调 `next()` 返回；只做标注/观察的**必须**委托。

3. **依赖靠 `inject` 声明，不靠启动顺序**：你不需要手写 bootstrap 序列。

## 速查：怎么做

- 想**拦截 / 加策略** → 用事件
- 想**直接调能力** → 用服务方法
- 行为归属：工具管线事件 → `ctx.tools`；模型流 → `ctx.llm`；活体 agent 协调 → `ctx.agents`

## 检查点

- [ ] 能自己写一个插件：往 `ctx.tools` 注册一个工具，往系统提示注册一个 section
- [ ] 在 `--dump-config` 里看到自己插件的行
- [ ] 能解释 waterfall 的 `next()` 语义，并写出一个「只观察必须委托」的监听器
- [ ] 能说出 `inject` 如何替代 boot 顺序

## 产出物

一个注册了工具 + 提示段的自写 Cordis 插件。

## 延伸

- `docs/glossary.md` —— 领域术语的**唯一规范定义**
- `docs/capability-seams.md` —— 生成的服务图（阶段 3 的地图）
