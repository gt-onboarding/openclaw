---
title: Zalouser
summary: "Plugin Zalo Personal: login tramite QR + messaggistica tramite zca-cli (installazione plugin + configurazione canale + CLI + tool)"
read_when:
  - Vuoi il supporto non ufficiale per Zalo Personal in OpenClaw
  - Stai configurando o sviluppando il plugin zalouser
---

<div id="zalo-personal-plugin">
  # Zalo Personal (plugin)
</div>

Supporto a Zalo Personal per OpenClaw tramite un plugin, utilizzando `zca-cli` per automatizzare un normale account utente Zalo.

> **Avvertenza:** l’automazione non ufficiale può portare alla sospensione o al ban dell’account. Usa questa funzione a tuo rischio e pericolo.

<div id="naming">
  ## Denominazione
</div>

L'id del canale è `zalouser` per indicare chiaramente che automatizza un **account utente Zalo personale** (non ufficiale). Manteniamo `zalo` riservato per una potenziale futura integrazione ufficiale con le API di Zalo.

<div id="where-it-runs">
  ## Dove viene eseguito
</div>

Questo plugin viene eseguito **all'interno del processo del Gateway**.

Se utilizzi un Gateway remoto, installa e configura il plugin sulla **macchina che esegue il Gateway**, quindi riavvia il Gateway.

<div id="install">
  ## Installazione
</div>

<div id="option-a-install-from-npm">
  ### Opzione A: installare da npm
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

Quindi riavvia il Gateway.


<div id="option-b-install-from-a-local-folder-dev">
  ### Opzione B: installazione da una cartella locale (dev)
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Quindi riavvia il Gateway.


<div id="prerequisite-zca-cli">
  ## Prerequisito: zca-cli
</div>

La macchina su cui gira il Gateway deve avere `zca` nel `PATH`:

```bash
zca --version
```


<div id="config">
  ## Config
</div>

La configurazione del canale è definita in `channels.zalouser` (non in `plugins.entries.*`):

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
  ## Strumento dell'agente
</div>

Tool name: `zalouser`

Actions: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`