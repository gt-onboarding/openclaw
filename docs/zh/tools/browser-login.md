---
title: 浏览器登录
summary: "用于浏览器自动化和在 X/Twitter 上发帖的手动登录"
read_when:
  - 你需要登录网站以便进行浏览器自动化
  - 你想在 X/Twitter 上发布更新
---

<div id="browser-login-xtwitter-posting">
  # 浏览器登录 + 在 X/Twitter 上发帖
</div>

<div id="manual-login-recommended">
  ## 手动登录（推荐）
</div>

当网站需要登录时，请在 **宿主** 浏览器配置文件（openclaw 浏览器）中**手动登录**。

**不要**把你的账户凭据交给模型。自动化登录往往会触发反机器人防御机制，并可能导致账户被锁定。

返回浏览器工具主文档：[Browser](/zh/tools/browser)。

<div id="which-chrome-profile-is-used">
  ## 使用的是哪个 Chrome 配置文件？
</div>

OpenClaw 会控制一个**专用的 Chrome 配置文件（Profile）**（名称为 `openclaw`，UI 为橙色主题）。它与你日常使用的浏览器配置文件是分开的、互不影响。

有两种简单的访问方式：

1. **让智能体打开浏览器**，然后你自己登录。
2. **通过 CLI 打开**：

```bash
openclaw browser start
openclaw browser open https://x.com
```

如果你有多个浏览器配置，请传递参数 `--browser-profile &lt;name&gt;`（默认值为 `openclaw`）。

<div id="xtwitter-recommended-flow">
  ## X/Twitter：推荐工作流
</div>

* **阅读/搜索/推文串：** 使用 **bird** CLI 技能（无需浏览器，更稳定）。
  * 仓库：https://github.com/steipete/bird
* **发布更新：** 使用 **host** 浏览器（手动登录）。

<div id="sandboxing-host-browser-access">
  ## 沙箱化与宿主浏览器访问
</div>

沙箱中的浏览器会话**更容易**触发反机器人检测机制。对于 X/Twitter（以及其他风控严格的网站），优先使用**宿主**浏览器。

如果智能体运行在沙箱中，浏览器工具会默认使用该沙箱。要改为由宿主机浏览器控制：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

然后将目标指向宿主浏览器：

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

或者为负责发布更新的智能体禁用沙箱。
