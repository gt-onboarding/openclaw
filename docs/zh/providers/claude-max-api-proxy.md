---
title: Claude Max API 代理
summary: "将 Claude Max/Pro 订阅用作兼容 OpenAI 的 API 端点"
read_when:
  - 你想在兼容 OpenAI 的工具中使用 Claude Max 订阅
  - 你想要一个封装 Claude Code CLI 的本地 API 服务器
  - 你想通过使用订阅而不是 API 密钥来节省费用
---

<div id="claude-max-api-proxy">
  # Claude Max API 代理
</div>

**claude-max-api-proxy** 是一个社区工具，它将你的 Claude Max/Pro 订阅提供为一个兼容 OpenAI 的 API 端点。这样你就可以在任何支持 OpenAI API 格式的工具中使用你的订阅。

<div id="why-use-this">
  ## 为什么要使用这个？
</div>

| 方式 | 费用 | 最适用场景 |
|----------|------|----------|
| Anthropic API | 按 token 计费（Opus 约为输入 $15/百万、输出 $75/百万） | 生产应用、大规模流量场景 |
| Claude Max 订阅 | 每月固定 $200 | 个人使用、开发、不限用量 |

如果你拥有 Claude Max 订阅，并希望将其配合 OpenAI 兼容工具一起使用，那么本代理可以为你节省可观的成本。

<div id="how-it-works">
  ## 工作原理
</div>

```
你的应用 → claude-max-api-proxy → Claude Code CLI → Anthropic (通过订阅)
     (OpenAI 格式)              (转换格式)      (使用你的登录信息)
```

该代理服务：

1. 在 `http://localhost:3456/v1/chat/completions` 接收 OpenAI 格式的请求
2. 将其转换为 Claude Code CLI 命令
3. 以 OpenAI 格式返回响应（支持流式响应）

<div id="installation">
  ## 安装
</div>

```bash
# 需要 Node.js 20+ 和 Claude Code CLI
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

<div id="usage">
  ## 使用方法
</div>

<div id="start-the-server">
  ### 启动服务端
</div>

```bash
claude-max-api
# 服务器运行于 http://localhost:3456
```

<div id="test-it">
  ### 进行测试
</div>

```bash
# Health check
curl http://localhost:3456/health

# List models
curl http://localhost:3456/v1/models

# 聊天完成
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

<div id="with-openclaw">
  ### 搭配 OpenClaw 使用
</div>

你可以将 OpenClaw 配置为指向该代理，把它当作自定义的 OpenAI 兼容端点来使用：

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed", // 无需实际密钥
    OPENAI_BASE_URL: "http://localhost:3456/v1"
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" }
    }
  }
}
```

<div id="available-models">
  ## 可用模型
</div>

| Model ID | 对应模型 |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## 在 macOS 上自动启动
</div>

创建一个 LaunchAgent，使代理服务自动运行：

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

<div id="links">
  ## 链接
</div>

* **npm：** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub：** https://github.com/atalovesyou/claude-max-api-proxy
* **问题（Issues）：** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## 注意事项
</div>

* 这是一个**社区开发的工具**，不受 Anthropic 或 OpenClaw 官方背书或支持
* 需要有效的 Claude Max/Pro 订阅，并完成 Claude Code CLI 的身份验证
* 代理在本地运行，不会将数据发送到任何第三方服务器
* 完整支持流式响应

<div id="see-also">
  ## 另请参阅
</div>

* [Anthropic 提供方](/zh/providers/anthropic) - 使用 setup-token 或 API key 的 OpenClaw 原生 Claude 集成
* [OpenAI 提供方](/zh/providers/openai) - 用于 OpenAI/Codex 订阅