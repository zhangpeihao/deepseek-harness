# 阶段 4：组合层 Profiles & Bundles（1–2 周）

## 为什么这是「打包企业平台」的地方

DSH 没有「应用」，只有 **profile + 有序 patch**。你的企业 AI 平台最终就是一个**自研 profile + 你的 bundle 堆叠 + patch 覆盖**。

## 核心概念

- **Profile**：存在 Harness home 里的命名组合。列出它堆叠的 bundle、树外插件、用户的 `cordis.patch.yml`。出厂模板：`web`、`headless`、`sdk`、`sdk-minimal`、`acp`。
- **Bundle**：Cordis config 行 + 它们挂载的代码的分发格式，使插入的内容可被上层 patch。
- 每个都通过自己 `package.json` 的 `dsh` 字段声明：`dsh.profile` 列出 profile 的 bundle，`dsh.bundle` 指向 bundle 的 patch 文件。

## 分层顺序（从底到顶）

```
profile 的有序 bundle
  → profile 的 cordis.patch.yml
  → home patch
  → --patch 覆盖层
```

各层作用于一个空 entry 列表。patch 按 **id** 定位一行、替换其**整个 config**，或插入新行。`preset` patch 在某个声明的 `config.plugins` 内应用同样操作，层优先级相同。

## 出厂 bundle 一览

| Bundle | 作用 |
|---|---|
| `dsh-base` | `web`/`headless`/`sdk`/`acp` 的**共享首层**：模型适配器、工具、持久化、沙箱与审批策略、设置、凭证、遥测 |
| `dsh-web-app` | 加浏览器应用 |
| `dsh-headless` | 加无 server 的一次性 runner |
| `dsh-sdk-app` | 加 SDK JSON-RPC server |
| `dsh-acp-app` | 加自动化专用 ACP server |
| `dsh-sdk-minimal` | **刻意的例外**：单个 bundle 拥有完整显式 SDK 树，不套用 `dsh-base` |

Base 还含 Plugin Manager（供 Web 与 agent 用）。

## 查看你机器的启动树

```sh
dsh --profile web --dump-config
```

**任何打印出的行都能被你自己的 patch 替换。**

## YAML 与 HMR

YAML 控制 HMR：base 启用 config-only 的 `dsh-hmr`；headless、SDK、ACP 禁用；`sdk-minimal` 省略。profile patch 可覆盖这些默认。

**cordis.yml 规则**：`config` 与 entry `disabled` 下允许 `!!js`（**绝不** `!js`）；其他元数据保持字面量，所以条件组合也要用 overlay。

## 强约束（务必记住）

**应用启动只能通过命名 `dsh` profile。** 自定义插件组合是「profile + 有序 patch 文件」，**不是**另一个可执行文件或内联应用树。`scripts/verify-application-entrypoints.ts` 会强制这一点。

## 阅读

- `docs/architecture.md#profiles-and-bundles`
- `packages/boot/app-boot/README.md#profiles`
- `packages/bundle/*/README.md`
- `docs/config-catalog.md`（生成，config 字段总目录）
- `docs/cordis-primer.md#loader-configuration`

## 检查点

- [ ] 能说出 profile / bundle / patch 的层优先级
- [ ] 能用 `--patch` 覆盖一个有 id 的行并验证生效
- [ ] 能解释 `sdk-minimal` 为何不套 `dsh-base`
- [ ] 明白为什么不能靠「另一个可执行文件」组合自定义插件

## 产出物

一个自研 profile，堆叠你的 bundle（可用一个最小 patch 验证）。
