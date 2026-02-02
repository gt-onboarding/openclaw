---
title: 更新
summary: "`openclaw update` 的 CLI 参考（相对安全的源码更新 + Gateway 自动重启）"
read_when:
  - 你想安全地更新本地源码检出
  - 你需要了解 `--update` 简写参数的行为
---

<div id="openclaw-update">
  # `openclaw update`
</div>

安全更新 OpenClaw，并在 stable/beta/dev 通道之间切换。

如果你是通过 **npm/pnpm** 安装的（全局安装、无 git 元数据），请通过包管理器的更新流程进行升级，参见 [Updating](/zh/install/updating)。

<div id="usage">
  ## 用法
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## 选项
</div>

* `--no-restart`：在成功更新后跳过重启 Gateway 服务。
* `--channel <stable|beta|dev>`：设置更新通道（git + npm；会持久化到配置中）。
* `--tag <dist-tag|version>`：仅针对本次更新覆盖 npm 的 dist-tag 或版本。
* `--json`：输出机器可读的 `UpdateRunResult` JSON。
* `--timeout <seconds>`：每个步骤的超时时间（默认 1200 秒）。

注意：降级操作需要确认，因为旧版本可能会破坏配置。

<div id="update-status">
  ## `update status`
</div>

显示当前激活的更新通道和 git 标签/分支/SHA（用于源码检出），以及更新可用情况。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

选项：

* `--json`：输出机器可读的状态 JSON。
* `--timeout <seconds>`：检查的超时时长（默认 3 秒）。

<div id="update-wizard">
  ## `update wizard`
</div>

交互式流程，用于选择更新通道，并确认更新后是否重启 Gateway（默认会重启）。如果你选择 `dev` 且当前没有 git checkout，它会提示你创建一个。

<div id="what-it-does">
  ## 功能说明
</div>

当你显式切换通道（`--channel ...`）时，OpenClaw 也会同步调整安装方式：

* `dev` → 确保存在一个 git checkout（默认：`~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 自定义），
  更新该仓库，并基于该 checkout 安装全局 CLI。
* `stable`/`beta` → 使用对应的 dist-tag 从 npm 安装。

<div id="git-checkout-flow">
  ## Git checkout 流程
</div>

通道：

* `stable`：checkout 最新的非 beta 标签，然后执行 build + doctor。
* `beta`：checkout 最新的 `-beta` 标签，然后执行 build + doctor。
* `dev`：checkout `main`，然后执行 fetch + rebase。

总体流程：

1. 需要一个干净的工作树（没有未提交的更改）。
2. 切换到选定的通道（tag 或分支）。
3. 拉取上游（仅 dev）。
4. 仅 dev：在临时工作树中执行预检 lint 和 TypeScript 构建；如果最新提交失败，则最多回退 10 个提交，以找到最新的可通过构建的提交。
5. 在选定的提交上执行 rebase（仅 dev）。
6. 安装依赖（优先使用 pnpm；回退到 npm）。
7. 构建核心并构建 Control UI。
8. 运行 `openclaw doctor` 作为最终的“安全更新”检查。
9. 将插件与当前通道同步（dev 使用内置扩展；stable/beta 使用 npm），并更新通过 npm 安装的插件。

<div id="update-shorthand">
  ## `--update` 简写形式
</div>

`openclaw --update` 等价于运行 `openclaw update`（对 shell 和启动脚本很有用）。

<div id="see-also">
  ## 另请参阅
</div>

* `openclaw doctor`（在基于 git 的安装中会优先提示先运行更新）
* [开发通道](/zh/install/development-channels)
* [更新](/zh/install/updating)
* [CLI 参考](/zh/cli)