---
title: 配置
summary: "`openclaw config` 的 CLI 参考（获取/设置/清除配置值）"
read_when:
  - 你想以非交互方式读取或编辑配置
---

<div id="openclaw-config">
  # `openclaw config`
</div>

配置助手：按路径获取/设置/清除值。直接运行（不带子命令）将启动配置向导（等同于 `openclaw configure`）。

<div id="examples">
  ## 示例
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## 路径
</div>

路径使用点号或方括号表示法：

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

使用智能体列表索引来指定特定的智能体：

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## 值
</div>

值会在可能的情况下解析为 JSON5；否则会被视为字符串。
使用 `--json` 以强制进行 JSON5 解析。

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

在完成修改后重启 Gateway。
