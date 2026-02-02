---
title: 严格配置
summary: "严格配置校验 + 仅通过 doctor 执行的迁移"
read_when:
  - 设计或实现配置验证行为时
  - 处理配置迁移或 doctor 工作流时
  - 处理插件配置 schema 或插件加载门控逻辑时
---

<div id="strict-config-validation-doctor-only-migrations">
  # 严格配置校验（仅由 doctor 执行的迁移）
</div>

<div id="goals">
  ## 目标
</div>

- **在所有层级拒绝未知配置键**（根级 + 嵌套）。
- **拒绝没有 schema 的插件配置**；不要加载该插件。
- **移除加载时的旧版自动迁移机制**；迁移仅通过 doctor 运行。
- **启动时自动以 dry-run 方式运行 doctor**；如果配置无效，阻止非诊断类命令的执行。

<div id="non-goals">
  ## 非目标
</div>

- 加载时的向后兼容性（不会在加载时自动迁移旧字段）。
- 静默丢弃无法识别的字段。

<div id="strict-validation-rules">
  ## 严格校验规则
</div>

- 配置在每一层级都必须与 schema 精确匹配。
- 未知键一律视为校验错误（根级或嵌套级都不允许透传）。
- `plugins.entries.<id>.config` 必须由对应插件的 schema 进行校验。
  - 如果某个插件缺少 schema，**拒绝加载该插件**，并返回清晰的错误信息。
- 未知的 `channels.<id>` 键一律视为错误，除非有插件 manifest 声明了该 channel id。
- 所有插件都必须提供插件 manifest（`openclaw.plugin.json`）。

<div id="plugin-schema-enforcement">
  ## 插件 schema 强制校验
</div>

- 每个插件都会为其配置提供一份严格的 JSON Schema（内联在 manifest 中）。
- 插件加载流程：
  1) 解析插件 manifest 和 schema（`openclaw.plugin.json`）。
  2) 根据 schema 校验配置。
  3) 如果缺少 schema 或配置无效：阻止加载插件，并记录错误。
- 错误信息包括：
  - 插件 id
  - 原因（缺少 schema / 配置无效）
  - 未通过校验的路径
- 被禁用的插件会保留其配置，但 Doctor 和日志中会显示一条警告。

<div id="doctor-flow">
  ## Doctor 流程
</div>

- 每次加载配置时，都会运行 Doctor（默认以 dry-run 方式执行）。
- 如果配置无效：
  - 输出摘要和可操作的错误信息。
  - 提示执行：`openclaw doctor --fix`。
- `openclaw doctor --fix`：
  - 执行迁移操作。
  - 移除未知键。
  - 写回更新后的配置。

<div id="command-gating-when-config-is-invalid">
  ## 命令限制（当配置无效时）
</div>

允许（仅限诊断用途）：

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

其他所有命令都必须直接报错并退出，提示：“配置无效。请运行 `openclaw doctor --fix`。”

<div id="error-ux-format">
  ## 错误 UX 展示格式
</div>

- 单一摘要标题。
- 分组部分：
  - 未知键（完整路径）
  - 旧版键 / 需要迁移
  - 插件加载失败（插件 ID + 原因 + 路径）

<div id="implementation-touchpoints">
  ## 实现接入点
</div>

- `src/config/zod-schema.ts`：移除根级 passthrough；所有对象都改为严格对象模式。
- `src/config/zod-schema.providers.ts`：确保严格的 channel schema。
- `src/config/validation.ts`：在存在未知键时直接失败；不再应用旧版迁移逻辑。
- `src/config/io.ts`：移除旧版自动迁移；始终运行 doctor 空跑（dry-run）。
- `src/config/legacy*.ts`：将其使用场景仅保留在 doctor 中。
- `src/plugins/*`：添加 schema 注册表和门控控制（gating）。
- 在 `src/cli` 中对 CLI 命令进行门控控制（gating）。

<div id="tests">
  ## 测试
</div>

- 拒绝未知键（根级 + 嵌套）。
- 插件缺少 schema → 阻止加载该插件，并给出清晰的错误信息。
- 无效配置 → 阻止 Gateway 启动（诊断命令除外）。
- Doctor 默认以 dry-run 方式运行；`doctor --fix` 会写入修正后的配置。