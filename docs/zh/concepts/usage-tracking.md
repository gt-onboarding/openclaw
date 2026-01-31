---
title: 使用情况跟踪
summary: "使用情况跟踪展示与凭证要求"
read_when:
  - 你正在接入提供方的使用/配额展示
  - 你需要解释使用情况跟踪行为或凭证要求
---

<div id="usage-tracking">
  # 使用情况跟踪
</div>

<div id="what-it-is">
  ## 功能说明
</div>

- 直接从提供方的用量 API 端点拉取用量/配额信息。
- 不进行成本预估；仅使用提供方报告的用量时间窗口。

<div id="where-it-shows-up">
  ## 显示位置
</div>

- 聊天中的 `/status`：带有会话 token 和预估费用（仅适用于 API key）的表情丰富状态卡片。在可用时，会显示**当前模型提供方**的用量数据。
- 聊天中的 `/usage off|tokens|full`：在每条回复结尾显示用量页脚（OAuth 只显示 token 数）。
- 聊天中的 `/usage cost`：基于 OpenClaw 会话日志汇总的本地费用统计。
- CLI：`openclaw status --usage` 输出按提供方划分的完整用量明细。
- CLI：`openclaw channels list` 输出与提供方配置并列显示的相同用量快照（使用 `--no-usage` 可跳过）。
- macOS 菜单栏：Context 下的 “Usage” 区域（仅在可用时显示）。

<div id="providers-credentials">
  ## 提供方与凭证
</div>

- **Anthropic (Claude)**：auth profile 中的 OAuth 令牌。
- **GitHub Copilot**：auth profile 中的 OAuth 令牌。
- **Gemini CLI**：auth profile 中的 OAuth 令牌。
- **Antigravity**：auth profile 中的 OAuth 令牌。
- **OpenAI Codex**：auth profile 中的 OAuth 令牌（如存在则使用 accountId）。
- **MiniMax**：API key（coding plan key；`MINIMAX_CODE_PLAN_KEY` 或 `MINIMAX_API_KEY`）；使用 5 小时的 coding plan 时间窗口。
- **z.ai**：通过环境变量 / 配置 / 认证存储（env/config/auth store）提供的 API key。

如果不存在匹配的 OAuth/API 凭证，则使用情况将被隐藏。