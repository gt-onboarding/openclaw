---
title: 代理集合
summary: "`openclaw agents` 命令的 CLI 参考文档（列出/添加/删除/设置身份）"
read_when:
  - 你希望使用多个相互隔离的智能体（工作区 + 路由 + 认证）
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

管理彼此隔离的智能体（工作区 + 身份认证 + 路由）。

相关内容：

* 多智能体路由：[Multi-Agent Routing](/zh/concepts/multi-agent)
* Agent 工作区：[Agent workspace](/zh/concepts/agent-workspace)

<div id="examples">
  ## 示例
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## 身份文件
</div>

每个智能体工作区都可以在工作区根目录下包含一个 `IDENTITY.md`：

* 示例路径：`~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` 会从工作区根目录读取（或从通过 `--identity-file` 显式指定的文件读取）

头像路径会相对于工作区根目录进行解析。

<div id="set-identity">
  ## 设置身份信息
</div>

`set-identity` 会将字段写入 `agents.list[].identity`：

* `name`
* `theme`
* `emoji`
* `avatar`（相对于工作区的路径、http(s) URL，或 data URI）

从 `IDENTITY.md` 加载：

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

显式指定要覆盖的字段：

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

配置示例：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png"
        }
      }
    ]
  }
}
```
