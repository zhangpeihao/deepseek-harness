# 阶段 3：能力缝 Capability Seam（2 周）

## 为什么这是企业平台的主战场

企业平台的绝大多数定制，本质都是「换一个 provider」或「挂一个策略消费者」——这些都在 seam 上完成，不碰内核。

## 核心概念

一个 seam = **Service Definition / Service Provider / Consumer** 三角色。

- **Service Definition**：拥有 `ctx.<key>` 与词汇类型的 Cordis `Service`（抽象类如 `ShellExecutor`，或具体注册表如 `WebRuntime`，**绝不是 TypeScript `interface`**）
- **Service Provider**：一个或多个实现
- **Consumer**：inject 该服务的消费者

**seam 是完整能力，永远不是单一角色。** 规范示例是 `packages/shell`：`dsh-shell`（Definition）→ `dsh-bash-local` / `dsh-bash-sandbox`（Providers）→ `dsh-tool-bash`（Consumer）。

## 地图

`docs/capability-seams.md`（692 行，脚本 `scripts/gen-doc-graphs.ts` 生成，**勿手改**）——服务图，标出拥有服务声明的包、已知实现包、直接消费包。**按需下钻，别顺序读。**

## 按企业需求排序的必修 seam

| 企业需求 | seam | Definition → Provider → Consumer |
|---|---|---|
| 模型网关 / 多模型路由 | `llm/llm` | 见 `docs/cookbook/adding-an-llm-adapter.md` |
| 工具治理 / 沙箱执行 | `shell/` | `dsh-shell` → `bash-local`/`bash-sandbox` → `tool-bash` |
| 联网能力 | `web/` | `ctx.web`（search + fetch 双操作单服务） |
| 权限 / 审批 | `interaction/` | 见 `docs/subsystems/approval.md`、`permission-presets.md` |
| 隔离执行 | `sandbox/`、`ssh/`、`e2b/` | — |
| 后台任务 | `jobs/`、`schedule/` | — |
| 可观测性 | `telemetry/` | 见 `docs/subsystems/otel.md` |

### 案例：`ctx.web` 为什么「一个能力两个操作」

search 与 fetch 不共享请求 schema，也被刻意放在同一个 `ctx.web` 中间层：**一个 provider 选择策略拥有者、一套 abort/error 词汇、一个对产品可见的「本 harness 如何上网」配置 API**。代价是服务上并行的 `searchX`/`fetchX` 方法对——这种并行是**故意的，不是漏抽**（见 `docs/subsystems/web.md`）。

**启示**：Provider 注册「能力」（`WebSearchProvider`），**不是工具**；对模型可见的名字、schema、提示引导、呈现都只在唯一的 consumer（`dsh-tool-web`）里。

## 必读 cookbook（照着做一遍）

| 文件 | 学到 |
|---|---|
| `docs/cookbook/adding-a-tool.md` | 设计工具 UI 呈现：Host presenter 保持纯函数，Web card 从原始事件 + 持久结果元数据派生 |
| `docs/cookbook/adding-an-llm-adapter.md` | 模型适配器 seam |
| `docs/cookbook/adding-a-package.md` | 新包结构 |
| `docs/cookbook/adding-a-remote-api.md` | 远程 BFF |
| `docs/cookbook/extension-cookbook.md` | 运行时自修改 |

## 两个必须掌握的约定

1. **显式优于隐式（包边界）**：默认值是拥有方实现里**显式的 `resolve(request): Spec` 步骤**，绝不是 `run()` 里隐藏的 `?? default`。`dsh-shell` 的 request/spec 分离是模板。

2. **无硬编码可调项**：随部署变化的取值是 cordis.yml 里可改的、经校验的 `Config` 字段；`DEFAULT_*` 常量或测试钩子**不算**可配置性。协议常量、外部规范、安全不变量保持不变。

## 检查点

- [ ] 能沿 `shell` seam 的模板（request/spec 分离）实现一个 provider
- [ ] 能在 `capability-seams.md` 里定位你实现的角色
- [ ] 能说清为什么 Provider 注册「能力」而非工具
- [ ] 读懂了 `web` seam 的「并行方法对是故意」的论证

## 产出物

自实现一个 seam provider（照 `shell` 模板）。
