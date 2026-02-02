---
title: 脚本
summary: "仓库脚本：用途、范围与安全注意事项"
read_when:
  - 从代码仓库运行脚本
  - 在 ./scripts 目录下添加或修改脚本
---

<div id="scripts">
  # 脚本
</div>

`scripts/` 目录包含用于本地工作流和运维任务的辅助脚本。
当某个任务与某个脚本有明确对应关系时再使用这些脚本；否则优先使用 CLI。

<div id="conventions">
  ## 约定
</div>

* 除非在文档或发布检查清单中被引用，否则脚本都是**可选的**。
* 当已有 CLI 界面时优先使用它（例如：认证监控使用 `openclaw models status --check`）。
* 假定脚本是针对特定主机环境编写的；在新机器上运行之前先阅读脚本内容。

<div id="git-hooks">
  ## Git 钩子
</div>

* `scripts/setup-git-hooks.js`：在 Git 仓库中运行时，尽可能为 `core.hooksPath` 进行自动配置。
* `scripts/format-staged.js`：对已暂存的 `src/` 和 `test/` 文件执行预提交格式化。

<div id="auth-monitoring-scripts">
  ## 身份验证监控脚本
</div>

有关身份验证监控脚本的文档请参见：
[/automation/auth-monitoring](/zh/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## 添加脚本时
</div>

* 保持脚本聚焦，并确保有文档记录。
* 在相关文档中添加一条简短说明（如果没有该文档则新建一个）。