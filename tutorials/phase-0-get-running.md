# 阶段 0：跑起来 + 建立仪表（2–3 天）

## 目标

不是读懂，而是**让系统在你手里动起来，建立反馈回路**。之后每学一个概念，都能立刻在运行的系统里验证。

## 动作

### 1. 构建并启动

```sh
pnpm install                      # pnpm workspaces，node ^22.19 || >=24
pnpm run build                    # 理解 src → lib 的「产物平面」
pnpm dsh --profile web            # 起 Web UI，命令会打印 URL
```

在 Web UI：**Settings → Models** 填 DeepSeek API key → **Choose workspace** 选你启动 `dsh` 的目录 → 发一个任务（如「总结这个仓库并识别主要包」）。

### 2. 关键动作：打印运行时地图

```sh
pnpm dsh --profile web --dump-config
```

**把这行输出保存下来，它是你的「运行时地图」。** 它打印出你机器实际启动的插件树——之后每学一个包，就回来看它在树里的位置和 `config` 字段。

### 3. 理解两个「平面」（重要概念）

- **源码平面（source plane）**：workspace 导入经 tsconfig `paths` 解析到 `src`，静态门禁与测试跑在干净树上。
- **产物平面（artifact plane）**：构建出的 `lib/`（tsc 出 types，tsdown 打包运行时）。

**二者永不混用**（见 `docs/development.md#typescript-project-layout`）。这解释了为什么 `pnpm run build` 是理解构建产物的一环。

### 4. 认识「应用启动」的强约束

支持的 Node 应用**只能通过命名 `dsh` profile 启动**：

```sh
dsh --profile web         # 或 dsh web
dsh --profile headless
dsh --profile sdk
dsh --profile sdk-minimal
dsh --profile acp
```

包里的 bin、demo、公开 SDK 的 argv 逃逸都被**明文禁止**——`scripts/verify-application-entrypoints.ts` 会拒绝任何绕过 `dsh` 的 Node 应用路径。**这条规则决定了你的企业平台不能 fork 出新可执行文件。**

## 阅读（不求全懂，通读一遍）

| 文件 | 作用 |
|---|---|
| 根 `README.md` | 项目定位、运行方式 |
| `docs/user/guide/index.md` | Web UI 上手 |
| `docs/architecture.md` | 总架构（165 行，先通读，标记不懂处） |

## 检查点

- [ ] `pnpm dsh --profile web` 成功起服务，能跑完一个真实任务
- [ ] `--dump-config` 输出已保存
- [ ] 能说清 source plane / artifact plane 的区别
- [ ] 能说出为什么不能 fork 出新可执行文件

## 产出物

`--dump-config` 运行时地图（保存为笔记，后续每阶段回填）。
