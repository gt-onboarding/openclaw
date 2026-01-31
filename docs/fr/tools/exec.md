---
title: Exec
summary: "Utilisation de l’outil Exec, modes stdin et prise en charge de TTY"
read_when:
  - Utilisation ou modification de l’outil Exec
  - Débogage du comportement de stdin ou de TTY
---

<div id="exec-tool">
  # Outil Exec
</div>

Exécute des commandes shell dans l’espace de travail. Prend en charge l’exécution au premier plan et en arrière-plan via `process`.
Si `process` est interdit, `exec` s’exécute de façon synchrone et ignore `yieldMs`/`background`.
Les sessions en arrière-plan sont propres à chaque agent ; `process` ne voit que les sessions du même agent.

<div id="parameters">
  ## Paramètres
</div>

* `command` (obligatoire)
* `workdir` (par défaut : cwd)
* `env` (remplacements clé/valeur)
* `yieldMs` (par défaut 10000) : passage automatique en arrière-plan après un délai
* `background` (bool) : passer immédiatement en arrière-plan
* `timeout` (secondes, par défaut 1800) : terminer le processus à l’expiration
* `pty` (bool) : exécuter dans un pseudo-terminal lorsqu’il est disponible (CLIs nécessitant un TTY, agents de développement, UIs de terminal)
* `host` (`sandbox | gateway | node`) : emplacement d’exécution
* `security` (`deny | allowlist | full`) : mode d’application pour `gateway`/`node`
* `ask` (`off | on-miss | always`) : demandes d’approbation pour `gateway`/`node`
* `node` (string) : id/nom de nœud pour `host=node`
* `elevated` (bool) : demander le mode avec privilèges élevés (hôte Gateway) ; `security=full` n’est imposé que lorsque `elevated` se résout à `full`

Notes :

* `host` a pour valeur par défaut `sandbox`.
* `elevated` est ignoré lorsque le sandboxing est désactivé (exec s’exécute déjà sur l’hôte).
* Les approbations `gateway`/`node` sont contrôlées par `~/.openclaw/exec-approvals.json`.
* `node` nécessite un nœud appairé (application compagnon ou hôte de nœud headless).
* Si plusieurs nœuds sont disponibles, définissez `exec.node` ou `tools.exec.node` pour en sélectionner un.
* Sur les hôtes non-Windows, exec utilise `SHELL` lorsqu’il est défini ; si `SHELL` vaut `fish`, il préfère `bash` (ou `sh`)
  depuis le `PATH` pour éviter les scripts incompatibles avec fish, puis revient à `SHELL` si aucun des deux n’existe.
* Important : le sandboxing est **désactivé par défaut**. Si le sandboxing est désactivé, `host=sandbox` s’exécute directement sur
  l’hôte Gateway (sans conteneur) et **ne nécessite pas d’approbations**. Pour exiger des approbations, exécutez avec
  `host=gateway` et configurez les approbations exec (ou activez le sandboxing).

<div id="config">
  ## Config
</div>

* `tools.exec.notifyOnExit` (par défaut : true) : lorsqu’il est réglé sur true, les sessions exec exécutées en arrière-plan ajoutent un événement système à la file d’attente et demandent un signal de vie à la fin.
* `tools.exec.approvalRunningNoticeMs` (par défaut : 10000) : émet une seule notification « en cours d’exécution » lorsqu’un exec soumis à approbation tourne plus longtemps que cette durée (0 désactive la fonctionnalité).
* `tools.exec.host` (par défaut : `sandbox`)
* `tools.exec.security` (par défaut : `deny` pour sandbox, `allowlist` pour Gateway + nœud lorsqu’il n’est pas défini)
* `tools.exec.ask` (par défaut : `on-miss`)
* `tools.exec.node` (par défaut : non défini)
* `tools.exec.pathPrepend` : liste de répertoires à ajouter en tête de `PATH` pour les exécutions.
* `tools.exec.safeBins` : binaires sûrs, uniquement via stdin, pouvant s’exécuter sans entrées explicites dans la liste d’autorisation.

Exemple :

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### Gestion de `PATH`
</div>

* `host=gateway` : fusionne le `PATH` de votre shell de connexion dans l&#39;environnement d&#39;exécution (sauf si l&#39;appel exec
  définit déjà `env.PATH`). Le démon lui-même s&#39;exécute toujours avec un `PATH` minimal :
  * macOS : `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux : `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox` : exécute `sh -lc` (shell de connexion) dans le conteneur, donc `/etc/profile` peut réinitialiser `PATH`.
  OpenClaw ajoute `env.PATH` en préfixe après le chargement du profil via une variable d&#39;environnement interne (sans interpolation
  par le shell) ; `tools.exec.pathPrepend` s&#39;applique aussi ici.
* `host=node` : seules les surcharges d&#39;environnement que vous fournissez sont envoyées au nœud. `tools.exec.pathPrepend` ne
  s&#39;applique que si l&#39;appel exec définit déjà `env.PATH`. Les hôtes nœud sans interface graphique acceptent `PATH` uniquement
  lorsqu&#39;il précède le `PATH` de l&#39;hôte nœud (sans remplacement). Les nœuds macOS ignorent complètement les surcharges de `PATH`.

Liaison d&#39;un nœud par agent (utilisez l&#39;index de la liste des agents dans la config) :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI : l’onglet Nœuds inclut un petit panneau « Liaison de nœud Exec » pour configurer ces mêmes paramètres.

<div id="session-overrides-exec">
  ## Substitutions de session (`/exec`)
</div>

Utilisez `/exec` pour définir des valeurs par défaut **spécifiques à la session** pour `host`, `security`, `ask` et `node`.
Envoyez `/exec` sans arguments pour afficher les valeurs actuelles.

Exemple :

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## Modèle d’autorisation
</div>

`/exec` n’est pris en compte que pour les **expéditeurs autorisés** (liste d’autorisation de canal/appairage plus `commands.useAccessGroups`).
Il met **uniquement à jour l’état de la session** et n’écrit aucune configuration. Pour désactiver complètement exec, refusez‑le via la
politique d’outils (`tools.deny: ["exec"]` ou par agent). Les approbations de l’hôte continuent de s’appliquer, sauf si vous définissez explicitement
`security=full` et `ask=off`.

<div id="exec-approvals-companion-app-node-host">
  ## Approbations Exec (application compagnon / hôte du nœud)
</div>

Les agents en sandbox peuvent exiger une approbation pour chaque requête avant que `exec` ne s’exécute sur le Gateway ou l’hôte du nœud.
Voir [Approbations Exec](/fr/tools/exec-approvals) pour la stratégie, la liste d’autorisation et le flux de la UI.

Lorsque des approbations sont requises, l’outil `exec` renvoie immédiatement
`status: "approval-pending"` et un identifiant d’approbation. Une fois l’approbation accordée (ou refusée / arrivée à expiration),
le Gateway émet des événements système (`Exec finished` / `Exec denied`). Si la commande est encore
en cours d’exécution après `tools.exec.approvalRunningNoticeMs`, une seule notification `Exec running` est émise.

<div id="allowlist-safe-bins">
  ## Liste d’autorisation + exécutables sûrs
</div>

L’application de la liste d’autorisation ne concerne **que les chemins binaires résolus** (aucune correspondance sur le nom de fichier de base). Quand
`security=allowlist`, les commandes shell sont automatiquement autorisées uniquement si chaque segment du pipeline est
présent dans la liste d’autorisation ou est un exécutable sûr. L’enchaînement (`;`, `&&`, `||`) et les redirections sont rejetés en
mode liste d’autorisation.

<div id="examples">
  ## Exemples
</div>

Premier plan :

```json
{"tool":"exec","command":"ls -la"}
```

Contexte et sondage :

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Envoyer des touches (type tmux) :

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Soumettre (envoyer uniquement le CR) :

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

Coller (avec crochets par défaut) :

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch (expérimental)
</div>

`apply_patch` est un sous-outil de `exec` qui permet d’effectuer des modifications structurées sur plusieurs fichiers.
Activez-le explicitement :

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

Remarques :

* Uniquement disponible pour les modèles OpenAI/OpenAI Codex.
* La politique des outils reste applicable ; `allow: ["exec"]` autorise implicitement `apply_patch`.
* La configuration se trouve dans `tools.exec.applyPatch`.
