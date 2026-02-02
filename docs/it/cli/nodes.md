---
title: Nodi
summary: "Riferimento alla CLI per `openclaw nodes` (list/status/approve/invoke, camera/canvas/screen)"
read_when:
  - Stai gestendo nodi associati (fotocamere, schermi, canvas)
  - Devi approvare le richieste o invocare comandi dei nodi
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

Gestisci i nodi (dispositivi) abbinati e invoca le capacità dei nodi.

Correlato:

* Panoramica sui nodi: [Nodi](/it/nodes)
* Fotocamera: [Nodi fotocamera](/it/nodes/camera)
* Immagini: [Nodi immagine](/it/nodes/images)

Opzioni comuni:

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## Comandi più comuni
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` stampa le tabelle dei nodi in attesa/associati. Le righe dei nodi associati includono il tempo trascorso dall&#39;ultima connessione (Last Connect).
Usa `--connected` per mostrare solo i nodi attualmente connessi. Usa `--last-connected <duration>` per
filtrare i nodi che si sono connessi entro una certa durata (ad es. `24h`, `7d`).

<div id="invoke-run">
  ## Invocare / eseguire
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Flag per l&#39;invocazione:

* `--params <json>`: stringa JSON dell&#39;oggetto (predefinito `{}`).
* `--invoke-timeout <ms>`: timeout di invocazione del nodo (predefinito `15000`).
* `--idempotency-key <key>`: chiave di idempotenza opzionale.

<div id="exec-style-defaults">
  ### Impostazioni predefinite in stile exec
</div>

`nodes run` rispecchia il comportamento exec del modello (valori predefiniti + approvazioni):

* Legge `tools.exec.*` (più gli override `agents.list[].tools.exec.*`).
* Utilizza le approvazioni exec (`exec.approval.request`) prima di invocare `system.run`.
* `--node` può essere omesso quando `tools.exec.node` è impostato.
* Richiede un nodo che esponga `system.run` (app companion macOS o host di nodo headless).

Flag:

* `--cwd <path>`: directory di lavoro.
* `--env <key=val>`: override delle variabili di ambiente (env) (ripetibile).
* `--command-timeout <ms>`: timeout del comando.
* `--invoke-timeout <ms>`: timeout di invocazione del nodo (predefinito `30000`).
* `--needs-screen-recording`: richiedi l&#39;autorizzazione alla registrazione dello schermo.
* `--raw <command>`: esegui una stringa di shell (`/bin/sh -lc` o `cmd.exe /c`).
* `--agent <id>`: approvazioni/lista di autorizzati con ambito agente (predefinito: l&#39;agente configurato).
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: override.