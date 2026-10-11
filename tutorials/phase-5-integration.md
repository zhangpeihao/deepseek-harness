# 阶段 5：集成边界 SDK & API Gateway（2 周）

## 目标

搞清**你的 Go/Python 控制面如何驱动 DSH**——这是企业架构的接缝。

## 主角：SDK 家族

`packages/sdk/` 让另一个进程通过 **newline-delimited JSON-RPC** 驱动一个完整的 DSH 运行时。

| 包 | 角色 |
|---|---|
| `protocol/` | 线协议：nd-JSON-RPC 传输 + 命名的 request / result / notification 类型 |
| `client/` | TypeScript 客户端：spawn 一个运行时子进程，经高层与协议层 API 驱动 agent turn |
| `server/` | `jsonrpc` 插件：经 stdio 服务外部 SDK 客户端 |

**能力**：客户端可打开 session、发 prompt，并观察 session 事件、agent 状态变化、subagent 完成。

## Python SDK（你的天然入口）

`python/` 下两个包：

| 目录 | Dist / module | 角色 |
|---|---|---|
| `sdk` | `deepseek-harness-sdk` / `deepseek_harness` | 高层 turns API + 低层 JSON-RPC client |
| `sdk-runtime` | `deepseek-harness-runtime-bin` / `deepseek_harness_runtime` | 打包的 `dsh` CLI 可执行文件 + 原生 sidecar |

**关键行为**：

- SDK 默认启动匹配的打包 `dsh --profile sdk` 运行时，除非调用方另选 `dsh` 或 profile
- runnable 最小示例选 `sdk-minimal` profile；同一运行时也打包 `dsh web` 及其前端资产
- **每次启动都要求显式选择 Harness home；Python 绝不静默读 `~/.dsh`**（企业多租户的关键）

Python SDK 与 TS 客户端**用同一协议**。想了解 runtime 选择、profile、patch、外部插件管理，读 `python/sdk/README.md` 与 `python/sdk-runtime/README.md`。

## API Gateway

`docs/api-gateway.md` + `packages/api/`——远程 BFF（Backend-For-Frontend）。当你要把 DSH 部署为多前端/公网服务时读它。配套 `docs/user/guide/public-deployments.md`（反向代理部署 Web UI）。

## 关键实验

用 Python SDK 起一个 `dsh --profile sdk` 子进程，编程式跑一个 session，**把事件流打到你自己（Go/Python）的日志系统**。

`python/sdk/examples/minimal.py` 是起点。

## 三种部署形态（按场景选 profile）

| 形态 | profile | 用途 |
|---|---|---|
| Web | `web` | 有 UI 的交互式产品 |
| Headless | `headless` | 无 server 的一次性任务 |
| SDK | `sdk` / `sdk-minimal` | 被你的控制面驱动 |

## 检查点

- [ ] 能用 Python SDK 起子进程、跑完一个 session
- [ ] 事件流已采集进你的日志系统
- [ ] 能解释「显式选 Harness home」对多租户的意义
- [ ] 能说出 TS client 与 Python SDK 共享的协议边界
- [ ] 读懂了 `api-gateway.md` 的 BFF 角色

## 产出物

Python SDK 驱动的端到端 session 采集。
