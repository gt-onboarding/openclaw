---
title: 应用补丁
summary: "使用 apply_patch 工具应用多文件补丁"
read_when:
  - 你需要在多个文件间进行结构化的文件编辑
  - 你想记录或调试基于补丁的编辑操作
---

<div id="apply_patch-tool">
  # apply_patch 工具
</div>

使用结构化补丁格式对文件进行修改。非常适合多文件
或多补丁块的编辑场景，此时单次 `edit` 调用往往不够可靠。

该工具接收一个 `input` 字符串，其中包含一个或多个文件操作：

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```


<div id="parameters">
  ## 参数
</div>

- `input`（必填）：补丁的完整内容，包括 `*** Begin Patch` 和 `*** End Patch`。

<div id="notes">
  ## 注意事项
</div>

- 路径相对于工作区根目录进行解析。
- 在 `*** Update File:` 块中使用 `*** Move to:` 来重命名文件。
- `*** End of File` 在需要时用于标记仅在文件末尾插入内容的操作。
- 为实验特性，默认禁用。通过 `tools.exec.applyPatch.enabled` 启用。
- 仅适用于 OpenAI（包括 OpenAI Codex）。可通过 `tools.exec.applyPatch.allowModels` 按模型进行限制。
- 配置仅位于 `tools.exec` 下。

<div id="example">
  ## 示例
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
