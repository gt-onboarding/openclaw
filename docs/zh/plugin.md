---
title: 插件
summary: "OpenClaw 插件/扩展：发现、配置和安全性"
read_when:
  - 添加或修改插件/扩展
  - 编写插件安装或加载规则相关文档时
---

<div id="plugins-extensions">
  # 插件（扩展功能）
</div>

<div id="quick-start-new-to-plugins">
  ## 快速上手（刚接触插件？）
</div>

插件只是一个**小型代码模块**，用来为 OpenClaw 增加额外功能（命令、工具和 Gateway RPC）。

大多数情况下，当你需要某个核心 OpenClaw 尚未内置的功能（或者你希望把可选功能与主安装环境分离）时，就会用到插件。

快速路径：

1. 查看当前已加载的内容：

```bash
openclaw plugins list
```

2. 安装官方插件（例如：Voice Call）：

```bash
openclaw plugins install @openclaw/voice-call
```

3. 重启 Gateway，然后在 `plugins.entries.<id>.config` 下进行配置。

参见 [Voice Call](/zh/plugins/voice-call)，了解一个具体的示例插件。

<div id="available-plugins-official">
  ## 可用插件（官方）
</div>

* 截至 2026.1.15，Microsoft Teams 仅通过插件方式支持；如果你使用 Teams，请安装 `@openclaw/msteams`。
* Memory (Core) — 内置的记忆搜索插件（通过 `plugins.slots.memory` 默认启用）
* Memory (LanceDB) — 内置的长期记忆插件（自动回忆/捕获；将 `plugins.slots.memory` 设置为 `"memory-lancedb"`）
* [Voice Call](/zh/plugins/voice-call) — `@openclaw/voice-call`
* [Zalo Personal](/zh/plugins/zalouser) — `@openclaw/zalouser`
* [Matrix](/zh/channels/matrix) — `@openclaw/matrix`
* [Nostr](/zh/channels/nostr) — `@openclaw/nostr`
* [Zalo](/zh/channels/zalo) — `@openclaw/zalo`
* [Microsoft Teams](/zh/channels/msteams) — `@openclaw/msteams`
* Google Antigravity OAuth（提供方认证）— 以内置插件形式提供 `google-antigravity-auth`（默认禁用）
* Gemini CLI OAuth（提供方认证）— 以内置插件形式提供 `google-gemini-cli-auth`（默认禁用）
* Qwen OAuth（提供方认证）— 以内置插件形式提供 `qwen-portal-auth`（默认禁用）
* Copilot Proxy（提供方认证）— 本地 VS Code Copilot Proxy 桥接；与内置的 `github-copilot` 设备登录不同（内置但默认禁用）

OpenClaw 插件是通过 jiti 在运行时加载的 **TypeScript 模块**。**配置
校验不会执行插件代码**；它只使用插件 manifest 和 JSON Schema。参见 [Plugin manifest](/zh/plugins/manifest)。

插件可以注册：

* Gateway RPC 方法
* Gateway HTTP 处理器
* Agent 代理工具
* CLI 命令
* 后台服务
* 可选的配置校验
* **技能**（通过在插件 manifest 中列出 `skills` 目录）
* **自动回复命令**（在不调用 AI 智能体的情况下执行）

插件与 Gateway **在同一进程中运行**，因此应将其视为受信任代码。
工具编写指南：[Plugin agent tools](/zh/plugins/agent-tools)。

<div id="runtime-helpers">
  ## 运行时辅助工具
</div>

插件可以通过 `api.runtime` 访问选定的核心辅助工具。针对电话场景下的 TTS：

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes:

* 使用核心 `messages.tts` 配置（OpenAI 或 ElevenLabs）。
* 返回 PCM 音频缓冲区和采样率。插件必须针对提供方自行执行重采样/编码。
* Edge TTS 不支持电话语音通话场景。

<div id="discovery-precedence">
  ## 发现与优先顺序
</div>

OpenClaw 按以下顺序进行扫描：

1. 配置路径

* `plugins.load.paths`（文件或目录）

2. 工作区扩展

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. 全局扩展

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. 内置扩展（随 OpenClaw 一起提供，**默认禁用**）

* `<openclaw>/extensions/*`

内置插件必须通过 `plugins.entries.<id>.enabled`
或 `openclaw plugins enable <id>` 显式启用。已安装的插件默认启用，
但也可以用同样的方式禁用。

每个插件必须在其根目录中包含一个 `openclaw.plugin.json` 文件。如果某个路径
指向的是文件，则插件根目录为该文件所在的目录，并且该目录中必须包含
该清单文件。

如果多个插件解析出的 id 相同，则按上述顺序最先匹配到的那个生效，
优先级更低的副本会被忽略。

<div id="package-packs">
  ### Package 打包
</div>

一个插件目录可以包含一个带有 `openclaw.extensions` 字段的 `package.json` 文件：

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

每个条目都会对应一个插件。如果该包列出了多个扩展，插件 ID 将变为 `name/<fileBase>`。

如果你的插件引入了 npm 依赖，请在该目录中安装它们，以确保存在 `node_modules`（`npm install` / `pnpm install`）。

<div id="channel-catalog-metadata">
  ### 通道目录元数据
</div>

通道插件可以通过 `openclaw.channel` 提供接入引导元数据，并通过
`openclaw.install` 提供安装提示信息。这样可以使核心目录本身不存放这些数据。

示例：

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (自托管)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "通过 Nextcloud Talk webhook 机器人实现自托管聊天。",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw 也可以合并**外部通道目录**（例如 MPM 注册表导出）。在以下任一位置放置一个 JSON 文件：

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

或者将 `OPENCLAW_PLUGIN_CATALOG_PATHS`（或 `OPENCLAW_MPM_CATALOG_PATHS`）指向一个或多个 JSON 文件（以逗号 / 分号 / `PATH` 风格分隔）。每个文件应包含 `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`。

<div id="plugin-ids">
  ## 插件 ID
</div>

默认插件 id：

* 包：`package.json` 中的 `name`
* 独立文件：文件基础名（`~/.../voice-call.ts` → `voice-call`）

如果一个插件导出了 `id` 字段，OpenClaw 会使用它，但当它与配置的 id 不匹配时会发出警告。

<div id="config">
  ## 配置
</div>

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } }
    }
  }
}
```

字段：

* `enabled`: 主开关（默认：true）
* `allow`: 允许列表（可选）
* `deny`: 拒绝列表（可选；`deny` 优先）
* `load.paths`: 额外的插件文件/目录
* `entries.<id>`: 每个插件的开关与配置

配置变更后**需要重启 Gateway**。

校验规则（严格）：

* `entries`、`allow`、`deny` 或 `slots` 中未知的插件 id 视为**错误**。
* 未知的 `channels.<id>` 键视为**错误**，除非某个插件清单声明了该 channel id。
* 插件配置会使用嵌入在 `openclaw.plugin.json`（`configSchema`）中的 JSON Schema 进行验证。
* 如果某个插件被禁用，其配置会被保留，并且会触发一条**警告**。

<div id="plugin-slots-exclusive-categories">
  ## 插件槽位（互斥类别）
</div>

某些插件类别是**互斥的**（任意时刻只能有一个处于启用状态）。使用
`plugins.slots` 来指定哪个插件占用该槽位：

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // 或使用 "none" 禁用内存插件
    }
  }
}
```

如果有多个插件声明为 `kind: "memory"`，则只有被选中的那个会加载。其他插件会被禁用，并会输出诊断信息。

<div id="control-ui-schema-labels">
  ## Control UI（schema + 标签）
</div>

Control UI 使用 `config.schema`（JSON Schema + `uiHints`）来渲染更好用的表单。

OpenClaw 会在运行时基于已发现的插件增强 `uiHints`：

* 为 `plugins.entries.<id>` / `.enabled` / `.config` 添加每个插件专属的标签
* 将插件可选提供的配置字段提示合并到：
  `plugins.entries.<id>.config.<field>`

如果你希望插件配置字段能显示良好的标签/占位符（并将机密信息标记为敏感），
请在插件 manifest 中与 JSON Schema 一并提供 `uiHints`。

示例：

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "区域", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # 将本地文件/目录复制到 ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # relative path ok
openclaw plugins install ./plugin.tgz           # install from a local tarball
openclaw plugins install ./plugin.zip           # install from a local zip
openclaw plugins install -l ./extensions/voice-call # link (no copy) for dev
openclaw plugins install @openclaw/voice-call # install from npm
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` 仅适用于在 `plugins.installs` 中跟踪的 npm 安装项。

插件也可以注册自己的顶级命令（例如：`openclaw voicecall`）。

<div id="plugin-api-overview">
  ## 插件 API（概览）
</div>

插件可以导出以下两种形式之一：

* 一个函数：`(api) => { ... }`
* 一个对象：`{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## 插件钩子
</div>

插件可以随自身一起提供钩子，并在运行时注册它们。这样插件就可以在无需单独安装钩子包的情况下，内置事件驱动的自动化能力。

<div id="example">
  ### 示例
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

注意事项：

* Hook 目录遵循常规 Hook 结构（`HOOK.md` + `handler.ts`）。
* Hook 的适用规则仍然生效（操作系统/二进制程序/环境变量/配置要求）。
* 由插件管理的 Hook 会在 `openclaw hooks list` 中以 `plugin:&lt;id&gt;` 的形式显示。
* 你不能通过 `openclaw hooks` 启用或禁用由插件管理的 Hook；请改为启用或禁用该插件本身。

<div id="provider-plugins-model-auth">
  ## 提供方插件（模型认证）
</div>

插件可以注册**模型提供方认证**流程，这样用户就可以在 OpenClaw 内部完成 OAuth 或
API 密钥的配置（无需额外的外部脚本）。

通过 `api.registerProvider(...)` 注册提供方。每个提供方会暴露一种或多种认证方式
（OAuth、API 密钥、设备代码等）。这些方式支持以下命令：

* `openclaw models auth login --provider <id> [--method <id>]`

示例：

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // 运行 OAuth 流程并返回身份验证配置文件。
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

备注：

* `run` 会接收一个 `ProviderAuthContext`，其中包含 `prompter`、`runtime`、
  `openUrl` 和 `oauth.createVpsAwareHandlers` 等辅助方法。
* 当你需要添加默认模型或提供方配置时，返回 `configPatch`。
* 返回 `defaultModel`，这样 `--set-default` 就可以更新智能体的默认模型。

<div id="register-a-messaging-channel">
  ### 注册消息通道
</div>

插件可以注册**通道插件**，其行为与内置通道（WhatsApp、Telegram 等）类似。
通道配置位于 `channels.<id>` 下，由你的通道插件代码负责校验。

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "演示频道插件。",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notes:

* 将配置放在 `channels.<id>` 下（而不是 `plugins.entries`）。
* `meta.label` 用于在 CLI/UI 列表中显示标签。
* `meta.aliases` 为规范化和 CLI 输入添加备用 ID。
* `meta.preferOver` 列出在与当前渠道同时配置时应跳过自动启用的渠道 ID。
* `meta.detailLabel` 和 `meta.systemImage` 使 UI 能展示更丰富的渠道标签/图标。

<div id="write-a-new-messaging-channel-stepbystep">
  ### 编写一个新的消息通道（分步说明）
</div>

当你需要一个**新的聊天入口/界面**（“消息通道”），而不是新的模型提供方时，使用本指南。
模型提供方文档位于 `/providers/*`。

1. 选择一个 id 和配置结构

* 所有通道配置都位于 `channels.<id>` 下。
* 对于多账号场景，优先使用 `channels.<id>.accounts.<accountId>`。

2. 定义通道元数据

* `meta.label`、`meta.selectionLabel`、`meta.docsPath`、`meta.blurb` 控制 CLI/UI 中的列表展示。
* `meta.docsPath` 应指向类似 `/channels/<id>` 的文档页面。
* `meta.preferOver` 允许某个插件替换另一个通道（在自动启用时优先选择它）。
* `meta.detailLabel` 和 `meta.systemImage` 被 UI 用于详情文本/图标。

3. 实现必备适配器

* `config.listAccountIds` + `config.resolveAccount`
* `capabilities`（聊天类型、媒体、线程等）
* `outbound.deliveryMode` + `outbound.sendText`（用于基础文本发送）

4. 按需添加可选适配器

* `setup`（向导）、`security`（私信策略）、`status`（健康检查/诊断）
* `gateway`（启动/停止/登录）、`mentions`、`threading`、`streaming`
* `actions`（消息操作）、`commands`（原生命令行为）

5. 在你的插件中注册通道

* `api.registerChannel({ plugin })`

最小配置示例：

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true }
      }
    }
  }
}
```

最简通道插件（仅发送）：

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat 消息通道。",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // 在此处将 `text` 投递到您的通道
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

加载插件（将其放入 extensions 目录或在 `plugins.load.paths` 中配置），重启 Gateway，
然后在你的配置中设置 `channels.<id>`。

<div id="agent-tools">
  ### Agent 工具
</div>

请参阅专门指南：[Agent 工具插件](/zh/plugins/agent-tools)。

<div id="register-a-gateway-rpc-method">
  ### 注册 Gateway RPC 方法
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### 注册 CLI 命令
</div>

```ts
export default function (api) {
  api.registerCli(({ program }) => {
    program.command("mycmd").action(() => {
      console.log("Hello");
    });
  }, { commands: ["mycmd"] });
}
```

<div id="register-auto-reply-commands">
  ### 注册自动回复命令
</div>

插件可以注册自定义斜杠命令，这些命令在**不调用 AI Agent 代理**的情况下即可执行。对于开关类命令、状态检查，或不需要 LLM 处理的快速操作，这非常有用。

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `插件正在运行!频道:${ctx.channel}`,
    }),
  });
}
```

命令处理器上下文：

* `senderId`: 发送者的 ID（如果可用）
* `channel`: 命令发送所在的 channel
* `isAuthorizedSender`: 发送者是否为已授权用户
* `args`: 命令之后传入的参数（如果 `acceptsArgs: true`）
* `commandBody`: 完整的命令文本
* `config`: 当前的 OpenClaw 配置

命令选项：

* `name`: 命令名称（不包含前缀 `/`）
* `description`: 在命令列表中显示的帮助文本
* `acceptsArgs`: 命令是否接受参数（默认：false）。如果为 false 且提供了参数，则命令不会匹配，该消息将继续交由其他处理器处理
* `requireAuth`: 是否要求发送者已授权（默认：true）
* `handler`: 返回 `{ text: string }` 的函数（可以是异步 async 函数）

带授权和参数的示例：

```ts
api.registerCommand({
  name: "setmode",
  description: "设置插件模式",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Notes:

* 插件命令会在内置命令和 AI Agent 代理**之前**被处理
* 命令在全局范围内注册，可在所有渠道中使用
* 命令名不区分大小写（`/MyStatus` 等同于 `/mystatus`）
* 命令名必须以字母开头，并且只能包含字母、数字、连字符和下划线
* 保留的命令名（如 `help`、`status`、`reset` 等）不能被插件覆盖
* 插件之间重复注册同一命令会失败，并触发诊断错误

<div id="register-background-services">
  ### 注册后台服务
</div>

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

<div id="naming-conventions">
  ## 命名规范
</div>

* Gateway 方法：`pluginId.action`（示例：`voicecall.status`）
* 工具：使用 `snake_case`（示例：`voice_call`）
* CLI 命令：使用 kebab-case 或 camelCase，但要避免与核心命令冲突

<div id="skills">
  ## 技能
</div>

插件可以在仓库中包含一个技能（`skills/<name>/SKILL.md`）。
通过 `plugins.entries.<id>.enabled`（或其他配置开关）启用它，并确保它存在于你的工作区/受管技能目录中。

<div id="distribution-npm">
  ## 分发（npm）
</div>

推荐的打包方式：

* 主包：`openclaw`（本仓库）
* 插件：位于 `@openclaw/*` 命名空间下的独立 npm 包（示例：`@openclaw/voice-call`）

发布规范：

* 插件的 `package.json` 必须包含 `openclaw.extensions`，其中列出一个或多个入口文件。
* 入口文件可以是 `.js` 或 `.ts`（jiti 会在运行时加载 TS 文件）。
* `openclaw plugins install <npm-spec>` 会使用 `npm pack`，将包解压到 `~/.openclaw/extensions/<id>/`，并在配置中启用。
* 配置键稳定性：带 scope 的包会被规范化为在 `plugins.entries.*` 中使用的**无 scope** id。

<div id="example-plugin-voice-call">
  ## 插件示例：语音通话
</div>

本仓库包含一个语音通话插件（Twilio 或日志回退实现）：

* 源码：`extensions/voice-call`
* 技能：`skills/voice-call`
* CLI：`openclaw voicecall start|status`
* 工具：`voice_call`
* RPC：`voicecall.start`、`voicecall.status`
* 配置（twilio）：`provider: "twilio"` + `twilio.accountSid/authToken/from`（可选 `statusCallbackUrl`、`twimlUrl`）
* 配置（开发）：`provider: "log"`（不进行网络访问）

有关安装和使用方法，请参阅 [Voice Call](/zh/plugins/voice-call) 和 `extensions/voice-call/README.md`。

<div id="safety-notes">
  ## 安全注意事项
</div>

插件与 Gateway 在同一进程中运行。请将它们视为可信代码：

* 只安装你信任的插件。
* 优先使用 `plugins.allow` 允许列表。
* 在更改后重启 Gateway。

<div id="testing-plugins">
  ## 测试插件
</div>

插件可以（而且应该）自带测试：

* 仓库内的插件可以将 Vitest 测试放在 `src/**` 目录下（例如：`src/plugins/voice-call.plugin.test.ts`）。
* 单独发布的插件应运行各自的 CI 流水线（lint/build/test），并验证 `openclaw.extensions` 指向已构建好的入口文件（`dist/index.js`）。