---
title: 技能配置
summary: "技能配置模式与示例"
read_when:
  - 添加或修改技能配置
  - 调整内置允许列表或安装行为
---

<div id="skills-config">
  # 技能配置
</div>

所有与技能相关的配置都定义在 `~/.openclaw/openclaw.json` 文件的 `skills` 部分下。

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (Gateway 运行时仍使用 Node;不推荐 bun)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## 字段
</div>

- `allowBundled`：仅针对 **捆绑** 技能的可选允许列表。设置后，只有该列表中的捆绑技能才可用（托管/工作区技能不受影响）。
- `load.extraDirs`：要额外扫描的技能目录（优先级最低）。
- `load.watch`：监听技能文件夹并刷新技能快照（默认：true）。
- `load.watchDebounceMs`：技能监听事件的防抖时间，单位为毫秒（默认：250）。
- `install.preferBrew`：在可用时优先使用 brew 安装器（默认：true）。
- `install.nodeManager`：Node.js 安装器偏好（`npm` | `pnpm` | `yarn` | `bun`，默认：npm）。
  这只影响**技能安装**；Gateway 运行时仍应使用 Node.js
  （不推荐将 Bun 用于 WhatsApp/Telegram）。
- `entries.<skillKey>`：按技能进行覆写的配置。

按技能设置的字段：

- `enabled`：设置为 `false` 可禁用某个技能，即使它已捆绑/已安装。
- `env`：在智能体运行时注入的环境变量（仅在尚未设置时注入）。
- `apiKey`：为声明了主环境变量的技能提供的可选便捷字段。

<div id="notes">
  ## 说明
</div>

- `entries` 下的键默认对应技能名称。如果某个技能定义了
  `metadata.openclaw.skillKey`，则优先使用该键。
- 启用监视器时，对技能的修改会在下一次智能体回合中被检测到并生效。

<div id="sandboxed-skills-env-vars">
  ### 沙箱化技能 + 环境变量
</div>

当会话处于**沙箱**状态时，技能进程会在 Docker 容器中运行。沙箱
**不会**继承宿主机的 `process.env`。

请选择以下方式之一：

- 使用 `agents.defaults.sandbox.docker.env`（或针对单个智能体的 `agents.list[].sandbox.docker.env`）
- 将环境变量直接烘焙进你的自定义沙箱镜像中

全局 `env` 和 `skills.entries.<skill>.env/apiKey` 仅适用于在**宿主机**上直接运行技能的情况。