---
title: 设备
summary: "`openclaw devices` 的 CLI 参考（设备配对 + 令牌轮换/吊销）"
read_when:
  - 你正在审批设备配对请求
  - 你需要轮换或吊销设备令牌
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

管理设备配对请求和以设备为 scope 的令牌。

<div id="commands">
  ## 命令
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

列出所有待处理的配对请求和已配对设备。

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

批准一个待处理的设备配对请求。

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

拒绝待处理的设备配对请求。

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; [--scope &lt;scope...&gt;]`
</div>

为指定角色轮换设备令牌（可选地更新 scope 设置）。

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

撤销特定角色的设备令牌。

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## 通用选项
</div>

- `--url <url>`: Gateway WebSocket URL（如果已配置，则默认使用 `gateway.remote.url`）。
- `--token <token>`: Gateway 令牌（如有需要）。
- `--password <password>`: Gateway 密码（密码验证）。
- `--timeout <ms>`: RPC 超时时间。
- `--json`: JSON 输出（建议用于脚本）。

<div id="notes">
  ## 注意事项
</div>

- 令牌轮换会生成一个新的敏感令牌。请将其视为机密信息。
- 这些命令需要 `operator.pairing`（或 `operator.admin`）scope。