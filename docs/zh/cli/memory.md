---
title: 记忆
summary: "`openclaw memory`（状态/索引/搜索）的 CLI 参考"
read_when:
  - 你需要对语义记忆进行索引或搜索
  - 你在调试记忆可用性或索引行为
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

管理语义记忆的索引和搜索。
由当前启用的内存插件提供（默认：`memory-core`；将 `plugins.slots.memory` 设为 `"none"` 可禁用）。

相关：

* 记忆概念：[Memory](/zh/concepts/memory)
* 插件：[Plugins](/zh/plugins)

<div id="examples">
  ## 示例
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## 选项
</div>

通用选项：

* `--agent <id>`：限定为单个智能体（默认：所有已配置的智能体）。
* `--verbose`：在探测和索引期间输出详细日志。

说明：

* `memory status --deep` 探测向量和 embedding 的可用性。
* `memory status --deep --index` 如果存储为脏状态则执行重新索引。
* `memory index --verbose` 打印各阶段的详细信息（提供方、模型、来源、批处理活动）。
* `memory status` 会包含通过 `memorySearch.extraPaths` 配置的任何额外路径。