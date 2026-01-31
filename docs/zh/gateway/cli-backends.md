---
title: CLI 后端
summary: "CLI 后端：通过本地 AI CLI 实现纯文本回退"
read_when:
  - 当 API 提供方故障时，你需要一个可靠的回退方案
  - 你正在运行 Claude Code CLI 或其他本地 AI CLI，并希望复用它们
  - 你需要一条纯文本、无工具的路径，但仍然支持会话和图像
---

<div id="cli-backends-fallback-runtime">
  # CLI backends（后备运行时）
</div>

OpenClaw 可以在 **API 提供方不可用**、**触发限流** 或 **短暂异常** 时，运行 **本地 AI CLI** 作为 **纯文本后备方案**。此模式刻意设计得比较保守：

- **禁用工具**（不进行 tool 调用）。
- **文本输入 → 文本输出**（可靠）。
- **支持会话**（保证后续轮次对话保持连贯）。
- **如果该 CLI 接受图像路径，则图像可以原样透传**。

这一设计是作为一种 **安全兜底机制**，而不是主要使用方案。当你需要在不依赖外部 API 的情况下，获得“始终可用”的文本回复时再使用它。

<div id="beginner-friendly-quick-start">
  ## 面向初学者的快速入门
</div>

你可以在**无需任何配置**的情况下使用 Claude Code CLI（OpenClaw 已内置一份默认配置）：

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI 也开箱即可使用：

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

如果你的 Gateway 是通过 launchd/systemd 运行的，并且 PATH 被裁剪得很小，只需添加命令的完整路径：

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

就这样。除了 CLI 本身，不需要任何密钥，也不需要额外的认证配置。


<div id="using-it-as-a-fallback">
  ## 将其作为兜底方案使用
</div>

在回退列表中添加一个 CLI 后端，使其仅在主模型失败时才运行：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

注意事项：

* 如果你使用 `agents.defaults.models`（允许列表），必须包含 `claude-cli/...`。
* 如果主提供方出现故障（认证失败、速率限制、超时等），OpenClaw 会
  尝试改用 CLI 后端。


<div id="configuration-overview">
  ## 配置概览
</div>

所有 CLI 后端都在以下位置：

```
agents.defaults.cliBackends
```

每个条目都以**提供方 ID**作为键名（例如 `claude-cli`、`my-cli`）。
该提供方 ID 会成为你模型引用（model ref）中的左侧部分：

```
<provider>/<model>
```


<div id="example-configuration">
  ### 配置示例
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## 工作原理
</div>

1) 根据提供方前缀（`claude-cli/...`）**选择后端**。
2) 使用同一套 OpenClaw 提示词和工作区上下文**构建系统提示词**。
3) **执行 CLI**，并携带会话 ID（若支持），以保证历史记录保持一致。
4) **解析输出**（JSON 或纯文本），并返回最终文本。
5) 按后端**持久化会话 ID**，这样后续请求会复用同一个 CLI 会话。

<div id="sessions">
  ## 会话
</div>

- 如果 CLI 支持会话，当需要将 ID 插入到多个参数中时，设置 `sessionArg`（例如 `--session-id`）或
  `sessionArgs`（占位符 `{sessionId}`）。
- 如果 CLI 使用带有不同参数的 **resume 子命令**，设置
  `resumeArgs`（在恢复时用于替换 `args`），并在需要时设置 `resumeOutput`
  （用于非 JSON 格式的恢复输出）。
- `sessionMode`：
  - `always`：始终发送会话 ID（如果之前未存储，则生成新的 UUID）。
  - `existing`：只有在之前已存储会话 ID 时才发送。
  - `none`：从不发送会话 ID。

<div id="images-pass-through">
  ## 图像（透传）
</div>

如果你的 CLI 支持图像路径输入，请设置 `imageArg`：

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw 会将 base64 编码的图像写入临时文件。如果设置了 `imageArg`，这些路径会作为 CLI 参数传入。如果未设置 `imageArg`，OpenClaw 会将文件路径追加到提示中（路径注入），对于会从纯路径自动加载本地文件的 CLI（例如 Claude Code CLI 的行为），这就已经足够了。


<div id="inputs-outputs">
  ## 输入 / 输出
</div>

- `output: "json"`（默认）尝试解析 JSON，并提取文本和会话 ID。
- `output: "jsonl"` 解析 JSONL 数据流（Codex CLI `--json`），并在存在时提取
  最后一条智能体消息以及 `thread_id`。
- `output: "text"` 将 stdout 视为最终的响应结果。

输入模式：

- `input: "arg"`（默认）将 prompt 作为最后一个 CLI 参数传递。
- `input: "stdin"` 通过 stdin 发送 prompt。
- 如果 prompt 非常长并且设置了 `maxPromptArgChars`，则会使用 stdin。

<div id="defaults-built-in">
  ## 默认值（内置）
</div>

OpenClaw 为 `claude-cli` 内置了一组默认配置：

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw 也为 `codex-cli` 内置了一组默认配置：

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

仅在需要时再进行覆写（常见用例：将 `command` 设置为绝对路径）。

<div id="limitations">
  ## 限制
</div>

- **无法使用 OpenClaw 工具**（CLI 后端不会接收到任何工具调用）。某些 CLI
  仍然可能运行它们自己的智能体工具链。
- **不支持流式输出**（CLI 输出会先被收集，然后再统一返回）。
- **结构化输出** 依赖于该 CLI 输出的 JSON 格式。
- **Codex CLI 会话** 通过文本输出恢复（没有 JSONL），其结构化程度低于初始使用 `--json` 运行时的输出。OpenClaw 会话仍然可以正常工作。

<div id="troubleshooting">
  ## 故障排查
</div>

- **找不到 CLI**：将 `command` 设置为完整路径。
- **模型名称错误**：使用 `modelAliases` 将 `provider/model` 映射到 CLI 模型。
- **会话无法保持连续**：确保已设置 `sessionArg` 且 `sessionMode` 不为
  `none`（Codex CLI 目前在 JSON 输出模式下无法恢复会话）。
- **图像被忽略**：设置 `imageArg`（并确认 CLI 支持文件路径）。