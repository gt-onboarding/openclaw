---
title: Zalouser
summary: "Plugin Zalo Personal : connexion via QR + messagerie via zca-cli (installation du plugin + configuration du canal + CLI + outil)"
read_when:
  - Vous souhaitez activer la prise en charge (non officielle) de Zalo Personal dans OpenClaw
  - Vous configurez ou développez le plugin zalouser
---

<div id="zalo-personal-plugin">
  # Zalo Personal (plugin)
</div>

Prise en charge de Zalo Personal dans OpenClaw via un plugin, en utilisant `zca-cli` pour automatiser un compte Zalo personnel standard.

> **Avertissement :** une automatisation non officielle peut entraîner la suspension ou le bannissement du compte. À utiliser à vos risques et périls.

<div id="naming">
  ## Convention de nommage
</div>

L'identifiant de canal est `zalouser` afin d'indiquer clairement qu'il automatise un **compte utilisateur Zalo personnel** (non officiel). Nous réservons `zalo` pour une éventuelle future intégration officielle de l'API Zalo.

<div id="where-it-runs">
  ## Où il s’exécute
</div>

Ce plugin s’exécute **à l’intérieur du processus du Gateway**.

Si vous utilisez un Gateway distant, installez/configurez-le sur la **machine sur laquelle s’exécute le Gateway**, puis redémarrez le Gateway.

<div id="install">
  ## Installation
</div>

<div id="option-a-install-from-npm">
  ### Option A : installation via npm
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

Redémarrez le Gateway ensuite.


<div id="option-b-install-from-a-local-folder-dev">
  ### Option B : installation à partir d’un dossier local (dev)
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Redémarrez Gateway ensuite.


<div id="prerequisite-zca-cli">
  ## Prérequis : zca-cli
</div>

La machine hébergeant le Gateway doit avoir `zca` dans son `PATH` :

```bash
zca --version
```


<div id="config">
  ## Config
</div>

La configuration du canal se trouve dans `channels.zalouser` (et non dans `plugins.entries.*`) :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "appairage"
    }
  }
}
```


<div id="cli">
  ## CLI
</div>

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```


<div id="agent-tool">
  ## Outil d’Agent
</div>

Nom de l’outil : `zalouser`

Actions : `send`, `image`, `link`, `friends`, `groups`, `me`, `status`