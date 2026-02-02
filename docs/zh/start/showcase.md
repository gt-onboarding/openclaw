---
title: "案例展示"
description: "来自社区的实际 OpenClaw 项目"
summary: "由 OpenClaw 驱动的社区项目与集成案例"
---

<div id="showcase">
  # 案例展示
</div>

来自社区的真实项目。看看大家在用 OpenClaw 构建什么。

<Info>
**想被收录展示吗？** 在 [Discord 的 #showcase 频道](https://discord.gg/clawd) 分享你的项目，或在 [X 上标记 @openclaw](https://x.com/openclaw)。
</Info>

<div id="openclaw-in-action">
  ## 🎥 OpenClaw 实际演示
</div>

由 VelvetShark 制作的完整安装全流程演示（28 分钟）。

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw：自托管的 AI，Siri 本该如此（完整安装）"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[在 YouTube 上观看](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="OpenClaw 功能展示视频"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[在 YouTube 上观看](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="OpenClaw 社区作品展示"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[在 YouTube 上观看](https://www.youtube.com/watch?v=5kkIJNUGFho)

<div id="fresh-from-discord">
  ## 🆕 Discord 最新动态
</div>

<CardGroup cols={2}>
  <Card title="PR 审查 → Telegram 反馈" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
    **@bangnokia** • `review` `github` `telegram`

    OpenCode 完成更改 → 提交 PR → OpenClaw 审查 diff，并在 Telegram 中回复带有「minor suggestions」的小建议，同时给出明确的合并结论（包括需要优先处理的关键修复项）。

    <img src="/assets/showcase/pr-review-telegram.jpg" alt="OpenClaw 在 Telegram 中发送的 PR 评审反馈" />
  </Card>

  <Card title="几分钟内完成 Wine Cellar 技能" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
    **@prades&#95;maxime** • `skills` `local` `csv`

    向 “Robby” (@openclaw) 请求了一个本地酒窖技能。它会先索要一个示例 CSV 导出文件及其存放位置，然后快速构建并测试该技能（示例中有 962 瓶酒）。

    <img src="/assets/showcase/wine-cellar-skill.jpg" alt="OpenClaw 从 CSV 构建本地酒窖技能" />
  </Card>

  <Card title="Tesco 购物自动驾驶助手" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
    **@marchattonhere** • `automation` `browser` `shopping`

    每周餐单 → 常购商品 → 预订配送时段 → 确认订单。无需 API，只通过浏览器控制。

    <img src="/assets/showcase/tesco-shop.jpg" alt="通过聊天实现 Tesco 购物自动化" />
  </Card>

  <Card title="SNAG 截图转 Markdown" icon="scissors" href="https://github.com/am-will/snag">
    **@am-will** • `devtools` `screenshots` `markdown`

    通过快捷键框选屏幕区域 → 交给 Gemini 视觉 → 立即在剪贴板生成 Markdown。

    <img src="/assets/showcase/snag.png" alt="SNAG 截图转 Markdown 工具" />
  </Card>

  <Card title="代理集合 UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
    **@kitze** • `ui` `skills` `sync`

    用于在 Agent 代理、Claude、Codex 和 OpenClaw 之间统一管理技能和命令的桌面应用。

    <img src="/assets/showcase/agents-ui.jpg" alt="Agent UI 应用" />
  </Card>

  <Card title="Telegram 语音消息（papla.media）" icon="microphone" href="https://papla.media/docs">
    **社区** • `voice` `tts` `telegram`

    封装 papla.media 的 TTS，将结果以 Telegram 语音消息发送（无恼人自动播放）。

    <img src="/assets/showcase/papla-tts.jpg" alt="来自 TTS 的 Telegram 语音消息输出" />
  </Card>

  <Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
    **@odrobnik** • `devtools` `codex` `brew`

    通过 Homebrew 安装的辅助工具，用于列出、检查和监视本地 OpenAI Codex 会话（CLI + VS Code）。

    <img src="/assets/showcase/codexmonitor.png" alt="ClawHub 上的 CodexMonitor" />
  </Card>

  <Card title="Bambu 3D 打印机控制" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
    **@tobiasbischoff** • `hardware` `3d-printing` `skill`

    控制和排查 BambuLab 打印机：状态、任务、摄像头、AMS、校准等。

    <img src="/assets/showcase/bambu-cli.png" alt="Bambu CLI skill on ClawHub" />
  </Card>

  <Card title="维也纳公共交通公司（Wiener Linien）" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
    **@hjanuschka** • `travel` `transport` `skill`

    维也纳公共交通的实时发车信息、运行中断、电梯状态和路线规划。

    <img src="/assets/showcase/wienerlinien.png" alt="ClawHub 上的 Wiener Linien 技能" />
  </Card>

  <Card title="ParentPay 校园餐饮" icon="utensils" href="#">
    **@George5562** • `automation` `browser` `parenting`

    通过 ParentPay 自动预约英国学校餐食。使用鼠标坐标以可靠地点击表格单元格。
  </Card>

  <Card title="R2 Upload（把文件发送给我）" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
    **@julianengel** • `files` `r2` `presigned-urls`

    上传到 Cloudflare R2/S3，并生成安全的预签名下载链接。非常适合远程部署的 OpenClaw 实例。
  </Card>

  <Card title="通过 Telegram 使用的 iOS 应用" icon="mobile" href="#">
    **@coard** • `ios` `xcode` `testflight`

    完全通过 Telegram 聊天构建了一个集成地图和语音录制功能的完整 iOS 应用，并将其部署到 TestFlight。

    <img src="/assets/showcase/ios-testflight.jpg" alt="iOS app on TestFlight" />
  </Card>

  <Card title="Oura Ring 健康助手" icon="heart-pulse" href="#">
    **@AS** • `health` `oura` `calendar`

    个人 AI 健康助手，将 Oura 戒指数据与日历、预约和健身计划集成使用。

    <img src="/assets/showcase/oura-health.png" alt="Oura ring health assistant" />
  </Card>

  <Card title="Kev 的梦幻团队（14+ 个 Agent 代理）" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
    **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

    在单个 Gateway 下运行 14+ 个智能体，由 Opus 4.5 协调器向 Codex worker 进行任务委派。完整的[技术详解](https://github.com/adam91holt/orchestrated-ai-articles)涵盖 Dream Team 阵容、模型选择、沙箱、webhook、心跳以及委派流程。[Clawdspace](https://github.com/adam91holt/clawdspace) 用于智能体沙箱。[博客文章](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/)。
  </Card>

  <Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
    **@NessZerra** • `devtools` `linear` `cli` `issues`

    Linear 的 CLI 工具，可与智能体工作流（Claude Code、OpenClaw）集成。直接在终端管理问题、项目和工作流。首个外部 PR 已合并！
  </Card>

  <Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
    **@jules** • `messaging` `beeper` `cli` `automation`

    通过 Beeper Desktop 执行 Read、发送和归档消息。使用 Beeper 本地 MCP API，使智能体可以在一个地方统一管理你所有的聊天（iMessage、WhatsApp 等）。
  </Card>
</CardGroup>

<div id="automation-workflows">
  ## 🤖 自动化与工作流
</div>

<CardGroup cols={2}>

<Card title="Winix Air Purifier Control" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

  Claude Code 发现并确认了净化器的控制方式，之后由 OpenClaw 接管，自动管理房间空气质量。

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="通过 OpenClaw 控制 Winix 空气净化器" />
</Card>

<Card title="Pretty Sky Camera Shots" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

  由屋顶摄像头触发：让 OpenClaw 在天空好看时自动拍照——它自己设计了一个技能并完成了拍摄。

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="由 OpenClaw 捕获的屋顶摄像头天空快照" />
</Card>

<Card title="Visual Morning Briefing Scene" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

  预设的定时提示词，每天早晨生成一张综合“场景”图片（天气、任务、日期、收藏帖子/名言），由一个 OpenClaw 角色负责。
</Card>

<Card title="Padel Court Booking" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Playtomic 场地可用性检查 + 预订 CLI。再也不会错过空闲球场。
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cli 截图" />
</Card>

<Card title="Accounting Intake" icon="file-invoice-dollar">
  **Community** • `automation` `email` `pdf`
  
  从邮件中收集 PDF，为税务顾问预处理文档。每月记账全程自动运行。
</Card>

<Card title="Couch Potato Dev Mode" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

  一边看 Netflix，一边通过 Telegram 重建了整个个人网站——Notion → Astro，迁移了 18 篇文章，DNS 切到 Cloudflare，全程不用打开笔记本电脑。
</Card>

<Card title="Job Search Agent" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

  搜索职位列表，与简历关键词匹配，并返回带链接的相关机会。用 JSearch API 在 30 分钟内就搭建完成。
</Card>

<Card title="Jira Skill Builder" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

  将 OpenClaw 接入 Jira，然后即时生成了一个全新的技能（在它出现在 ClawHub 之前）。
</Card>

<Card title="Todoist Skill via Telegram" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

  将 Todoist 任务自动化，并让 OpenClaw 直接在 Telegram 对话中生成该技能。
</Card>

<Card title="TradingView Analysis" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

  通过浏览器自动化登录 TradingView，截取图表截图，并按需执行技术分析。无需 API——只用浏览器控制即可。
</Card>

<Card title="Slack Auto-Support" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

  监控公司 Slack 频道，自动做出有用回复，并将通知转发到 Telegram。无需提醒，就自主修复了已部署应用中的线上生产环境 bug。
</Card>

</CardGroup>

<div id="knowledge-memory">
  ## 🧠 知识与记忆
</div>

<CardGroup cols={2}>

<Card title="xuezh 中文学习" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  基于 OpenClaw 的中文学习引擎，支持发音反馈和系统化学习流程。
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh 发音反馈" />
</Card>

<Card title="WhatsApp 记忆金库" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  导入完整的 WhatsApp 导出数据，转录 1k+ 条语音消息，与 git 日志交叉比对，生成带链接的 Markdown 报告。
</Card>

<Card title="Karakeep 语义搜索" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  使用 Qdrant 和 OpenAI/Ollama embeddings，为 Karakeep 书签添加向量搜索功能。
</Card>

<Card title="Inside-Out-2 Memory" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  独立的记忆管理器，将会话文件转化为记忆 → 信念 → 不断演化的自我模型。
</Card>

</CardGroup>

<div id="voice-phone">
  ## 🎙️ 语音与电话
</div>

<CardGroup cols={2}>

<Card title="Clawdia 电话桥" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Vapi 语音助手 ↔ OpenClaw HTTP 桥接。通过你的智能体进行近实时电话通话。
</Card>

<Card title="OpenRouter 转录" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

  通过 OpenRouter（Gemini 等）进行多语言音频转录。可在 ClawHub 上获取。
</Card>

</CardGroup>

<div id="infrastructure-deployment">
  ## 🏗️ 基础设施与部署
</div>

<CardGroup cols={2}>

<Card title="Home Assistant 插件" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  在 Home Assistant OS 上运行的 OpenClaw Gateway，支持 SSH 隧道和持久化状态。
</Card>

<Card title="Home Assistant 技能" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  通过自然语言控制和自动化 Home Assistant 设备。
</Card>

<Card title="Nix 打包" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  开箱即用的 Nix 化 OpenClaw 配置，用于可复现部署。
</Card>

<Card title="CalDAV 日历" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  使用 khal/vdirsyncer 的日历技能，自托管日历集成。
</Card>

</CardGroup>

<div id="home-hardware">
  ## 🏠 家庭与硬件
</div>

<CardGroup cols={2}>

<Card title="GoHome Automation" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  基于 Nix 原生构建的家庭自动化方案，以 OpenClaw 作为接口，并配有精美的 Grafana 仪表盘。
  
  <img src="/assets/showcase/gohome-grafana.png" alt="GoHome Grafana 仪表盘" />
</Card>

<Card title="Roborock Vacuum" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  通过自然语言对话控制你的 Roborock 扫地机器人。
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Roborock 状态" />
</Card>

</CardGroup>

<div id="community-projects">
  ## 🌟 社区项目
</div>

<CardGroup cols={2}>

<Card title="StarSwap Marketplace" icon="star" href="https://star-swap.com/">
  **社区** • `marketplace` `astronomy` `webapp`
  
  全面的天文装备交易平台。基于并围绕 OpenClaw 生态系统构建。
</Card>

</CardGroup>

---

<div id="submit-your-project">
  ## 提交你的项目
</div>

有作品想分享？我们很乐意帮你展示！

<Steps>
  <Step title="分享项目">
    在 [Discord 的 #showcase 频道](https://discord.gg/clawd) 发帖，或 [在 X 上发推 @openclaw](https://x.com/openclaw)
  </Step>
  <Step title="补充详细信息">
    告诉我们它能做什么，附上代码仓库/演示链接，如果有的话再配上一张截图
  </Step>
  <Step title="获取展示机会">
    我们会把表现突出的项目添加到本页
  </Step>
</Steps>