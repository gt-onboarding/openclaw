---
title: Configurazione
summary: "Riferimento CLI per `openclaw config` (visualizzare/impostare/eliminare valori di configurazione)"
read_when:
  - Vuoi leggere o modificare la configurazione in modo non interattivo
---

<div id="openclaw-config">
  # `openclaw config`
</div>

Utility di configurazione: ottieni/imposta/rimuovi valori per path. Esegui il comando senza un sottocomando per aprire la procedura guidata di configurazione (equivalente a `openclaw configure`).

<div id="examples">
  ## Esempi
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## Percorsi
</div>

I percorsi utilizzano la notazione con punto o con parentesi quadre:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Usa l&#39;indice dell&#39;elenco degli agenti per indirizzare uno specifico agente:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## Valori
</div>

I valori vengono interpretati come JSON5 quando possibile; in caso contrario, vengono trattati come stringhe.
Usa `--json` per forzare l&#39;interpretazione come JSON5.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

Riavvia il Gateway dopo aver apportato le modifiche.
