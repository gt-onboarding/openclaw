---
title: 发布流程
summary: "针对 npm 与 macOS 应用的逐步发布检查清单"
read_when:
  - 准备发布新的 npm 版本
  - 准备发布新的 macOS 应用版本
  - 在发布前核对元数据
---

<div id="release-checklist-npm-macos">
  # 发布检查清单（npm + macOS）
</div>

在仓库根目录使用 `pnpm`（Node 22+）。在打标签/发布之前保持工作区干净。

<div id="operator-trigger">
  ## 运维触发操作
</div>

当运维人员说 “release” 时，立刻执行以下预检步骤（除非被阻塞，否则不要额外提问）：

* 阅读本文档和 `docs/platforms/mac/release.md`。
* 从 `~/.profile` 加载环境变量，并确认已设置 `SPARKLE_PRIVATE_KEY_FILE` 和 App Store Connect 相关变量（`SPARKLE_PRIVATE_KEY_FILE` 应该保存在 `~/.profile` 中）。
* 如有需要，使用 `~/Library/CloudStorage/Dropbox/Backup/Sparkle` 中的 Sparkle 密钥。

1. **版本与元数据**

* [ ] 更新 `package.json` 版本号（例如：`2026.1.29`）。
* [ ] 运行 `pnpm plugins:sync` 以对齐扩展包版本和变更日志。
* [ ] 更新 CLI/版本字符串：[`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) 以及 [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts) 中的 Baileys user agent。
* [ ] 确认包元数据（name、description、repository、keywords、license）正确，并且 `bin` 映射中 `openclaw` 指向 [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)。
* [ ] 如果依赖有变更，运行 `pnpm install`，确保 `pnpm-lock.yaml` 已更新。

2. **构建与制品**

* [ ] 如果 A2UI 输入有改动，运行 `pnpm canvas:a2ui:bundle` 并提交更新后的 [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js)（如有）。
* [ ] 运行 `pnpm run build`（重新生成 `dist/`）。
* [ ] 检查 npm 包的 `files` 字段是否包含所有必需的 `dist/*` 目录（特别是用于无头节点和 ACP CLI 的 `dist/node-host/**` 与 `dist/acp/**`）。
* [ ] 确认存在 `dist/build-info.json`，并包含预期的 `commit` 哈希（CLI 横幅在 npm 安装时会使用它）。
* [ ] 可选：构建后运行 `npm pack --pack-destination /tmp`；检查生成的 tarball 内容，并在 GitHub 发布时备用（**不要**提交该文件）。

3. **变更日志与文档**

* [ ] 更新 `CHANGELOG.md`，加入面向用户的亮点（如果缺失则创建该文件）；确保条目按版本严格降序排列。
* [ ] 确认 README 中的示例/参数与当前 CLI 行为一致（特别是新增的命令或选项）。

4. **验证**

* [ ] `pnpm lint`
* [ ] `pnpm test`（如需要覆盖率输出可用 `pnpm test:coverage`）
* [ ] `pnpm run build`（测试结束后的最后一次健全性检查）
* [ ] `pnpm release:check`（验证 npm 打包内容）
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke`（Docker 安装冒烟测试，快速路径；发布前必须执行）
  * 如果已知上一版 npm 发布是损坏的，为预安装步骤设置 `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` 或 `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1`。
* [ ] （可选）完整安装器冒烟测试（增加非 root 场景 + CLI 覆盖）：`pnpm test:install:smoke`
* [ ] （可选）安装器端到端测试（Docker，运行 `curl -fsSL https://openclaw.bot/install.sh | bash`，完成引导，然后执行真实工具调用）：
  * `pnpm test:install:e2e:openai`（需要 `OPENAI_API_KEY`）
  * `pnpm test:install:e2e:anthropic`（需要 `ANTHROPIC_API_KEY`）
  * `pnpm test:install:e2e`（需要两个密钥；同时运行两个提供方）
* [ ] （可选）如果你的改动影响发送/接收路径，抽查一次 Web Gateway。

5. **macOS 应用（Sparkle）**

* [ ] 构建并签名 macOS 应用，然后将其压缩为 ZIP 包以便分发。
* [ ] 生成 Sparkle appcast（通过 [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) 生成 HTML 更新说明），并更新 `appcast.xml`。
* [ ] 保留应用 ZIP（以及可选的 dSYM ZIP），以便稍后附加到 GitHub release。
* [ ] 按照 [macOS release](/zh/platforms/mac/release) 中的说明执行，使用完全一致的命令和所需的环境变量。
  * `APP_BUILD` 必须是数值且单调递增（不能包含 `-beta`），这样 Sparkle 才能正确比较版本。
  * 如果需要进行公证，请使用由 App Store Connect API 环境变量创建的 `openclaw-notary` 钥匙串配置文件（参见 [macOS release](/zh/platforms/mac/release)）。

6. **发布（npm）**

* [ ] 确认 git 工作区干净；按需提交并推送。
* [ ] 如有需要，执行 `npm login`（验证 2FA）。
* [ ] 执行 `npm publish --access public`（预发布版本使用 `--tag beta`）。
* [ ] 在 npm 注册表中验证：`npm view openclaw version`、`npm view openclaw dist-tags`，以及 `npx -y openclaw@X.Y.Z --version`（或 `--help`）。

<div id="troubleshooting-notes-from-200-beta2-release">
  ### 疑难排查（2.0.0-beta2 发布记录）
</div>

* **`npm pack/publish` 卡住或生成超大 tarball**：`dist/OpenClaw.app` 中的 macOS 应用包（以及用于发布的 zip 包）被一起打进了 npm 包。通过在 `package.json` 的 `files` 字段中显式列出要发布的内容来修复（只包含 dist 子目录、文档、技能；排除 app bundle）。使用 `npm pack --dry-run` 确认 `dist/OpenClaw.app` 没有出现在列表中。
* **用于 dist-tags 的 npm 认证在网页中反复循环**：使用 legacy 认证以获得 OTP 提示：
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **`npx` 验证失败，错误为 `ECOMPROMISED: Lock compromised`**：使用全新缓存重试：
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **在最后补丁之后需要重新指向 tag**：强制更新并推送该 tag，然后确保 GitHub release 资源仍然匹配：
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub 发布 + appcast**

* [ ] 打 tag 并推送：`git tag vX.Y.Z && git push origin vX.Y.Z`（或 `git push --tags`）。
* [ ] 为 `vX.Y.Z` 创建/刷新 GitHub release，**标题使用 `openclaw X.Y.Z`**（而不仅仅是 tag）；正文应包含该版本的**完整**变更日志部分（Highlights + Changes + Fixes），以内联文本方式给出（不要只放裸链接），并且**正文中不得重复标题**。
* [ ] 附加制品：`npm pack` 生成的 tarball（可选）、`OpenClaw-X.Y.Z.zip` 和 `OpenClaw-X.Y.Z.dSYM.zip`（如果有生成）。
* [ ] 提交更新后的 `appcast.xml` 并推送（Sparkle 会从 main 分支拉取 feed）。
* [ ] 在一个干净的临时目录中（没有 `package.json`），运行 `npx -y openclaw@X.Y.Z send --help`，以确认安装/CLI 入口命令可以正常工作。
* [ ] 公布/分享发布说明。

<div id="plugin-publish-scope-npm">
  ## 插件发布 scope（npm）
</div>

我们只会在 `@openclaw/*` 这个 npm scope 下发布**已经存在于 npm 上的插件**。未在 npm
上的捆绑插件保持为**仅存在于磁盘目录中**（仍然随发行版一起打包在
`extensions/**` 中）。

生成该列表的流程：

1. 运行 `npm search @openclaw --json` 并获取所有包名。
2. 将其与 `extensions/*/package.json` 中的名称进行对比。
3. 只发布两者的**交集**（已经在 npm 上的）。

当前 npm 插件列表（按需更新）：

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

发布说明中还必须明确指出**新的可选捆绑插件**，这些插件**默认不启用**
（例如：`tlon`）。