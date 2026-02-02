---
title: Approbations
summary: "Référence CLI pour `openclaw approvals` (approbations d'exécution pour les hôtes Gateway ou les nœuds)"
read_when:
  - Vous souhaitez modifier les approbations d'exécution depuis la CLI
  - Vous devez gérer des listes d’autorisation sur des hôtes exécutant Gateway ou des nœuds
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

Gérer les approbations d&#39;exécution pour l’**hôte local**, l’**hôte Gateway** ou un **hôte de nœud**.
Par défaut, les commandes ciblent le fichier local des approbations sur le disque. Utilisez `--gateway` pour cibler le Gateway, ou `--node` pour cibler un nœud spécifique.

À voir également :

* Approbations d&#39;exécution : [Approbations d&#39;exécution](/fr/tools/exec-approvals)
* Nœuds : [Nœuds](/fr/nodes)

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## Remplacer les autorisations depuis un fichier
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## Outils pour la liste d’autorisation
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## Notes
</div>

* `--node` utilise le même mécanisme de résolution que `openclaw nodes` (ID, nom, IP ou préfixe d’ID).
* `--agent` vaut par défaut `"*"`, ce qui s’applique à tous les agents.
* L’hôte du nœud doit annoncer `system.execApprovals.get/set` (application macOS ou hôte de nœud sans interface graphique).
* Les fichiers d’approbation sont stockés par hôte dans `~/.openclaw/exec-approvals.json`.