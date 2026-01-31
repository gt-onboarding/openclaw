---
title: 沙箱 CLI
summary: "管理沙箱容器并检查实际生效的沙箱策略"
read_when: "当你需要管理沙箱容器或调试沙箱/工具策略行为时"
status: active
---

<div id="sandbox-cli">
  # 沙箱 CLI
</div>

管理基于 Docker 的沙箱容器，用于在隔离环境中运行智能体。

<div id="overview">
  ## 概述
</div>

出于安全考虑，OpenClaw 可以在隔离的 Docker 容器中运行智能体。`sandbox` 命令帮助你管理这些容器，特别是在进行更新或修改配置之后。

<div id="commands">
  ## 命令
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

检查**有效**的沙箱模式/scope/工作区访问权限、沙箱工具策略，以及所有提权门控（附带可用于修复的配置键路径）。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

列出所有沙箱容器及其各自的状态和配置信息。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # 仅列出浏览器容器
openclaw sandbox list --json     # JSON 输出
```

**输出内容包括：**

* 容器名称和状态（运行中/已停止）
* Docker 镜像以及是否与配置一致
* 存在时长（自创建以来的时间）
* 空闲时间（自上次使用以来的时间）
* 关联的会话/智能体

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

删除沙箱容器，以强制使用更新后的镜像和配置重新创建。

```bash
openclaw sandbox recreate --all                # 重新创建所有容器
openclaw sandbox recreate --session main       # 指定会话
openclaw sandbox recreate --agent mybot        # 指定智能体
openclaw sandbox recreate --browser            # 仅浏览器容器
openclaw sandbox recreate --all --force        # 跳过确认
```

**选项：**

* `--all`: 重新创建所有沙箱容器
* `--session <key>`: 为特定会话重新创建容器
* `--agent <id>`: 为特定智能体重新创建容器
* `--browser`: 仅重新创建浏览器容器
* `--force`: 跳过确认提示

**重要说明：** 下次使用该智能体时，这些容器会自动重新创建。

<div id="use-cases">
  ## 使用场景
</div>

<div id="after-updating-docker-images">
  ### 更新 Docker 镜像后
</div>

```bash
# 拉取新镜像
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 更新配置以使用新镜像
# 编辑配置:agents.defaults.sandbox.docker.image (或 agents.list[].sandbox.docker.image)

# 重新创建容器
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### 在更改沙箱配置后
</div>

```bash
# 编辑配置：agents.defaults.sandbox.* (或 agents.list[].sandbox.*)

# 重新创建沙箱以应用新配置
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### 修改 setupCommand 之后
</div>

```bash
openclaw sandbox recreate --all
# 或仅针对单个智能体:
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### 仅适用于特定智能体
</div>

```bash
# 仅更新单个智能体的容器
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## 为什么需要这个命令？
</div>

**问题：** 当你更新沙箱 Docker 镜像或配置时：

* 现有容器会继续以旧配置运行
* 容器只会在连续 24 小时无活动后才会被清理
* 经常使用的智能体会让旧容器一直运行下去

**解决方案：** 使用 `openclaw sandbox recreate` 强制移除旧容器。下次需要时，它们会根据当前配置自动重新创建。

提示：优先使用 `openclaw sandbox recreate`，而不是手动执行 `docker rm`。它会使用 Gateway 的容器命名规则，并在 scope/会话键变更时避免不匹配问题。

<div id="configuration">
  ## 配置
</div>

沙箱设置位于 `~/.openclaw/openclaw.json` 文件中的 `agents.defaults.sandbox` 键下（每个智能体的覆盖配置放在 `agents.list[].sandbox` 中）：

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... 更多 Docker 选项
        },
        "prune": {
          "idleHours": 24,               // 空闲 24 小时后自动清理
          "maxAgeDays": 7                // 7 天后自动清理
        }
      }
    }
  }
}
```

<div id="see-also">
  ## 另请参阅
</div>

* [沙箱文档](/zh/gateway/sandboxing)
* [Agent 代理配置](/zh/concepts/agent-workspace)
* [Doctor 命令](/zh/gateway/doctor) - 检查沙箱配置