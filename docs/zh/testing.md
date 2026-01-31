---
title: 测试
summary: "测试工具套件：单元/e2e/实时测试套件、Docker 运行环境，以及各类测试的覆盖范围"
read_when:
  - 当你在本地或 CI 中运行测试时
  - 当你为模型/提供方缺陷添加回归测试时
  - 当你调试 Gateway 和智能体行为时
---

<div id="testing">
  # 测试
</div>

OpenClaw 提供三个 Vitest 测试套件（单元/集成、e2e、live），以及少量 Docker runner。

本文档是一份“我们如何测试”的指南：

* 每个测试套件的覆盖范围（以及它*刻意*不覆盖的内容）
* 在常见工作流（本地、pre-push、调试）中需要运行哪些命令
* live 测试如何获取凭据并选择模型/提供方
* 如何为真实环境中的模型/提供方问题添加回归测试

<div id="quick-start">
  ## 快速开始
</div>

大多数情况下：

* 完整门控（推送前建议执行）：`pnpm lint && pnpm build && pnpm test`

当你改动了测试或需要更多信心时：

* 覆盖率门控：`pnpm test:coverage`
* E2E 测试套件：`pnpm test:e2e`

在调试真实提供方/模型时（需要真实凭证）：

* 联机测试套件（模型 + Gateway 工具/图像探针）：`pnpm test:live`

提示：当你只需要定位某一个失败用例时，优先通过下文所述的允许列表环境变量来缩小联机测试范围。

<div id="test-suites-what-runs-where">
  ## 测试套件（在哪运行什么）
</div>

可以把这些套件看作是“越来越接近真实环境”（同时不稳定性和成本也随之增加）：

<div id="unit-integration-default">
  ### 单元 / 集成（默认）
</div>

* 命令：`pnpm test`
* 配置：`vitest.config.ts`
* 文件：`src/**/*.test.ts`
* Scope：
  * 纯单元测试
  * 进程内集成测试（Gateway 认证、路由、工具、解析、配置）
  * 已知缺陷的确定性回归用例
* 预期：
  * 在 CI 中运行
  * 无需真实密钥
  * 应当快速且稳定

<div id="e2e-gateway-smoke">
  ### E2E（Gateway 冒烟测试）
</div>

* Command: `pnpm test:e2e`
* Config: `vitest.e2e.config.ts`
* Files: `src/**/*.e2e.test.ts`
* Scope:
  * 多实例 Gateway 的端到端行为
  * WebSocket/HTTP 接口、节点配对，以及更复杂的网络交互
* Expectations:
  * 在 CI 流水线中运行（启用后）
  * 不需要真实 API 密钥
  * 涉及的组件比单元测试更多（可能更慢）

<div id="live-real-providers-real-models">
  ### 在线（真实提供方 + 真实模型）
</div>

* 命令：`pnpm test:live`
* 配置：`vitest.live.config.ts`
* 文件：`src/**/*.live.test.ts`
* 默认：通过 `pnpm test:live` **启用**（设置 `OPENCLAW_LIVE_TEST=1`）
* Scope：
  * “这个提供方/模型在**今天**用真实凭据真的能正常工作吗？”
  * 捕捉提供方格式变更、工具调用怪异行为、认证问题以及限流行为
* 预期：
  * 设计上就**不会**在 CI 中保持稳定（真实网络、真实提供方策略、配额、故障）
  * 会花钱 / 会占用速率限制配额
  * 建议只运行收窄后的子集，而不是“一股脑全跑”
  * 在线测试会读取 `~/.profile` 来获取缺失的 API 密钥
  * Anthropic 密钥轮换：设置 `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."`（或 `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`），或设置多个 `ANTHROPIC_API_KEY*` 变量；测试会在遇到限流时自动重试

<div id="which-suite-should-i-run">
  ## 我应该运行哪个测试套件？
</div>

使用下面的决策表：

* 编辑逻辑或测试时：运行 `pnpm test`（如果改动较多，也运行 `pnpm test:coverage`）
* 修改 Gateway 网络栈 / WS 协议 / 配对流程时：另外加上 `pnpm test:e2e`
* 排查“我的 bot 掉线了” / 特定提供方的故障 / 工具调用问题时：运行范围收窄的 `pnpm test:live`

<div id="live-model-smoke-profile-keys">
  ## 在线：模型冒烟测试（配置档密钥）
</div>

在线测试被分成两层，这样可以隔离不同类型的失败：

* “直接模型”告诉我们，在给定密钥下，该提供方/模型是否至少能返回响应。
* “Gateway 冒烟测试”告诉我们，对该模型而言，完整的 Gateway + Agent 代理流水线是否工作正常（会话、历史记录、工具、沙箱策略等）。

<div id="layer-1-direct-model-completion-no-gateway">
  ### 第 1 层：直接模型补全（不经由 Gateway）
</div>

* 测试：`src/agents/models.profiles.live.test.ts`
* 目标：
  * 枚举已发现的模型
  * 使用 `getApiKeyForModel` 选择你有凭据的模型
  * 对每个模型运行一次小型补全请求（以及在需要时执行有针对性的回归测试）
* 如何启用：
  * 运行 `pnpm test:live`（或在直接调用 Vitest 时设置 `OPENCLAW_LIVE_TEST=1`）
* 将 `OPENCLAW_LIVE_MODELS=modern`（或 `all`，即 modern 的别名）设置为实际运行这一测试套件；否则会跳过，以便让 `pnpm test:live` 主要聚焦于 Gateway 冒烟测试
* 如何选择模型：
  * `OPENCLAW_LIVE_MODELS=modern` 运行 modern 允许列表（Opus/Sonnet/Haiku 4.5、GPT-5.x + Codex、Gemini 3、GLM 4.7、MiniMax M2.1、Grok 4）
  * `OPENCLAW_LIVE_MODELS=all` 是 modern 允许列表的别名
  * 或者 `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."`（逗号分隔的允许列表）
* 如何选择提供方：
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"`（逗号分隔的允许列表）
* 密钥来源：
  * 默认：profile 存储和环境变量回退
  * 设置 `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` 以强制仅使用 **profile 存储**
* 设置此层的目的：
  * 将“提供方 API 异常 / 密钥无效”与“Gateway 智能体流水线异常”区分开来
  * 覆盖一些小而独立的回归场景（例如：OpenAI Responses/Codex Responses 的推理重放 + 工具调用流程）

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### 第 2 层：Gateway + dev 智能体冒烟测试（“@openclaw” 实际在做什么）
</div>

* 测试：`src/gateway/gateway-models.profiles.live.test.ts`
* 目标：
  * 启动一个进程内 Gateway
  * 创建/更新一个 `agent:dev:*` 会话（每次运行可覆盖模型）
  * 遍历带密钥配置的模型并断言：
    * 返回“有意义”的响应（不使用工具）
    * 一次真实的工具调用可正常工作（`read` 探针）
    * 可选的额外工具探针（`exec+read` 探针）
    * OpenAI 回归测试路径（仅工具调用 → 后续对话）保持可用
* 探针详情（便于你快速解释失败原因）：
  * `read` 探针：测试在工作区写入一个 nonce（一次性随机值）文件，并让智能体 `read` 该文件并回显该 nonce。
  * `exec+read` 探针：测试让智能体通过 `exec` 将 nonce 写入一个临时文件，然后再 `read` 回来。
  * 图像探针：测试附加一个生成的 PNG（猫 + 随机代码），并期望模型返回 `cat <CODE>`。
  * 实现参考：`src/gateway/gateway-models.profiles.live.test.ts` 和 `src/gateway/live-image-probe.ts`。
* 如何启用：
  * `pnpm test:live`（或者在直接调用 Vitest 时设置 `OPENCLAW_LIVE_TEST=1`）
* 如何选择模型：
  * 默认：现代允许列表（Opus/Sonnet/Haiku 4.5、GPT-5.x + Codex、Gemini 3、GLM 4.7、MiniMax M2.1、Grok 4）
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` 是现代允许列表的别名
  * 或者设置 `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"`（或逗号分隔列表）以收窄范围
* 如何选择提供方（避免“OpenRouter 全都上”）：
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"`（逗号分隔允许列表）
* 工具 + 图像探针在此实时测试中始终启用：
  * `read` 探针 + `exec+read` 探针（工具压力测试）
  * 当模型声明支持图像输入时运行图像探针
  * 流程（高层级）：
    * 测试生成一个很小的 PNG，内容为“CAT” + 随机代码（`src/gateway/live-image-probe.ts`）
    * 通过 `agent` 以 `attachments: [{ mimeType: "image/png", content: "<base64>" }]` 发送
    * Gateway 将附件解析为 `images[]`（`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`）
    * 嵌入式智能体将多模态用户消息转发给模型
    * 断言：回复中包含 `cat` + 该代码（OCR 容错：允许轻微错误）

提示：要查看在你的机器上可以测试哪些内容（以及精确的 `provider/model` ID），请运行：

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## 联机：Anthropic setup-token 冒烟测试
</div>

* 测试：`src/agents/anthropic.setup-token.live.test.ts`
* 目标：验证 Claude Code CLI 的 setup-token（或粘贴的 setup-token 配置文件）可以成功完成一个 Anthropic 提示词。
* 启用：
  * `pnpm test:live`（或在直接调用 Vitest 时设置 `OPENCLAW_LIVE_TEST=1`）
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* Token 来源（任选其一）：
  * 配置文件：`OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * 原始 token：`OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* 模型覆写（可选）：
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

配置示例：

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## 实时：CLI 后端冒烟测试（Claude Code CLI 或其他本地 CLI）
</div>

* 测试：`src/gateway/gateway-cli-backend.live.test.ts`
* 目标：在不修改默认配置的前提下，使用本地 CLI 后端验证 Gateway + 智能体处理流水线。
* 启用：
  * `pnpm test:live`（或在直接调用 Vitest 时设置 `OPENCLAW_LIVE_TEST=1`）
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* 默认值：
  * 模型：`claude-cli/claude-sonnet-4-5`
  * 命令：`claude`
  * 参数：`["-p","--output-format","json","--dangerously-skip-permissions"]`
* 覆盖（可选）：
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * 将 `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` 设为 1，以发送真实的图片附件（路径会注入到提示词中）。
  * 将 `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` 设为 `--image`，以便将图片文件路径作为 CLI 参数传递，而不是通过提示词注入。
  * 将 `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"`（或 `"list"`）设为对应值，以在设置了 `IMAGE_ARG` 时控制图片参数的传递方式。
  * 将 `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` 设为 1，以发送第二轮对话并验证恢复流程。
* 将 `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` 设为 0，以保持 Claude Code CLI MCP 配置处于启用状态（默认会通过临时空文件禁用 MCP 配置）。

示例：

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### 推荐的实时测试方案
</div>

范围小且明确的允许列表速度最快、最不容易出问题：

* 单一模型，直连（不经过 Gateway）：
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* 单一模型，Gateway 冒烟测试：
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* 跨多个提供方的工具调用：
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* 以 Google 为主（Gemini API key + Antigravity）：
  * Gemini（API key）：`OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity（OAuth）：`OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

说明：

* `google/...` 使用 Gemini API（API key）。
* `google-antigravity/...` 使用 Antigravity OAuth 桥接（Cloud Code Assist 风格的 Agent 代理端点）。
* `google-gemini-cli/...` 使用你本机上的本地 Gemini CLI（单独的认证机制 + 工具使用上的一些特有行为）。
* Gemini API 与 Gemini CLI 的区别：
  * API：OpenClaw 通过 HTTP 调用 Google 托管的 Gemini API（使用 API key / profile 认证）；这也是大多数用户提到 “Gemini” 时所指的接口。
  * CLI：OpenClaw 通过本地 `gemini` 可执行文件进行调用；它有自己独立的认证方式，并且在行为上可能不同（例如流式输出、工具支持、版本差异等）。

<div id="live-model-matrix-what-we-cover">
  ## 实时：模型矩阵（覆盖内容）
</div>

没有固定的“CI 模型列表”（实时测试是按需开启的），但以下是**推荐**在已配置密钥的开发机上定期覆盖的模型。

<div id="modern-smoke-set-tool-calling-image">
  ### 现代冒烟测试集（工具调用 + 图像）
</div>

这是我们预期会一直保持可用的「常用模型」组合：

* OpenAI（非 Codex）：`openai/gpt-5.2`（可选：`openai/gpt-5.1`）
* OpenAI Codex：`openai-codex/gpt-5.2`（可选：`openai-codex/gpt-5.2-codex`）
* Anthropic：`anthropic/claude-opus-4-5`（或 `anthropic/claude-sonnet-4-5`）
* Google（Gemini API）：`google/gemini-3-pro-preview` 和 `google/gemini-3-flash-preview`（避免使用较旧的 Gemini 2.x 模型）
* Google（Antigravity）：`google-antigravity/claude-opus-4-5-thinking` 和 `google-antigravity/gemini-3-flash`
* Z.AI（GLM）：`zai/glm-4.7`
* MiniMax：`minimax/minimax-m2.1`

使用工具 + 图像对 Gateway 运行冒烟测试：
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### 基线：工具调用（Read + 可选 Exec）
</div>

为每个提供方系列至少选择一个：

* OpenAI：`openai/gpt-5.2`（或 `openai/gpt-5-mini`）
* Anthropic：`anthropic/claude-opus-4-5`（或 `anthropic/claude-sonnet-4-5`）
* Google：`google/gemini-3-flash-preview`（或 `google/gemini-3-pro-preview`）
* Z.AI（GLM）：`zai/glm-4.7`
* MiniMax：`minimax/minimax-m2.1`

可选的额外覆盖（有更好，没有也行）：

* xAI：`xai/grok-4`（或当前可用的最新版本）
* Mistral：`mistral/`…（选择一个你已启用的、支持“tools”的模型）
* Cerebras：`cerebras/`…（如果你有访问权限）
* LM Studio：`lmstudio/`…（本地；工具调用取决于 API 模式）

<div id="vision-image-send-attachment-multimodal-message">
  ### 视觉：发送图像（附件 → 多模态消息）
</div>

在 `OPENCLAW_LIVE_GATEWAY_MODELS` 中至少包含一个支持图像的模型（如具备视觉能力的 Claude / Gemini / OpenAI 变体等），以便验证图像探测功能。

<div id="aggregators-alternate-gateways">
  ### 聚合器 / 其他 Gateway
</div>

如果你已经配置好密钥，我们也支持通过以下方式进行测试：

* OpenRouter：`openrouter/...`（数百个模型；使用 `openclaw models scan` 查找具备 tool + image 能力的候选模型）
* OpenCode Zen：`opencode/...`（通过 `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY` 进行认证）

如果你已经准备好凭据/配置，还可以在实时矩阵中加入更多提供方：

* 内置：`openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* 通过 `models.providers`（自定义端点）：`minimax`（云/API），以及任何兼容 OpenAI/Anthropic 的代理（LM Studio、vLLM、LiteLLM 等）

提示：不要尝试在文档中硬编码“所有模型”。权威列表是你本机上 `discoverModels(...)` 的返回结果 + 当前可用的所有密钥。

<div id="credentials-never-commit">
  ## 凭据（切勿提交到版本库）
</div>

实时测试会以与 CLI 相同的方式获取凭据。实际含义是：

* 如果 CLI 能正常工作，实时测试也应该能找到相同的密钥。

* 如果实时测试提示 “no creds”，就按你调试 `openclaw models list` / 模型选择时的方式进行排查。

* Profile 存储目录：`~/.openclaw/credentials/`（推荐；测试中所说的 “profile keys” 指的就是这里）

* 配置文件：`~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）

如果你打算依赖环境变量中的密钥（例如在 `~/.profile` 中导出），请在执行 `source ~/.profile` 之后再运行本地测试，或者使用下面的 Docker runner（它们可以把 `~/.profile` 挂载进容器）。

<div id="deepgram-live-audio-transcription">
  ## Deepgram 实时（音频转写）
</div>

* 测试：`src/media-understanding/providers/deepgram/audio.live.test.ts`
* 启用：`DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Docker 运行器（可选“在 Linux 上可用”检查）
</div>

这些命令会在仓库的 Docker 镜像中运行 `pnpm test:live`，挂载你的本地配置目录和工作区（如果有挂载则加载 `~/.profile`）：

* 直接模型：`pnpm test:docker:live-models`（脚本：`scripts/test-live-models-docker.sh`）
* Gateway + 开发用智能体：`pnpm test:docker:live-gateway`（脚本：`scripts/test-live-gateway-models-docker.sh`）
* 入门向导（TTY，完整脚手架）：`pnpm test:docker:onboard`（脚本：`scripts/e2e/onboard-docker.sh`）
* Gateway 网络（两个容器，WS 认证 + 健康检查）：`pnpm test:docker:gateway-network`（脚本：`scripts/e2e/gateway-network-docker.sh`）
* 插件（自定义扩展加载 + 注册表冒烟测试）：`pnpm test:docker:plugins`（脚本：`scripts/e2e/plugins-docker.sh`）

有用的环境变量：

* `OPENCLAW_CONFIG_DIR=...`（默认：`~/.openclaw`）挂载到 `/home/node/.openclaw`
* `OPENCLAW_WORKSPACE_DIR=...`（默认：`~/.openclaw/workspace`）挂载到 `/home/node/.openclaw/workspace`
* `OPENCLAW_PROFILE_FILE=...`（默认：`~/.profile`）挂载到 `/home/node/.profile` 并在运行测试前加载
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` 用于缩小测试范围
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` 确保凭据来自 profile 存储（而不是环境变量）

<div id="docs-sanity">
  ## 文档快速检查
</div>

在修改文档后运行检查命令：`pnpm docs:list`。

<div id="offline-regression-ci-safe">
  ## 离线回归（适用于 CI）
</div>

这些是没有真实提供方参与的“真实流水线”回归测试：

* Gateway 工具调用（模拟 OpenAI，真实 Gateway + 智能体循环）：`src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Gateway 向导（WS `wizard.start`/`wizard.next`，写入配置并强制执行认证）：`src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## Agent 可靠性评测（技能）
</div>

我们已经有了一些可安全用于 CI 的测试，它们的行为类似于 “Agent 可靠性评测”：

* 通过真实 Gateway + 智能体循环进行模拟工具调用（`src/gateway/gateway.tool-calling.mock-openai.test.ts`）。
* 端到端向导流程，用于验证会话串联和配置生效情况（`src/gateway/gateway.wizard.e2e.test.ts`）。

在技能方面（参见 [Skills](/zh/tools/skills)），目前仍然缺少的是：

* **决策能力（Decisioning）：** 当技能列表出现在提示词中时，智能体是否会选择正确的技能（或避免无关技能）？
* **合规性（Compliance）：** 智能体在使用前是否会读取 `SKILL.md`，并遵循其中要求的步骤和参数？
* **工作流契约（Workflow contracts）：** 多轮场景，用于断言工具调用顺序、会话历史传递以及沙箱边界。

后续评测应优先保持确定性：

* 使用模拟提供方的场景运行器，用于断言工具调用及其顺序、技能文件读取以及会话串联。
* 一小组以技能为中心的场景（使用 vs 避免、门控、提示词注入（prompt injection））。
* 可选的在线/真实环境评测（显式选择加入，由环境变量门控），仅在 CI 安全测试套件到位之后再启用。

<div id="adding-regressions-guidance">
  ## 添加回归测试（指导）
</div>

当你修复在线上发现的提供方 / 模型问题时：

* 如果可能，添加一个适用于 CI 的回归测试（使用 mock / 桩实现的提供方，或捕获精确的请求形状转换）
* 如果问题本质上只能在线上复现（限流、鉴权策略），就让线上测试范围尽可能窄，并通过环境变量显式开启
* 优先针对能捕获该 bug 的最小层级：
  * 提供方请求转换 / 回放 bug → 直接的模型层测试
  * Gateway 会话 / 历史 / tool 流水线 bug → Gateway 线上冒烟测试或适用于 CI 的 Gateway mock 测试