---
title: 技能
summary: "技能：托管型与工作区型、门控规则，以及配置/环境变量绑定"
read_when:
  - 添加或修改技能
  - 更改技能门控或加载规则
---

<div id="skills-openclaw">
  # 技能（OpenClaw）
</div>

OpenClaw 使用与 **[AgentSkills](https://agentskills.io) 兼容** 的技能目录来教智能体如何使用工具。每个技能对应一个目录，其中包含带有 YAML 前置元数据和说明的 `SKILL.md` 文件。OpenClaw 会加载**内置技能**以及可选的本地覆盖项，并在加载时根据环境、配置以及相关二进制文件是否存在对它们进行筛选。

<div id="locations-and-precedence">
  ## 位置与优先级
</div>

技能会从**三个**位置加载：

1. **内置技能**：随安装包提供（npm 包或 OpenClaw.app）
2. **受管/本地技能**：`~/.openclaw/skills`
3. **工作区技能**：`<workspace>/skills`

如果技能名称发生冲突，优先级为：

`<workspace>/skills`（最高） → `~/.openclaw/skills` → 内置技能（最低）

此外，你可以通过在 `~/.openclaw/openclaw.json` 中配置 `skills.load.extraDirs` 来添加额外的技能目录（最低优先级）。

<div id="per-agent-vs-shared-skills">
  ## 按智能体划分的技能 vs 共享技能
</div>

在 **多智能体（multi-agent）** 配置中，每个智能体都有自己的工作区。这意味着：

* **按智能体划分的技能（Per-agent skills）** 只存在于该智能体的 `<workspace>/skills` 目录中。
* **共享技能（Shared skills）** 位于 `~/.openclaw/skills`（托管/本地），并且在同一台机器上的
  **所有智能体** 都可见。
* 如果你希望多个智能体使用一套通用技能包，还可以通过 `skills.load.extraDirs` 添加
  **共享文件夹**（优先级最低）。

如果同名技能在多个位置同时存在，则会应用常规的优先级规则：工作区优先，其次是托管/本地，最后是内置捆绑。

<div id="plugins-skills">
  ## 插件 + 技能
</div>

插件可以通过在 `openclaw.plugin.json` 中列出 `skills` 目录（相对于插件根目录的路径），来自带它们自己的技能。插件技能会在插件启用时加载，并参与正常的技能优先级规则。你可以通过插件配置项中的 `metadata.openclaw.requires.config` 来对其进行启用控制。有关发现/配置，请参见 [Plugins](/zh/plugin)，有关这些技能暴露的工具能力范围，请参见 [Tools](/zh/tools)。

<div id="clawhub-install-sync">
  ## ClawHub（安装与同步）
</div>

ClawHub 是 OpenClaw 的公共技能注册表。你可以在 https://clawhub.com 浏览。使用它来发现、安装、更新和备份技能。
完整指南：[ClawHub](/zh/tools/clawhub)。

常用操作：

* 将技能安装到你的工作区：
  * `clawhub install <skill-slug>`
* 更新所有已安装的技能：
  * `clawhub update --all`
* 同步（扫描并发布更新）：
  * `clawhub sync --all`

默认情况下，`clawhub` 会安装到你当前工作目录下的 `./skills`（或回退到已配置的 OpenClaw 工作区）。OpenClaw 会在下一次会话中将其识别为 `<workspace>/skills`。

<div id="security-notes">
  ## 安全注意事项
</div>

* 将第三方技能视同**受信任代码**对待。启用前先审阅其代码。
* 对不受信任的输入和高风险工具，应优先在沙箱中运行。参见 [Sandboxing](/zh/gateway/sandboxing)。
* `skills.entries.*.env` 和 `skills.entries.*.apiKey` 会将机密注入该智能体当前回合的**宿主**进程
  （而不是沙箱）。避免在提示词和日志中暴露机密。
* 如需更全面的威胁模型和检查清单，参见 [Security](/zh/gateway/security)。

<div id="format-agentskills-pi-compatible">
  ## 格式（AgentSkills + Pi 兼容）
</div>

`SKILL.md` 至少应包含：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图片
---
```

Notes:

* 我们在布局和意图上遵循 AgentSkills 规范。
* 嵌入式智能体使用的解析器只支持 **单行** frontmatter 键。
* `metadata` 必须是一个 **单行 JSON 对象**。
* 在说明中使用 `{baseDir}` 来引用技能文件夹路径。
* 可选的 frontmatter 键：
  * `homepage` — 在 macOS Skills UI 中以“Website”形式展示的 URL（也可通过 `metadata.openclaw.homepage` 配置）。
  * `user-invocable` — `true|false`（默认：`true`）。当为 `true` 时，该技能会作为用户可用的斜杠命令暴露出来。
  * `disable-model-invocation` — `true|false`（默认：`false`）。当为 `true` 时，该技能不会包含在传给模型的提示中（但仍可通过用户调用使用）。
  * `command-dispatch` — `tool`（可选）。当设为 `tool` 时，斜杠命令会绕过模型，直接派发到某个工具。
  * `command-tool` — 在设置了 `command-dispatch: tool` 时要调用的工具名称。
  * `command-arg-mode` — `raw`（默认）。用于工具派发时，将原始参数字符串转发给工具（不做核心解析）。

    工具调用时会传入以下参数：
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## 门控（加载时过滤）
</div>

OpenClaw 会在**加载时**使用 `metadata`（单行 JSON）对技能进行过滤：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图像
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

位于 `metadata.openclaw` 下的字段：

* `always: true` — 始终包含该技能（跳过其他 gate 检查）。
* `emoji` — 可选，用于 macOS Skills UI 的 emoji。
* `homepage` — 可选，在 macOS Skills UI 中以 “Website” 显示的 URL。
* `os` — 可选的平台列表（`darwin`、`linux`、`win32`）。如果设置了，该技能只会在这些操作系统上可用。
* `requires.bins` — 列表；其中每个可执行文件都必须存在于 `PATH` 中。
* `requires.anyBins` — 列表；其中至少一个可执行文件必须存在于 `PATH` 中。
* `requires.env` — 列表；对应的环境变量必须存在，**或者** 在配置中显式提供。
* `requires.config` — `openclaw.json` 中其值必须为 truthy 的字段路径列表。
* `primaryEnv` — 与 `skills.entries.<name>.apiKey` 关联的环境变量名称。
* `install` — 由 macOS Skills UI 使用的可选安装器规范数组（brew/node/go/uv/download）。

关于沙箱的说明：

* `requires.bins` 会在加载技能时在**宿主机**上进行检查。
* 如果某个智能体在沙箱中运行，则该可执行文件也必须**存在于容器内部**。
  通过 `agents.defaults.sandbox.docker.setupCommand`（或自定义镜像）在容器中安装。
  `setupCommand` 会在容器创建后运行一次。
  安装软件包还需要具备出站网络访问、可写的根文件系统以及沙箱中的 root 用户权限。
  示例：`summarize` 技能（`skills/summarize/SKILL.md`）需要在沙箱容器内提供 `summarize` CLI，
  才能在其中运行。

安装器示例：

```markdown
---
name: gemini
description: 使用 Gemini CLI 进行编码辅助和 Google 搜索查询。
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Install Gemini CLI (brew)"}]}}
---
```

Notes:

* 如果列出了多个安装器，Gateway 会选择**一个**首选选项（优先使用 brew，如不可用则使用 node）。
* 如果所有安装器都是 `download`，OpenClaw 会列出每个条目，方便你查看可用构件。
* 安装器配置可以包含 `os: ["darwin"|"linux"|"win32"]`，用于按平台筛选可选项。
* Node 安装会遵循 `openclaw.json` 中的 `skills.install.nodeManager`（默认值：npm；可选值：npm/pnpm/yarn/bun）。
  这只会影响**技能安装**；Gateway 运行时仍应使用 Node
  （不推荐在 WhatsApp/Telegram 场景使用 Bun）。
* Go 安装：如果缺少 `go` 且 `brew` 可用，Gateway 会先通过 Homebrew 安装 Go，并在可能的情况下将 `GOBIN` 设置为 Homebrew 的 `bin`。
* Download 安装：`url`（必填）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（默认：检测到归档时自动）、`stripComponents`、`targetDir`（默认：`~/.openclaw/tools/<skillKey>`）。

如果不存在 `metadata.openclaw`，该技能始终视为可用（除非在配置中被禁用，或对于内置技能被 `skills.allowBundled` 阻止）。

<div id="config-overrides-openclawopenclawjson">
  ## 配置覆盖（`~/.openclaw/openclaw.json`）
</div>

内置/受管技能可以通过开关启用或禁用，并为其提供环境变量值：

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

注意：如果技能名称包含连字符，请为键加上引号（JSON5 允许带引号的键）。

配置键默认与**技能名称**一致。如果某个技能定义了
`metadata.openclaw.skillKey`，则在 `skills.entries` 下使用该键。

规则：

* `enabled: false` 会禁用该技能，即便它已被打包/安装。
* `env`：**仅在**该变量尚未在进程中设置时才会注入。
* `apiKey`：为声明了 `metadata.openclaw.primaryEnv` 的技能提供的便捷字段。
* `config`：可选的、用于存放按技能划分的自定义字段的容器；自定义键必须放在这里。
* `allowBundled`：仅用于**打包内置**技能的可选允许列表。如果设置了该字段，则只有
  列表中的打包技能才会被采用（托管/工作区技能不受影响）。

<div id="environment-injection-per-agent-run">
  ## 环境注入（按单次智能体运行）
</div>

当一次智能体运行开始时，OpenClaw 会：

1. 读取技能元数据。
2. 将任何 `skills.entries.<key>.env` 或 `skills.entries.<key>.apiKey` 注入到
   `process.env`。
3. 使用**符合条件的**技能构建系统提示词（system prompt）。
4. 在本次运行结束后恢复原始环境。

这**仅作用于本次智能体运行**，而不是全局 shell 环境。

<div id="session-snapshot-performance">
  ## 会话快照（性能）
</div>

OpenClaw 会在**会话开始时**对符合条件的技能进行快照，并在同一会话中的后续轮次复用该列表。对技能或配置的更改会在下一次新会话中生效。

当启用了技能监视器，或者有新的符合条件的远程节点出现时，技能也可以在会话中途刷新（见下文）。你可以将这视为一种**热重载（hot reload）**：刷新后的列表会在下一次智能体轮次中被使用。

<div id="remote-macos-nodes-linux-gateway">
  ## 远程 macOS 节点（Linux Gateway）
</div>

如果 Gateway 运行在 Linux 上，但有一个**macOS 节点**已连接且**允许 `system.run`**（执行审批安全策略未设置为 `deny`），当该节点上存在所需的二进制可执行文件时，OpenClaw 可以将仅适用于 macOS 的技能视为可用候选项。智能体应通过 `nodes` 工具（通常是 `nodes.run`）来执行这些技能。

这依赖于节点报告其命令支持情况，以及通过 `system.run` 进行的二进制探测。如果 macOS 节点随后离线，这些技能仍会保持可见；在节点重新连接之前，调用可能会失败。

<div id="skills-watcher-auto-refresh">
  ## 技能监视器（自动刷新）
</div>

默认情况下，OpenClaw 会监视技能目录，一旦 `SKILL.md` 文件发生变更就会刷新技能快照。可在 `skills.load` 中进行配置：

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## Token 开销（技能列表）
</div>

当技能处于可用状态时，OpenClaw 会在系统提示词（system prompt）中注入一个紧凑的可用技能 XML 列表（通过 `pi-coding-agent` 中的 `formatSkillsForPrompt` 实现）。其字符开销是确定性的：

* **基础开销（仅在技能数量 ≥ 1 时）：** 195 个字符。
* **每个技能：** 97 个字符 + XML 转义后的 `<name>`、`<description>` 和 `<location>` 值的长度。

公式（按字符计）：

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notes:

* XML 转义会将 `& < > " '` 展开为实体（`&amp;`、`&lt;` 等），从而增加长度。
* token 计数会因具体模型的 tokenizer 而有所不同。粗略按 OpenAI 风格估算约为每个 token ≈ 4 个字符，因此 **97 个字符 ≈ 24 个 token**，再加上你各字段的实际长度。

<div id="managed-skills-lifecycle">
  ## 受管技能生命周期
</div>

OpenClaw 在安装时（npm 包或 OpenClaw.app）会提供一组基础的 **捆绑技能**。`~/.openclaw/skills` 用于本地覆盖（例如，在不修改捆绑副本的情况下固定版本或打补丁某个技能）。工作区技能由用户管理，并在名称冲突时优先于前两者。

<div id="config-reference">
  ## 配置参考
</div>

完整的配置架构请参见 [技能配置](/zh/tools/skills-config)。

<div id="looking-for-more-skills">
  ## 想要更多技能？
</div>

前往 https://clawhub.com 查看。

***