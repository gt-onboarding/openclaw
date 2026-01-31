---
summary: "按智能体划分的沙箱与工具限制、优先级和示例"
title: 多智能体沙箱与工具
read_when: "当你需要在多智能体 Gateway 中为各个智能体配置沙箱或工具允许/禁止策略时阅读。"
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # 多智能体沙箱和工具配置
</div>

<div id="overview">
  ## 概览
</div>

在多智能体设置中，每个智能体现在都可以拥有自己的：

* **沙箱配置**（`agents.list[].sandbox` 会覆盖 `agents.defaults.sandbox`）
* **工具限制**（`tools.allow` / `tools.deny`，以及 `agents.list[].tools`）

这使你可以并行运行具有不同安全策略的多个智能体：

* 拥有完整访问权限的个人助理
* 仅允许使用受限工具的家庭 / 工作智能体
* 运行在沙箱中的面向公众的智能体

`setupCommand` 位于 `sandbox.docker` 下（全局或按智能体配置），并会在
容器创建时执行一次。

认证按智能体隔离：每个智能体都会从其自己的 `agentDir` 认证存储中读取，路径为：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

不同智能体之间**不会**共享凭据。切勿在多个智能体之间复用同一个 `agentDir`。
如果你想共享凭据，请将 `auth-profiles.json` 复制到另一个智能体的 `agentDir` 中。

关于沙箱在运行时的行为，请参见 [Sandboxing](/zh/gateway/sandboxing)。
若要调试“为什么会被拦截？”，请参见 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated) 和 `openclaw sandbox explain`。

***

<div id="configuration-examples">
  ## 配置示例
</div>

<div id="example-1-personal-restricted-family-agent">
  ### 示例 1：个人 + 权限受限家庭 Agent 代理
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**结果：**

* `main` 智能体：在宿主机上运行，可访问全部工具
* `family` 智能体：在 Docker 中运行（每个智能体一个容器），仅有 `read` 工具

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### 示例 2：使用共享沙箱的工作 Agent 代理
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### 示例 2b：全局编码配置 + 仅消息型智能体
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**结果：**

* 默认智能体具备编程工具
* `support` 智能体仅支持消息收发（+ Slack 工具）

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### 示例 3：为不同 Agent 代理设置不同的沙箱模式
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // 覆盖:public 始终沙箱化
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## 配置优先级
</div>

当同时存在全局配置（`agents.defaults.*`）和智能体级配置（`agents.list[].*`）时：

<div id="sandbox-config">
  ### 沙箱配置
</div>

Agent 代理级别的设置优先于全局设置：

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意：**

* 对于该智能体，`agents.list[].sandbox.{docker,browser,prune}.*` 会覆盖 `agents.defaults.sandbox.{docker,browser,prune}.*`（当沙箱的 scope 解析为 `"shared"` 时将被忽略）。

<div id="tool-restrictions">
  ### 工具限制
</div>

筛选顺序为：

1. **工具配置文件**（`tools.profile` 或 `agents.list[].tools.profile`）
2. **提供方工具配置文件**（`tools.byProvider[provider].profile` 或 `agents.list[].tools.byProvider[provider].profile`）
3. **全局工具策略**（`tools.allow` / `tools.deny`）
4. **提供方工具策略**（`tools.byProvider[provider].allow/deny`）
5. **特定 Agent 的工具策略**（`agents.list[].tools.allow/deny`）
6. **特定 Agent 的提供方工具策略**（`agents.list[].tools.byProvider[provider].allow/deny`）
7. **沙箱工具策略**（`tools.sandbox.tools` 或 `agents.list[].tools.sandbox.tools`）
8. **子智能体工具策略**（如适用，`tools.subagents.tools`）

每一层都可以进一步收紧可用工具，但不能重新放行在更早层级中已被拒绝的工具。
如果设置了 `agents.list[].tools.sandbox.tools`，则对于该 Agent，会替代 `tools.sandbox.tools`。
如果设置了 `agents.list[].tools.profile`，则对于该 Agent，会覆盖 `tools.profile`。
提供方工具键可以接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。

<div id="tool-groups-shorthands">
  ### 工具分组（简写）
</div>

工具策略（全局、智能体、沙箱）支持使用 `group:*` 条目，这类条目会展开为多个具体工具：

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供方插件）

<div id="elevated-mode">
  ### 提权模式
</div>

`tools.elevated` 是全局基线（基于发送方的允许列表）。`agents.list[].tools.elevated` 可以针对特定智能体进一步收紧提权（两处都必须允许）。

缓解策略：

* 对不受信任的智能体拒绝 `exec`（`agents.list[].tools.deny: ["exec"]`）
* 避免把会被路由到受限智能体的发送方加入允许列表
* 如果你只希望使用沙箱执行，可全局禁用提权（`tools.elevated.enabled: false`）
* 对敏感智能体在智能体级别禁用提权（`agents.list[].tools.elevated.enabled: false`）

***

<div id="migration-from-single-agent">
  ## 从单个智能体迁移
</div>

**迁移前（单智能体）：**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**之后（多智能体，使用不同配置）：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

旧版 `agent.*` 配置会由 `openclaw doctor` 自动迁移；后续请优先使用 `agents.defaults` 与 `agents.list` 组合。

***

<div id="tool-restriction-examples">
  ## 工具使用限制示例
</div>

<div id="read-only-agent">
  ### 只读 Agent 代理
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### 安全执行 Agent 代理（不修改文件）
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### 仅用于通信的 Agent 代理
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## 常见陷阱：&quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` 是基于 `session.mainKey`（默认为 `"main"`），
而不是基于智能体 ID。群组/频道会话总是会使用它们各自的键，因此
会被视为 &quot;non-main&quot; 并被沙箱化。如果你希望某个智能体始终不启用沙箱，
请设置 `agents.list[].sandbox.mode: "off"`。

***

<div id="testing">
  ## 测试
</div>

在完成多智能体沙箱和工具配置之后：

1. **检查智能体解析：**
   ```exec
   openclaw agents list --bindings
   ```

2. **验证沙箱容器：**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **测试工具限制：**
   * 发送一条需要使用受限工具的消息
   * 验证该智能体无法使用被禁止的工具

4. **监控日志：**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### 即使设置了 `mode: "all"`，Agent 代理仍未在沙箱中运行
</div>

* 检查是否存在覆盖该设置的全局配置 `agents.defaults.sandbox.mode`
* Agent 代理的专用配置优先级更高，因此请设置 `agents.list[].sandbox.mode: "all"`

<div id="tools-still-available-despite-deny-list">
  ### 工具在被列入拒绝列表后仍然可用
</div>

* 检查工具过滤顺序：全局 → 智能体 → 沙箱 → 子智能体
* 每一层只能进一步收紧，不能重新放行已被禁止的工具
* 通过日志验证：`[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### 容器未按智能体级别隔离
</div>

* 在特定智能体的沙箱配置中设置 `scope: "agent"`
* 默认为 `"session"`，即为每个会话创建一个容器

***

<div id="see-also">
  ## 另请参见
</div>

* [多智能体路由](/zh/concepts/multi-agent)
* [沙箱配置](/zh/gateway/configuration#agentsdefaults-sandbox)
* [会话管理](/zh/concepts/session)