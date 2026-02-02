---
title: Freigaben
summary: "CLI-Referenz für `openclaw approvals` (Ausführungsfreigaben für Gateway- oder Knoten-Hosts)"
read_when:
  - Du Ausführungsfreigaben in der CLI bearbeiten möchtest
  - Du Allowlists auf Gateway- oder Knoten-Hosts verwalten musst
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

Verwalte Exec-Genehmigungen für den **lokalen Host**, den **Gateway-Host** oder einen **Knoten-Host**.
Standardmäßig wirken Befehle auf die lokale Genehmigungsdatei auf dem Datenträger. Verwende `--gateway`, um das Gateway anzusprechen, oder `--node`, um einen bestimmten Knoten anzusprechen.

Verwandte Themen:

* Exec-Genehmigungen: [Exec-Genehmigungen](/de/tools/exec-approvals)
* Knoten: [Knoten](/de/nodes)

<div id="common-commands">
  ## Häufig verwendete Befehle
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## Genehmigungen aus einer Datei neu setzen
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## Allowlist-Hilfsfunktionen
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## Hinweise
</div>

* `--node` verwendet denselben Resolver wie `openclaw nodes` (ID, Name, IP oder ID-Präfix).
* `--agent` ist standardmäßig auf `"*"` gesetzt und gilt damit für alle Agenten.
* Der Knotenhost muss `system.execApprovals.get/set` bereitstellen (macOS-App oder Headless-Knotenhost).
* Genehmigungsdateien werden pro Host unter `~/.openclaw/exec-approvals.json` gespeichert.