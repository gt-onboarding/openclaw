---
title: ClawHub
summary: "ClawHub 指南：公共技能注册中心 + CLI 工作流程"
read_when:
  - 向新用户介绍 ClawHub
  - 安装、搜索或发布技能
  - 解释 ClawHub CLI 标志和同步行为
---

<div id="clawhub">
  # ClawHub
</div>

ClawHub 是 **OpenClaw 的公共技能注册表**。它是一个免费服务：所有技能都是公开的、开放的，对所有人可见，便于共享和复用。一个技能本质上就是一个包含 `SKILL.md` 文件（以及一些支持性文本文件）的文件夹。你可以在 Web 应用中浏览技能，或者使用 CLI 来搜索、安装、更新和发布技能。

网站：[clawhub.com](https://clawhub.com)

<div id="who-this-is-for-beginner-friendly">
  ## 适合哪些人（适合新手）
</div>

如果你想为自己的 OpenClaw Agent 代理添加新功能，ClawHub 是查找和安装技能的最简单方式。你不需要了解后端具体是如何工作的。你可以：

* 使用自然语言搜索技能。
* 将技能安装到你的工作区中。
* 之后只需一条命令就能更新技能。
* 通过发布来备份你自己的技能。

<div id="quick-start-non-technical">
  ## 快速开始（面向非技术用户）
</div>

1. 安装 CLI（见下一节）。
2. 搜索你需要的内容：
   * `clawhub search "calendar"`
3. 安装一个技能：
   * `clawhub install <skill-slug>`
4. 启动一个新的 OpenClaw 会话，以便加载这个新技能。

<div id="install-the-cli">
  ## 安装 CLI
</div>

请选择一种方式：

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="how-it-fits-into-openclaw">
  ## 它在 OpenClaw 中的定位
</div>

默认情况下，CLI 会把技能安装到当前工作目录下的 `./skills` 中。如果已经配置了 OpenClaw 工作区，`clawhub` 会优先使用该工作区，除非你显式指定 `--workdir`（或 `CLAWHUB_WORKDIR`）。OpenClaw 会从 `<workspace>/skills` 加载该工作区中的技能，并在**下一次**会话中生效。如果你已经在使用 `~/.openclaw/skills` 或随附的技能，则工作区技能会优先生效。

关于技能如何加载、共享和访问控制的更多细节，请参阅
[技能](/zh/tools/skills)。

<div id="what-the-service-provides-features">
  ## 服务提供的功能（特性）
</div>

* **公开浏览**技能及其 `SKILL.md` 内容。
* 由嵌入向量（向量检索）驱动的**搜索**，而不仅仅是关键词匹配。
* 基于 semver 的**版本管理**，包含更新日志和标签（包括 `latest`）。
* 按版本提供的 zip 包**下载**。
* 用于社区反馈的**星标（Stars）和评论**。
* 用于审批和审计的**审核（Moderation）钩子**。
* 适配 CLI 使用的**API**，便于自动化和脚本集成。

<div id="cli-commands-and-parameters">
  ## CLI commands and parameters
</div>

全局选项（适用于所有命令）：

* `--workdir &lt;dir&gt;`: 工作目录（默认：当前目录；否则回退到 OpenClaw 工作区）。
* `--dir &lt;dir&gt;`: 技能目录，相对于 workdir（默认：`skills`）。
* `--site &lt;url&gt;`: 站点基础 URL（用于浏览器登录）。
* `--registry &lt;url&gt;`: Registry API 基础 URL。
* `--no-input`: 禁用交互提示（非交互模式）。
* `-V, --cli-version`: 打印 CLI 版本。

认证：

* `clawhub login`（浏览器流程）或 `clawhub login --token &lt;token&gt;`
* `clawhub logout`
* `clawhub whoami`

选项：

* `--token &lt;token&gt;`: 粘贴一个 API token。
* `--label &lt;label&gt;`: 为浏览器登录 token 存储的标签（默认：`CLI token`）。
* `--no-browser`: 不打开浏览器（需要同时提供 `--token`）。

搜索：

* `clawhub search "query"`
* `--limit &lt;n&gt;`: 最大结果数。

安装：

* `clawhub install &lt;slug&gt;`
* `--version &lt;version&gt;`: 安装指定版本。
* `--force`: 如果文件夹已存在则覆盖。

更新：

* `clawhub update &lt;slug&gt;`
* `clawhub update --all`
* `--version &lt;version&gt;`: 更新到指定版本（仅单个 slug 时可用）。
* `--force`: 当本地文件与任意已发布版本都不匹配时强制覆盖。

列出：

* `clawhub list`（读取 `.clawhub/lock.json`）

发布：

* `clawhub publish &lt;path&gt;`
* `--slug &lt;slug&gt;`: 技能 slug。
* `--name &lt;name&gt;`: 显示名称。
* `--version &lt;version&gt;`: 符合 SemVer 的版本号。
* `--changelog &lt;text&gt;`: 更新日志文本（可为空）。
* `--tags &lt;tags&gt;`: 逗号分隔的标签（默认：`latest`）。

删除/撤销删除（仅所有者/管理员）：

* `clawhub delete &lt;slug&gt; --yes`
* `clawhub undelete &lt;slug&gt; --yes`

同步（扫描本地技能并发布新增/更新项）：

* `clawhub sync`
* `--root &lt;dir...&gt;`: 额外扫描根路径。
* `--all`: 无提示上传所有内容。
* `--dry-run`: 仅显示将要上传的内容。
* `--bump &lt;type&gt;`: 更新版本号类型：`patch|minor|major`（默认：`patch`）。
* `--changelog &lt;text&gt;`: 非交互式更新时使用的更新日志。
* `--tags &lt;tags&gt;`: 逗号分隔的标签（默认：`latest`）。
* `--concurrency &lt;n&gt;`: Registry 检查并发数（默认：4）。

<div id="common-workflows-for-agents">
  ## 智能体的常见工作流程
</div>

<div id="search-for-skills">
  ### 查找技能
</div>

```bash
clawhub search "postgres backups"
```

<div id="download-new-skills">
  ### 下载新技能
</div>

```bash
clawhub install my-skill-pack
```

<div id="update-installed-skills">
  ### 更新已安装的技能
</div>

```bash
clawhub update --all
```

<div id="back-up-your-skills-publish-or-sync">
  ### 备份你的技能（发布或同步）
</div>

对于单个技能目录：

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

若要同时扫描和备份多个技能：

```bash
clawhub sync --all
```

<div id="advanced-details-technical">
  ## 高级技术细节
</div>

<div id="versioning-and-tags">
  ### 版本管理与标签
</div>

* 每次发布都会创建一个新的 **语义化版本（semver）** `SkillVersion`。
* 标签（如 `latest`）指向具体版本；通过移动标签可以执行回滚。
* 变更日志按版本关联，在同步或发布更新时可以为空。

<div id="local-changes-vs-registry-versions">
  ### 本地变更 vs 注册表版本
</div>

更新会通过内容哈希，将本地技能的内容与注册表中的版本进行比较。如果本地文件与任一已发布版本都不匹配，CLI 会在覆盖前提示确认（在非交互式运行中则需要使用 `--force`）。

<div id="sync-scanning-and-fallback-roots">
  ### 同步扫描与回退根路径
</div>

`clawhub sync` 会优先扫描你当前的工作目录。如果未找到任何技能，则会回退到已知的旧版路径（例如 `~/openclaw/skills` 和 `~/.openclaw/skills`）。这样设计是为了在无需额外标志或选项的情况下发现较早安装的技能。

<div id="storage-and-lockfile">
  ### 存储和锁文件
</div>

* 已安装的技能会记录在工作目录下的 `.clawhub/lock.json` 中。
* 认证令牌存储在 ClawHub CLI 配置文件中（可通过 `CLAWHUB_CONFIG_PATH` 覆盖）。

<div id="telemetry-install-counts">
  ### 遥测（安装计数）
</div>

当你在登录状态下运行 `clawhub sync` 时，CLI 会发送一个精简快照用于统计安装计数。你可以完全禁用这一行为：

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

<div id="environment-variables">
  ## 环境变量
</div>

* `CLAWHUB_SITE`: 用于覆盖站点 URL。
* `CLAWHUB_REGISTRY`: 用于覆盖注册表 API 的 URL。
* `CLAWHUB_CONFIG_PATH`: 用于覆盖 CLI 存储令牌/配置的路径。
* `CLAWHUB_WORKDIR`: 用于覆盖默认工作目录。
* `CLAWHUB_DISABLE_TELEMETRY=1`: 在执行 `sync` 时禁用遥测数据收集。