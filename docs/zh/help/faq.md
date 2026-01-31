---
title: 常见问题
summary: "关于 OpenClaw 安装、配置和使用的常见问答"
---

<div id="faq">
  # 常见问题解答（FAQ）
</div>

面向实际部署场景（本地开发、VPS、多智能体、OAuth/API 密钥、模型故障切换）提供快速解答和更深入的故障排查指引。运行时诊断请参见 [故障排查](/zh/gateway/troubleshooting)。完整配置参考请参见 [配置](/zh/gateway/configuration)。

<div id="table-of-contents">
  ## 目录
</div>

* [快速入门与首次运行设置](#quick-start-and-firstrun-setup)
  * [我卡住了——最快的解决办法是什么？](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  * [推荐的 OpenClaw 安装和初始化配置方式是什么？](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  * [完成引导后，如何打开控制台（dashboard）？](#how-do-i-open-the-dashboard-after-onboarding)
  * [在本机（localhost）和远程环境下，如何为控制台进行 token 认证？](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  * [我需要什么运行时环境？](#what-runtime-do-i-need)
  * [它能在 Raspberry Pi 上运行吗？](#does-it-run-on-raspberry-pi)
  * [在 Raspberry Pi 上安装有什么技巧？](#any-tips-for-raspberry-pi-installs)
  * [卡在 “wake up my friend” / 引导流程一直无法“孵化”出来怎么办？](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  * [我能在不重新走引导流程的情况下，把当前配置迁移到新机器（Mac mini）吗？](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  * [在哪里可以查看最新版本的更新内容？](#where-do-i-see-whats-new-in-the-latest-version)
  * [我无法访问 docs.openclaw.ai（SSL 错误），该怎么办？](#i-cant-access-docsopenclawai-ssl-error-what-now)
  * [稳定版（stable）和测试版（beta）有什么区别？](#whats-the-difference-between-stable-and-beta)
* [如何安装 beta 版本？beta 和 dev 有什么区别？](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  * [如何试用最新开发版本？](#how-do-i-try-the-latest-bits)
  * [安装和上手流程通常需要多长时间？](#how-long-does-install-and-onboarding-usually-take)
  * [安装程序卡住了？如何获取更多输出信息？](#installer-stuck-how-do-i-get-more-feedback)
  * [Windows 安装时提示找不到 git 或无法识别 openclaw](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  * [文档没能解决我的问题——我该如何找到更好的答案？](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  * [如何在 Linux 上安装 OpenClaw？](#how-do-i-install-openclaw-on-linux)
  * [如何在 VPS 上安装 OpenClaw？](#how-do-i-install-openclaw-on-a-vps)
  * [云服务/VPS 安装指南在哪里？](#where-are-the-cloudvps-install-guides)
  * [我可以让 OpenClaw 自动更新吗？](#can-i-ask-openclaw-to-update-itself)
  * [入门向导实际上做了些什么？](#what-does-the-onboarding-wizard-actually-do)
  * [我需要订阅 Claude 或 OpenAI 才能运行它吗？](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  * [我可以在没有 API 密钥的情况下使用 Claude Max 订阅服务吗](#can-i-use-claude-max-subscription-without-an-api-key)
  * [Anthropic “setup-token” 认证如何工作？](#how-does-anthropic-setuptoken-auth-work)
  * [在哪里可以找到 Anthropic 的 setup-token？](#where-do-i-find-an-anthropic-setuptoken)
  * [是否支持 Claude 订阅授权（Claude Code OAuth）？](#do-you-support-claude-subscription-auth-claude-code-oauth)
  * [为什么会出现来自 Anthropic 的 `HTTP 429: rate_limit_error` 错误？](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  * [是否支持 AWS Bedrock？](#is-aws-bedrock-supported)
  * [Codex 认证机制是如何运作的？](#how-does-codex-auth-work)
  * [是否支持 OpenAI 订阅认证（Codex OAuth）？](#do-you-support-openai-subscription-auth-codex-oauth)
  * [如何配置 Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  * [本地模型适合日常聊天吗？](#is-a-local-model-ok-for-casual-chats)
  * [如何确保托管模型流量只在特定区域内传输？](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  * [我一定要买一台 Mac mini 才能安装吗？](#do-i-have-to-buy-a-mac-mini-to-install-this)
  * [支持 iMessage 一定要用 Mac mini 吗？](#do-i-need-a-mac-mini-for-imessage-support)
  * [如果我买一台 Mac mini 用于运行 OpenClaw，可以把它连接到我的 MacBook Pro 吗？](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  * [可以使用 Bun 吗？](#can-i-use-bun)
  * [Telegram：`allowFrom` 中应填写什么？](#telegram-what-goes-in-allowfrom)
  * [多人是否可以在不同的 OpenClaw 实例上共用同一个 WhatsApp 号码？](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  * [我可以同时运行 &quot;fast chat&quot; 智能体和 &quot;Opus for coding&quot; 智能体吗？](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  * [Homebrew 可以在 Linux 上使用吗？](#does-homebrew-work-on-linux)
  * [可 hack（git）安装与 npm 安装有什么区别？](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  * [以后可以在 npm 安装和 git 安装之间切换吗？](#can-i-switch-between-npm-and-git-installs-later)
  * [我应该在自己的笔记本电脑上运行 Gateway，还是在 VPS 上运行？](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  * [在专用主机上运行 OpenClaw 有多重要？](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  * [VPS 的最低要求和推荐的操作系统是什么？](#what-are-the-minimum-vps-requirements-and-recommended-os)
  * [我可以在虚拟机中运行 OpenClaw 吗？有哪些系统要求？](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
* [什么是 OpenClaw？](#what-is-openclaw)
  * [用一段话介绍 OpenClaw 是什么？](#what-is-openclaw-in-one-paragraph)
  * [它的价值主张是什么？](#whats-the-value-proposition)
  * [我刚完成安装，首先应该做什么？](#i-just-set-it-up-what-should-i-do-first)
  * [OpenClaw 最常见的五个日常使用场景是什么？](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  * [OpenClaw 能帮忙做 SaaS 的获客外联、广告投放和博客内容吗？](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  * [和 Claude Code 相比，在 Web 开发方面有什么优势？](#what-are-the-advantages-vs-claude-code-for-web-development)
* [技能和自动化](#skills-and-automation)
  * [如何在不让仓库变“脏”的情况下自定义技能？](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  * [我可以从自定义文件夹加载技能吗？](#can-i-load-skills-from-a-custom-folder)
  * [如何为不同任务使用不同模型？](#how-can-i-use-different-models-for-different-tasks)
  * [机器人在执行繁重任务时会卡住。我该如何把这些工作卸载/转移出去？](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  * [Cron 或提醒没有触发。我应该检查什么？](#cron-or-reminders-do-not-fire-what-should-i-check)
  * [如何在 Linux 上安装技能？](#how-do-i-install-skills-on-linux)
  * [OpenClaw 可以按计划或在后台持续运行任务吗？](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  * [我可以在 Linux 上运行仅适用于 Apple/macOS 的技能吗？](#can-i-run-applemacosonly-skills-from-linux)
  * [是否提供 Notion 或 HeyGen 集成？](#do-you-have-a-notion-or-heygen-integration)
  * [如何安装用于接管浏览器的 Chrome 扩展程序？](#how-do-i-install-the-chrome-extension-for-browser-takeover)
* [沙箱和内存](#sandboxing-and-memory)
  * [有没有专门介绍沙箱的文档？](#is-there-a-dedicated-sandboxing-doc)
  * [如何将主机文件夹挂载到沙箱中？](#how-do-i-bind-a-host-folder-into-the-sandbox)
  * [记忆系统是如何工作的？](#how-does-memory-work)
  * [记忆总是“忘事”。如何让它更持久？](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  * [记忆会一直保留吗？有什么限制？](#does-memory-persist-forever-what-are-the-limits)
  * [语义记忆搜索是否需要 OpenAI 的 API 密钥？](#does-semantic-memory-search-require-an-openai-api-key)
* [磁盘上的目录结构](#where-things-live-on-disk)
  * [使用 OpenClaw 时的所有数据都会保存在本地吗？](#is-all-data-used-with-openclaw-saved-locally)
  * [OpenClaw 将数据存储在哪里？](#where-does-openclaw-store-its-data)
  * [AGENTS.md / SOUL.md / USER.md / MEMORY.md 应该放在什么位置？](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  * [推荐的备份策略是什么？](#whats-the-recommended-backup-strategy)
  * [如何完全卸载 OpenClaw？](#how-do-i-completely-uninstall-openclaw)
  * [智能体可以在工作区之外工作吗？](#can-agents-work-outside-the-workspace)
  * [我处于远程模式——会话存储在哪里？](#im-in-remote-mode-where-is-the-session-store)
* [基础配置](#config-basics)
  * [配置使用什么格式？在哪里？](#what-format-is-the-config-where-is-it)
  * [我把 `gateway.bind: "lan"`（或 `"tailnet"`）设置好了，但现在没有任何端口在监听 / UI 显示“unauthorized”](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  * [为什么现在在 localhost 上也需要 token？](#why-do-i-need-a-token-on-localhost-now)
  * [修改配置后必须重启吗？](#do-i-have-to-restart-after-changing-config)
  * [如何启用 web search（以及 web fetch）？](#how-do-i-enable-web-search-and-web-fetch)
  * [`config.apply` 把我的配置清空了。如何恢复并避免再次发生？](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  * [如何运行一个中心 Gateway，并在多设备上部署专用 worker？](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  * [OpenClaw 浏览器可以以无头模式运行吗？](#can-the-openclaw-browser-run-headless)
  * [如何使用 Brave 进行浏览器控制？](#how-do-i-use-brave-for-browser-control)
* [远程 Gateway 和节点](#remote-gateways-nodes)
  * [命令如何在 Telegram、Gateway 和节点之间传递？](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  * [如果 Gateway 托管在远程服务器上，我的智能体如何访问我的电脑？](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  * [Tailscale 已连接但我收不到任何回复，该怎么办？](#tailscale-is-connected-but-i-get-no-replies-what-now)
  * [两个 OpenClaw 实例可以互相通信吗（本地 + VPS）？](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  * [多个智能体需要单独的 VPS 吗？](#do-i-need-separate-vpses-for-multiple-agents)
  * [在我的个人笔记本上运行节点，相比从 VPS 使用 SSH 有什么好处吗？](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  * [节点会运行 Gateway 服务吗？](#do-nodes-run-a-gateway-service)
  * [有没有通过 api/RPC 应用配置的方式？](#is-there-an-api-rpc-way-to-apply-config)
  * [首次安装时，最小且“合理”的配置是什么？](#whats-a-minimal-sane-config-for-a-first-install)
  * [如何在 VPS 上设置 Tailscale 并从我的 Mac 连接？](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  * [如何将 Mac 节点连接到远程 Gateway（Tailscale Serve）？](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  * [我应该在第二台笔记本上安装，还是只添加一个节点？](#should-i-install-on-a-second-laptop-or-just-add-a-node)
* [环境变量和 .env 加载](#env-vars-and-env-loading)
  * [OpenClaw 如何加载环境变量？](#how-does-openclaw-load-environment-variables)
  * [“我通过服务启动了 Gateway，结果环境变量全没了。” 现在该怎么办？](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  * [我已经设置了 `COPILOT_GITHUB_TOKEN`，但模型状态显示 “Shell env: off”。这是为什么？](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
* [会话与多路聊天](#sessions-multiple-chats)
  * [如何开始一段全新的对话？](#how-do-i-start-a-fresh-conversation)
  * [如果我从不发送 `/new`，会话会自动重置吗？](#do-sessions-reset-automatically-if-i-never-send-new)
  * [有没有办法让一组 OpenClaw 实例组成“一个 CEO、多个智能体”的团队？](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  * [为什么上下文在任务中途被截断了？如何防止这种情况？](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  * [如何在保留安装的情况下完全重置 OpenClaw？](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  * [我遇到“context too large”错误——如何重置或压缩？](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  * [为什么我会看到“LLM request rejected: messages.N.content.X.tool&#95;use.input: Field required”？](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  * [为什么我每 30 分钟会收到一次心跳消息？](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  * [我需要把一个“bot 账号”添加到 WhatsApp 群组里吗？](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  * [如何获取 WhatsApp 群组的 JID？](#how-do-i-get-the-jid-of-a-whatsapp-group)
  * [为什么 OpenClaw 不在群组里回复？](#why-doesnt-openclaw-reply-in-a-group)
  * [群组/线程会和私信共享上下文吗？](#do-groupsthreads-share-context-with-dms)
  * [我可以创建多少个工作区和智能体？](#how-many-workspaces-and-agents-can-i-create)
  * [我能否在 Slack 上同时运行多个机器人或聊天？应该如何配置？](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
* [模型：默认值、选择、别名与切换](#models-defaults-selection-aliases-switching)
  * [什么是“默认模型”？](#what-is-the-default-model)
  * [推荐使用哪种模型？](#what-model-do-you-recommend)
  * [如何在不清空配置的情况下切换模型？](#how-do-i-switch-models-without-wiping-my-config)
  * [我可以使用自托管模型（llama.cpp、vLLM、Ollama）吗？](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  * [OpenClaw、Flawd 和 Krill 使用哪些模型？](#what-do-openclaw-flawd-and-krill-use-for-models)
  * [如何在不重启的情况下即时切换模型？](#how-do-i-switch-models-on-the-fly-without-restarting)
  * [我可以用 GPT 5.2 处理日常任务，用 Codex 5.2 专门写代码吗？](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  * [为什么我会看到“Model … is not allowed”，然后就没有回复了？](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  * [为什么我会看到“Unknown model: minimax/MiniMax-M2.1”？](#why-do-i-see-unknown-model-minimaxminimaxm21)
  * [我可以将 MiniMax 作为默认模型，把 OpenAI 用于复杂任务吗？](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  * [opus / sonnet / gpt 是内置快捷别名吗？](#are-opus-sonnet-gpt-builtin-shortcuts)
  * [如何定义或覆盖模型快捷别名（alias）？](#how-do-i-defineoverride-model-shortcuts-aliases)
  * [如何添加来自 OpenRouter 或 Z.AI 等其他提供方的模型？](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
* [模型故障切换与“所有模型均失败”](#model-failover-and-all-models-failed)
  * [故障切换是如何工作的？](#how-does-failover-work)
  * [这个错误是什么意思？](#what-does-this-error-mean)
  * [`No credentials found for profile "anthropic:default"` 的修复排查清单](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  * [为什么还会尝试使用 Google Gemini 并失败？](#why-did-it-also-try-google-gemini-and-fail)
* [认证配置文件：是什么以及如何管理](#auth-profiles-what-they-are-and-how-to-manage-them)
  * [什么是认证配置？](#what-is-an-auth-profile)
  * [常见的配置 ID 有哪些？](#what-are-typical-profile-ids)
  * [我可以控制优先尝试哪个认证配置吗？](#can-i-control-which-auth-profile-is-tried-first)
  * [OAuth 与 API 密钥：有什么区别？](#oauth-vs-api-key-whats-the-difference)
* [Gateway：端口、“已在运行”提示和远程模式](#gateway-ports-already-running-and-remote-mode)
  * [Gateway 使用什么端口？](#what-port-does-the-gateway-use)
  * [为什么 `openclaw gateway status` 显示为 `Runtime: running`，但 `RPC probe: failed`？](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  * [为什么 `openclaw gateway status` 中的 `Config (cli)` 和 `Config (service)` 不一样？](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  * [“another gateway instance is already listening” 是什么意思？](#what-does-another-gateway-instance-is-already-listening-mean)
  * [如何以远程模式运行 OpenClaw（客户端连接到其他位置的 Gateway）？](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  * [Control UI 显示 “unauthorized”（或一直在重新连接），该怎么办？](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  * [我设置了 `gateway.bind: "tailnet"`，但无法绑定 / 没有任何端口在监听](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  * [我可以在同一主机上运行多个 Gateway 吗？](#can-i-run-multiple-gateways-on-the-same-host)
  * [“invalid handshake” / 代码 1008 是什么意思？](#what-does-invalid-handshake-code-1008-mean)
* [日志和调试](#logging-and-debugging)
  * [日志在哪里？](#where-are-logs)
  * [如何启动/停止/重启 Gateway 服务？](#how-do-i-startstoprestart-the-gateway-service)
  * [我在 Windows 上关闭了终端——如何重新启动 OpenClaw？](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  * [Gateway 已经启动但始终收不到回复，我应该检查什么？](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  * [“Disconnected from gateway: no reason”——现在该怎么办？](#disconnected-from-gateway-no-reason-what-now)
  * [Telegram setMyCommands 因网络错误失败，我应该检查什么？](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  * [TUI 没有任何输出，我应该检查什么？](#tui-shows-no-output-what-should-i-check)
  * [如何先彻底停止再重新启动 Gateway？](#how-do-i-completely-stop-then-start-the-gateway)
  * [ELI5：`openclaw gateway restart` 和 `openclaw gateway` 有什么区别？](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  * [在出错时，最快获得更多详细信息的方法是什么？](#whats-the-fastest-way-to-get-more-details-when-something-fails)
* [媒体和附件](#media-attachments)
  * [我的技能生成了图像/PDF，但什么都没有发出去](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
* [安全与访问控制](#security-and-access-control)
  * [对外开放 OpenClaw 接收私信是否安全？](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  * [提示注入只是在公开机器人场景下才需要担心吗？](#is-prompt-injection-only-a-concern-for-public-bots)
  * [我的机器人是否应该有自己专用的电子邮箱、GitHub 账号或电话号码？](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  * [我可以让它自主处理我的短信吗？这样安全吗？](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  * [我可以为个人助理类任务使用更便宜的模型吗？](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  * [我在 Telegram 里执行了 `/start`，但没有收到配对码](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  * [WhatsApp：它会主动给我的联系人发消息吗？配对是如何工作的？](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
* [聊天命令、终止任务和「停不下来」的情况](#chat-commands-aborting-tasks-and-it-wont-stop)
  * [如何隐藏聊天中的内部系统消息](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  * [如何停止/取消正在运行的任务？](#how-do-i-stopcancel-a-running-task)
  * [如何从 Telegram 发送 Discord 消息？（“Cross-context messaging denied”）](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  * [为什么感觉机器人会“忽略”快速连发的消息？](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

<div id="first-60-seconds-if-somethings-broken">
  ## 出问题后的前 60 秒该做什么
</div>

1. **快速状态（首要检查）**
   ```bash
   openclaw status
   ```
   本地快速概览：操作系统 + 更新状态、Gateway/服务可达性、智能体/会话、提供方配置 + 运行时问题（在 Gateway 可达时）。

2. **可粘贴报告（适合分享）**
   ```bash
   openclaw status --all
   ```
   只读诊断，并附带日志尾部（令牌已打码）。

3. **守护进程 + 端口状态**
   ```bash
   openclaw gateway status
   ```
   显示 supervisor 运行状态与 RPC 可达性、探测目标 URL，以及服务很可能使用的配置。

4. **深度探测**
   ```bash
   openclaw status --deep
   ```
   运行 Gateway 健康检查 + 提供方探测（需要 Gateway 可达）。参见 [Health](/zh/gateway/health)。

5. **跟踪最新日志**
   ```bash
   openclaw logs --follow
   ```
   如果 RPC 不可用，回退到：
   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```
   文件日志独立于服务日志；参见 [Logging](/zh/logging) 和 [Troubleshooting](/zh/gateway/troubleshooting)。

6. **运行 doctor（修复）**
   ```bash
   openclaw doctor
   ```
   修复/迁移配置与状态，并运行健康检查。参见 [Doctor](/zh/gateway/doctor)。

7. **Gateway 快照**
   ```bash
   openclaw health --json
   openclaw health --verbose   # 出错时显示目标 URL 和配置路径
   ```
   向正在运行的 Gateway 请求完整快照（仅通过 WS）。参见 [Health](/zh/gateway/health)。

<div id="quick-start-and-first-run-setup">
  ## 快速入门和首次运行设置
</div>

<div id="im-stuck-whats-the-fastest-way-to-get-unstuck">
  ### 我卡住了——最快的解困方式是什么
</div>

使用一个能**直接查看你本机环境**的本地 AI 智能体。这样比在 Discord 里提问有效得多，因为大多数“我卡住了”的情况本质上都是**本地配置或环境问题**，远程帮忙的人没法直接检查你的机器。

* **Claude Code**: https://www.anthropic.com/claude-code/
* **OpenAI Codex**: https://openai.com/codex/

这些工具可以读取代码仓库、运行命令、检查日志，并帮助修复你机器层面的设置（PATH、服务、权限、认证文件）。通过可自由修改的（git）安装方式，把**完整源码检出**提供给它们：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

这会**从 git checkout 安装 OpenClaw**，因此智能体可以读取代码和文档，并针对你当前运行的精确版本进行推理。你随时可以通过在不带 `--install-method git` 的情况下重新运行安装程序切换回稳定版本。

提示：让智能体**规划并监督**修复过程（逐步执行），然后只执行必要的命令。这样可以让变更更小、更容易审计。

如果你发现了真正的 bug 或修复方案，请提交 GitHub issue 或 PR：
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

先从这些命令开始（在寻求帮助时请共享这些命令的输出）：

```bash
openclaw status
openclaw models status
openclaw doctor
```

它们的作用：

* `openclaw status`：快速获取 Gateway/智能体的健康状况和基础配置快照。
* `openclaw models status`：检查提供方授权状态和模型可用性。
* `openclaw doctor`：验证并修复常见的配置/状态问题。

其他有用的 CLI 检查：`openclaw status --all`、`openclaw logs --follow`、
`openclaw gateway status`、`openclaw health --verbose`。

快速调试流程：[如果出问题了，前 60 秒该做什么](#first-60-seconds-if-somethings-broken)。
安装文档：[安装](/zh/install)、[安装器参数](/zh/install/installer)、[更新](/zh/install/updating)。

<div id="whats-the-recommended-way-to-install-and-set-up-openclaw">
  ### 推荐的 OpenClaw 安装和配置方式是什么
</div>

该仓库推荐从源码运行，并使用上手向导完成配置：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
openclaw onboard --install-daemon
```

该向导也可以自动构建 UI 资源。在完成引导流程后，你通常会在 **18789** 端口上运行 Gateway。

从源码运行（贡献者/开发者）：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # 首次运行时自动安装 UI 依赖项
openclaw onboard
```

如果你还没有进行全局安装，可以使用 `pnpm openclaw onboard` 来运行。

<div id="how-do-i-open-the-dashboard-after-onboarding">
  ### 完成引导后如何打开仪表盘
</div>

向导现在会在引导完成后，自动在你的浏览器中打开带有令牌的仪表盘 URL，并在摘要中显示完整链接（包含令牌）。请保持该标签页处于打开状态；如果它没有自动打开，请在同一台机器上复制粘贴输出的 URL。令牌始终只保留在你的主机本地——不会通过浏览器发送到任何地方。

<div id="how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote">
  ### 我该如何在本机 localhost 与远程环境中完成 dashboard token 认证
</div>

**Localhost（同一台机器）：**

* 打开 `http://127.0.0.1:18789/`。
* 如果提示需要认证，运行 `openclaw dashboard` 并使用其中带有 token 的链接（`?token=...`）。
* 该 token 与 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）的值相同，并会在首次加载后由 UI 保存。

**非 localhost：**

* **Tailscale Serve**（推荐）：保持绑定到回环地址，运行 `openclaw gateway --tailscale serve`，然后打开 `https://<magicdns>/`。如果 `gateway.auth.allowTailscale` 为 `true`，则身份标头即可满足认证（无需 token）。
* **Tailnet 绑定**：运行 `openclaw gateway --bind tailnet --token "<token>"`，打开 `http://<tailscale-ip>:18789/`，在 dashboard 设置中粘贴该 token。
* **SSH 隧道**：`ssh -N -L 18789:127.0.0.1:18789 user@host`，然后在本地浏览器中打开由 `openclaw dashboard` 提供的 `http://127.0.0.1:18789/?token=...` 链接。

有关绑定模式和认证详情，请参见 [Dashboard](/zh/web/dashboard) 和 [Web surfaces](/zh/web)。

<div id="what-runtime-do-i-need">
  ### 我需要什么运行时环境
</div>

需要 Node **&gt;= 22**。推荐使用 `pnpm`。不推荐将 Bun 用于 Gateway。

<div id="does-it-run-on-raspberry-pi">
  ### 它能在 Raspberry Pi 上运行吗
</div>

可以。Gateway 非常轻量——文档中列出 **512MB-1GB 内存**、**1 核 CPU**，以及大约 **500MB**
磁盘空间就足以满足个人使用，并指出 **Raspberry Pi 4 就能运行它**。

如果你想留出更多余量（日志、媒体、其他服务），**推荐 2GB 内存**，但这不是
严格的最低要求。

提示：一台小型 Pi/VPS 就可以托管 Gateway，你可以在笔记本电脑/手机上配对 **节点**，
用于本地屏幕/相机/canvas 或命令执行。参见 [Nodes](/zh/nodes)。

<div id="any-tips-for-raspberry-pi-installs">
  ### 在 Raspberry Pi 上安装有什么建议吗
</div>

简短回答：可以用，但要预期有些粗糙的地方。

* 使用 **64 位** 操作系统，并确保 Node &gt;= 22。
* 优先选择 **基于 git 的可修改安装方式**，这样你可以查看日志并快速更新。
* 先在不启用 channels/技能 的情况下启动，然后再逐个添加。
* 如果遇到奇怪的二进制问题，通常是 **ARM 兼容性** 问题。

文档：[Linux](/zh/platforms/linux)，[Install](/zh/install)。

<div id="it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now">
  ### 卡在 “Wake up, my friend” 引导流程界面，永远孵化不出来。现在怎么办？
</div>

那个界面依赖 Gateway 可访问且已通过认证。TUI 也会在首次孵化（hatch）时自动发送
&quot;Wake up, my friend!&quot;。如果你看到这行但**没有任何回复**，并且 token 一直为 0，说明 Agent 代理从未启动过。

1. 重启 Gateway：

```bash
openclaw gateway restart
```

2. 检查状态和身份验证：

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. 如果仍然无响应，请运行：

```bash
openclaw doctor
```

如果 Gateway 运行在远程主机上，确保隧道/Tailscale 连接处于正常状态，并且 UI 已指向正确的 Gateway。参见[远程访问](/zh/gateway/remote)。

<div id="can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding">
  ### 我可以把当前配置迁移到一台新的 Mac mini，而不用重新走引导流程吗
</div>

可以。复制 **state 目录** 和 **工作区（workspace）**，然后运行一次 Doctor。只要你同时复制这两个位置，你的机器人就会保持“完全一样”（记忆、会话历史、认证信息和渠道状态）：

1. 在新机器上安装 OpenClaw。
2. 从旧机器复制 `$OPENCLAW_STATE_DIR`（默认：`~/.openclaw`）。
3. 复制你的工作区（默认：`~/.openclaw/workspace`）。
4. 运行 `openclaw doctor` 并重启 Gateway 服务。

这样会保留配置、认证配置、WhatsApp 凭证、会话和记忆。如果你处于远程模式，记住 Gateway 主机拥有会话存储和工作区。

**重要提示：** 如果你只是把工作区提交/推送到 GitHub，你只备份了**记忆 + 引导（bootstrap）文件**，但**没有**备份会话历史或认证信息。那些都在 `~/.openclaw/` 下（例如 `~/.openclaw/agents/<agentId>/sessions/`）。

相关内容：[迁移](/zh/install/migrating)、[磁盘上的数据位置](/zh/help/faq#where-does-openclaw-store-its-data)、[Agent 工作区](/zh/concepts/agent-workspace)、[Doctor](/zh/gateway/doctor)、[远程模式](/zh/gateway/remote)。

<div id="where-do-i-see-whats-new-in-the-latest-version">
  ### 在哪里可以查看最新版本有哪些更新
</div>

查看 GitHub 上的更新日志：\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

最新的条目在最上方。如果最上面的部分标记为 **Unreleased**，则其下方第一个带日期的部分就是最新已发布的版本。条目按 **Highlights**、**Changes** 和 **Fixes** 分组（必要时还会有文档/其他相关部分）。

<div id="i-cant-access-docsopenclawai-ssl-error-what-now">
  ### 我无法访问 docs.openclaw.ai，出现 SSL 错误，该怎么办
</div>

某些通过 Comcast/Xfinity 的连接会因为 Xfinity Advanced Security 而错误地拦截 `docs.openclaw.ai`。请禁用该功能或在允许列表中加入 `docs.openclaw.ai`，然后重试。更多详情见：[Troubleshooting](/zh/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity)。
请通过在此处提交报告来帮助我们解除拦截：https://spa.xfinity.com/check&#95;url&#95;status。

如果你仍然无法访问该站点，文档在 GitHub 上有镜像：
https://github.com/openclaw/openclaw/tree/main/docs

<div id="whats-the-difference-between-stable-and-beta">
  ### 稳定版和测试版有什么区别
</div>

**Stable** 和 **beta** 是 **npm 的 dist‑tag（发行标签）**，而不是单独的代码分支：

* `latest` = 稳定版
* `beta` = 用于测试的早期构建版本

我们会先将构建发布到 **beta**，进行测试，一旦该构建足够稳定，就会**将同一个版本提升为 `latest`**。因此，beta 和 stable 有时会指向**同一个版本**。

查看变更内容：\
https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

<div id="how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev">
  ### 我该如何安装 Beta 测试版？Beta 和 Dev 有什么区别
</div>

**Beta** 是 npm 的 dist‑tag `beta`（可能与 `latest` 相同）。
**Dev** 是 `main`（git）分支的最新进度；在发布到 npm 时，它使用 dist‑tag `dev`。

单行命令（macOS/Linux）：

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.bot/install.sh | bash -s -- --install-method git
```

Windows 安装程序（PowerShell）：
https://openclaw.ai/install.ps1

更多详情请参见：[开发通道](/zh/install/development-channels) 和 [安装程序参数](/zh/install/installer)。

<div id="how-long-does-install-and-onboarding-usually-take">
  ### 安装和引导配置通常需要多长时间
</div>

大致情况如下：

* **安装：** 2-5 分钟
* **引导配置（Onboarding）：** 5-15 分钟，取决于你要配置多少个渠道/模型

如果过程中卡住，请参考 [安装程序卡住](/zh/help/faq#installer-stuck-how-do-i-get-more-feedback)
以及 [我卡住了](/zh/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck) 中的快速调试流程。

<div id="how-do-i-try-the-latest-bits">
  ### 我如何体验最新版本？
</div>

有两种方式：

1. **开发通道（Git checkout）：**

```bash
openclaw update --channel dev
```

这会切换到 `main` 分支并从源码更新。

2. **可改造的安装方式（通过安装器站点）：**

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

这会为你创建一个本地仓库，你可以在本地进行编辑，然后通过 git 更新。

如果你更喜欢手动进行一次干净的克隆，请使用：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

文档：[更新](/zh/cli/update)、[开发通道](/zh/install/development-channels)、[安装](/zh/install)。

<div id="installer-stuck-how-do-i-get-more-feedback">
  ### 安装程序卡住了，如何获取更多输出信息？
</div>

使用**详细输出**重新运行安装程序：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

启用详细输出的 Beta 安装：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

如需可自行修改源码的 Git 安装方式：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --verbose
```

更多选项：[安装器参数](/zh/install/installer)。

<div id="windows-install-says-git-not-found-or-openclaw-not-recognized">
  ### Windows 安装提示找不到 git 或无法识别 openclaw
</div>

两个常见的 Windows 问题：

**1) npm 报错 spawn git / 找不到 git**

* 安装 **Git for Windows**，并确保 `git` 已加入你的 PATH。
* 关闭并重新打开 PowerShell，然后重新运行安装程序。

**2) 安装后无法识别 openclaw**

* 你的 npm 全局 bin 目录未在 PATH 中。
* 检查路径：
  ```powershell
  npm config get prefix
  ```
* 确保 `<prefix>\\bin` 在 PATH 中（在大多数系统上为 `%AppData%\\npm`）。
* 更新 PATH 后，关闭并重新打开 PowerShell。

如果你想要最顺畅的 Windows 环境搭建体验，推荐使用 **WSL2** 而不是原生 Windows。
文档： [Windows](/zh/platforms/windows)。

<div id="the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer">
  ### 文档没有解决我的问题，我该如何获得更好的解答
</div>

使用 **可 hack 的（git）安装方式**，这样你就可以在本地拥有完整的源码和文档，然后在 *该目录下* 向你的 bot（或 Claude/Codex）提问，这样它就能读取整个仓库并给出更精确的回答。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

更多详细信息请参阅：[安装](/zh/install) 和 [安装程序命令行参数](/zh/install/installer)。

<div id="how-do-i-install-openclaw-on-linux">
  ### 如何在 Linux 上安装 OpenClaw
</div>

简要回答：按照 Linux 指南操作，然后运行上手向导。

* Linux 快速流程 + 服务安装： [Linux](/zh/platforms/linux)。
* 完整操作指南： [Getting Started](/zh/start/getting-started)。
* 安装器与更新： [Install &amp; updates](/zh/install/updating)。

<div id="how-do-i-install-openclaw-on-a-vps">
  ### 如何在 VPS 上安装 OpenClaw
</div>

任何 Linux VPS 都可以用。在服务器上安装，然后通过 SSH 或 Tailscale 连接到 Gateway。

安装指南：[exe.dev](/zh/platforms/exe-dev)、[Hetzner](/zh/platforms/hetzner)、[Fly.io](/zh/platforms/fly)。\
远程访问：[Gateway 远程访问](/zh/gateway/remote)。

<div id="where-are-the-cloudvps-install-guides">
  ### cloud VPS 安装指南在哪里
</div>

我们维护了一个包含常见提供方的 **托管汇总页**。选一个并按照对应指南操作：

* [VPS hosting](/zh/vps)（所有提供方集中在一处）
* [Fly.io](/zh/platforms/fly)
* [Hetzner](/zh/platforms/hetzner)
* [exe.dev](/zh/platforms/exe-dev)

在云端的工作方式：**Gateway 运行在服务器上**，你通过笔记本/手机上的 Control UI（或 Tailscale/SSH）进行访问。你的状态和工作区都保存在服务器上，因此要将该主机视为权威数据源并做好备份。

你可以将 **节点**（Mac/iOS/Android/无头环境）配对到该云端 Gateway，以访问本地屏幕/摄像头/canvas，或在你的笔记本上执行命令，同时仍将 Gateway 保持在云端。

Hub：[Platforms](/zh/platforms)。远程访问：[Gateway remote](/zh/gateway/remote)。
节点：[Nodes](/zh/nodes)，[Nodes CLI](/zh/cli/nodes)。

<div id="can-i-ask-openclaw-to-update-itself">
  ### 我可以让 OpenClaw 自行更新吗
</div>

简要回答：**可以，但不推荐**。更新流程可能会重启 Gateway（这会导致当前活跃会话丢失），可能需要一个干净的 git checkout，并且可能会提示你进行确认。更安全的做法：作为运维/操作人员在 shell 中手动运行更新。

使用 CLI：

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

如果你确实需要由智能体发起自动化操作：

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

文档：[更新](/zh/cli/update)、[更新指南](/zh/install/updating)。

<div id="what-does-the-onboarding-wizard-actually-do">
  ### 上手向导实际会做什么
</div>

`openclaw onboard` 是推荐的初始化配置方式。在 **本地模式** 下，它会引导你完成：

* **模型/认证配置**（推荐使用 Anthropic 的 **setup-token** 来配置 Claude 订阅，支持 OpenAI Codex OAuth，可选使用 API keys，支持 LM Studio 本地模型）
* **工作区** 位置 + 引导/初始化文件
* **Gateway 设置**（绑定地址/端口/认证/Tailscale）
* **提供方/渠道**（WhatsApp、Telegram、Discord、Mattermost（插件）、Signal、iMessage）
* **守护进程安装**（macOS 上为 LaunchAgent；Linux/WSL2 上为 systemd 用户单元）
* **健康检查** 和 **技能** 选择

如果你配置的模型未知或缺少认证信息，它也会给出警告。

<div id="do-i-need-a-claude-or-openai-subscription-to-run-this">
  ### 运行这个时我需要 Claude 或 OpenAI 订阅吗
</div>

不需要。你可以使用 **API keys**（Anthropic/OpenAI 等）来运行 OpenClaw，或者使用
**纯本地模型**，这样你的数据会始终保留在你的设备上。订阅（Claude
Pro/Max 或 OpenAI Codex）只是对这些提供方进行身份验证的一种可选方式。

文档：[Anthropic](/zh/providers/anthropic)、[OpenAI](/zh/providers/openai)、
[本地模型](/zh/gateway/local-models)、[模型](/zh/concepts/models)。

<div id="can-i-use-claude-max-subscription-without-an-api-key">
  ### 我可以在没有 API key 的情况下使用 Claude Max 订阅吗
</div>

可以。你可以使用 **setup-token** 来完成身份验证，
而不是使用 API key。这是订阅用户的推荐路径。

Claude Pro/Max 订阅 **不包含 API key**，因此对于订阅账号来说，
这是正确的使用方式。重要说明：你必须向 Anthropic 确认此类用法
是否被其订阅政策和条款所允许。\
如果你希望使用最明确且官方支持的方式，请使用 Anthropic 的 API key。

<div id="how-does-anthropic-setuptoken-auth-work">
  ### Anthropic setuptoken 认证是如何运作的
</div>

`claude setup-token` 会通过 Claude Code CLI 生成一个 **令牌字符串**（网页版控制台中不可用）。你可以在 **任意机器** 上运行该命令。然后在向导中选择 **Anthropic token (paste setup-token)**，或者通过 `openclaw models auth paste-token --provider anthropic` 粘贴该令牌。该令牌会作为 **anthropic** 提供方的认证配置文件（auth profile）进行保存，并像 API key 一样使用（不会自动刷新）。更多详情参见：[OAuth](/zh/concepts/oauth)。

<div id="where-do-i-find-an-anthropic-setuptoken">
  ### 在哪里可以找到 Anthropic 设置令牌
</div>

它**不在** Anthropic Console 中。`setup-token` 是通过 **Claude Code CLI** 在**任意一台机器**上生成的：

```bash
claude setup-token
```

复制命令输出的 token，然后在向导中选择 **Anthropic token (paste setup-token)**。如果你想在 Gateway 主机上运行，使用 `openclaw models auth setup-token --provider anthropic`。如果你在其他地方运行了 `claude setup-token`，请在 Gateway 主机上执行 `openclaw models auth paste-token --provider anthropic` 并粘贴该 token。参见 [Anthropic](/zh/providers/anthropic)。

<div id="do-you-support-claude-subscription-auth-claude-promax">
  ### 是否支持 Claude 订阅认证（Claude Pro/Max）
</div>

支持 —— 通过 **setup-token**。OpenClaw 不再复用 Claude Code CLI OAuth 令牌；请使用 setup-token 或 Anthropic API key。你可以在任意位置生成该令牌，然后粘贴到 Gateway 所在主机上。参见 [Anthropic](/zh/providers/anthropic) 和 [OAuth](/zh/concepts/oauth)。

注意：Claude 订阅访问受 Anthropic 的使用条款约束。对于生产环境或多用户负载，通常更安全的做法是使用 API key。

<div id="why-am-i-seeing-http-429-ratelimiterror-from-anthropic">
  ### 为什么我会看到来自 Anthropic 的 HTTP 429 ratelimiterror
</div>

这意味着你在当前时间窗口内的 **Anthropic 配额/速率限制** 已经用完。\
如果你使用的是 **Claude 订阅**（setup‑token 或 Claude Code OAuth），请等待该时间窗口重置，或升级你的套餐。\
如果你使用的是 **Anthropic API key**，请在 Anthropic Console 中检查用量/计费情况，并根据需要提高限额。

提示：设置一个 **回退模型（fallback model）**，这样当某个提供方触发速率限制时，OpenClaw 仍然可以继续回复。\
参见 [Models](/zh/cli/models) 和 [OAuth](/zh/concepts/oauth)。

<div id="is-aws-bedrock-supported">
  ### 是否支持 AWS Bedrock
</div>

支持——通过 pi‑ai 的 **Amazon Bedrock (Converse)** 提供方，并采用**手动配置**方式。你必须在 Gateway 主机上配置 AWS 凭证和区域（region），并在模型配置中添加一个 Bedrock 提供方条目。参见 [Amazon Bedrock](/zh/bedrock) 和 [Model providers](/zh/providers/models)。如果你更偏好托管密钥的使用流程，在 Bedrock 前面加一层兼容 OpenAI 的代理依然是可行选项。

<div id="how-does-codex-auth-work">
  ### Codex 认证如何工作
</div>

OpenClaw 通过 OAuth（使用 ChatGPT 账号登录）支持 **OpenAI Code（Codex）**。向导可以运行 OAuth 授权流程，并在合适的情况下将默认模型设置为 `openai-codex/gpt-5.2`。参见 [模型提供方](/zh/concepts/model-providers) 和 [向导](/zh/start/wizard)。

<div id="do-you-support-openai-subscription-auth-codex-oauth">
  ### 是否支持 OpenAI 订阅认证 Codex OAuth
</div>

支持。OpenClaw 完全支持 **OpenAI Code（Codex）订阅 OAuth**。引导向导
可以为你执行整个 OAuth 授权流程。

参见 [OAuth](/zh/concepts/oauth)、[模型提供方](/zh/concepts/model-providers) 和 [向导](/zh/start/wizard)。

<div id="how-do-i-set-up-gemini-cli-oauth">
  ### 如何配置 Gemini CLI 的 OAuth
</div>

Gemini CLI 使用的是**插件授权流程（plugin auth flow）**，而不是在 `openclaw.json` 中配置客户端 ID 或密钥（client id / secret）。

步骤：

1. 启用插件：`openclaw plugins enable google-gemini-cli-auth`
2. 登录：`openclaw models auth login --provider google-gemini-cli --set-default`

这会将 OAuth 令牌存储在 Gateway 主机上的认证配置文件（auth profiles）中。详情参见：[模型提供方](/zh/concepts/model-providers)。

<div id="is-a-local-model-ok-for-casual-chats">
  ### 本地模型适合随意聊天吗
</div>

通常不适合。OpenClaw 需要大的上下文窗口和强安全性；上下文窗口太小会导致截断和信息泄漏。非用不可的话，请在本地（LM Studio）运行你能跑得动的 **最大** MiniMax M2.1 版本，并参考 [/gateway/local-models](/zh/gateway/local-models)。更小／量化程度更高的模型会增加提示词注入风险——参见 [安全](/zh/gateway/security)。

<div id="how-do-i-keep-hosted-model-traffic-in-a-specific-region">
  ### 我该如何将托管模型的流量限制在特定地区内
</div>

选择绑定到特定地区的 endpoint。OpenRouter 为 MiniMax、Kimi 和 GLM 提供了托管在美国的选项；选择美国托管版本即可让数据留在该地区。你仍然可以通过使用 `models.mode: "merge"` 将 Anthropic/OpenAI 与这些一起列出，这样既能保留回退能力，又能遵守你所选择的区域化提供方的限制。

<div id="do-i-have-to-buy-a-mac-mini-to-install-this">
  ### 我必须买一台 Mac mini 才能安装这个吗
</div>

不需要。OpenClaw 可以运行在 macOS 或 Linux 上（Windows 通过 WSL2）。Mac mini 是可选的——有些人会买一台作为常开主机，但小型 VPS、家庭服务器或 Raspberry Pi 级别的小机器也同样可行。

你只在需要使用 **仅限 macOS 的工具** 时才需要一台 Mac。对于 iMessage，你可以让 Gateway 继续在 Linux 上运行，然后在任意一台 Mac 上通过 SSH 运行 `imsg`，并将 `channels.imessage.cliPath` 指向一个 SSH 封装脚本。如果你需要其他仅限 macOS 的工具，可以直接在 Mac 上运行 Gateway，或者配对一个 macOS 节点。

文档：[iMessage](/zh/channels/imessage)、[Nodes](/zh/nodes)、[Mac remote mode](/zh/platforms/mac/remote)。

<div id="do-i-need-a-mac-mini-for-imessage-support">
  ### 我需要一台 Mac mini 才能支持 iMessage 吗
</div>

你需要**某台运行 macOS 的设备**登录“信息”App。**不**一定非得是 Mac mini——任何 Mac 都可以。OpenClaw 的 iMessage 集成运行在 macOS 上（BlueBubbles 或 `imsg`），而 Gateway 可以部署在其他地方。

常见部署方式：

* 在 Linux/VPS 上运行 Gateway，并将 `channels.imessage.cliPath` 指向一个通过 SSH 在 Mac 上运行 `imsg` 的包装脚本。
* 如果你想要最简单的单机部署，就把所有内容都跑在同一台 Mac 上。

文档： [iMessage](/zh/channels/imessage)、[BlueBubbles](/zh/channels/bluebubbles)、[Mac 远程模式](/zh/platforms/mac/remote)。

<div id="if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro">
  ### 如果我买一台 Mac mini 来运行 OpenClaw，可以把它连接到我的 MacBook Pro 吗
</div>

可以。**Mac mini 可以运行 Gateway**，而你的 MacBook Pro 可以作为
**节点**（辅助设备）连接上来。节点本身不运行 Gateway——它们只是在该设备上提供额外能力，比如屏幕/摄像头/画布，以及通过 `system.run` 在该设备上执行命令。

常见模式：

* Gateway 运行在 Mac mini 上（常开）。
* MacBook Pro 运行 macOS 应用或节点进程，并与 Gateway 配对。
* 使用 `openclaw nodes status` / `openclaw nodes list` 查看。

文档：[Nodes](/zh/nodes)，[Nodes CLI](/zh/cli/nodes)。

<div id="can-i-use-bun">
  ### 我可以使用 Bun 吗
</div>

**不推荐** 使用 Bun。我们观察到它在运行时会出现问题/bug，尤其是在配合 WhatsApp 和 Telegram 使用时。
要获得稳定的 Gateway，请使用 **Node**。

如果你仍然想用 Bun 做实验，请在非生产 Gateway 上进行，
并且不要连接 WhatsApp/Telegram。

<div id="telegram-what-goes-in-allowfrom">
  ### Telegram 的 allowFrom 应该填什么
</div>

`channels.telegram.allowFrom` 是**真人发送者的 Telegram 用户 ID**（推荐使用数字形式）或 `@username`。注意，这里不是机器人的用户名。

更安全的方式（不依赖第三方机器人）：

* 给你的机器人发一条私信，然后运行 `openclaw logs --follow`，查看日志中的 `from.id`。

使用官方 Bot API：

* 给你的机器人发一条私信，然后调用 `https://api.telegram.org/bot<bot_token>/getUpdates`，在响应中查看 `message.from.id`。

使用第三方工具（隐私性较差）：

* 私信 `@userinfobot` 或 `@getidsbot`。

详见 [/channels/telegram](/zh/channels/telegram#access-control-dms--groups)。

<div id="can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances">
  ### 多个人能否使用同一个 WhatsApp 号码搭配不同的 OpenClaw 实例使用
</div>

可以，通过**多智能体路由（multi‑agent routing）**实现。将每个发送方的 WhatsApp **DM**（对端 `kind: "dm"`，发送方的 E.164 号码例如 `+15551234567`）绑定到不同的 `agentId`，这样每个人都会拥有自己的工作区和会话存储。回复仍然来自**同一个 WhatsApp 账号**，并且 DM 访问控制（`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`）是在 WhatsApp 账号级别生效的全局设置。参见 [Multi-Agent Routing](/zh/concepts/multi-agent) 和 [WhatsApp](/zh/channels/whatsapp)。

<div id="can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent">
  ### 我可以同时运行一个用于快速聊天的智能体和一个用于编程的 Opus 智能体吗？
</div>

可以。使用多智能体路由功能：为每个智能体设置各自的默认模型，然后将入站路由（提供方账号或特定对端）绑定到对应的智能体。示例配置见 [Multi-Agent Routing](/zh/concepts/multi-agent)。另请参阅 [Models](/zh/concepts/models) 和 [Configuration](/zh/gateway/configuration)。

<div id="does-homebrew-work-on-linux">
  ### Homebrew 能在 Linux 上使用吗
</div>

可以。Homebrew 支持 Linux（Linuxbrew）。快速安装步骤：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

如果你通过 systemd 运行 OpenClaw，请确保该服务的 PATH 包含 `/home/linuxbrew/.linuxbrew/bin`（或你的 brew 前缀），这样通过 `brew` 安装的工具才能在非登录 shell 中被正确找到并使用。
较新的构建还会在 Linux 的 systemd 服务中预先添加常见的用户 bin 目录（例如 `~/.local/bin`、`~/.npm-global/bin`、`~/.local/share/pnpm`、`~/.bun/bin`），并在设置了 `PNPM_HOME`、`NPM_CONFIG_PREFIX`、`BUN_INSTALL`、`VOLTA_HOME`、`ASDF_DATA_DIR`、`NVM_DIR` 和 `FNM_DIR` 时自动使用这些环境变量指定的路径。

<div id="whats-the-difference-between-the-hackable-git-install-and-npm-install">
  ### “可修改”的 git 安装 和 npm 安装 有什么区别
</div>

* **可修改（git）安装：** 完整源码检出，可编辑，最适合贡献者。
  你在本地运行构建，并可以打补丁修改代码/文档。
* **npm 安装：** 全局 CLI 安装，不包含仓库，最适合“直接跑起来就好”的场景。
  更新通过 npm 的 dist‑tags 分发。

文档：[快速上手](/zh/start/getting-started)、[升级/更新](/zh/install/updating)。

<div id="can-i-switch-between-npm-and-git-installs-later">
  ### 之后可以在 npm 安装和 git 安装之间切换吗
</div>

可以。先安装另一种安装方式，然后运行 Doctor 命令，这样 Gateway 服务就会指向新的入口点。
这**不会删除你的数据**——它只会更改 OpenClaw 代码的安装。你的状态目录
（`~/.openclaw`）和工作区（`~/.openclaw/workspace`）都会保持不变。

从 npm → git：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

从 Git 到 npm：

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor 检测到 Gateway 服务入口点不匹配，并可将服务配置重写为与当前安装一致（在自动化场景中使用 `--repair`）。

备份提示：参见 [备份策略](/zh/help/faq#whats-the-recommended-backup-strategy)。

<div id="should-i-run-the-gateway-on-my-laptop-or-a-vps">
  ### 我应该在笔记本电脑上运行 Gateway，还是在 VPS 上运行？
</div>

简短结论：**如果你需要 24/7 的可靠性，请使用 VPS**。如果你想要最省事的方案，并且可以接受睡眠/重启带来的中断，可以在本机运行。

**笔记本电脑（本地 Gateway）**

* **优点：** 没有服务器成本，可直接访问本地文件，有实时浏览器窗口。
* **缺点：** 休眠/网络掉线 = 断连，系统更新/重启会打断运行，机器必须始终保持唤醒。

**VPS / 云环境**

* **优点：** 始终在线，网络稳定，没有笔记本睡眠问题，更容易保持持续运行。
* **缺点：** 通常以无头方式运行（需要用截图查看界面），只能远程访问文件，更新时需要通过 SSH 操作。

**OpenClaw 相关说明：** WhatsApp/Telegram/Slack/Mattermost（插件）/Discord 在 VPS 上都能正常工作。唯一真正的权衡是 **无头浏览器** 和可见窗口之间的取舍。参见 [Browser](/zh/tools/browser)。

**推荐默认选择：** 如果你之前遇到过 Gateway 断连，建议使用 VPS。当你在主动使用 Mac，并且需要访问本地文件或使用带可见浏览器窗口的 UI 自动化时，本地运行会非常合适。

<div id="how-important-is-it-to-run-openclaw-on-a-dedicated-machine">
  ### 将 OpenClaw 运行在独立机器上有多重要
</div>

不是硬性要求，但**出于可靠性和隔离性，我们强烈推荐这样做**。

* **独立主机（VPS/Mac mini/Pi）：** 长期开机、较少睡眠/重启中断、更干净的权限边界、更容易保持持续运行。
* **共享笔记本/台式机：** 非常适合用于测试和日常主动使用，但当机器睡眠或系统更新时，要预期会出现暂停或中断。

如果你想兼顾两方面的优势，可以把 Gateway 部署在独立主机上，然后将你的笔记本配对为用于本地屏幕/摄像头/命令执行工具的**节点**。参见 [Nodes](/zh/nodes)。
有关安全方面的指导，请阅读 [Security](/zh/gateway/security)。

<div id="what-are-the-minimum-vps-requirements-and-recommended-os">
  ### 最低 VPS 要求和推荐的操作系统是什么
</div>

OpenClaw 本身比较轻量。对于基础的 Gateway + 一个聊天通道：

* **绝对最低配置：** 1 vCPU，1GB 内存，约 500MB 磁盘空间。
* **推荐配置：** 1–2 vCPU，2GB 或以上内存，以预留一定冗余空间（日志、媒体、多个通道）。节点工具和浏览器自动化可能会比较吃资源。

操作系统：使用 **Ubuntu LTS**（或任意较新的 Debian/Ubuntu 发行版）。Linux 安装流程在这些系统上经过了最充分的测试。

文档： [Linux](/zh/platforms/linux)、[VPS hosting](/zh/vps)。

<div id="can-i-run-openclaw-in-a-vm-and-what-are-the-requirements">
  ### 我可以在虚拟机中运行 OpenClaw 吗？有什么要求？
</div>

可以。把虚拟机当作 VPS 来对待：它需要始终运行、可访问，并且有足够的 RAM 来运行 Gateway 和你启用的所有通道。

基线建议：

* **绝对最低配置：** 1 vCPU，1GB RAM。
* **推荐配置：** 如果你运行多个通道、浏览器自动化或媒体工具，建议 2GB RAM 或更多。
* **操作系统：** Ubuntu LTS 或其他现代的 Debian/Ubuntu 发行版。

如果你使用 Windows，**WSL2 是最简单的虚拟机式方案**，并且具有最好的工具兼容性。参见 [Windows](/zh/platforms/windows)、[VPS hosting](/zh/vps)。
如果你在虚拟机中运行 macOS，参见 [macOS VM](/zh/platforms/macos-vm)。

<div id="what-is-openclaw">
  ## OpenClaw 是什么？
</div>

<div id="what-is-openclaw-in-one-paragraph">
  ### 用一段话概括 OpenClaw 是什么
</div>

OpenClaw 是你可以在自己设备上运行的个人 AI 助理。它会在你已经在使用的消息平台上回复你（WhatsApp、Telegram、Slack、Mattermost（插件）、Discord、Google Chat、Signal、iMessage、WebChat），并且在支持的平台上还能进行语音交互以及实时 Canvas 展示。**Gateway** 是始终在线的控制平面；助理本身就是产品。

<div id="whats-the-value-proposition">
  ### 价值主张是什么
</div>

OpenClaw 不只是一个 “Claude 封装器（wrapper）”。它是一个**本地优先的控制平面**，让你可以在**自己的硬件**上运行一个强大的助手，通过你已经在用的聊天应用访问，并且具备有状态的会话、记忆和工具——而无需把你的工作流控制权交给托管的 SaaS。

亮点：

* **你的设备，你的数据：** 在任意位置运行 Gateway（Mac、Linux、VPS），并将工作区和会话历史保存在本地。
* **真实消息通道，而不是 web 沙箱：** 支持 WhatsApp/Telegram/Slack/Discord/Signal/iMessage 等通道，以及在支持的平台上使用移动语音和 Canvas。
* **模型无关：** 可使用 Anthropic、OpenAI、MiniMax、OpenRouter 等，支持按智能体路由和故障转移。
* **纯本地选项：** 运行本地模型，如果你愿意，**所有数据都可以留在你的设备上**。
* **多智能体路由：** 按通道、账号或任务拆分不同的智能体，每个都有自己的工作区和默认配置。
* **开源且易于扩展：** 可检查、扩展并自托管，无厂商锁定。

文档： [Gateway](/zh/gateway)、[Channels](/zh/channels)、[Multi‑agent](/zh/concepts/multi-agent)、[Memory](/zh/concepts/memory)。

<div id="i-just-set-it-up-what-should-i-do-first">
  ### 我刚把它设置好，现在首先该做什么
</div>

适合作为入门练手的项目：

* 搭建一个网站（WordPress、Shopify，或者一个简单的静态站点）。
* 设计一个移动应用原型（大纲、界面、API 方案）。
* 整理文件和文件夹（清理、命名、打标签）。
* 连接 Gmail，并自动生成摘要或跟进邮件。

它可以处理大型任务，但更推荐你先把任务拆分成多个阶段，并使用子智能体并行处理。

<div id="what-are-the-top-five-everyday-use-cases-for-openclaw">
  ### OpenClaw 的五大日常使用场景是什么
</div>

日常高频、最实用的用法通常包括：

* **个人简报：** 汇总你关心的收件箱、日历和新闻。
* **调研与写作：** 快速调研、生成摘要，以及撰写邮件或文档的初稿。
* **提醒与跟进：** 由 cron 或心跳（heartbeat）驱动的提醒和检查清单。
* **浏览器自动化：** 填写表单、收集数据并重复执行网页任务。
* **跨设备协同：** 在手机上发送一个任务，让 Gateway 在服务器上运行，并在聊天中拿回结果。

<div id="can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas">
  ### OpenClaw 能否帮助为 SaaS 做获客外联广告和博客内容
</div>

可以，用于**调研、资格筛选和撰写草稿**。它可以扫描站点、建立候选名单、
总结潜在客户，并撰写外联或广告文案草稿。

对于**外联活动或广告投放**，务必确保有人在环节中把关。避免发送垃圾信息，遵守本地法律和
平台政策，并在发送前审阅所有内容。最安全的模式是让
OpenClaw 负责起草，由你来审批。

文档：[Security](/zh/gateway/security)。

<div id="what-are-the-advantages-vs-claude-code-for-web-development">
  ### 与 Claude Code 相比，用于 Web 开发有什么优势
</div>

OpenClaw 是一个**个人助手**和协调层，而不是 IDE 替代品。\
在代码仓库中需要最快速的直接编码迭代时，请使用 Claude Code 或 Codex。\
当你需要持久记忆、跨设备访问和工具编排时，请使用 OpenClaw。

优势：

* 跨会话的**持久记忆 + 工作区**
* **多平台访问**（WhatsApp、Telegram、TUI、WebChat）
* **工具编排**（浏览器、文件、调度、hooks）
* **始终在线的 Gateway**（运行在 VPS 上，可从任意位置访问）
* 用于本地 browser/screen/camera/exec 的 **节点**

展示页：https://openclaw.ai/showcase

<div id="skills-and-automation">
  ## 技能与自动化
</div>

<div id="how-do-i-customize-skills-without-keeping-the-repo-dirty">
  ### 如何在不弄脏仓库的情况下自定义技能
</div>

使用托管覆盖（managed overrides），而不是直接修改仓库中的副本。将你的更改放在 `~/.openclaw/skills/<name>/SKILL.md` 中（或者在 `~/.openclaw/openclaw.json` 里通过 `skills.load.extraDirs` 添加一个文件夹）。优先级顺序为 `<workspace>/skills` &gt; `~/.openclaw/skills` &gt; 内置（打包）版本，因此托管覆盖会在不动 git 的情况下优先生效。只有真正值得上游合并的修改才应该保留在仓库中，并通过 PR 提交。

<div id="can-i-load-skills-from-a-custom-folder">
  ### 我可以从自定义文件夹加载技能吗
</div>

可以。通过在 `~/.openclaw/openclaw.json` 中配置 `skills.load.extraDirs` 来添加额外目录（优先级最低）。默认搜索优先级顺序保持为：`<workspace>/skills` → `~/.openclaw/skills` → 内置 → `skills.load.extraDirs`。`clawhub` 默认安装到 `./skills`，OpenClaw 会将其视为 `<workspace>/skills`。

<div id="how-can-i-use-different-models-for-different-tasks">
  ### 如何为不同任务使用不同的模型
</div>

目前支持的方式包括：

* **Cron jobs**：相互独立的任务可以为每个任务单独设置 `model` 覆盖。
* **Sub-agents**：将任务路由到具有不同默认模型的独立子智能体。
* **按需切换**：使用 `/model` 随时切换当前会话所使用的模型。

参见 [Cron jobs](/zh/automation/cron-jobs)、[Multi-Agent Routing](/zh/concepts/multi-agent) 和 [Slash commands](/zh/tools/slash-commands)。

<div id="the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that">
  ### 机器人在执行繁重任务时会卡住 我该如何把这些任务分出去
</div>

对耗时长或需要并行的任务使用 **子智能体（sub-agents）**。子智能体在各自独立的会话中运行，
返回摘要结果，同时让你的主对话保持响应。

让你的机器人“为这个任务创建一个子智能体”，或者使用 `/subagents`。
在聊天中使用 `/status` 查看 Gateway 此刻在做什么（以及它是否正忙）。

Token 小提示：长任务和子智能体都会消耗 token。若你在意成本，可以通过 `agents.defaults.subagents.model`
为子智能体设置更便宜的模型。

文档：[子智能体](/zh/tools/subagents)。

<div id="cron-or-reminders-do-not-fire-what-should-i-check">
  ### Cron 或提醒没有触发：我应该检查什么？
</div>

Cron 在 Gateway 进程内运行。如果 Gateway 没有持续运行，
计划任务将不会执行。

排查清单：

* 确认已启用 cron（`cron.enabled`），且未设置 `OPENCLAW_SKIP_CRON`。
* 检查 Gateway 是否 24/7 持续运行（无睡眠/无重启）。
* 核实该任务的时区设置（`--tz` 与宿主机时区是否一致）。

调试：

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

相关文档：[Cron 任务](/zh/automation/cron-jobs)、[Cron 与心跳对比](/zh/automation/cron-vs-heartbeat)。

<div id="how-do-i-install-skills-on-linux">
  ### 如何在 Linux 上安装技能
</div>

使用 **ClawHub**（CLI），或者将技能直接放入你的工作区。macOS 技能 UI 在 Linux 上不可用。
在 https://clawhub.com 浏览技能。

安装 ClawHub CLI（选择一个包管理器）：

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

<div id="can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background">
  ### OpenClaw 能否按计划运行任务，或在后台持续运行任务
</div>

可以。请使用 Gateway 的调度器：

* **Cron 任务**：用于计划或周期性任务（重启后依然保留）。
* **Heartbeat**：用于“主会话”的周期性检查。
* **隔离任务**：用于自主智能体，它们会发布摘要或推送到聊天。

文档参见：[Cron jobs](/zh/automation/cron-jobs)、[Cron vs Heartbeat](/zh/automation/cron-vs-heartbeat)、[Heartbeat](/zh/gateway/heartbeat)。

**我能在 Linux 上运行仅支持 Apple macOS 的技能吗**

不能直接运行。macOS 技能会受 `metadata.openclaw.os` 以及所需二进制文件的限制，且只有在 **Gateway 主机** 上满足条件时，这些技能才会出现在系统提示中。在 Linux 上，`darwin` 专用技能（如 `imsg`、`apple-notes`、`apple-reminders`）不会加载，除非你覆盖这一限制。

你有三种受支持的方式：

**方案 A——在 Mac 上运行 Gateway（最简单）。**\
在存在 macOS 二进制文件的机器上运行 Gateway，然后从 Linux 通过[远程模式](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)或通过 Tailscale 连接。由于 Gateway 主机是 macOS，技能会正常加载。

**方案 B——使用 macOS 节点（无需 SSH）。**\
在 Linux 上运行 Gateway，配对一个 macOS 节点（菜单栏应用），并在 Mac 上将 **Node Run Commands** 设置为 &quot;Always Ask&quot; 或 &quot;Always Allow&quot;。当节点上存在所需二进制文件时，OpenClaw 会将仅限 macOS 的技能视为可用。智能体通过 `nodes` 工具运行这些技能。如果选择 &quot;Always Ask&quot;，在提示中批准 &quot;Always Allow&quot; 会将该命令加入允许列表。

**方案 C——通过 SSH 代理 macOS 二进制文件（高级用法）。**\
保持 Gateway 运行在 Linux 上，但让所需的 CLI 二进制解析到在 Mac 上运行的 SSH 包装脚本。然后覆盖技能配置以允许 Linux，使其保持可用。

1. 为该二进制创建一个 SSH 包装脚本（例如：`imsg`）：
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2. 将包装脚本放到 Linux 主机的 `PATH` 中（例如 `~/bin/imsg`）。
3. 覆盖技能的元数据（在工作区或 `~/.openclaw/skills` 中）以允许 Linux：
   ```markdown
   ---
   name: imsg
   description: iMessage/SMS CLI for listing chats, history, watch, and sending.
   metadata: {"openclaw":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4. 启动一个新会话，以便刷新技能快照。

对于 iMessage，你也可以将 `channels.imessage.cliPath` 指向一个 SSH 包装脚本（OpenClaw 只需要标准输入输出 stdio）。参见 [iMessage](/zh/channels/imessage)。

<div id="do-you-have-a-notion-or-heygen-integration">
  ### 你们有 Notion 或 HeyGen 集成吗
</div>

目前还没有内置支持。

可选方案：

* **自定义技能 / 插件：** 适合需要稳定 API 访问的场景（Notion/HeyGen 都提供 API）。
* **浏览器自动化：** 不需要写代码，但更慢，也更脆弱。

如果你想按客户保持上下文（例如代理机构的工作流），一个简单的模式是：

* 每个客户一个 Notion 页面（上下文 + 偏好 + 当前工作）。
* 在每次会话开始时，让智能体先获取该页面。

如果你需要原生集成，可以提交一个功能请求，或者自己构建一个
面向这些 API 的技能。

安装技能：

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub 会安装到你当前目录下的 `./skills`（否则会回退到你在 OpenClaw 中配置的工作区）；在下一次会话中，OpenClaw 会将其视为 `<workspace>/skills`。对于要在多个智能体之间共享的技能，请将它们放在 `~/.openclaw/skills/<name>/SKILL.md` 中。某些技能需要通过 Homebrew 安装的二进制文件；在 Linux 上则意味着使用 Linuxbrew（参见上文 Homebrew Linux 的常见问题条目）。参见 [Skills](/zh/tools/skills) 和 [ClawHub](/zh/tools/clawhub)。

<div id="how-do-i-install-the-chrome-extension-for-browser-takeover">
  ### 如何安装用于接管浏览器的 Chrome 扩展程序
</div>

使用内置安装向导，然后在 Chrome 中加载未打包的扩展程序：

```bash
openclaw browser extension install
openclaw browser extension path
```

然后在 Chrome 中打开 `chrome://extensions`，启用“开发者模式”，点击“加载已解压的扩展程序”，选择该文件夹。

完整指南（包括远程 Gateway 场景和安全说明）：[Chrome extension](/zh/tools/chrome-extension)

如果 Gateway 和 Chrome 运行在同一台机器上（默认设置），通常**不需要**额外配置。
如果 Gateway 运行在其他机器上，则需要在浏览器所在的机器上运行一个节点，这样 Gateway 才能代理浏览器操作。
你仍然需要在想要控制的标签页上手动点击扩展按钮（它不会自动附加到标签页）。

<div id="sandboxing-and-memory">
  ## 沙箱和内存
</div>

<div id="is-there-a-dedicated-sandboxing-doc">
  ### 有单独的沙箱文档吗
</div>

有。参见 [Sandboxing](/zh/gateway/sandboxing)。关于 Docker 相关的设置（在 Docker 中运行完整 Gateway 或沙箱镜像），参见 [Docker](/zh/install/docker)。

**我能否让私信保持私密，但在同一个 Agent 中让群组以公共沙箱方式运行**

可以——前提是你的私有流量是 **DMs**，公共流量是 **groups**。

使用 `agents.defaults.sandbox.mode: "non-main"`，这样群组/频道会话（非 main 键）会在 Docker 中运行，而主 DM 会话保留在宿主机上。然后通过 `tools.sandbox.tools` 限制沙箱会话中可用的工具。

配置演练 + 示例配置： [Groups: personal DMs + public groups](/zh/concepts/groups#pattern-personal-dms-public-groups-single-agent)

关键配置参考： [Gateway configuration](/zh/gateway/configuration#agentsdefaultssandbox)

<div id="how-do-i-bind-a-host-folder-into-the-sandbox">
  ### 如何将宿主机文件夹挂载到沙箱中
</div>

将 `agents.defaults.sandbox.docker.binds` 设置为 `["host:path:mode"]`（例如 `"/home/user/src:/src:ro"`）。全局挂载与每个智能体的挂载会合并；当 `scope: "shared"` 时，会忽略每个智能体单独配置的挂载。对于任何敏感路径一律使用 `:ro`，并牢记这些挂载会绕过沙箱的文件系统隔离。示例和安全注意事项参见 [Sandboxing](/zh/gateway/sandboxing#custom-bind-mounts) 与 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check)。

<div id="how-does-memory-work">
  ### 记忆是如何工作的
</div>

OpenClaw 的记忆本质上就是智能体工作区里的 Markdown 文件：

* `memory/YYYY-MM-DD.md` 中的每日笔记
* `MEMORY.md` 中精心整理的长期笔记（仅适用于主会话和私有会话）

OpenClaw 还会运行一次**静默的预压缩记忆刷新**，在自动压缩前提醒模型
把需要持久化的笔记写入文件。仅当工作区可写时才会运行（只读沙箱会跳过）。
参见 [Memory](/zh/concepts/memory)。

<div id="memory-keeps-forgetting-things-how-do-i-make-it-stick">
  ### 记忆老是忘东西 我该如何让它记住？
</div>

让机器人 **把这个事实写入记忆**。长期笔记应该写在 `MEMORY.md` 中，
短期上下文则写入 `memory/YYYY-MM-DD.md`。

这部分功能我们还在持续改进中。你可以提醒模型把内容存进记忆；
它会知道该怎么做。如果它还是一直在忘，检查一下 Gateway 在每次运行时
是否都在使用同一个工作区。

文档：[Memory](/zh/concepts/memory)、[Agent 工作区](/zh/concepts/agent-workspace)。

<div id="does-semantic-memory-search-require-an-openai-api-key">
  ### 语义记忆搜索是否需要 OpenAI API 密钥
</div>

只有在你使用 **OpenAI embeddings** 时才需要。Codex OAuth 覆盖的是对话/补全，
**不会**授予 embeddings 访问权限，因此**使用 Codex 登录（OAuth 或 Codex CLI 登录）**
对语义记忆搜索没有帮助。OpenAI embeddings 仍然需要实际的 API 密钥
（`OPENAI_API_KEY` 或 `models.providers.openai.apiKey`）。

如果你没有显式设置提供方，OpenClaw 会在能解析到 API 密钥时自动选择提供方
（认证配置、`models.providers.*.apiKey` 或环境变量）。
如果能解析到 OpenAI 密钥，则优先使用 OpenAI；否则在能解析到 Gemini 密钥时使用 Gemini。
如果两个密钥都不可用，记忆搜索会保持禁用状态，直到你完成配置。
如果你配置并提供了本地模型路径，OpenClaw 会优先选择 `local`。

如果你希望保持本地优先，设置 `memorySearch.provider = "local"`（并可选设置
`memorySearch.fallback = "none"`）。如果你想使用 Gemini embeddings，
设置 `memorySearch.provider = "gemini"` 并提供 `GEMINI_API_KEY`（或
`memorySearch.remote.apiKey`）。我们支持 **OpenAI、Gemini 或本地** embedding 模型——
有关详细配置请参见 [Memory](/zh/concepts/memory)。

<div id="does-memory-persist-forever-what-are-the-limits">
  ### 记忆会永久保留吗？有哪些限制？
</div>

记忆文件保存在磁盘上，会一直存在，直到你手动删除它们。限制来自你的存储空间，而不是模型本身。**会话上下文**仍然受模型上下文窗口限制，所以很长的对话可能会被压缩或截断。这就是提供记忆搜索功能的原因——它只会把相关的部分重新加入到上下文中。

文档：[Memory](/zh/concepts/memory)、[Context](/zh/concepts/context)。

<div id="where-things-live-on-disk">
  ## 磁盘上的数据存放位置
</div>

<div id="is-all-data-used-with-openclaw-saved-locally">
  ### 使用 OpenClaw 的所有数据都会保存在本地吗
</div>

不会——**OpenClaw 的状态是本地的**，但**外部服务仍然能看到你发送给它们的内容**。

* **默认本地：** 会话、记忆文件、配置和工作区都存储在 Gateway 主机上
  （`~/.openclaw` + 你的工作区目录）。
* **因需要而远程：** 你发送给模型提供方（Anthropic/OpenAI 等）的消息会发到它们的
  API，聊天平台（WhatsApp/Telegram/Slack 等）会在它们的服务器上存储消息数据。
* **你可以控制数据足迹：** 使用本地模型可以让提示词保留在你的机器上，但频道流量仍会通过
  各频道的服务器。

相关内容：[Agent 工作区](/zh/concepts/agent-workspace)、[记忆](/zh/concepts/memory)。

<div id="where-does-openclaw-store-its-data">
  ### OpenClaw 在哪里存储其数据
</div>

所有内容都位于 `$OPENCLAW_STATE_DIR` 下（默认：`~/.openclaw`）：

| Path | Purpose |
|------|---------|
| `$OPENCLAW_STATE_DIR/openclaw.json` | 主配置文件（JSON5） |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json` | 旧版 OAuth 导入（首次使用时复制到认证配置中） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | 认证配置（OAuth + API keys） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json` | 运行时认证缓存（自动管理） |
| `$OPENCLAW_STATE_DIR/credentials/` | 提供方状态数据（例如 `whatsapp/<accountId>/creds.json`） |
| `$OPENCLAW_STATE_DIR/agents/` | 每个智能体的状态目录（agentDir + 会话） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` | 会话历史与状态（按智能体划分） |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json` | 会话元数据（按智能体划分） |

旧版单智能体路径：`~/.openclaw/agent/*`（由 `openclaw doctor` 迁移）。

你的 **工作区**（AGENTS.md、memory 文件、技能等）是单独分开的，通过 `agents.defaults.workspace` 配置（默认：`~/.openclaw/workspace`）。

<div id="where-should-agentsmd-soulmd-usermd-memorymd-live">
  ### `AGENTS.md`、`SOUL.md`、`USER.md`、`MEMORY.md` 应该放在哪里
</div>

这些文件应位于 **智能体工作区（agent workspace）**，而不是 `~/.openclaw`。

* **工作区（按智能体划分）**：`AGENTS.md`、`SOUL.md`、`IDENTITY.md`、`USER.md`、
  `MEMORY.md`（或 `memory.md`）、`memory/YYYY-MM-DD.md`、可选的 `HEARTBEAT.md`。
* **状态目录（`~/.openclaw`）**：配置、凭据、认证配置、会话、日志，
  以及共享技能（`~/.openclaw/skills`）。

默认工作区目录为 `~/.openclaw/workspace`，可通过以下方式进行配置：

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

如果机器人在重启后“忘记”了内容，确认 Gateway 在每次启动时都使用同一个
工作区（并且记住：remote 模式使用的是 **Gateway 主机的**
工作区，而不是你本地笔记本上的）。

提示：如果你希望某个行为或偏好是持久的，应该让机器人**把它写入
AGENTS.md 或 MEMORY.md 文件中**，而不是只依赖聊天记录。

参见 [Agent 工作区](/zh/concepts/agent-workspace) 和 [Memory](/zh/concepts/memory)。

<div id="whats-the-recommended-backup-strategy">
  ### 推荐的备份策略
</div>

将你的 **Agent 工作区（agent workspace）** 存放在一个 **私有** 的 git 仓库中，并额外备份到某个私有位置
（例如 GitHub 私有仓库）。这样可以包含 memory 以及 AGENTS/SOUL/USER
文件，使你之后可以恢复助理的“⼤脑”。

**不要** 将 `~/.openclaw` 下的任何内容（credentials、sessions、tokens）提交到仓库。
如果你需要完全恢复，请分别备份工作区和 state 目录
（参见上面的迁移问题）。

文档：[Agent workspace](/zh/concepts/agent-workspace)。

<div id="how-do-i-completely-uninstall-openclaw">
  ### 如何完全卸载 OpenClaw
</div>

请参阅专门的指南：[卸载](/zh/install/uninstall)。

<div id="can-agents-work-outside-the-workspace">
  ### 智能体可以在工作区之外工作吗
</div>

可以。工作区是**默认 cwd（当前工作目录）**和记忆锚点，而不是严格意义上的沙箱。
相对路径会在工作区内解析，但绝对路径可以访问主机上的其他位置，除非启用了沙箱。
如果你需要隔离，请使用 [`agents.defaults.sandbox`](/zh/gateway/sandboxing)
或为单个智能体配置沙箱设置。如果你希望某个仓库成为默认工作目录，
就把该智能体的 `workspace` 指向该仓库根目录。OpenClaw 仓库只是源代码；
请保持工作区独立，除非你确实希望智能体在仓库内部工作。

示例（将仓库作为默认 cwd）：

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo"
    }
  }
}
```

<div id="im-in-remote-mode-where-is-the-session-store">
  ### 我在远程模式下，会话存储在哪里？
</div>

会话状态由 **Gateway 主机** 管理。 如果你处于远程模式，会话存储位于远程机器上，而不是你的本地电脑。参见 [会话管理](/zh/concepts/session)。

<div id="config-basics">
  ## 配置基础知识
</div>

<div id="what-format-is-the-config-where-is-it">
  ### 配置文件是什么格式？存放在哪里？
</div>

OpenClaw 会从 `$OPENCLAW_CONFIG_PATH` 读取一个可选的 **JSON5** 配置文件（默认：`~/.openclaw/openclaw.json`）：

```
$OPENCLAW_CONFIG_PATH
```

如果该文件不存在，系统会使用相对安全的默认配置（其中包括将 `~/.openclaw/workspace` 作为默认工作区）。

<div id="i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized">
  ### 我把 gatewaybind 设置为 lan 或 tailnet 后，现在没有任何服务在监听，而且 UI 显示“unauthorized”
</div>

绑定到非回环地址时**必须启用认证**。请配置 `gateway.auth.mode` 和 `gateway.auth.token`（或使用环境变量 `OPENCLAW_GATEWAY_TOKEN`）。

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me"  // 替换为你的令牌
    }
  }
}
```

Notes:

* `gateway.remote.token` 仅用于 **远程 CLI 调用**；它不会启用本地 Gateway 的身份验证。
* Control UI 通过 `connect.params.auth.token`（存储在应用/UI 设置中）进行身份验证。避免在 URL 中直接包含 token。

<div id="why-do-i-need-a-token-on-localhost-now">
  ### 为什么现在在 localhost 上也需要令牌？
</div>

向导默认会生成一个 Gateway 令牌（即使是在回环地址上），因此**本地 WS 客户端必须先进行身份验证**。这样可以阻止其他本地进程随意调用 Gateway。将该令牌粘贴到 Control UI 的设置中（或你的客户端配置中）即可连接。

如果你**真的**想开放回环访问，可以从配置中移除 `gateway.auth`。`openclaw doctor` 命令可以在任何时候为你生成令牌：`openclaw doctor --generate-gateway-token`。

<div id="do-i-have-to-restart-after-changing-config">
  ### 更改配置后必须重启吗
</div>

Gateway 会监视配置并支持热重载：

* `gateway.reload.mode: "hybrid"`（默认）：对安全的配置变更进行热应用，关键变更则需要重启
* 同时也支持 `hot`、`restart`、`off`

<div id="how-do-i-enable-web-search-and-web-fetch">
  ### 如何启用 web search 和 web fetch
</div>

`web_fetch` 在没有 API 密钥的情况下也能工作。`web_search` 需要 Brave Search 的 API
密钥。**推荐做法：**运行 `openclaw configure --section web` 将其存储到
`tools.web.search.apiKey` 中。环境变量方式：为 Gateway 进程设置 `BRAVE_API_KEY`。

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5
      },
      fetch: {
        enabled: true
      }
    }
  }
}
```

注意：

* 如果你启用了允许列表，添加 `web_search`/`web_fetch` 或 `group:web`。
* `web_fetch` 默认启用（除非显式禁用）。
* 守护进程会从 `~/.openclaw/.env`（或服务的环境中）读取环境变量。

文档：[Web 工具](/zh/tools/web)。

<div id="how-do-i-run-a-central-gateway-with-specialized-workers-across-devices">
  ### 如何在多设备上运行中心 Gateway 并配合专用 worker
</div>

常见模式是 **一个 Gateway**（例如 Raspberry Pi）加上 **节点** 和 **智能体**：

* **Gateway（中心）：** 管理通道（Signal/WhatsApp）、路由和会话。
* **节点（设备）：** Mac/iOS/Android 作为外设连接到 Gateway，并暴露本地工具（`system.run`、`canvas`、`camera`）。
* **Agent 代理（worker）：** 拥有各自独立的大脑/工作区，用于承担特定角色（例如 “Hetzner 运维”、“Personal data”）。
* **子智能体：** 当你需要并行处理时，从主智能体中派生后台任务。
* **TUI：** 连接到 Gateway 并在智能体/会话之间切换。

文档： [节点](/zh/nodes)、[远程访问](/zh/gateway/remote)、[多智能体路由](/zh/concepts/multi-agent)、[子智能体](/zh/tools/subagents)、[TUI](/zh/tui)。

<div id="can-the-openclaw-browser-run-headless">
  ### OpenClaw 浏览器可以以无头模式运行吗
</div>

可以。这是一个可配置项：

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } }
    }
  }
}
```

默认值为 `false`（有界面/headful）。无头模式（headless）在部分网站上更有可能触发反机器人检测。参见 [Browser](/zh/tools/browser)。

无头模式使用**相同的 Chromium 引擎**，对大多数自动化场景（表单、点击、爬取、登录）都能正常工作。主要差异在于：

* 没有可见的浏览器窗口（如果需要可视画面，请使用截图）。
* 某些网站在无头模式下对自动化检测更为严格（CAPTCHA、人机验证、反机器人机制）。
  例如，X/Twitter 经常会阻止无头模式下的会话。

<div id="how-do-i-use-brave-for-browser-control">
  ### 如何使用 Brave 进行浏览器控制
</div>

将 `browser.executablePath` 设置为 Brave 的可执行文件路径（或任意基于 Chromium 的浏览器），然后重启 Gateway。
完整配置示例参见 [Browser](/zh/tools/browser#use-brave-or-another-chromium-based-browser)。

<div id="remote-gateways-nodes">
  ## 远程 Gateway 和节点
</div>

<div id="how-do-commands-propagate-between-telegram-the-gateway-and-nodes">
  ### How do commands propagate between Telegram the gateway and nodes
</div>

命令如何在 Telegram、Gateway 和节点之间传播

Telegram messages are handled by the **gateway**. The gateway runs the agent and
only then calls nodes over the **Gateway WebSocket** when a node tool is needed:

**Gateway** 负责处理 Telegram 消息。Gateway 会运行 Agent 代理，并且只有在需要节点工具时，才会通过 **Gateway WebSocket** 调用节点：

Telegram → Gateway → Agent → `node.*` → Node → Gateway → Telegram

Nodes don’t see inbound provider traffic; they only receive node RPC calls.

节点不会直接看到提供方的入站流量；它们只会接收发往节点的 RPC 调用。

<div id="how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely">
  ### 如果 Gateway 部署在远程，我的智能体如何访问我的电脑
</div>

简短回答：**把你的电脑配对为一个节点**。Gateway 虽然运行在其他地方，但它可以通过 Gateway 的 WebSocket，在你的本地电脑上调用 `node.*` 工具（screen、camera、system）。

典型配置：

1. 在一台常开主机（VPS / 家用服务器）上运行 Gateway。
2. 将 Gateway 主机和你的电脑加入同一个 tailnet。
3. 确保 Gateway 的 WS 可达（通过 tailnet 绑定或 SSH 隧道）。
4. 在本地打开 macOS 应用，并以 **Remote over SSH** 模式（或直接通过 tailnet）连接，
   这样它就可以注册为一个节点。
5. 在 Gateway 上批准该节点：
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

不需要额外的 TCP 桥接；节点通过 Gateway 的 WebSocket 进行连接。

安全提醒：将一个 macOS 节点配对后，就允许在那台机器上执行 `system.run`。只配对你信任的设备，并查看 [Security](/zh/gateway/security)。

文档：[Nodes](/zh/nodes)、[Gateway protocol](/zh/gateway/protocol)、[macOS remote mode](/zh/platforms/mac/remote)、[Security](/zh/gateway/security)。

<div id="tailscale-is-connected-but-i-get-no-replies-what-now">
  ### Tailscale 已连接但收不到回复 接下来怎么办
</div>

先检查基础情况：

* Gateway 是否在运行：`openclaw gateway status`
* Gateway 健康状况：`openclaw status`
* Channel 健康状况：`openclaw channels status`

然后确认认证和路由：

* 如果你使用 Tailscale Serve，确保 `gateway.auth.allowTailscale` 设置正确。
* 如果你通过 SSH 隧道连接，确认本地隧道已建立且指向正确的端口。
* 确认你的允许列表（私信或群组）中包含你的账号。

文档：[Tailscale](/zh/gateway/tailscale)、[远程访问](/zh/gateway/remote)、[频道](/zh/channels)。

<div id="can-two-openclaw-instances-talk-to-each-other-local-vps">
  ### 本地或 VPS 上的两个 OpenClaw 实例能互相通信吗
</div>

可以。虽然没有内置的 “bot-to-bot” 桥接机制，但你可以通过几种可靠的方式自行搭建：

**最简单的方法：** 使用两个机器人都能访问的普通聊天通道（Telegram/Slack/WhatsApp）。
让 Bot A 向 Bot B 发送一条消息，然后让 Bot B 像平常一样回复。

**CLI 桥接（通用）：** 运行一个脚本，使用
`openclaw agent --message ... --deliver` 调用另一个 Gateway，
目标是另一个机器人正在监听的聊天会话。如果其中一个机器人运行在远程 VPS 上，
可以通过 SSH/Tailscale 将你的 CLI 指向那个远程 Gateway（参见 [Remote access](/zh/gateway/remote)）。

示例模式（在一台可以访问目标 Gateway 的机器上运行）：

```bash
openclaw agent --message "来自本地机器人的问候" --deliver --channel telegram --reply-to <chat-id>
```

提示：添加安全护栏，避免两个机器人互相无限循环对话（仅限提及、频道允许列表，或者 &quot;不要回复机器人消息&quot; 规则）。

文档：[远程访问](/zh/gateway/remote)、[Agent 代理 CLI](/zh/cli/agent)、[Agent 发送](/zh/tools/agent-send)。

<div id="do-i-need-separate-vpses-for-multiple-agents">
  ### 针对多个智能体，我需要单独的 VPS 吗
</div>

不需要。一个 Gateway 可以托管多个智能体，每个都有自己的工作区、模型默认设置和路由。这是常见的部署方式，比为每个智能体各自运行一个 VPS 要便宜得多，也简单得多。

只有在你需要强隔离（严格的安全边界），或有你不希望共享、差异很大的配置时，才使用单独的 VPS。否则，就保留一个 Gateway，并使用多个智能体或子智能体。

<div id="is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps">
  ### 在我的个人笔记本上运行节点，相比从 VPS 用 SSH，有什么好处吗
</div>

有的——节点是从远程 Gateway 访问你笔记本的标准方式，而且提供的不仅仅是 shell 访问。Gateway 运行在 macOS/Linux（Windows 通过 WSL2），本身很轻量（小型 VPS 或 Raspberry Pi 级别的机器就够用；4 GB 内存就已经很宽裕），所以常见的部署是：一台始终在线的主机，再加上你的笔记本作为一个节点。

* **不需要入站 SSH。** 节点会主动连接到 Gateway 的 WS（WebSocket）并使用设备配对。
* **更安全的执行控制。** `system.run` 会受该笔记本上的节点允许列表/审批机制保护。
* **更多设备工具。** 节点除了暴露 `system.run` 之外，还会暴露 `canvas`、`camera` 和 `screen`。
* **本地浏览器自动化。** 可以把 Gateway 放在 VPS 上，但在本地运行 Chrome，通过 Chrome 扩展 + 在笔记本上运行的节点中继控制。

SSH 适合临时的 shell 访问，但对于持续性的智能体工作流和设备自动化，节点更简单好用。

文档： [Nodes](/zh/nodes)、[Nodes CLI](/zh/cli/nodes)、[Chrome extension](/zh/tools/chrome-extension)。

<div id="should-i-install-on-a-second-laptop-or-just-add-a-node">
  ### 我应该在第二台笔记本上再安装一个 Gateway，还是只添加一个节点
</div>

如果你只需要在第二台笔记本上使用 **本地工具**（screen/camera/exec），就把这台机器添加为一个
**节点**。这样可以只保留一个 Gateway，避免重复配置。本地节点工具目前仅支持 macOS，但我们计划扩展到其他操作系统。

只有当你需要 **强隔离** 或者两个完全独立的助手时，才需要在第二台机器上再安装一个 Gateway。

文档：[Nodes](/zh/nodes)、[Nodes CLI](/zh/cli/nodes)、[Multiple gateways](/zh/gateway/multiple-gateways)。

<div id="do-nodes-run-a-gateway-service">
  ### 节点会运行 Gateway 服务吗
</div>

不会。除非你有意运行彼此隔离的配置文件（参见 [多个 Gateway](/zh/gateway/multiple-gateways)），否则**每台主机上只应运行一个 Gateway**。节点是连接到 Gateway 的外围设备（iOS/Android 节点，或 macOS 菜单栏应用中的“节点模式”）。对于无头节点主机和 CLI 控制，参见 [Node host CLI](/zh/cli/node)。

对 `gateway`、`discovery` 和 `canvasHost` 的更改都需要完全重启。

<div id="is-there-an-api-rpc-way-to-apply-config">
  ### 是否有以 API/RPC 方式应用配置的方法
</div>

有。`config.apply` 会在该操作中校验并写入完整配置，并重启 Gateway。

<div id="configapply-wiped-my-config-how-do-i-recover-and-avoid-this">
  ### configapply 把我的配置清空了，如何恢复并避免再次发生？
</div>

`config.apply` 会替换**整个配置**。如果你只提交了部分配置对象，其余所有内容都会被删除。

恢复方法：

* 从备份中恢复（git 或已拷贝的 `~/.openclaw/openclaw.json`）。
* 如果没有备份，重新运行 `openclaw doctor` 并重新配置 channels/models。
* 如果这是意外情况，请提交一个 bug 报告，并附上你最后一次已知的配置或任何备份。
* 本地编码 Agent 代理通常可以根据日志或历史记录重建一个可用的配置。

避免这种情况：

* 对于小改动，使用 `openclaw config set`。
* 对于交互式编辑，使用 `openclaw configure`。

文档：[Config](/zh/cli/config)、[Configure](/zh/cli/configure)、[Doctor](/zh/gateway/doctor)。

<div id="whats-a-minimal-sane-config-for-a-first-install">
  ### 初次安装时，推荐的最小合理配置是什么
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

这将设置你的工作区，并限制谁可以触发机器人。

<div id="how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac">
  ### 如何在 VPS 上设置 Tailscale 并从 Mac 连接
</div>

最简步骤：

1. **在 VPS 上安装并登录**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. **在你的 Mac 上安装并登录**
   * 使用 Tailscale 应用并登录到同一个 Tailnet。
3. **启用 MagicDNS（推荐）**
   * 在 Tailscale 管理控制台中启用 MagicDNS，使 VPS 拥有稳定的主机名。
4. **使用 Tailnet 主机名**
   * SSH：`ssh user@your-vps.tailnet-xxxx.ts.net`
   * Gateway WS：`ws://your-vps.tailnet-xxxx.ts.net:18789`

如果你希望在不使用 SSH 的情况下访问 Control UI，请在 VPS 上使用 Tailscale Serve：

```bash
openclaw gateway --tailscale serve
```

这会让 Gateway 仅绑定在本地回环地址上，并通过 Tailscale 提供 HTTPS 服务。参见 [Tailscale](/zh/gateway/tailscale)。

<div id="how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve">
  ### 如何通过 Tailscale Serve 将 Mac 节点连接到远程 Gateway
</div>

Serve 会暴露 **Gateway Control UI + WS**。节点通过同一个 Gateway 的 WS 端点进行连接。

推荐设置方式：

1. **确保 VPS 和 Mac 位于同一个 tailnet 中**。
2. **在 macOS 应用中使用 Remote 模式**（SSH 目标可以是 tailnet 主机名）。
   应用会为 Gateway 端口建立隧道，并作为一个节点进行连接。
3. **在 Gateway 上批准该节点**：
   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

文档：[Gateway protocol](/zh/gateway/protocol)、[Discovery](/zh/gateway/discovery)、[macOS remote mode](/zh/platforms/mac/remote)。

<div id="env-vars-and-env-loading">
  ## 环境变量与 .env 文件加载
</div>

<div id="how-does-openclaw-load-environment-variables">
  ### OpenClaw 如何加载环境变量
</div>

OpenClaw 会从父进程（shell、launchd/systemd、CI 等）读取环境变量，并另外加载：

* 当前工作目录下的 `.env`
* 全局回退用的 `.env`：`~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`）

这两个 `.env` 文件都不会覆盖已经存在的环境变量。

你也可以在配置中内联定义环境变量（仅在进程环境中缺失时才会生效）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

有关完整的优先级顺序及其来源，请参见 [/environment](/zh/environment)。

<div id="i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now">
  ### 我通过服务启动 Gateway 后发现环境变量都没了，现在怎么办
</div>

有两种常见的解决方法：

1. 把缺失的键写到 `~/.openclaw/.env` 里，这样即使服务没有继承你的 shell 环境变量也能加载到。
2. 启用从 shell 导入（需要主动开启的便捷选项）：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

这会运行你的登录 shell，并且只导入缺失的预期键（绝不会覆盖已有值）。对应的环境变量为：
`OPENCLAW_LOAD_SHELL_ENV=1`，`OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`。

<div id="i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why">
  ### 我设置了 COPILOTGITHUBTOKEN，但 `models status` 显示 Shell env off，为什么？
</div>

`openclaw models status` 会报告 **shell env import** 是否已启用。“Shell env: off”
**并不**表示你的环境变量缺失——这只意味着 OpenClaw 不会自动加载
你的登录 shell。

如果 Gateway 作为服务运行（launchd/systemd），它不会继承你的 shell
环境。可以通过以下任一方式解决：

1. 把 token 放到 `~/.openclaw/.env` 中：
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2. 或者启用 shell 环境导入（`env.shellEnv.enabled: true`）。
3. 或者把它添加到配置文件中的 `env` 块（仅在缺失时才生效）。

然后重启 Gateway 并重新检查：

```bash
openclaw models status
```

Copilot 令牌从 `COPILOT_GITHUB_TOKEN`（以及 `GH_TOKEN` / `GITHUB_TOKEN`）中读取。
参见 [/concepts/model-providers](/zh/concepts/model-providers) 和 [/environment](/zh/environment)。

<div id="sessions-multiple-chats">
  ## 会话与多个对话
</div>

<div id="how-do-i-start-a-fresh-conversation">
  ### 如何开始一段全新的对话
</div>

将 `/new` 或 `/reset` 作为一条独立消息发送。参见[会话管理](/zh/concepts/session)。

<div id="do-sessions-reset-automatically-if-i-never-send-new">
  ### 如果我一直不发送新消息，会话会自动重置吗
</div>

会的。会话在空闲 `session.idleMinutes` 分钟后过期（默认 **60**）。**下一条**
消息会为该 chat key 启动一个新的会话 ID。此操作不会删除
对话记录——它只会开始一个新会话。

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

<div id="is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents">
  ### 有没有办法把一组 OpenClaw 实例组织成一个“CEO + 多个智能体”的团队
</div>

可以，通过 **多智能体路由** 和 **子智能体** 实现。你可以创建一个协调者
智能体，以及若干各自拥有独立工作区和模型的工作智能体。

不过，需要强调的是，这更适合作为一个**有趣的实验**。它的 token 开销很大，而且
通常不如使用一个机器人配合多个会话高效。我们设想的典型模式是：你只与一个机器人对话，
并为并行任务使用不同的会话。这个机器人也可以在需要时派生子智能体。

文档：[多智能体路由](/zh/concepts/multi-agent)、[子智能体](/zh/tools/subagents)、[Agents CLI](/zh/cli/agents)。

<div id="why-did-context-get-truncated-midtask-how-do-i-prevent-it">
  ### 为什么上下文会在任务中途被截断？我该如何避免？
</div>

会话上下文受模型上下文窗口限制。较长的对话、大量工具输出或过多文件都会触发压缩或截断。

可以尝试：

* 让助手总结当前状态并把摘要写入一个文件。
* 在长任务前使用 `/compact`，在切换话题时使用 `/new`。
* 把重要上下文放到工作区里，并让助手按需读取。
* 对于较长或并行的工作，使用子智能体，这样主会话可以保持更精简。
* 如果这种情况经常发生，选择一个具有更大上下文窗口的模型。

<div id="how-do-i-completely-reset-openclaw-but-keep-it-installed">
  ### 如何在不卸载的前提下彻底重置 OpenClaw
</div>

使用 reset 命令：

```bash
openclaw reset
```

非交互式完全重置：

```bash
openclaw reset --scope full --yes --non-interactive
```

然后重新运行引导向导：

```bash
openclaw onboard --install-daemon
```

注意：

* 如果检测到已有配置，初始化向导也会提供 **Reset**。参见 [Wizard](/zh/start/wizard)。
* 如果你使用了 profile（`--profile` / `OPENCLAW_PROFILE`），请重置每个状态目录（默认是 `~/.openclaw-<profile>`）。
* 开发环境重置：`openclaw gateway --dev --reset`（仅限开发环境；会清除开发配置、凭据、会话和工作区）。

<div id="im-getting-context-too-large-errors-how-do-i-reset-or-compact">
  ### 我遇到“上下文过大”错误，该如何重置或压缩？
</div>

使用以下任一方式：

* **压缩（Compact）**（保留对话但会对较早的轮次做摘要）：
  ```
  /compact
  ```
  或使用 `/compact <instructions>` 来指导摘要方式。

* **重置（Reset）**（为同一个聊天 key 生成新的会话 ID）：
  ```
  /new
  /reset
  ```

如果问题仍然出现：

* 启用或调优 **会话修剪（session pruning）**（`agents.defaults.contextPruning`）以清理旧的工具输出。
* 使用上下文窗口更大的模型。

文档： [压缩（Compaction）](/zh/concepts/compaction)、[会话修剪（Session pruning）](/zh/concepts/session-pruning)、[会话管理（Session management）](/zh/concepts/session)。

<div id="why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required">
  ### 为什么会出现“LLM request rejected messagesNcontentXtooluseinput Field required”这样的消息
</div>

这是一个提供方验证错误：模型输出了一个 `tool_use` 块，但缺少必需的
`input`。这通常意味着会话历史已经过期或损坏（常见于对话很长之后，或在变更了 tool/schema 之后）。

修复方法：使用 `/new` 启动一个新的会话（作为一条单独发送的消息）。

<div id="why-am-i-getting-heartbeat-messages-every-30-minutes">
  ### 为什么我每隔 30 分钟就会收到心跳消息
</div>

心跳默认每 **30 分钟**运行一次。你可以调整其频率或将其禁用：

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // 或 "0m" 以禁用
      }
    }
  }
}
```

如果 `HEARTBEAT.md` 存在但实际上是空的（只有空行和类似 `# Heading` 的 markdown
标题），OpenClaw 会跳过心跳运行以节省 API 调用。
如果该文件不存在，心跳仍然会运行，由模型自行决定要执行的操作。

每个智能体的覆盖配置使用 `agents.list[].heartbeat`。文档参见：[Heartbeat](/zh/gateway/heartbeat)。

<div id="do-i-need-to-add-a-bot-account-to-a-whatsapp-group">
  ### 我需要把一个机器人账号加进 WhatsApp 群吗
</div>

不需要。OpenClaw 基于**你的个人账号**运行，所以只要你在这个群里，OpenClaw 就能看到该群。
默认情况下，在你允许发件人之前（`groupPolicy: "allowlist"`），群内回复会被阻止。

如果你只想让**你自己**能够触发群内回复：

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    }
  }
}
```

<div id="how-do-i-get-the-jid-of-a-whatsapp-group">
  ### 如何获取 WhatsApp 群组的 JID
</div>

选项 1（最快）：使用 tail 命令查看日志，并在群组中发送一条测试消息：

```bash
openclaw logs --follow --json
```

查找以 `@g.us` 结尾的 `chatId`（或 `from`），例如：
`1234567890-1234567890@g.us`。

选项 2（如果已经配置/已在允许列表中）：从配置中列出群组：

```bash
openclaw directory groups list --channel whatsapp
```

相关文档： [WhatsApp](/zh/channels/whatsapp)、[Directory](/zh/cli/directory)、[Logs](/zh/cli/logs)。

<div id="why-doesnt-openclaw-reply-in-a-group">
  ### 为什么 OpenClaw 在群聊里不回复
</div>

常见原因有两个：

* 已启用 mention gating（提及门控，默认设置）。你必须 @提及机器人（或匹配 `mentionPatterns`）。
* 你配置了 `channels.whatsapp.groups`，但没有包含 `"*"`，并且该群聊未在允许列表中。

参见 [Groups](/zh/concepts/groups) 和 [Group messages](/zh/concepts/group-messages)。

<div id="do-groupsthreads-share-context-with-dms">
  ### 群组/话题会与私信共享上下文吗
</div>

默认情况下，直接聊天会折叠到主会话中。群组/频道有各自的会话 key，而 Telegram 话题 / Discord 线程则是独立的会话。参见 [Groups](/zh/concepts/groups) 和 [Group messages](/zh/concepts/group-messages)。

<div id="how-many-workspaces-and-agents-can-i-create">
  ### 我可以创建多少个工作区和智能体
</div>

没有硬性上限。几十个（甚至上百个）都没问题，但需要注意：

* **磁盘占用增长：** 会话和聊天记录位于 `~/.openclaw/agents/<agentId>/sessions/` 下。
* **Token 成本：** 智能体越多，并发模型使用就越多。
* **运维开销：** 每个智能体的认证配置（profile）、工作区和通道路由。

建议：

* 每个智能体只保留一个**活动的**工作区（`agents.defaults.workspace`）。
* 如果磁盘占用增长，清理旧会话（删除 JSONL 或存储条目）。
* 使用 `openclaw doctor` 查找多余工作区和配置档案不匹配问题。

<div id="can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up">
  ### 我可以在 Slack 中同时运行多个机器人或会话吗？应该如何配置？
</div>

可以。使用 **多智能体路由（Multi‑Agent Routing）** 来运行多个相互隔离的智能体，并按
channel/account/peer（频道 / 账号 / 对端）维度路由入站消息。Slack 已作为一个 channel 受支持，并且可以绑定到特定智能体。

浏览器访问能力很强，但并不等于“能做任何人类能做的事”——反机器人机制、CAPTCHA 和 MFA
仍然可以阻止自动化。若要获得最可靠的浏览器控制，请在运行浏览器的那台机器上使用 Chrome 扩展中继，
而 Gateway 可以部署在任意位置。

推荐实践配置：

* 始终在线的 Gateway 主机（VPS / Mac mini）。
* 每个角色对应一个智能体（通过绑定实现）。
* 将 Slack 频道绑定到对应的智能体。
* 需要时通过扩展中继（或一个节点）使用本地浏览器。

文档： [多智能体路由](/zh/concepts/multi-agent)、[Slack](/zh/channels/slack)、
[浏览器](/zh/tools/browser)、[Chrome 扩展](/zh/tools/chrome-extension)、[节点](/zh/nodes)。

<div id="models-defaults-selection-aliases-switching">
  ## 模型：默认设置、选择、别名与切换
</div>

<div id="what-is-the-default-model">
  ### 默认模型是什么
</div>

OpenClaw 的默认模型由你在以下位置所做的设置决定：

```
agents.defaults.model.primary
```

模型以 `provider/model` 的形式引用（示例：`anthropic/claude-opus-4-5`）。如果你省略了提供方，OpenClaw 目前会作为临时的弃用兼容回退而默认使用 `anthropic`，但你仍然应该**显式**设置完整的 `provider/model`。

<div id="what-model-do-you-recommend">
  ### 你推荐使用哪个模型
</div>

**推荐默认值：** `anthropic/claude-opus-4-5`。\
**不错的替代：** `anthropic/claude-sonnet-4-5`。\
**可靠（个性稍弱）：** `openai/gpt-5.2` —— 几乎和 Opus 一样好，只是“个性”弱一点。\
**预算优先：** `zai/glm-4.7`。

MiniMax M2.1 有单独的文档：[MiniMax](/zh/providers/minimax) 和
[本地模型](/zh/gateway/local-models)。

经验法则：对高风险、高价值任务，使用**你负担得起的最强模型**；日常聊天或摘要，用便宜一些的模型即可。你可以按智能体配置路由到不同模型，并使用子智能体来并行处理耗时较长的任务（每个子智能体都会消耗 tokens）。参见 [Models](/zh/concepts/models) 和
[Sub-agents](/zh/tools/subagents)。

强烈警告：较弱或过度量化的模型更容易受到提示注入攻击，并表现出不安全行为。参见 [Security](/zh/gateway/security)。

更多背景：参见 [Models](/zh/concepts/models)。

<div id="can-i-use-selfhosted-models-llamacpp-vllm-ollama">
  ### 我可以使用自托管模型（llamacpp、vLLM、Ollama）吗
</div>

可以。如果你的本地服务器提供 OpenAI 兼容的 api，你可以将一个自定义提供方指向它。Ollama 提供了直接支持，是最简单的方案。

安全提示：较小或重度量化的模型更容易受到提示注入攻击。我们强烈建议，对任何可以使用工具的机器人都使用**大模型**。如果你仍然想使用小模型，请启用沙箱并配置严格的工具允许列表。

文档：[Ollama](/zh/providers/ollama)、[本地模型](/zh/gateway/local-models)、[模型提供方](/zh/concepts/model-providers)、[安全性](/zh/gateway/security)、[沙箱](/zh/gateway/sandboxing)。

<div id="how-do-i-switch-models-without-wiping-my-config">
  ### 如何在不清空配置的情况下切换模型
</div>

使用 **模型相关命令** 或只编辑 **model** 字段。避免整份配置整体替换。

推荐的安全做法：

* 在对话中使用 `/model`（快速，作用于当前会话）
* 使用 `openclaw models set ...`（仅更新模型相关配置）
* 使用 `openclaw configure --section models`（交互式）
* 编辑 `~/.openclaw/openclaw.json` 中的 `agents.defaults.model`

除非你确实打算替换整个配置，否则避免在只传入部分对象时使用 `config.apply`。
如果你已经覆盖了配置，请从备份恢复，或重新运行 `openclaw doctor` 进行修复。

文档：[Models](/zh/concepts/models)、[Configure](/zh/cli/configure)、[Config](/zh/cli/config)、[Doctor](/zh/gateway/doctor)。

<div id="what-do-openclaw-flawd-and-krill-use-for-models">
  ### OpenClaw、Flawd 和 Krill 使用哪些模型？
</div>

* **OpenClaw + Flawd：**Anthropic Opus（`anthropic/claude-opus-4-5`）——请参见 [Anthropic](/zh/providers/anthropic)。
* **Krill：**MiniMax M2.1（`minimax/MiniMax-M2.1`）——请参见 [MiniMax](/zh/providers/minimax)。

<div id="how-do-i-switch-models-on-the-fly-without-restarting">
  ### 如何在无需重启的情况下随时切换模型
</div>

将 `/model` 命令作为一条独立消息发送即可：

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

你可以使用 `/model`、`/model list` 或 `/model status` 列出可用的模型。

`/model`（以及 `/model list`）会显示一个简洁的编号选择器。通过编号进行选择：

```
/model 3
```

你也可以为该提供方强制指定特定的身份验证配置文件（按会话划分）：

```
/model opus@anthropic:default
/model opus@anthropic:work
```

提示：`/model status` 会显示当前激活的是哪个智能体、正在使用哪个 `auth-profiles.json` 文件，以及接下来会尝试哪个认证配置。
如果可用，它还会显示已配置的提供方端点（`baseUrl`）和 API 模式（`api`）。

**如何取消我用 profile 固定的配置**

重新运行 `/model`，**不要**带上 `@profile` 后缀：

```
/model anthropic/claude-opus-4-5
```

如果你想恢复为默认设置，可以从 `/model` 中选择它（或者发送 `/model <默认提供方/模型>`）。
使用 `/model status` 来确认当前激活的是哪个身份验证配置（auth profile）。

<div id="can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding">
  ### 我可以用 GPT 5.2 处理日常任务，用 Codex 5.2 编码吗？
</div>

可以。将其中一个设为默认值，并按需切换：

* **快速切换（按会话）：** 日常任务用 `/model gpt-5.2`，编码用 `/model gpt-5.2-codex`。
* **默认值 + 切换：** 将 `agents.defaults.model.primary` 设为 `openai-codex/gpt-5.2`，然后在编码时切换到 `openai-codex/gpt-5.2-codex`（或反过来）。
* **子智能体：** 将编码任务路由到默认模型不同的子智能体。

参见 [Models](/zh/concepts/models) 和 [Slash commands](/zh/tools/slash-commands)。

<div id="why-do-i-see-model-is-not-allowed-and-then-no-reply">
  ### 为什么会看到 “Model is not allowed”，然后就没有回复
</div>

如果设置了 `agents.defaults.models`，它会被视为 `/model` 和任何会话级覆盖配置的**允许列表**。选择一个不在该列表中的模型会返回：

```
Model "provider/model" is not allowed. Use /model to list available models.
```

系统会返回该错误，**而不是**正常回复。解决方法：将该模型添加到
`agents.defaults.models`，移除允许列表，或者从 `/model list` 中选择一个模型。

<div id="why-do-i-see-unknown-model-minimaxminimaxm21">
  ### 为什么会看到 Unknown model minimaxMiniMaxM21
</div>

这表示**提供方尚未配置**（未找到 MiniMax 提供方配置或认证
profile），因此无法解析该模型。针对这一检测的修复包含在 **2026.1.12** 版本中（撰写时尚未发布）。

排查清单：

1. 升级到 **2026.1.12**（或从源码分支 `main` 运行），然后重启 Gateway。
2. 确认已完成 MiniMax 配置（通过向导或 JSON），或者在环境变量/认证 profile 中
   已存在 MiniMax API key，以便能够注入该提供方配置。
3. 使用**精确匹配**的模型 id（区分大小写）：`minimax/MiniMax-M2.1` 或
   `minimax/MiniMax-M2.1-lightning`。
4. 运行：
   ```bash
   openclaw models list
   ```
   然后从列表中选择（或在聊天中使用 `/model list`）。

参见 [MiniMax](/zh/providers/minimax) 和 [Models](/zh/concepts/models)。

<div id="can-i-use-minimax-as-my-default-and-openai-for-complex-tasks">
  ### 我可以把 MiniMax 设为默认模型，并在复杂任务时使用 OpenAI 吗
</div>

可以。使用 **MiniMax 作为默认模型**，在需要时**按会话**切换模型。
回退机制是用于处理**错误**的，而不是为“困难任务”准备的，因此请使用 `/model` 或单独的智能体。

**方案 A：按会话切换**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" }
      }
    }
  }
}
```

然后：

```
/model gpt
```

**选项 B：独立智能体**

* 智能体 A 默认：MiniMax
* 智能体 B 默认：OpenAI
* 按智能体进行路由，或使用 `/agent` 进行切换

文档：[模型](/zh/concepts/models)、[多智能体路由](/zh/concepts/multi-agent)、[MiniMax](/zh/providers/minimax)、[OpenAI](/zh/providers/openai)。

<div id="are-opus-sonnet-gpt-builtin-shortcuts">
  ### `opus`、`sonnet`、`gpt` 是内置快捷别名吗
</div>

是的。OpenClaw 内置了一些默认简写别名（仅当该模型存在于 `agents.defaults.models` 中时才会生效）：

* `opus` → `anthropic/claude-opus-4-5`
* `sonnet` → `anthropic/claude-sonnet-4-5`
* `gpt` → `openai/gpt-5.2`
* `gpt-mini` → `openai/gpt-5-mini`
* `gemini` → `google/gemini-3-pro-preview`
* `gemini-flash` → `google/gemini-3-flash-preview`

如果你用相同名称设置了自己的别名，以你配置的值为准。

<div id="how-do-i-defineoverride-model-shortcuts-aliases">
  ### 如何定义或覆盖模型快捷别名
</div>

别名来自 `agents.defaults.models.<modelId>.alias`。示例：

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-5" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" }
      }
    }
  }
}
```

这样 `/model sonnet`（或在支持别名时使用 `/<alias>`）就会解析为该模型 ID。

<div id="how-do-i-add-models-from-other-providers-like-openrouter-or-zai">
  ### 如何从其他提供方（例如 OpenRouter 或 ZAI）添加模型
</div>

OpenRouter（按 token 计费；提供多种模型）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} }
    }
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." }
}
```

Z.AI（GLM 模型）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  },
  env: { ZAI_API_KEY: "..." }
}
```

如果你引用了某个提供方/模型，但缺少所需的提供方密钥，你会在运行时遇到身份验证错误（例如：`No API key found for provider "zai"`）。

**在添加新智能体后找不到提供方的 API key**

这通常意味着**新智能体**的认证存储是空的。身份验证信息是按智能体隔离存储的，位于：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

修复方案：

* 运行 `openclaw agents add <id>`，并在向导中配置身份验证。
* 或者将主智能体的 `agentDir` 中的 `auth-profiles.json` 复制到新智能体的 `agentDir` 中。

**不要**在多个智能体之间复用同一个 `agentDir`；这会导致身份验证/会话冲突。

<div id="model-failover-and-all-models-failed">
  ## 模型故障切换与“所有模型全部失败”
</div>

<div id="how-does-failover-work">
  ### 故障切换如何工作
</div>

故障切换分两个阶段进行：

1. 在同一提供方内进行**认证配置轮换（Auth profile rotation）**。
2. **模型回退（Model fallback）**到 `agents.defaults.model.fallbacks` 中的下一个模型。

冷却时间会应用到失败的配置文件（采用指数退避），因此即使某个提供方被限流或出现短暂故障，OpenClaw 也能继续返回响应。

<div id="what-does-this-error-mean">
  ### 这个错误表示什么
</div>

```
No credentials found for profile "anthropic:default"
```

这意味着系统尝试使用认证配置文件 ID `anthropic:default`，但在预期的认证存储中找不到对应的凭据。

<div id="fix-checklist-for-no-credentials-found-for-profile-anthropicdefault">
  ### 针对 “No credentials found for profile anthropicdefault” 的排查清单
</div>

* **确认鉴权配置文件所在位置**（新版路径 vs 旧版路径）
  * 当前路径：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  * 旧版路径：`~/.openclaw/agent/*`（由 `openclaw doctor` 迁移）
* **确认环境变量已被 Gateway 加载**
  * 如果你在 shell 中设置了 `ANTHROPIC_API_KEY`，但通过 systemd/launchd 运行 Gateway，它可能不会继承该变量。请将其写入 `~/.openclaw/.env`，或者启用 `env.shellEnv`。
* **确保你正在编辑的是正确的智能体**
  * 多智能体（multi‑agent）配置意味着可能存在多个 `auth-profiles.json` 文件。
* **基本自检模型/鉴权状态**
  * 使用 `openclaw models status` 查看已配置的模型，以及各提供方是否已完成鉴权。

**针对 “No credentials found for profile anthropic” 的排查清单**

这表示本次运行被绑定到某个 Anthropic 鉴权配置（auth profile），但 Gateway
在其鉴权存储中找不到该配置。

* **使用 setup-token**
  * 运行 `claude setup-token`，然后使用 `openclaw models auth setup-token --provider anthropic` 粘贴该 token。
  * 如果 token 是在另一台机器上创建的，使用 `openclaw models auth paste-token --provider anthropic`。
* **如果你想改为使用 API 密钥**
  * 在**Gateway 主机**上，将 `ANTHROPIC_API_KEY` 写入 `~/.openclaw/.env`。
  * 清除任何强制使用缺失配置的固定顺序：
    ```bash
    openclaw models auth order clear --provider anthropic
    ```
* **确认你是在 Gateway 主机上执行命令**
  * 在远程模式下，鉴权配置保存在 Gateway 所在的机器上，而不是你的笔记本电脑上。

<div id="why-did-it-also-try-google-gemini-and-fail">
  ### 为什么它还会尝试 Google Gemini 并且失败
</div>

如果你的模型配置中将 Google Gemini 设为回退项（或你切换成了 Gemini 的简写），OpenClaw 会在模型回退时尝试使用它。若你尚未配置 Google 凭据，你会看到 `No API key found for provider "google"`。

解决方法：要么提供 Google 认证，要么在 `agents.defaults.model.fallbacks` / aliases 中移除或避免使用 Google 模型，这样回退就不会路由到那里。

**LLM 请求被拒绝，错误信息提示 thinking 签名是 Google Antigravity 所必需的**

原因：会话历史中包含了**没有签名的 thinking 块**（通常来自已中止/部分的流式响应）。Google Antigravity 要求 thinking 块必须带有签名。

解决方法：OpenClaw 现在会为 Google Antigravity Claude 清理掉未签名的 thinking 块。如果仍然出现该问题，请开启一个**新会话**，或为该智能体设置 `/thinking off`。

<div id="auth-profiles-what-they-are-and-how-to-manage-them">
  ## 认证配置文件：是什么以及如何管理
</div>

相关内容：[/concepts/oauth](/zh/concepts/oauth)（OAuth 流程、令牌存储、多账户模式）

<div id="what-is-an-auth-profile">
  ### 什么是 auth profile
</div>

auth profile 是一个命名的凭据记录（OAuth 或 API key），与某个提供方关联。Profiles 存放在：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

<div id="what-are-typical-profile-ids">
  ### 常见的 profile ID 格式
</div>

OpenClaw 使用带有提供方前缀的 ID，例如：

* `anthropic:default`（在没有邮件身份时常见）
* `anthropic:&lt;email&gt;` 用于 OAuth 身份
* 你自定义的 ID（例如 `anthropic:work`）

<div id="can-i-control-which-auth-profile-is-tried-first">
  ### 我可以控制首先尝试哪个认证配置吗
</div>

可以。配置支持为认证配置添加可选元数据，并支持按每个提供方设置顺序（`auth.order.<provider>`）。这并**不会**存储密钥；它只是将 ID 映射到提供方/模式，并设定轮转顺序。

如果某个配置处于短暂的**冷却**状态（限流/超时/认证失败）或更长的**禁用**状态（计费/额度不足），OpenClaw 可能会暂时跳过它。要检查当前状态，运行 `openclaw models status --json` 并查看 `auth.unusableProfiles`。可调参数：`auth.cooldowns.billingBackoffHours*`。

你也可以为**每个智能体**单独设置顺序覆盖（存储在该智能体的 `auth-profiles.json` 中），通过 CLI 完成：

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# 清除覆盖(回退到配置 auth.order / 轮询)
openclaw models auth order clear --provider anthropic
```

要指定某个特定的智能体：

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

<div id="oauth-vs-api-key-whats-the-difference">
  ### OAuth 和 API key 有什么区别？
</div>

OpenClaw 支持两种方式：

* **OAuth** 通常采用订阅制访问（在适用的情况下）。
* **API keys** 采用按 token 计费的方式。

配置向导内置支持 Anthropic setup-token 和 OpenAI Codex 的 OAuth，并且可以帮你保存 API key。

<div id="gateway-ports-already-running-and-remote-mode">
  ## Gateway：端口、“已在运行”状态和远程模式
</div>

<div id="what-port-does-the-gateway-use">
  ### Gateway 使用哪个端口
</div>

`gateway.port` 控制用于 WebSocket 和 HTTP（Control UI、hooks 等）的单一多路复用端口。

优先级：

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

<div id="why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed">
  ### 为什么 `openclaw gateway status` 显示 Runtime running 但 RPC probe failed
</div>

因为 “running” 是**服务管理器**（launchd/systemd/schtasks）看到的状态。而 RPC 探针则是 CLI 实际去连接 Gateway 的 WebSocket 并调用 `status` 的结果。

使用 `openclaw gateway status`，重点看这些行：

* `Probe target:`（探针实际使用的 URL）
* `Listening:`（该端口上实际绑定的内容）
* `Last gateway error:`（当进程存活但端口未监听时常见的根本原因）

<div id="why-does-openclaw-gateway-status-show-config-cli-and-config-service-different">
  ### 为什么 `openclaw gateway status` 会显示 Config cli 和 Config service 不一致
</div>

你正在编辑的配置文件和服务实际正在使用的配置文件不是同一个（通常是 `--profile` / `OPENCLAW_STATE_DIR` 设置不匹配）。

修复方法：

```bash
openclaw gateway install --force
```

在你希望服务使用的同一 `--profile` / 环境下运行该命令。

<div id="what-does-another-gateway-instance-is-already-listening-mean">
  ### “another gateway instance is already listening” 是什么意思
</div>

OpenClaw 在启动时会立即通过绑定 WebSocket 监听端口（默认 `ws://127.0.0.1:18789`）来启用运行时锁。如果绑定时因 `EADDRINUSE` 失败，它会抛出 `GatewayLockError`，表示已经有另一个实例在该端口上监听。

解决方法：停止另一个实例以释放端口，或者使用 `openclaw gateway --port <port>` 在其他端口上运行。

<div id="how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere">
  ### 如何以远程模式运行 OpenClaw（客户端连接到远程 Gateway）
</div>

将 `gateway.mode: "remote"` 设置为远程模式，并指向一个远程 WebSocket URL，可选地附带令牌/密码：

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Notes:

* 只有当 `gateway.mode` 设置为 `local` 时，`openclaw gateway` 才会启动（除非你传入覆盖参数）。
* macOS 应用程序会监视配置文件，并在这些值发生变化时实时切换模式。

<div id="the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now">
  ### Control UI 显示 unauthorized 或不停重连，现在怎么办
</div>

你的 Gateway 在启用鉴权的状态下运行（`gateway.auth.*`），但 UI 没有发送匹配的 token/密码。

代码层面的事实：

* Control UI 会把 token 存在浏览器 localStorage 键 `openclaw.control.settings.v1` 中。
* UI 可以从 `?token=...`（以及/或者 `?password=...`）中导入一次，然后会从 URL 中移除这些参数。

修复方法：

* 最快方式：运行 `openclaw dashboard`（打印并复制带 token 的链接，尝试自动在浏览器中打开；如果是无头环境会给出 SSH 提示）。
* 如果你还没有 token：运行 `openclaw doctor --generate-gateway-token`。
* 如果是远程 Gateway，先建立隧道：`ssh -N -L 18789:127.0.0.1:18789 user@host`，然后打开 `http://127.0.0.1:18789/?token=...`。
* 在 Gateway 主机上设置 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
* 在 Control UI 的设置中粘贴相同的 token（或使用一次性的 `?token=...` 链接刷新）。
* 仍然卡住？运行 `openclaw status --all` 并参考 [Troubleshooting](/zh/gateway/troubleshooting)。鉴权细节见 [Dashboard](/zh/web/dashboard)。

<div id="i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens">
  ### 我把 gatewaybind 设成 tailnet，但它无法绑定，端口上也没有任何监听
</div>

`tailnet` 绑定会从你的网络接口中选择一个 Tailscale IP（100.64.0.0/10）。如果这台机器没有加入 Tailscale（或者对应接口是关闭的），就没有任何地址可以绑定。

解决办法：

* 在该主机上启动 Tailscale（这样它就会有一个 100.x 地址），或者
* 切换到 `gateway.bind: "loopback"` / `"lan"`。

注意：`tailnet` 需要显式配置。`auto` 会优先选择 loopback；当你只想做 tailnet 专用绑定时，使用 `gateway.bind: "tailnet"`。

<div id="can-i-run-multiple-gateways-on-the-same-host">
  ### 我可以在同一台主机上运行多个 Gateway 吗
</div>

一般不需要——单个 Gateway 就可以同时运行多个消息通道和智能体。只有在需要冗余（例如救援 bot）或强隔离时才建议使用多个 Gateway。

可以，但你必须对以下内容做隔离：

* `OPENCLAW_CONFIG_PATH`（每个实例独立的配置）
* `OPENCLAW_STATE_DIR`（每个实例独立的状态）
* `agents.defaults.workspace`（工作区隔离）
* `gateway.port`（唯一端口）

快速设置（推荐）：

* 对每个实例使用 `openclaw --profile <name> …`（会自动创建 `~/.openclaw-<name>`）。
* 在每个 profile 的配置中设置唯一的 `gateway.port`（或在手动运行时传入 `--port`）。
* 为每个 profile 安装独立服务：`openclaw --profile <name> gateway install`。

Profile 也会给服务名添加后缀（`bot.molt.<profile>`；旧格式为 `com.openclaw.*`、`openclaw-gateway-<profile>.service`、`OpenClaw Gateway (<profile>)`）。
完整指南：[多 Gateway 部署](/zh/gateway/multiple-gateways)。

<div id="what-does-invalid-handshake-code-1008-mean">
  ### invalid handshake code 1008 是什么意思
</div>

Gateway 是一个 **WebSocket 服务器**，它期望收到的第一条消息是
一个 `connect` 帧。如果收到的是其他内容，它会关闭连接，并返回
**代码 1008**（策略违规）。

常见原因：

* 你在浏览器中打开了 **HTTP** URL（`http://...`），而不是用 WS 客户端连接。
* 你使用了错误的端口或路径。
* 某个代理或隧道移除了认证头，或者发送的请求并不是给 Gateway 的。

快速修复方法：

1. 使用 WS URL：`ws://<host>:18789`（如果是 HTTPS，则使用 `wss://...`）。
2. 不要在普通浏览器标签页中直接打开 WS 端口。
3. 如果启用了认证，在 `connect` 帧中包含 token/密码。

如果你在使用 CLI 或 TUI，URL 应该类似：

```
openclaw tui --url ws://<host>:18789 --token <token>
```

协议详细说明：[Gateway 协议](/zh/gateway/protocol)。

<div id="logging-and-debugging">
  ## 日志与调试
</div>

<div id="where-are-logs">
  ### 日志存放在哪里
</div>

文件日志（结构化）：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

你可以通过 `logging.file` 设置固定的日志文件路径。文件日志级别由 `logging.level` 控制。控制台日志输出的详细程度由 `--verbose` 和 `logging.consoleLevel` 控制。

查看日志尾部的最快方式：

```bash
openclaw logs --follow
```

服务/守护进程日志（当 Gateway 通过 launchd/systemd 运行时）：

* macOS：`$OPENCLAW_STATE_DIR/logs/gateway.log` 和 `gateway.err.log`（默认：`~/.openclaw/logs/...`；使用 profile 时为 `~/.openclaw-<profile>/logs/...`）
* Linux：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

更多信息请参见 [故障排查](/zh/gateway/troubleshooting#log-locations)。

<div id="how-do-i-startstoprestart-the-gateway-service">
  ### 我该如何启动、停止或重启 Gateway 服务
</div>

使用 Gateway 辅助命令：

```bash
openclaw gateway status
openclaw gateway restart
```

如果你是手动运行 Gateway，可以使用 `openclaw gateway --force` 来重新占用该端口。参见 [Gateway](/zh/gateway)。

<div id="i-closed-my-terminal-on-windows-how-do-i-restart-openclaw">
  ### 在 Windows 上把终端关掉了，如何重新启动 OpenClaw
</div>

Windows 有 **两种安装模式**：

**1) WSL2（推荐）：** Gateway 在 Linux 环境中运行。

打开 PowerShell，进入 WSL，然后重新启动：

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

如果你尚未安装该服务，请以前台方式启动它：

```bash
openclaw gateway run
```

**2) 原生 Windows（不推荐）：** Gateway 直接在 Windows 上运行。

打开 PowerShell，并运行：

```powershell
openclaw gateway status
openclaw gateway restart
```

如果你是手动运行它（未作为服务运行），请使用：

```powershell
openclaw gateway run
```

文档：[Windows（WSL2）](/zh/platforms/windows)、[Gateway 服务运行手册](/zh/gateway)。

<div id="the-gateway-is-up-but-replies-never-arrive-what-should-i-check">
  ### Gateway 已运行但始终收不到回复 我该检查什么？
</div>

先从一次快速健康检查开始：

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

常见原因：

* 模型认证未在 **Gateway 主机** 上加载（检查 `models status`）。
* 频道配对/允许列表阻止了回复（检查频道配置和日志）。
* WebChat/Dashboard 已打开但令牌不正确。

如果你在远程环境，请确认隧道/Tailscale 连接已建立，并且 Gateway WebSocket 可访问。

文档：[Channels](/zh/channels)、[Troubleshooting](/zh/gateway/troubleshooting)、[Remote access](/zh/gateway/remote)。

<div id="disconnected-from-gateway-no-reason-what-now">
  ### 与 Gateway 无故断开连接，接下来怎么办
</div>

这通常意味着 UI 丢失了 WebSocket 连接。请检查：

1. Gateway 是否正在运行？`openclaw gateway status`
2. Gateway 是否处于健康状态？`openclaw status`
3. UI 是否拥有正确的令牌（token）？`openclaw dashboard`
4. 如果是远程连接，隧道/Tailscale 链接是否正常？

然后实时跟踪日志：

```bash
openclaw logs --follow
```

文档：[仪表盘](/zh/web/dashboard)、[远程访问](/zh/gateway/remote)、[故障排除](/zh/gateway/troubleshooting)。

<div id="telegram-setmycommands-fails-with-network-errors-what-should-i-check">
  ### Telegram setMyCommands 因网络错误失败 我应该检查什么？
</div>

先从日志和通道状态入手：

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

如果你在 VPS 上或位于代理之后，确认已允许出站 HTTPS 流量，并且 DNS 解析正常。
如果 Gateway 在远程主机上，确保你正在查看该 Gateway 主机上的日志。

文档：[Telegram](/zh/channels/telegram)，[频道故障排查](/zh/channels/troubleshooting)。

<div id="tui-shows-no-output-what-should-i-check">
  ### 当 TUI 没有任何输出时，我应该检查什么
</div>

首先确认 Gateway 可访问，并且智能体可以运行：

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

在 TUI 中，使用 `/status` 查看当前状态。如果你希望在某个聊天通道中收到回复，请确保已启用消息投递（`/deliver on`）。

文档： [TUI](/zh/tui)、[斜杠命令](/zh/tools/slash-commands)。

<div id="how-do-i-completely-stop-then-start-the-gateway">
  ### 我该如何完全停止并重新启动 Gateway
</div>

如果你是把它安装为系统服务：

```bash
openclaw gateway stop
openclaw gateway start
```

这会停止/启动**受监督的服务**（macOS 上的 launchd，Linux 上的 systemd）。
当 Gateway 作为守护进程在后台运行时，请使用这种方式。

如果你在前台运行，先用 Ctrl‑C 停止，然后：

```bash
openclaw gateway run
```

文档：[Gateway 服务运行手册](/zh/gateway)。

<div id="eli5-openclaw-gateway-restart-vs-openclaw-gateway">
  ### 用小学生也能听懂的 openclaw gateway restart 与 openclaw gateway 区别
</div>

* `openclaw gateway restart`：重启**后台服务**（launchd/systemd）。
* `openclaw gateway`：在当前终端会话中**以前台方式**运行 Gateway。

如果你已经安装了服务，就使用这些 Gateway 命令。当你只想在前台临时运行一次时，使用 `openclaw gateway`。

<div id="whats-the-fastest-way-to-get-more-details-when-something-fails">
  ### 当出现故障时，如何最快获取更多详细信息
</div>

使用 `--verbose` 启动 Gateway，以获得更详细的控制台输出。然后检查日志文件中的通道认证、模型路由和 RPC 错误。

<div id="media-attachments">
  ## 媒体和附件
</div>

<div id="my-skill-generated-an-imagepdf-but-nothing-was-sent">
  ### 我的技能生成了一个 imagePDF，但没有发送出去
</div>

来自智能体的外发附件必须在单独一行包含 `MEDIA:<path-or-url>`。参见 [OpenClaw 助手设置](/zh/start/openclaw) 和 [Agent 发送](/zh/tools/agent-send)。

通过 CLI 发送：

```bash
openclaw message send --target +15555550123 --message "给你" --media /path/to/file.png
```

另请检查：

* 目标通道支持发送媒体内容，且未被允许列表限制。
* 文件在提供方的大小限制范围内（图像会被调整到最大边长 2048 像素）。

参见 [Images](/zh/nodes/images)。

<div id="security-and-access-control">
  ## 安全与访问控制
</div>

<div id="is-it-safe-to-expose-openclaw-to-inbound-dms">
  ### 允许外部私信直接接入 OpenClaw 是否安全
</div>

应将入站私信一律视为不受信任的输入。默认设置旨在降低风险：

* 在支持私信的通道上，默认行为是**配对**：
  * 未知发送者会收到一个配对码；机器人不会处理他们的消息。
  * 使用以下命令批准：`openclaw pairing approve <channel> <code>`
  * 每个通道的待处理请求上限为 **3 个**；如果没收到配对码，请检查 `openclaw pairing list <channel>`。
* 将私信对外公开需要你显式选择加入（`dmPolicy: "open"`，并将 allowlist 设置为 `"*"`，表示允许任何用户发送消息）。

运行 `openclaw doctor` 以发现存在风险的私信策略配置。

<div id="is-prompt-injection-only-a-concern-for-public-bots">
  ### 提示注入只是公共机器人才需要担心的问题吗？
</div>

不是。提示注入的核心在于**不受信任的内容**，而不只是谁可以给机器人发私信（DM）。
如果你的助手会读取外部内容（网页搜索/抓取、浏览器页面、电子邮件、
文档、附件、粘贴的日志），这些内容里就可能包含试图劫持模型的指令。即便**只有你一个发送者**，这种情况也可能发生。

最大风险是在启用工具时：模型可能会被诱骗去窃取上下文，或者代你调用工具。可以通过以下方式减小影响范围：

* 使用只读或禁用工具的「阅读器」智能体来总结不受信任的内容
* 对启用工具的智能体关闭 `web_search` / `web_fetch` / `browser`
* 使用沙箱并配置严格的工具允许列表

详细说明见：[Security](/zh/gateway/security)。

<div id="should-my-bot-have-its-own-email-github-account-or-phone-number">
  ### 我的机器人是否应该有自己独立的邮箱、GitHub 账号或电话号码
</div>

在大多数场景下，答案是是的。通过为机器人使用单独的账号和电话号码进行隔离，
可以在出现问题时减小影响范围。这也让你更容易轮换凭据或撤销访问权限，而不会影响你的个人账号。

从小范围开始。只给当前确实需要的工具和账号授予访问权限，如果有需要，再逐步扩展。

文档：[Security](/zh/gateway/security)、[Pairing](/zh/start/pairing)。

<div id="can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe">
  ### 我可以让它自主管理我的短信吗？这样安全吗？
</div>

我们**不**建议让它对你的个人消息拥有完全自主权。最安全的模式是：

* 将私信保持在**配对模式**，或使用严格的允许列表。
* 如果你希望它代表你发消息，请使用**单独的号码或账号**。
* 让它先起草，然后由你在**发送前确认**。

如果你想做实验，请在一个专用账号上进行，并保持与其他账号隔离。参见
[安全](/zh/gateway/security)。

<div id="can-i-use-cheaper-models-for-personal-assistant-tasks">
  ### 我可以在个人助理任务中使用更便宜的模型吗
</div>

可以，**前提是**该智能体仅用于聊天且输入是可信的。较小档次的模型更容易被指令劫持，因此应避免用于启用工具的智能体，或在处理不受信任内容时使用。如果必须用较小模型，请锁定工具，并在沙箱中运行。参见 [安全](/zh/gateway/security)。

<div id="i-ran-start-in-telegram-but-didnt-get-a-pairing-code">
  ### 我在 Telegram 里发送了 /start，但没有收到配对码
</div>

只有当有未知发送者向 bot 发送消息且启用了 `dmPolicy: "pairing"` 时，才会**发送**配对码。单独发送 `/start` 并不会生成配对码。

检查待处理请求：

```bash
openclaw pairing list telegram
```

如果你想立即获得访问权限，可以将你的发送者 ID 添加到允许列表中，或者为该账户设置 `dmPolicy: "open"`（允许从任何用户不受限制地接收消息）。

<div id="whatsapp-will-it-message-my-contacts-how-does-pairing-work">
  ### WhatsApp 会不会给我的联系人发消息？配对是如何工作的？
</div>

不会。默认的 WhatsApp 私信策略是**配对**。未知发件人只会收到一个配对码，他们的消息**不会被处理**。OpenClaw 只会回复它接收到的聊天，或由你显式触发的发送操作。

使用以下方式批准配对：

```bash
openclaw pairing approve whatsapp <code>
```

列出当前待处理的请求：

```bash
openclaw pairing list whatsapp
```

向导中的电话号码提示：用于设置你的 **allowlist/owner（允许列表/所有者）**，以便你的私信（DM）被允许通过。它不会用于自动发送消息。如果你是在自己的 WhatsApp 个人号上运行，请使用该号码，并启用 `channels.whatsapp.selfChatMode`。

<div id="chat-commands-aborting-tasks-and-it-wont-stop">
  ## 聊天命令、终止任务和“停不下来”的情况
</div>

<div id="how-do-i-stop-internal-system-messages-from-showing-in-chat">
  ### 如何禁止在聊天中显示内部系统消息
</div>

大多数内部或工具消息只有在该会话启用了 **verbose** 或 **reasoning**
时才会显示。

在你看到这些消息的那个聊天中按如下方式处理：

```
/verbose off
/reasoning off
```

如果输出仍然过多，请在 Control UI 中检查会话设置，并将 verbose 设置为 **inherit**。同时确认你没有在配置中使用将 `verboseDefault` 设为 `on` 的机器人配置文件。

文档：[Thinking and verbose](/zh/tools/thinking)，[Security](/zh/gateway/security#reasoning--verbose-output-in-groups)。

<div id="how-do-i-stopcancel-a-running-task">
  ### 如何停止或取消正在运行的任务
</div>

将下列任意一条**单独作为一条消息发送**（前面不要加斜杠）：

```
stop
abort
esc
wait
exit
interrupt
```

这些是中止触发器（不是斜杠命令）。

对于后台进程（来自 `exec` 工具），你可以让智能体执行：

```
process action:kill sessionId:XXX
```

斜杠命令概览：参见 [斜杠命令](/zh/tools/slash-commands)。

大多数命令必须作为以 `/` 开头的**独立**消息发送，但少数快捷命令（如 `/status`）对允许列表中的发送方也支持内联使用。

<div id="how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied">
  ### 我如何从 Telegram 向 Discord 发送消息（Crosscontext messaging denied：跨提供方消息被拒绝）
</div>

OpenClaw 默认会阻止**跨提供方**消息。如果某个工具调用绑定在 Telegram 上，除非你显式允许，否则它不会向 Discord 发送消息。

为该智能体启用跨提供方消息功能：

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[来自 {channel}] " }
          }
        }
      }
    }
  }
}
```

在编辑配置后重启 Gateway。如果你只想让它对单个智能体生效，请改为在 `agents.list[].tools.message` 下进行设置。

<div id="why-does-it-feel-like-the-bot-ignores-rapidfire-messages">
  ### 为什么感觉机器人会忽略你连续快速发的消息
</div>

队列模式控制新消息如何与正在进行的运行交互。使用 `/queue` 来切换模式：

* `steer` - 新消息会重定向当前任务
* `followup` - 按顺序一次只处理一条消息
* `collect` - 将多条消息打包后一次性回复（默认）
* `steer-backlog` - 先根据新消息调整当前方向，然后再处理积压消息
* `interrupt` - 中断当前运行并重新开始

你可以为这类跟进模式添加类似 `debounce:2s cap:25 drop:summarize` 的选项。

<div id="answer-the-exact-question-from-the-screenshotchat-log">
  ## 回答截图 / 聊天记录里的原始问题
</div>

**问：“使用 API key 时，Anthropic 的默认模型是什么？”**

**答：**在 OpenClaw 中，凭据和模型选择是分离的。设置 `ANTHROPIC_API_KEY`（或在 auth profiles 中存储 Anthropic 的 API key）只负责启用身份验证，真正的“默认模型”取决于你在 `agents.defaults.model.primary` 中配置的内容（例如 `anthropic/claude-sonnet-4-5` 或 `anthropic/claude-opus-4-5`）。如果你看到 `No credentials found for profile "anthropic:default"`，这表示 Gateway 在当前运行的智能体对应的 `auth-profiles.json` 中，没有找到 Anthropic 凭据。

***

还是没搞定？可以在 [Discord](https://discord.com/invite/clawd) 提问，或者发起一个 [GitHub discussion](https://github.com/openclaw/openclaw/discussions)。