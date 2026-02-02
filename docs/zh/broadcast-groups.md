---
title: 广播组
summary: "向多个智能体广播一条 WhatsApp 消息"
read_when:
  - 配置广播组
  - 调试 WhatsApp 中的多智能体回复
status: experimental
---

<div id="broadcast-groups">
  # 广播组
</div>

**状态：** 实验阶段\
**版本：** 自 2026.1.9 起新增

<div id="overview">
  ## 概览
</div>

Broadcast Groups 允许多个智能体同时处理并回复同一条消息。这样你就可以创建专门化的 Agent 代理团队，在同一个 WhatsApp 群组或私信中协同工作——并且只使用一个电话号码。

当前适用范围：**仅限 WhatsApp**（Web 渠道）。

Broadcast groups 的评估顺序在渠道允许列表和群组激活规则之后。在 WhatsApp 群组中，这意味着只会在 OpenClaw 本来就会回复的情况下触发广播（例如被 @ 提及时，具体取决于你的群组设置）。

<div id="use-cases">
  ## 使用场景
</div>

<div id="1-specialized-agent-teams">
  ### 1. 专门化 Agent 代理团队
</div>

部署多个职责单一且高度聚焦的智能体：

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

每个智能体都会处理同一条消息，并从各自的专业角度给出结果。

<div id="2-multi-language-support">
  ### 2. 多语言支持
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. 质量保证流程
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

<div id="4-task-automation">
  ### 4. 任务自动化
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

<div id="configuration">
  ## 配置
</div>

<div id="basic-setup">
  ### 基本设置
</div>

添加一个顶层的 `broadcast` 部分（与 `bindings` 同级）。键为 WhatsApp 会话标识（peer id）：

* 群聊：群 JID（例如 `120363403215116621@g.us`）
* 私聊：E.164 电话号码（例如 `+15551234567`）

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**结果：** 当 OpenClaw 需要在此聊天中回复时，它会运行全部三个智能体。

<div id="processing-strategy">
  ### 处理策略
</div>

控制智能体处理消息的方式：

<div id="parallel-default">
  #### 并行（默认）
</div>

所有智能体将同时处理：

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### 顺序执行
</div>

智能体按顺序逐个处理（后一个会等待前一个完成）：

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### 完整示例
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## 工作原理
</div>

<div id="message-flow">
  ### 消息流
</div>

1. **传入消息** 到达某个 WhatsApp 群组
2. **广播检查**：系统检查该 peer ID 是否在 `broadcast` 中
3. **如果在广播列表中**：
   * 所有列出的 Agent 代理都会处理该消息
   * 每个 Agent 代理都有自己的会话键和隔离的上下文
   * Agent 代理默认并行处理（也可顺序处理）
4. **如果不在广播列表中**：
   * 使用常规路由规则（第一个匹配的绑定）

注意：广播组不会绕过通道的允许列表或群组激活规则（提及/@mention、命令等）。它们只会改变在一条消息符合处理条件时 *有哪些 Agent 代理会运行*。

<div id="session-isolation">
  ### 会话隔离
</div>

广播组中的每个智能体都维护彼此完全独立的：

* **会话键**（`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`）
* **对话历史**（智能体看不到其他智能体的消息）
* **工作区**（如已配置，则使用单独的沙箱）
* **工具访问权限**（不同的允许/拒绝列表）
* **记忆/上下文**（独立的 IDENTITY.md、SOUL.md 等）
* **群组上下文缓冲区**（用于上下文的最近群组消息）按对端（peer）共享，因此所有广播组中的智能体在被触发时看到的上下文相同

这使得每个智能体可以拥有：

* 不同的性格/人格
* 不同的工具访问权限（例如，只读 vs 读写）
* 不同的模型（例如：opus vs sonnet）
* 不同的已安装技能

<div id="example-isolated-sessions">
  ### 示例：相互隔离的会话
</div>

在群组 `120363403215116621@g.us` 中，包含智能体 `["alfred", "baerbel"]`：

**Alfred 的上下文：**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [用户消息, alfred 的先前回复]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel 的背景：**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

<div id="best-practices">
  ## 最佳实践
</div>

<div id="1-keep-agents-focused">
  ### 1. 让 Agent 代理各自专注
</div>

为每个 Agent 代理设计单一且清晰的职责：

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **推荐：** 每个智能体只负责一项任务
❌ **不推荐：** 一个通用的「dev-helper」智能体

<div id="2-use-descriptive-names">
  ### 2. 使用具有描述性的名称
</div>

明确说明每个智能体的作用：

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. 配置不同的工具访问权限
</div>

仅为智能体配置其真正需要的工具：

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // 读写
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. 监控性能
</div>

在存在大量智能体的情况下，可以考虑：

* 为了提高速度，使用 `"strategy": "parallel"`（默认）
* 将广播组限制在 5–10 个智能体
* 为更简单的智能体使用更快的模型

<div id="5-handle-failures-gracefully">
  ### 5. 优雅地处理故障
</div>

智能体是相互独立的。一个智能体的错误不会阻塞其他智能体：

```
消息 → [Agent A ✓, Agent B ✗ 错误, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

<div id="compatibility">
  ## 兼容性
</div>

<div id="providers">
  ### 提供方
</div>

广播组当前支持：

* ✅ WhatsApp（已实现）
* 🚧 Telegram（规划中）
* 🚧 Discord（规划中）
* 🚧 Slack（规划中）

<div id="routing">
  ### 路由
</div>

广播组会与现有路由机制协同工作：

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: 仅 alfred 响应（正常路由）
* `GROUP_B`: agent1 和 agent2 都响应（广播）

**优先级：**`broadcast` 的优先级高于 `bindings`。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="agents-not-responding">
  ### 多个 Agent 代理未响应
</div>

**检查：**

1. Agent 代理 ID 存在于 `agents.list` 中
2. Peer ID 格式正确（例如 `120363403215116621@g.us`）
3. Agent 代理不在拒绝列表中

**调试：**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### 只有一个 Agent 代理在响应
</div>

**原因：**Peer ID 可能在 `bindings` 中，但不在 `broadcast` 中。

**解决方法：**将其添加到 broadcast 配置中，或从 bindings 中移除。

<div id="performance-issues">
  ### 性能问题
</div>

**如果在有大量智能体时变慢：**

* 减少每个组中的智能体数量
* 使用更轻量的模型（用 sonnet 替代 opus）
* 检查沙箱启动时间

<div id="examples">
  ## 示例
</div>

<div id="example-1-code-review-team">
  ### 示例 1：代码审查团队
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**用户发送：** 代码片段
**响应：**

* code-formatter: &quot;已修正缩进并添加类型提示&quot;
* security-scanner: &quot;⚠️ 第 12 行存在 SQL 注入漏洞&quot;
* test-coverage: &quot;覆盖率为 45%，缺少错误情况的测试&quot;
* docs-checker: &quot;函数 `process_data` 缺少文档字符串（docstring）&quot;

<div id="example-2-multi-language-support">
  ### 示例 2：多语言支持
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## API 参考文档
</div>

<div id="config-schema">
  ### 配置架构
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### 字段
</div>

* `strategy`（可选）：如何处理智能体
  * `"parallel"`（默认）：所有智能体并行处理
  * `"sequential"`：智能体按数组顺序依次处理

* `[peerId]`：WhatsApp 群组 JID、E.164 号码或其他对端 ID
  * 值：应处理消息的智能体 ID 数组

<div id="limitations">
  ## 限制
</div>

1. **最大智能体数量：** 没有硬性上限，但 10 个以上的智能体可能会变慢
2. **共享上下文：** 各智能体不会看到彼此的回复（这是刻意的设计）
3. **消息顺序：** 并行回复可能会以任意顺序到达
4. **速率限制：** 所有智能体都会计入 WhatsApp 的速率限制

<div id="future-enhancements">
  ## 后续增强
</div>

计划功能：

* [ ] 共享上下文模式（智能体可以看到彼此的响应）
* [ ] Agent 代理协同（智能体可以相互发送信号）
* [ ] 动态智能体选择（根据消息内容选择智能体）
* [ ] 智能体优先级（某些智能体会优先于其他智能体响应）

<div id="see-also">
  ## 另请参阅
</div>

* [多智能体配置](/zh/multi-agent-sandbox-tools)
* [路由配置](/zh/concepts/channel-routing)
* [会话管理](/zh/concepts/sessions)