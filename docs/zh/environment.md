---
title: 环境
summary: "OpenClaw 从哪些位置加载环境变量以及加载时的优先级顺序"
read_when:
  - 你需要了解会加载哪些环境变量，以及它们被加载的先后顺序
  - 你正在调试 Gateway 中缺失的 API 密钥
  - 你正在撰写有关提供方认证或部署环境的文档
---

<div id="environment-variables">
  # 环境变量
</div>

OpenClaw 会从多个来源读取环境变量。原则是：**绝不覆盖现有值**。

<div id="precedence-highest-lowest">
  ## 优先级（从高到低）
</div>

1. **进程环境**（Gateway 进程从父级 shell/守护进程继承的环境）。
2. **当前工作目录中的 `.env`**（dotenv 默认行为；不会覆盖已有值）。
3. **全局 `.env`**，路径为 `~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`；不会覆盖已有值）。
4. `~/.openclaw/openclaw.json` 中的 **配置文件 `env` 块**（仅在对应变量缺失时生效）。
5. **可选的登录 shell 环境导入**（`env.shellEnv.enabled` 或 `OPENCLAW_LOAD_SHELL_ENV=1`），只对缺失的预期环境变量键生效。

如果配置文件完全不存在，将跳过第 4 步；如果已启用，仍会执行 shell 导入。

<div id="config-env-block">
  ## 配置 `env` 块
</div>

有两种等价方式以内联形式设置环境变量（两者都不会覆盖已有的环境变量值）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## Shell 环境导入
</div>

`env.shellEnv` 会运行你的登录 shell，并且只会导入那些**缺失的**预期环境变量键：

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

对应的环境变量：

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## 配置中的环境变量替换
</div>

你可以在配置的字符串值中直接使用 `${VAR_NAME}` 语法来引用环境变量：

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

有关完整说明，请参见[配置：环境变量替换](/zh/gateway/configuration#env-var-substitution-in-config)。

<div id="related">
  ## 相关内容
</div>

* [Gateway 配置](/zh/gateway/configuration)
* [常见问题：环境变量和 .env 加载](/zh/help/faq#env-vars-and-env-loading)
* [模型概览](/zh/concepts/models)