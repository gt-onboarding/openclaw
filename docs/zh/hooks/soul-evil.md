---
title: SOUL Evil
summary: "SOUL Evil 钩子（将 SOUL.md 替换为 SOUL_EVIL.md）"
read_when:
  - 当你想启用或调整 SOUL Evil 钩子时
  - 当你想要启用清理窗口期或按随机概率进行人格切换时
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

在清理窗口期内，或在随机情况下，SOUL Evil hook 会将**注入的** `SOUL.md` 内容替换为 `SOUL_EVIL.md`。它**不会**修改磁盘上的任何文件。

<div id="how-it-works">
  ## 工作原理
</div>

当执行 `agent:bootstrap` 时，该 hook 可以在系统提示词被组装之前，
在内存中替换 `SOUL.md` 的内容。如果 `SOUL_EVIL.md` 缺失或为空，
OpenClaw 会记录一条警告日志并保留正常的 `SOUL.md`。

子智能体在其 bootstrap 文件中**不会**包含 `SOUL.md`，因此这个 hook
对子智能体没有任何影响。

<div id="enable">
  ## 启用
</div>

```bash
openclaw hooks enable soul-evil
```

然后进行如下配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

在智能体工作区根目录下创建 `SOUL_EVIL.md`（与 `SOUL.md` 同级）。

<div id="options">
  ## 选项
</div>

* `file` (string)：备用 SOUL 文件名（默认：`SOUL_EVIL.md`）
* `chance` (number 0–1)：每次运行使用 `SOUL_EVIL.md` 的随机概率
* `purge.at` (HH:mm)：每日清理开始时间（24 小时制）
* `purge.duration` (duration)：时间窗口长度（例如 `30s`、`10m`、`1h`）

**优先级：**清理时间窗口优先于概率。

**时区：**若已配置，则使用 `agents.defaults.userTimezone`；否则使用主机时区。

<div id="notes">
  ## 注意事项
</div>

* 不会向磁盘写入或修改任何文件。
* 如果 `SOUL.md` 不在 bootstrap 列表中，该 hook 不会执行任何操作。

<div id="see-also">
  ## 参见
</div>

* [Hooks](/zh/hooks)