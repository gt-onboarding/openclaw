---
title: 创建自定义技能
---

<div id="creating-custom-skills">
  # 创建自定义技能 🛠
</div>

OpenClaw 的设计目标是便于扩展。“技能”是为你的助手添加新能力的主要方式。

<div id="what-is-a-skill">
  ## 什么是 Skill？
</div>

一个 Skill 是一个目录，其中包含一个 `SKILL.md` 文件（为 LLM 提供说明和工具定义），并且可以选择性地包含一些脚本或资源。

<div id="step-by-step-your-first-skill">
  ## 分步指南：你的第一个技能
</div>

<div id="1-create-the-directory">
  ### 1. 创建目录
</div>

技能位于你的工作区中，通常在 `~/.openclaw/workspace/skills/` 路径下。为你的技能创建一个新的目录：

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. 定义 `SKILL.md`
</div>

在该目录中创建一个 `SKILL.md` 文件。该文件使用 YAML frontmatter 提供元数据，并使用 Markdown 编写具体说明。

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. 添加工具（可选）
</div>

你可以在 frontmatter 中定义自定义工具，或者让智能体使用现有的系统工具（例如 `bash` 或 `browser`）。

<div id="4-refresh-openclaw">
  ### 4. 刷新 OpenClaw
</div>

让你的智能体执行 “refresh skills” 命令，或者重启 Gateway。OpenClaw 会识别到新的目录，并为 `SKILL.md` 建立索引。

<div id="best-practices">
  ## 最佳实践
</div>

- **保持简洁**：告诉模型要*做什么*，而不是教它如何当一个 AI。
- **安全优先**：如果你的技能使用 `bash`，确保提示词不会让不受信任的用户输入能注入任意命令。
- **本地测试**：使用 `openclaw agent --message "use my new skill"` 进行测试。

<div id="shared-skills">
  ## 共享技能
</div>

你也可以在 [ClawHub](https://clawhub.com) 浏览他人共享的技能，并贡献自己的技能。