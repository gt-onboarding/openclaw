---
title: Plugin
summary: "Riferimento CLI per `openclaw plugins` (elenco, installazione, abilitazione/disabilitazione, doctor)"
read_when:
  - Vuoi installare o gestire i plugin Gateway in-process
  - Vuoi eseguire il debug dei problemi di caricamento dei plugin
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

Gestisci i plugin/estensioni del Gateway (caricati all&#39;interno del processo).

Correlati:

* Sistema di plugin: [Plugins](/it/plugin)
* Manifest del plugin + schema: [Plugin manifest](/it/plugins/manifest)
* Hardening della sicurezza: [Security](/it/gateway/security)

<div id="commands">
  ## Comandi
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

I plugin forniti in bundle con OpenClaw sono inizialmente disabilitati. Usa `plugins enable` per
attivarli.

Tutti i plugin devono includere un file `openclaw.plugin.json` con uno Schema JSON inline
(`configSchema`, anche se vuoto). Manifest o schemi mancanti/non validi impediscono
il caricamento del plugin e causano il fallimento della validazione della configurazione.

<div id="install">
  ### Installa
</div>

```bash
openclaw plugins install <path-or-spec>
```

Nota sulla sicurezza: tratta l&#39;installazione dei plugin come se stessi eseguendo codice. Preferisci versioni bloccate (pinned).

Archivi supportati: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Usa `--link` per evitare di copiare una directory locale (aggiunge a `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### Aggiorna
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Gli aggiornamenti si applicano solo ai plugin installati tramite npm (tracciati in `plugins.installs`).
