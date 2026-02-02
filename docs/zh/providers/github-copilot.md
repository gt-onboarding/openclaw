---
title: "GitHub Copilot"
summary: "使用设备授权流程通过 OpenClaw 登录 GitHub Copilot"
read_when:
  - 你想将 GitHub Copilot 用作模型提供方
  - 你需要使用 `openclaw models auth login-github-copilot` 设备授权流程
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## 什么是 GitHub Copilot？
</div>

GitHub Copilot 是 GitHub 的 AI 编程助手。它会根据你的 GitHub 账号和订阅计划，提供对 Copilot 模型的访问。OpenClaw 可以通过两种不同方式将 Copilot 用作模型提供方。

<div id="two-ways-to-use-copilot-in-openclaw">
  ## 在 OpenClaw 中使用 Copilot 的两种方式
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) 内置 GitHub Copilot 提供方 (`github-copilot`)
</div>

使用 GitHub 的设备登录流程获取 GitHub 令牌，然后在运行 OpenClaw 时将其兑换为
Copilot API 令牌。由于不需要 VS Code，这是**默认**且最简单的方式。

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Copilot Proxy 插件 (`copilot-proxy`)
</div>

使用 **Copilot Proxy** VS Code 扩展程序作为本地桥梁。OpenClaw 会与
该代理的 `/v1` 端点通信，并使用你在其中配置的模型列表。如果你已经在 VS Code 中运行 Copilot Proxy，或需要通过它进行路由，请选择此方式。
你必须启用该插件，并保持 VS Code 扩展程序处于运行状态。

将 GitHub Copilot 用作模型提供方（`github-copilot`）。`login` 命令会运行
GitHub 设备授权流程，保存一个授权配置文件，并更新你的配置以使用该
配置文件。

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw models auth login-github-copilot
```

系统会提示你访问一个 URL 并输入一次性验证码。在流程完成之前，请保持终端窗口打开。


<div id="optional-flags">
  ### 可选参数
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## 设置默认模型
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### 配置示例代码
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## 注意事项
</div>

- 需要交互式 TTY；请直接在终端中运行。
- Copilot 模型的可用性取决于你的订阅计划；如果某个模型不可用，尝试
  另一个 ID（例如 `github-copilot/gpt-4.1`）。
- 登录会在身份验证配置存储中保存一个 GitHub 令牌，并在 OpenClaw 运行时
  将其换取 Copilot API 令牌。