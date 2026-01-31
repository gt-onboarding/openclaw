---
title: Agent 工作区
summary: "Agent 工作区：位置、布局和备份策略"
read_when:
  - 你需要说明智能体工作区或其文件布局
  - 你想要备份或迁移某个智能体工作区
---

<div id="agent-workspace">
  # Agent 工作区
</div>

工作区是 Agent 代理 的“家”。它是文件工具和工作区上下文唯一使用的工作目录。请保持其私密性，并将其视作记忆空间。

这与 `~/.openclaw/` 目录不同，后者用于存储配置、凭据和会话。

**重要：**工作区是**默认 cwd（当前工作目录）**，而不是一个严格意义上的沙箱环境。工具会基于工作区解析相对路径，但在未启用沙箱的情况下，绝对路径仍然可以访问主机上的其他位置。如果你需要隔离，请使用 [`agents.defaults.sandbox`](/zh/gateway/sandboxing)（和/或为单个智能体配置沙箱）。当启用沙箱且 `workspaceAccess` 不是 `"rw"` 时，工具会在 `~/.openclaw/sandboxes` 下的沙箱工作区中运行，而不是在你的主机工作区中运行。

<div id="default-location">
  ## 默认位置
</div>

* 默认值：`~/.openclaw/workspace`
* 如果设置了 `OPENCLAW_PROFILE` 且其值不是 `"default"`，默认位置会变为
  `~/.openclaw/workspace-<profile>`。
* 可在 `~/.openclaw/openclaw.json` 中进行覆盖配置：

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`、`openclaw configure` 或 `openclaw setup` 会在缺失时创建
工作区并初始化引导文件。

如果你已经自行管理工作区文件，可以关闭自动创建引导文件：

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## 额外的工作区文件夹
</div>

较早的安装可能创建过 `~/openclaw`。在系统中保留多个工作区目录会导致认证或状态出现令人困惑的不一致，因为任意时刻只有一个工作区处于激活状态。

**建议：** 只保留一个处于激活状态的工作区。如果你不再使用这些额外文件夹，请将它们归档或移到回收站（例如 `trash ~/openclaw`）。如果你有意保留多个工作区，请确保
`agents.defaults.workspace` 指向当前激活的那个。

当检测到多余的工作区目录时，`openclaw doctor` 会发出警告。

<div id="workspace-file-map-what-each-file-means">
  ## 工作区文件说明（每个文件的含义）
</div>

这些是 OpenClaw 期望在工作区内存在的标准文件：

* `AGENTS.md`
  * 关于智能体的操作说明，以及它应该如何使用记忆。
  * 在每个会话开始时加载。
  * 适合作为规则、优先级和“行为方式”等细节的存放位置。

* `SOUL.md`
  * 人设、语气和边界。
  * 每个会话都会加载。

* `USER.md`
  * 用户是谁以及应如何称呼他们。
  * 每个会话都会加载。

* `IDENTITY.md`
  * 智能体的名称、风格和 emoji。
  * 在引导流程（bootstrap ritual）期间创建/更新。

* `TOOLS.md`
  * 关于本地工具和约定的备注。
  * 不控制工具的可用性；仅作为指导说明。

* `HEARTBEAT.md`
  * 用于心跳运行的可选精简检查清单。
  * 保持内容简短以避免 token 消耗过多。

* `BOOT.md`
  * 在启用内部钩子时，于 Gateway 重启时执行的可选启动检查清单。
  * 保持简短；对外发送请使用消息工具。

* `BOOTSTRAP.md`
  * 一次性首次运行流程。
  * 仅在全新的工作区中创建。
  * 流程完成后将其删除。

* `memory/YYYY-MM-DD.md`
  * 每日记忆日志（每天一个文件）。
  * 建议在会话开始时读取今天和昨天的文件。

* `MEMORY.md`（可选）
  * 精选的长期记忆。
  * 仅在主私有会话中加载（不要在共享/群组上下文中加载）。

有关工作流程和自动记忆刷新的说明，请参见 [Memory](/zh/concepts/memory)。

* `skills/`（可选）
  * 工作区专用技能。
  * 当名称冲突时，会覆盖托管/内置的技能。

* `canvas/`（可选）
  * 用于节点显示的 Canvas UI 文件（例如 `canvas/index.html`）。

如果缺少任何引导文件，OpenClaw 会向会话中注入一个“缺失文件”标记并继续执行。较大的引导文件在注入时会被截断；可通过 `agents.defaults.bootstrapMaxChars`（默认值：20000）调整该限制。
`openclaw setup` 可以在不覆盖现有文件的情况下重新创建缺失的默认文件。

<div id="what-is-not-in-the-workspace">
  ## 工作区中不包含的内容
</div>

这些内容位于 `~/.openclaw/` 目录下，且不要提交到工作区仓库：

* `~/.openclaw/openclaw.json`（配置）
* `~/.openclaw/credentials/`（OAuth 令牌、API 密钥）
* `~/.openclaw/agents/<agentId>/sessions/`（会话记录和元数据）
* `~/.openclaw/skills/`（受管技能）

如果你需要迁移会话或配置，请单独复制这些内容，并将它们
排除在版本控制之外。

<div id="git-backup-recommended-private">
  ## Git 备份（推荐，私有）
</div>

将工作区视为私有记忆。把它放在一个 **私有** 的 Git 仓库中，这样它就能被备份并且可恢复。

在运行 Gateway 的那台机器上执行这些步骤（也就是工作区所在的机器）。

<div id="1-initialize-the-repo">
  ### 1) 初始化仓库
</div>

如果已安装 git，全新的工作区会自动初始化为仓库。如果这个
工作区还不是一个仓库，运行：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "添加代理工作区"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) 添加私有远程（适合初学者的选项）
</div>

选项 A：GitHub Web UI

1. 在 GitHub 上创建一个新的 **私有** 仓库。
2. 不要初始化 README 文件（可避免合并冲突）。
3. 复制该 HTTPS 远程仓库的 URL。
4. 添加该远程并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

方案 B：GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

选项 C：GitLab Web UI

1. 在 GitLab 上创建一个新的**私有**仓库。
2. 不要在仓库中初始化 README（可避免合并冲突）。
3. 复制该仓库的 HTTPS 远程 URL。
4. 添加远程仓库并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3）后续更新
</div>

```bash
git status
git add .
git commit -m "更新内存"
git push
```

<div id="do-not-commit-secrets">
  ## 不要将敏感信息提交到版本库
</div>

即使是在私有仓库中，也要避免在工作区中存储敏感信息：

* API 密钥、OAuth 令牌、密码或其他私密凭证。
* `~/.openclaw/` 下的任何内容。
* 聊天记录的原始导出或敏感附件。

如果你必须存储与敏感信息相关的引用，请使用占位符，并将真实的敏感信息保存在其他地方（密码管理器、环境变量或 `~/.openclaw/` 中）。

推荐的 `.gitignore` 初始配置示例：

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## 将工作区迁移到新机器
</div>

1. 将仓库克隆到目标路径（默认路径为 `~/.openclaw/workspace`）。
2. 在 `~/.openclaw/openclaw.json` 中将 `agents.defaults.workspace` 设置为该路径。
3. 运行 `openclaw setup --workspace <path>` 以初始化任何缺失的文件。
4. 如果你需要会话，单独从旧机器复制 `~/.openclaw/agents/<agentId>/sessions/` 目录。

<div id="advanced-notes">
  ## 高级说明
</div>

* 多智能体路由可以为每个智能体使用不同的工作区。有关路由配置，请参见
  [Channel routing](/zh/concepts/channel-routing)。
* 如果启用了 `agents.defaults.sandbox`，非主会话可以在 `agents.defaults.sandbox.workspaceRoot` 下使用按会话隔离的沙箱工作区。