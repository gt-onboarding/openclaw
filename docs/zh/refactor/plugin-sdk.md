---
title: 插件 SDK
summary: "计划：为所有消息通道连接器提供一个统一、干净的插件 SDK 与运行时"
read_when:
  - 定义或重构插件架构时
  - 将通道连接器迁移到插件 SDK/运行时时
---

<div id="plugin-sdk-runtime-refactor-plan">
  # 插件 SDK 与运行时重构计划
</div>

目标：每个消息连接器都是一个使用单一稳定 API 的插件（内置或外部）。
任何插件都不能直接从 `src/**` 导入。所有依赖必须通过 SDK 或运行时访问。

<div id="why-now">
  ## 为什么是现在
</div>

* 当前的连接器混用多种模式：直接导入核心模块、仅针对 dist 的桥接层，以及自定义 helper/辅助工具。
* 这使得升级变得脆弱、易出问题，并阻碍了构建干净的外部插件接口层。

<div id="target-architecture-two-layers">
  ## 目标架构（两层结构）
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) 插件 SDK（编译时，稳定，可发布）
</div>

范围：类型、辅助方法和配置工具。无运行时状态，无副作用。

内容（示例）：

* 类型：`ChannelPlugin`、适配器、`ChannelMeta`、`ChannelCapabilities`、`ChannelDirectoryEntry`。
* 配置辅助方法：`buildChannelConfigSchema`、`setAccountEnabledInConfigSection`、`deleteAccountFromConfigSection`、
  `applyAccountNameToChannelSection`。
* 配对辅助方法：`PAIRING_APPROVED_MESSAGE`、`formatPairingApproveHint`。
* 上手 / 引导辅助方法：`promptChannelAccessConfig`、`addWildcardAllowFrom`、上手相关类型。
* 工具参数辅助方法：`createActionGate`、`readStringParam`、`readNumberParam`、`readReactionParams`、`jsonResult`。
* 文档链接辅助方法：`formatDocsLink`。

交付：

* 以 `openclaw/plugin-sdk` 形式发布（或从 core 中以 `openclaw/plugin-sdk` 进行导出）。
* 使用语义化版本（SemVer），并提供明确的稳定性保证。

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) 插件运行时（执行层，注入式）
</div>

范围：所有会触及核心运行时行为的内容。
通过 `OpenClawPluginApi.runtime` 访问，因此插件永远不直接 import `src/**`。

拟议的运行时 API 面（精简但完整）：

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: { text?: string; mediaUrls?: string[]; mediaUrl?: string }) =>
            void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // Teams 风格流的适配器
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: "dm" | "group" | "channel"; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(cfg: OpenClawConfig, channel: string, accountId: string, groupId: string): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```

Notes:

* Runtime 是访问核心行为的唯一入口。
* SDK 被有意设计得精简且稳定。
* 每个 Runtime 方法都一一对应到现有的核心实现（不做重复实现）。

<div id="migration-plan-phased-safe">
  ## 迁移计划（分阶段且安全）
</div>

<div id="phase-0-scaffolding">
  ### 阶段 0：脚手架搭建
</div>

* 引入 `openclaw/plugin-sdk`。
* 在 `OpenClawPluginApi` 中添加 `api.runtime`，其 API 形态如上所述。
* 在过渡期内保留现有的导入语句（同时给出弃用警告）。

<div id="phase-1-bridge-cleanup-low-risk">
  ### 阶段 1：bridge 清理（低风险）
</div>

* 将各扩展中的 `core-bridge.ts` 替换为 `api.runtime`。
* 优先迁移 BlueBubbles、Zalo、Zalo Personal（这些已经基本就绪）。
* 移除重复的 bridge 代码。

<div id="phase-2-light-direct-import-plugins">
  ### 阶段 2：轻量级直接导入型插件
</div>

* 将 Matrix 迁移到 SDK + 运行时。
* 验证引导流程、目录、群组 @ 提及逻辑。

<div id="phase-3-heavy-direct-import-plugins">
  ### 阶段 3：重度依赖 direct-import 的插件
</div>

* 迁移 MS Teams（其拥有规模最大的运行时 helper 集合）。
* 确保回复/输入状态语义与当前行为一致。

<div id="phase-4-imessage-pluginization">
  ### 第 4 阶段：iMessage 插件化
</div>

* 将 iMessage 移动到 `extensions/imessage` 中。
* 用 `api.runtime` 替代对核心的直接调用。
* 保持配置键、CLI 行为和文档不变。

<div id="phase-5-enforcement">
  ### 第 5 阶段：强制执行
</div>

* 添加 lint 规则 / CI 检查：禁止在 `src/**` 中导入 `extensions/**`。
* 添加插件 SDK 版本兼容性检查（运行时与 SDK 的 semver）。

<div id="compatibility-and-versioning">
  ## 兼容性与版本管理
</div>

* SDK：遵循 semver，发布并文档化变更。
* Runtime：随核心发行版进行版本管理。添加 `api.runtime.version`。
* 插件需声明所需的 runtime 版本范围（例如 `openclawRuntime: ">=2026.2.0"`）。

<div id="testing-strategy">
  ## 测试策略
</div>

* 适配器级单元测试（在真实核心实现上运行 runtime 函数）。
* 针对每个插件的 golden 测试：确保行为无漂移（路由、配对、允许列表、基于 @ 提及的门控）。
* 在 CI 中使用单个端到端插件示例（安装 + 运行 + 冒烟测试）。

<div id="open-questions">
  ## 未决问题
</div>

* SDK 类型应放在哪里：单独的包还是由 core 导出？
* 运行时类型的分发方式：放在 SDK（仅类型）还是放在 core？
* 如何为内置插件和外部插件提供文档链接入口？
* 过渡期间是否允许仓库内插件以受限方式直接从 core 导入？

<div id="success-criteria">
  ## 成功标准
</div>

* 所有通道连接器都作为插件，基于 SDK + 运行时 实现。
* 不再从 `src/**` 导入 `extensions/**`。
* 新的连接器模板只依赖 SDK + 运行时。
* 外部插件可以在无需访问核心源码的情况下独立开发和更新。

相关文档：[插件](/zh/plugin)、[通道](/zh/channels/index)、[配置](/zh/gateway/configuration)。