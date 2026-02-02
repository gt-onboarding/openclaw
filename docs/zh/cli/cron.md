---
title: Cron 定时任务
summary: "`openclaw cron` 的 CLI 参考（调度并运行后台作业）"
read_when:
  - 你需要配置计划任务和唤醒
  - 你正在调试 cron 的执行情况和日志
---

<div id="openclaw-cron">
  # `openclaw cron`
</div>

管理 Gateway 调度器中的 cron 任务。

相关内容：

* Cron 任务：[Cron jobs](/zh/automation/cron-jobs)

提示：运行 `openclaw cron --help` 以查看完整的命令说明。

<div id="common-edits">
  ## 常见编辑
</div>

在不修改消息内容的情况下更新发送设置：

```bash
openclaw cron edit <job-id> --deliver --channel telegram --to "123456789"
```

为单个作业禁用发送：

```bash
openclaw cron edit <job-id> --no-deliver
```
