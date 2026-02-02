---
title: Approvazioni
summary: "Riferimento CLI per `openclaw approvals` (approvazioni di esecuzione per host Gateway o nodo)"
read_when:
  - Vuoi modificare le approvazioni di esecuzione dalla CLI
  - Devi gestire la lista di autorizzati sugli host Gateway o nodo
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

Gestisci le approvazioni exec per l&#39;**host locale**, l&#39;**host Gateway** o l&#39;**host di un nodo**.
Per impostazione predefinita, i comandi agiscono sul file locale delle approvazioni presente su disco. Usa `--gateway` per operare sul Gateway oppure `--node` per operare su uno specifico nodo.

Correlati:

* Exec approvals: [Approvazioni exec](/it/tools/exec-approvals)
* Nodes: [Nodi](/it/nodes)

<div id="common-commands">
  ## Comandi più comuni
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## Sostituire le approvazioni da un file
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## Helper per la lista di autorizzati
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## Note
</div>

* `--node` utilizza lo stesso resolver di `openclaw nodes` (id, nome, ip o prefisso dell&#39;id).
* `--agent` ha come valore predefinito `"*"`, che si applica a tutti gli agenti.
* L&#39;host del nodo deve esporre `system.execApprovals.get/set` (app macOS o host nodo headless).
* I file di approvazione vengono memorizzati per host in `~/.openclaw/exec-approvals.json`.