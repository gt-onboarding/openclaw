---
title: 钩子
summary: "`openclaw hooks`（智能体钩子）的 CLI 参考文档"
read_when:
  - 你需要管理智能体钩子时
  - 你需要安装或更新钩子时
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

管理智能体 Hook（用于 `/new`、`/reset` 等命令以及 Gateway 启动的事件驱动自动化）。

相关内容：

* Hooks：[Hooks](/zh/hooks)
* 插件 Hook：[Plugins](/zh/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## 列出所有钩子
</div>

```bash
openclaw hooks list
```

列出在工作区、托管目录和内置目录中发现的所有 hooks。

**选项：**

* `--eligible`: 仅显示符合条件的 hooks（已满足要求）
* `--json`: 以 JSON 格式输出
* `-v, --verbose`: 显示详细信息，包括缺失的要求

**示例输出：**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**示例（详细模式）：**

```bash
openclaw hooks list --verbose
```

显示当前不符合条件的 hook 所缺少的前置条件。

**示例（JSON）：**

```bash
openclaw hooks list --json
```

返回用于编程使用的结构化 JSON。

<div id="get-hook-information">
  ## 获取 Hook 详情
</div>

```bash
openclaw hooks info <name>
```

显示指定 hook 的详细信息。

**参数：**

* `<name>`：Hook 名称（例如：`session-memory`）

**选项：**

* `--json`：以 JSON 格式输出

**示例：**

```bash
openclaw hooks info session-memory
```

**输出：**

```
💾 session-memory ✓ 就绪

当执行 /new 命令时将会话上下文保存到内存

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## 检查 Hook 可用性
</div>

```bash
openclaw hooks check
```

显示 hook 可用状态的摘要（已就绪与未就绪的数量）。

**选项：**

* `--json`: 以 JSON 输出

**示例输出：**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## 启用 Hook
</div>

```bash
openclaw hooks enable <name>
```

通过将特定 hook 添加到你的配置文件（`~/.openclaw/config.json`）中来启用该 hook。

**注意：** 由插件管理的 hooks 在 `openclaw hooks list` 中会显示为 `plugin:<id>`，
并且无法通过此命令启用或禁用。请改为启用或禁用对应的插件。

**参数：**

* `<name>`：Hook 名称（例如：`session-memory`）

**示例：**

```bash
openclaw hooks enable session-memory
```

**输出：**

```
✓ 已启用钩子：💾 session-memory
```

**作用：**

* 检查 hook 是否存在且可用
* 在你的配置文件中将 `hooks.internal.entries.<name>.enabled = true`
* 将配置保存到磁盘

**启用后：**

* 重启 Gateway 以重新加载 hooks（在 macOS 上重启菜单栏应用，或在开发环境中重启你的 Gateway 进程）。

<div id="disable-a-hook">
  ## 停用 Hook
</div>

```bash
openclaw hooks disable <name>
```

通过更新配置文件来禁用指定的 hook。

**参数：**

* `<name>`：Hook 名称（例如 `command-logger`）

**示例：**

```bash
openclaw hooks disable command-logger
```

**输出：**

```
⏸ 已禁用钩子：📝 command-logger
```

**禁用后：**

* 重启 Gateway 以让 hooks 重新加载

<div id="install-hooks">
  ## 安装 Hooks
</div>

```bash
openclaw hooks install <path-or-spec>
```

从本地文件夹、归档文件或 npm 安装 hook 包。

**作用：**

* 将该 hook 包复制到 `~/.openclaw/hooks/<id>`
* 在 `hooks.internal.entries.*` 中启用已安装的 hooks
* 将此次安装记录在 `hooks.internal.installs` 下

**选项：**

* `-l, --link`：链接到本地目录而不是复制（会将其添加到 `hooks.internal.load.extraDirs`）

**支持的归档格式：** `.zip`、`.tgz`、`.tar.gz`、`.tar`

**示例：**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# 链接本地目录而不复制
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## 更新 Hook
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

更新已安装的 hook 包（仅限通过 npm 安装的）。

**选项：**

* `--all`: 更新所有已跟踪的 hook 包
* `--dry-run`: 显示将要发生的变更但不实际写入

<div id="bundled-hooks">
  ## 内置 Hook
</div>

<div id="session-memory">
  ### session-memory
</div>

在你发出 `/new` 命令时，将会话上下文保存到内存中。

**启用：**

```bash
openclaw hooks enable session-memory
```

**输出：** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**参见：** [会话内存文档](/zh/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

将所有命令事件记录到集中式审计日志文件中。

**启用：**

```bash
openclaw hooks enable command-logger
```

**输出：** `~/.openclaw/logs/commands.log`

**查看日志：**

```bash
# 查看最近的命令
tail -n 20 ~/.openclaw/logs/commands.log

# 格式化打印
cat ~/.openclaw/logs/commands.log | jq .

# 按 action 字段过滤
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**请参阅：** [command-logger 文档](/zh/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

在清理时间窗口期间或以一定随机概率，将注入的 `SOUL.md` 内容替换为 `SOUL_EVIL.md`。

**启用：**

```bash
openclaw hooks enable soul-evil
```

**参见：** [SOUL Evil Hook](/zh/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

当 Gateway 启动时（在各个 channel 启动之后）运行 `BOOT.md`。

**事件**: `gateway:startup`

**启用**:

```bash
openclaw hooks enable boot-md
```

**请参阅：** [boot-md 文档](/zh/hooks#boot-md)
