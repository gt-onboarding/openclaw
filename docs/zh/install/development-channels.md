---
title: 开发通道
summary: "stable、beta 和 dev 通道：语义、切换与标记"
read_when:
  - 你想在 stable/beta/dev 之间切换
  - 你正在为预发布版本打标签或发布预发布版本
---

<div id="development-channels">
  # 开发通道
</div>

最后更新：2026-01-21

OpenClaw 提供三个更新通道：

- **stable**：npm dist-tag `latest`。
- **beta**：npm dist-tag `beta`（测试中的构建）。
- **dev**：始终指向 `main`（git）分支最新提交的移动头。npm dist-tag：`dev`（发布时）。

我们先将构建发布到 **beta**，完成测试后，**在不更改版本号的前提下，将验证通过的构建提升到 `latest`** —— 对于 npm 安装而言，dist-tag 是版本选择的唯一依据。

<div id="switching-channels">
  ## 切换通道
</div>

Git checkout 命令：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` 检出最新的匹配标签（通常是同一个标签）。
* `dev` 切换到 `main` 分支并基于上游仓库进行 rebase。

npm/pnpm 全局安装：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

这会通过对应的 npm dist-tag（`latest`、`beta`、`dev`）进行更新。

当你使用 `--channel` **显式**切换通道时，OpenClaw 也会同步切换安装方式：

* `dev` 会确保存在一个 git checkout 工作副本（默认路径为 `~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 覆盖），
  更新该副本，并从该 checkout 安装全局 CLI。
* `stable`/`beta` 会使用匹配的 dist-tag 从 npm 安装。

提示：如果你想同时使用 stable 和 dev，请保留两个代码副本，并将你的 Gateway 指向 stable 的那个。


<div id="plugins-and-channels">
  ## 插件和通道
</div>

当你使用 `openclaw update` 切换通道时，OpenClaw 也会同步插件来源：

- `dev` 会优先使用当前 Git 检出中自带的插件。
- `stable` 和 `beta` 会恢复通过 npm 安装的插件包。

<div id="tagging-best-practices">
  ## 标记最佳实践
</div>

- 为你希望 git 检出落到的发布版本打标签（`vYYYY.M.D` 或 `vYYYY.M.D-<patch>`）。
- 保持标签不可变：绝不要移动或重复使用任何标签。
- npm dist-tags 仍然是 npm 安装的权威来源：
  - `latest` → 稳定版
  - `beta` → 候选构建
  - `dev` → main 分支快照（可选）

<div id="macos-app-availability">
  ## macOS 应用可用性
</div>

Beta 和开发版构建可能**不会**包含 macOS 应用发布。这没问题：

- 仍然可以发布 git tag 和 npm dist-tag。
- 在发布说明或变更日志中明确标注“此 beta 版本没有 macOS 构建”。