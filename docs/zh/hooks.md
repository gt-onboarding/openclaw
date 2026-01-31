---
title: Hooks 钩子
summary: "Hooks：用于命令和生命周期事件的事件驱动自动化"
read_when:
  - 你需要为 /new、/reset、/stop 以及 Agent 代理生命周期事件实现事件驱动自动化时
  - 你需要构建、安装或调试 Hooks 钩子时
---

<div id="hooks">
  # Hooks
</div>

Hooks 提供了一套可扩展的事件驱动系统，用于在响应智能体的命令和事件时自动执行操作。Hooks 会从目录中自动扫描和发现，并且可以通过 CLI 命令进行管理，其方式类似于 OpenClaw 中技能的工作方式。

<div id="getting-oriented">
  ## 入门指引
</div>

Hooks 是在特定事件发生时运行的小脚本。它们有两种类型：

* **Hooks**（本页）：在 Gateway 内部运行，当智能体事件触发时执行，比如 `/new`、`/reset`、`/stop` 或其他生命周期事件。
* **Webhooks**：外部 HTTP webhook，允许其他系统在 OpenClaw 中触发任务。参见 [Webhook Hooks](/zh/automation/webhook)，或使用 `openclaw webhooks` 获取 Gmail 辅助命令。

Hooks 也可以打包到插件中；参见 [Plugins](/zh/plugin#plugin-hooks)。

常见用法：

* 在重置会话时保存一次记忆快照
* 保留命令审计记录，用于排障或合规
* 在会话开始或结束时触发后续自动化流程
* 在事件触发时向智能体工作区写入文件，或调用外部 API

只要你会写一个简单的 TypeScript 函数，就能写出一个 hook。Hooks 会被自动发现，你可以通过 CLI 启用或禁用它们。

<div id="overview">
  ## 概览
</div>

Hooks 系统允许你：

* 在发出 `/new` 时将会话上下文保存到内存
* 记录所有命令用于审计
* 在智能体生命周期事件发生时触发自定义自动化流程
* 在不修改核心代码的情况下扩展 OpenClaw 的行为

<div id="getting-started">
  ## 入门
</div>

<div id="bundled-hooks">
  ### 内置 Hooks
</div>

OpenClaw 自带四个预置 hook，系统会自动发现：

* **💾 session-memory**：当你运行 `/new` 时，将会话上下文保存到你的智能体工作区（默认路径 `~/.openclaw/workspace/memory/`）
* **📝 command-logger**：将所有命令事件记录到 `~/.openclaw/logs/commands.log`
* **🚀 boot-md**：在 Gateway 启动时运行 `BOOT.md`（需要启用内部 hooks）
* **😈 soul-evil**：在清理窗口期或随机情况下，将注入的 `SOUL.md` 内容替换为 `SOUL_EVIL.md`

列出可用 hooks：

```bash
openclaw hooks list
```

启用 Hook：

```bash
openclaw hooks enable session-memory
```

查看 hook 状态：

```bash
openclaw hooks check
```

查看详细信息：

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### 初始引导（Onboarding）
</div>

在初始引导（`openclaw onboard`）过程中，系统会提示你启用推荐的 hooks。向导会自动检测所有符合条件的 hooks，并将它们展示出来供你选择。

<div id="hook-discovery">
  ## Hook 发现
</div>

Hook 会按以下优先级从三个目录中自动发现：

1. **工作区 hooks**：`<workspace>/hooks/`（按智能体划分，优先级最高）
2. **托管 hooks**：`~/.openclaw/hooks/`（用户安装，在各工作区间共享）
3. **内置 hooks**：`<openclaw>/dist/hooks/bundled/`（随 OpenClaw 一同发布）

托管 hook 目录既可以是一个 **单个 hook**，也可以是一个 **hook 包**（包目录）。

每个 hook 都是一个目录，目录中包含：

```
my-hook/
├── HOOK.md          # 元数据 + 文档
└── handler.ts       # 处理器实现
```

<div id="hook-packs-npmarchives">
  ## Hook 包（npm/归档）
</div>

Hook 包是标准的 npm 包，它们通过 `package.json` 中的 `openclaw.hooks` 字段导出一个或多个 hook。使用以下命令进行安装：

```bash
openclaw hooks install <path-or-spec>
```

示例 `package.json`：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

每个条目都指向一个包含 `HOOK.md` 和 `handler.ts`（或 `index.ts`）的 hook 目录。
Hook 包可以自带依赖，这些依赖会被安装到 `~/.openclaw/hooks/&lt;id&gt;` 目录下。

<div id="hook-structure">
  ## Hook 的结构
</div>

<div id="hookmd-format">
  ### HOOK.md 格式
</div>

`HOOK.md` 文件由开头的 YAML frontmatter 元数据和后面的 Markdown 文档组成：

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

<div id="metadata-fields">
  ### 元数据字段
</div>

`metadata.openclaw` 对象支持：

* **`emoji`**：在 CLI 中显示的表情符号（例如：`"💾"`）
* **`events`**：要监听的事件数组（例如：`["command:new", "command:reset"]`）
* **`export`**：要使用的具名导出（默认为 `"default"`）
* **`homepage`**：文档 URL
* **`requires`**：可选依赖条件
  * **`bins`**：PATH 中要求存在的可执行文件（例如：`["git", "node"]`）
  * **`anyBins`**：这些可执行文件中至少有一个必须存在
  * **`env`**：必需的环境变量
  * **`config`**：必需的配置路径（例如：`["workspace.dir"]`）
  * **`os`**：要求的操作系统平台（例如：`["darwin", "linux"]`）
* **`always`**：跳过可用性检查（布尔值）
* **`install`**：安装方法（对于打包的 hooks：`[{"id":"bundled","kind":"bundled"}]`）

<div id="handler-implementation">
  ### 处理器实现
</div>

`handler.ts` 文件导出一个 `HookHandler` 函数：

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // 仅在 'new' 命令时触发
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  会话: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // 在此处添加自定义逻辑

  // 可选:向用户发送消息
  event.messages.push('✨ My hook executed!');
};

export default myHandler;
```

<div id="event-context">
  #### 事件上下文
</div>

每个事件包含：

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // 例如:'whatsapp'、'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## 事件类型
</div>

<div id="command-events">
  ### 命令事件
</div>

在发出智能体命令时触发：

* **`command`**：所有命令事件（通用监听器）
* **`command:new`**：当发出 `/new` 命令时
* **`command:reset`**：当发出 `/reset` 命令时
* **`command:stop`**：当发出 `/stop` 命令时

<div id="agent-events">
  ### Agent 事件
</div>

* **`agent:bootstrap`**: 在注入工作区引导文件之前（钩子可以修改 `context.bootstrapFiles`）

<div id="gateway-events">
  ### Gateway 事件
</div>

当 Gateway 启动时触发：

* **`gateway:startup`**：在通道启动且钩子加载完成之后

<div id="tool-result-hooks-plugin-api">
  ### 工具结果 Hook（插件 API）
</div>

这些 Hook 并不是事件流监听器；它们允许插件在 OpenClaw 持久化工具结果之前以同步方式调整结果。

* **`tool_result_persist`**：在将工具结果写入会话记录之前对其进行转换。必须是同步的；返回更新后的工具结果载荷，或返回 `undefined` 以保持不变。参见 [Agent 循环](/zh/concepts/agent-loop)。

<div id="future-events">
  ### 未来事件
</div>

计划中的事件类型：

* **`session:start`**: 当新会话开始时
* **`session:end`**: 当会话结束时
* **`agent:error`**: 当智能体发生错误时
* **`message:sent`**: 当消息被发送时
* **`message:received`**: 当消息被接收时

<div id="creating-custom-hooks">
  ## 创建自定义钩子
</div>

<div id="1-choose-location">
  ### 1. 选择位置
</div>

* **工作区 hooks** (`<workspace>/hooks/`): 按智能体划分，优先级最高
* **托管 hooks** (`~/.openclaw/hooks/`): 在所有工作区之间共享

<div id="2-create-directory-structure">
  ### 2. 创建目录结构
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. 创建 HOOK.md 文件
</div>

```markdown
---
name: my-hook
description: "执行某些有用的操作"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# 我的自定义钩子

当你执行 `/new` 命令时,此钩子会执行某些有用的操作。
```

<div id="4-create-handlerts">
  ### 4. 创建 handler.ts 文件
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // 在此处编写您的逻辑
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. 启用和测试
</div>

```bash
# 验证 hook 已被发现
openclaw hooks list

# 启用 hook
openclaw hooks enable my-hook

# 重启 Gateway 进程(macOS 上重启菜单栏应用,或重启开发进程)

# 触发事件
# 通过消息通道发送 /new
```

<div id="configuration">
  ## 配置
</div>

<div id="new-config-format-recommended">
  ### 新的配置格式（推荐）
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true }, // 会话内存
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### 单个 Hook 的配置
</div>

每个 Hook 都可以拥有自定义配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### 附加目录
</div>

从附加目录加载 hooks：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### 旧版配置格式（仍受支持）
</div>

旧的配置格式依然可用，用于向后兼容：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**迁移**：新建 hook 时，请使用新的基于发现的系统。旧版处理器会在基于目录的 hook 之后加载。

<div id="cli-commands">
  ## CLI 命令
</div>

<div id="list-hooks">
  ### 列出所有 Hooks
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# 详细输出（显示缺失的依赖项）
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### Hook 信息
</div>

```bash
# 显示钩子的详细信息
openclaw hooks info session-memory

# JSON 输出
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### 检查适用条件
</div>

```bash
# 显示资格摘要
openclaw hooks check

# JSON 输出
openclaw hooks check --json
```

<div id="enabledisable">
  ### 启用/禁用
</div>

```bash
# 启用钩子
openclaw hooks enable session-memory

# 禁用钩子
openclaw hooks disable command-logger
```

## 内置 Hooks

<div id="session-memory">
  ### session-memory
</div>

当你执行 `/new` 时，将会话上下文保存到记忆中。

**事件**: `command:new`

**要求**: 必须配置 `workspace.dir`

**输出**: `<workspace>/memory/YYYY-MM-DD-slug.md`（默认值为 `~/.openclaw/workspace`）

**功能说明**:

1. 使用重置前的会话记录来定位正确的对话内容
2. 提取会话中最后 15 行内容
3. 使用 LLM 生成具有描述性的文件名 slug
4. 将会话元数据保存到按日期命名的记忆文件中

**示例输出**:

```markdown
# 会话：2026-01-16 14:30:00 UTC

- **会话键**：agent:main:main
- **会话 ID**：abc123def456
- **来源**：telegram
```

**文件名示例**：

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md`（在 slug 生成失败时作为回退时间戳使用）

**启用**：

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

将所有命令事件记录到集中审计日志文件。

**事件**: `command`

**要求**: 无

**输出**: `~/.openclaw/logs/commands.log`

**作用**:

1. 捕获事件详情（命令操作、时间戳、会话 key、发送方 ID、来源）
2. 以 JSONL 格式追加写入日志文件
3. 在后台静默运行

**示例日志记录**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**查看日志**：

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# 按操作类型筛选
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**启用**：

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

在清理时间窗口内或以一定的随机概率，将注入的 `SOUL.md` 内容替换为 `SOUL_EVIL.md`。

**事件**: `agent:bootstrap`

**文档**: [SOUL Evil Hook](/zh/hooks/soul-evil)

**输出**: 不写入任何文件；替换操作仅在内存中进行。

**启用**:

```bash
openclaw hooks enable soul-evil
```

**配置**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

在 Gateway 启动时（在通道启动之后）运行 `BOOT.md`。
必须启用内部 hook，此功能才会运行。

**事件（Events）**: `gateway:startup`

**要求（Requirements）**: 必须配置 `workspace.dir`

**功能说明（What it does）**:

1. 从你的工作区读取 `BOOT.md`
2. 通过智能体运行器执行其中的指令
3. 通过消息工具发送所有请求发送的出站消息

**启用方式（Enable）**:

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## 最佳实践
</div>

<div id="keep-handlers-fast">
  ### 保持处理程序快速
</div>

Hooks 会在命令处理过程中运行，请尽量让它们保持轻量：

```typescript
// ✓ 良好 - 异步工作,立即返回
const handler: HookHandler = async (event) => {
  void processInBackground(event); // 触发后即忘
};

// ✗ 不良 - 阻塞命令处理
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### 优雅处理错误
</div>

始终为可能出错的操作加上一层封装：

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // 不要抛出 - 让其他处理器运行
  }
};
```

<div id="filter-events-early">
  ### 尽早筛选事件
</div>

如果事件不相关，就立即返回：

```typescript
const handler: HookHandler = async (event) => {
  // 仅处理 'new' 命令
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // 你的逻辑代码
};
```

<div id="use-specific-event-keys">
  ### 使用特定事件键
</div>

尽可能在元数据中精确指定具体事件：

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # 具体事件
```

而不是：

```yaml
metadata: {"openclaw":{"events":["command"]}}      # 通用 - 开销较大
```

<div id="debugging">
  ## 调试
</div>

<div id="enable-hook-logging">
  ### 启用 Hook 日志
</div>

Gateway 在启动时会记录 Hook 的加载情况：

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### 检查发现情况
</div>

列出所有已发现的 hooks：

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### 检查注册
</div>

在你的 handler 中添加日志以记录其被调用的时机：

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // 你的业务逻辑
};
```

<div id="verify-eligibility">
  ### 验证生效条件
</div>

检查某个 hook 为什么不符合生效条件：

```bash
openclaw hooks info my-hook
```

检查输出中是否有遗漏的需求。

<div id="testing">
  ## 测试
</div>

<div id="gateway-logs">
  ### Gateway 日志
</div>

监控 Gateway 日志以查看 Hook 的执行情况：

```bash
# macOS
./scripts/clawlog.sh -f

# 其他平台
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### 直接测试 Hooks
</div>

在隔离环境下单独测试你的处理函数：

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // 断言副作用
});
```

<div id="architecture">
  ## 架构
</div>

<div id="core-components">
  ### 核心组件
</div>

* **`src/hooks/types.ts`**: 类型定义
* **`src/hooks/workspace.ts`**: 目录扫描与加载
* **`src/hooks/frontmatter.ts`**: HOOK.md 元数据解析
* **`src/hooks/config.ts`**: 生效条件检查
* **`src/hooks/hooks-status.ts`**: 状态报告
* **`src/hooks/loader.ts`**: 动态模块加载器
* **`src/cli/hooks-cli.ts`**: CLI 命令
* **`src/gateway/server-startup.ts`**: 在 Gateway 启动时加载 hooks
* **`src/auto-reply/reply/commands-core.ts`**: 触发命令事件

<div id="discovery-flow">
  ### 发现流程
</div>

```
Gateway 启动
    ↓
扫描目录(工作区 → 托管 → 内置)
    ↓
解析 HOOK.md 文件
    ↓
检查资格(二进制文件、环境、配置、操作系统)
    ↓
从符合条件的钩子加载处理器
    ↓
为事件注册处理器
```

<div id="event-flow">
  ### 事件流程
</div>

```
用户发送 /new
    ↓
命令验证
    ↓
创建钩子事件
    ↓
触发钩子(所有已注册的处理器)
    ↓
命令处理继续
    ↓
会话重置
```

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="hook-not-discovered">
  ### 未检测到 Hook
</div>

1. 检查目录结构：
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # 应该显示：HOOK.md, handler.ts
   ```

2. 检查 HOOK.md 格式：
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # 应包含带有名称和元数据的 YAML frontmatter 头部
   ```

3. 列出所有已发现的 Hook：
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook 不满足条件
</div>

检查前置条件：

```bash
openclaw hooks info my-hook
```

检查是否存在以下缺失情况：

* 二进制可执行文件（检查 PATH）
* 环境变量
* 配置项
* 与操作系统的兼容性

<div id="hook-not-executing">
  ### Hook 未执行
</div>

1. 确认已启用该 hook：
   ```bash
   openclaw hooks list
   # 在已启用的 hooks 旁边应显示 ✓
   ```

2. 重启你的 Gateway 进程以重新加载 hooks。

3. 检查 Gateway 日志中是否有错误：
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### 处理器错误
</div>

检查是否存在 TypeScript 或导入错误：

```bash
# 直接测试导入
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## 迁移指南
</div>

<div id="from-legacy-config-to-discovery">
  ### 从旧版配置过渡到 Discovery
</div>

**之前**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**之后**：

1. 创建 hook 目录：
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. 创建 HOOK.md：
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   Does something useful.
   ```

3. 更新配置：
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. 验证并重启你的 Gateway 进程：
   ```bash
   openclaw hooks list
   # 应显示：🎯 my-hook ✓
   ```

**迁移的好处**：

* 自动发现
* CLI 管理
* 可用性检查
* 更好的文档支持
* 一致的结构

<div id="see-also">
  ## 另见
</div>

* [CLI 参考：hooks](/zh/cli/hooks)
* [内置 hooks 的 README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Webhook 钩子](/zh/automation/webhook)
* [配置](/zh/gateway/configuration#hooks)