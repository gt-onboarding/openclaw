---
title: Opencode
summary: "在 OpenClaw 中使用 OpenCode Zen（精选模型）"
read_when:
  - 你希望使用 OpenCode Zen 来访问模型
  - 你想要一份适合编程的精选模型列表
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen 是由 OpenCode 团队为编码智能体推荐的**精选模型列表**。
它是一个可选的托管模型访问路径，使用 api key 和 `opencode` 提供方。
Zen 目前处于测试阶段。

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# 或者使用非交互模式
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## 配置示例
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## 注意事项
</div>

- 同样支持 `OPENCODE_ZEN_API_KEY`。
- 你需要登录 Zen，添加账单信息，然后复制你的 API 密钥。
- OpenCode Zen 按请求计费；详细信息请查看 OpenCode 仪表板。