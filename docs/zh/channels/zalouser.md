---
title: Zalo 个人账号
summary: "通过 zca-cli（扫码登录）使用 Zalo 个人账号的支持、功能与配置"
read_when:
  - 为 OpenClaw 设置 Zalo 个人账号
  - 调试 Zalo 个人账号的登录或消息流
---

<div id="zalo-personal-unofficial">
  # Zalo 个人账号（非官方）
</div>

状态：实验性。此集成通过 `zca-cli` 对**个人 Zalo 账号**进行自动化控制。

> **警告：**这是非官方集成，可能导致账号被暂停或封禁。使用风险自负。

<div id="plugin-required">
  ## 所需插件
</div>

Zalo Personal 以插件形式提供，不包含在核心安装中。

* 通过 CLI 安装：`openclaw plugins install @openclaw/zalouser`
* 或从源码检出目录安装：`openclaw plugins install ./extensions/zalouser`
* 详情参见：[插件](/zh/plugin)

<div id="prerequisite-zca-cli">
  ## 先决条件：zca-cli
</div>

Gateway 机器的 `PATH` 中必须能找到 `zca` 可执行文件。

* 检查：`zca --version`
* 如果没有，请安装 zca-cli（参见 `extensions/zalouser/README.md` 或上游 zca-cli 文档）。

<div id="quick-setup-beginner">
  ## 快速设置（初学者）
</div>

1. 安装插件（见上文）。
2. 登录（在运行 Gateway 的机器上，通过二维码）：
   * `openclaw channels login --channel zalouser`
   * 使用 Zalo 手机应用扫描终端中的二维码完成登录。
3. 启用该频道：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```

4. 重启 Gateway（或完成初始设置）。
5. 私信访问默认走配对流程；首次联系时请批准配对代码。

<div id="what-it-is">
  ## 是什么
</div>

* 使用 `zca listen` 接收传入消息。
* 使用 `zca msg ...` 发送回复（文本/媒体/链接）。
* 面向 Zalo Bot API 不可用时的“个人账号”场景设计。

<div id="naming">
  ## 命名
</div>

Channel id 为 `zalouser`，以明确表明该通道是用来自动化**个人 Zalo 用户账号**（非官方）的。我们保留 `zalo`，以便在将来可能集成官方 Zalo API 时使用。

<div id="finding-ids-directory">
  ## 查找 ID（目录）
</div>

使用 directory CLI 发现联系人/群组及其 ID：

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## 限制
</div>

* 出站文本会被切分为约 2000 个字符（受 Zalo 客户端限制）。
* 默认情况下禁用流式传输。

<div id="access-control-dms">
  ## 访问控制（私信）
</div>

`channels.zalouser.dmPolicy` 支持以下取值：`pairing | allowlist | open | disabled`（其中 `open` 表示允许从任何用户不受限制地接收消息；默认值：`pairing`）。
`channels.zalouser.allowFrom` 接受用户 ID 或用户名。向导在可用时会通过 `zca friend find` 将名称解析为 ID。

通过以下命令批准配对：

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## 群组访问（可选）
</div>

* 默认值：`channels.zalouser.groupPolicy = "open"`（`open` 表示允许来自任何用户的群组消息，不做限制）。使用 `channels.defaults.groupPolicy` 在未显式设置时覆盖默认值。
* 限制为允许列表：
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups`（键为群组 ID 或名称）
* 阻止所有群组：`channels.zalouser.groupPolicy = "disabled"`。
* 配置向导会提示你配置群组允许列表。
* 启动时，OpenClaw 会将允许列表中的群组/用户名称解析为 ID 并记录映射；无法解析的条目会按原样保留。

示例：

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## 多账号
</div>

账号会映射到 zca 的 profile。示例：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## 故障排查
</div>

**找不到 `zca`:**

* 安装 `zca-cli`，并确保运行 Gateway 进程的 `PATH` 环境变量中包含它。

**登录状态无法保持：**

* `openclaw channels status --probe`
* 重新登录：`openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`