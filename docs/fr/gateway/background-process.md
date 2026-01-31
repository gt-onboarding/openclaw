---
title: Processus en arrière-plan
summary: "Exécution background exec et gestion des processus"
read_when:
  - Ajout ou modification du comportement de background exec
  - Débogage de tâches exec de longue durée
---

<div id="background-exec-process-tool">
  # Outil exec en arrière‑plan + process
</div>

OpenClaw exécute des commandes shell via l'outil `exec` et maintient en mémoire les tâches de longue durée. L'outil `process` gère ces sessions en arrière‑plan.

<div id="exec-tool">
  ## outil exec
</div>

Paramètres principaux :

- `command` (obligatoire)
- `yieldMs` (par défaut 10000) : passage automatique en arrière‑plan après ce délai
- `background` (bool) : passage immédiat en arrière‑plan
- `timeout` (secondes, par défaut 1800) : tue le processus après ce délai
- `elevated` (bool) : s’exécute sur l’hôte si le mode avec privilèges élevés est activé/autorisé
- Besoin d’un vrai TTY ? Définissez `pty: true`.
- `workdir`, `env`

Comportement :

- Les exécutions au premier plan renvoient directement la sortie.
- Lorsqu’il est relégué en arrière‑plan (explicitement ou par expiration du délai), l’outil renvoie `status: "running"` + `sessionId` et un court extrait de fin de sortie.
- La sortie est conservée en mémoire jusqu’à ce que la session soit interrogée ou effacée.
- Si l’outil `process` est interdit, `exec` s’exécute de manière synchrone et ignore `yieldMs`/`background`.

<div id="child-process-bridging">
  ## Interfaçage avec les processus enfants
</div>

Lorsque vous lancez des processus enfants de longue durée en dehors des outils exec/process (par exemple, des relances de la CLI ou des helpers du Gateway), attachez l’assistant passerelle de sous‑processus afin que les signaux de terminaison soient transmis et que les écouteurs soient détachés à l’arrêt ou en cas d’erreur. Cela évite les processus orphelins sous systemd et maintient un comportement d’arrêt cohérent entre les plates‑formes.

Surcharges d’environnement :

- `PI_BASH_YIELD_MS` : délai d’attente par défaut (ms)
- `PI_BASH_MAX_OUTPUT_CHARS` : limite de sortie en mémoire (caractères)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS` : limite stdout/stderr en attente par flux (caractères)
- `PI_BASH_JOB_TTL_MS` : TTL pour les sessions terminées (ms, borné entre 1 min et 3 h)

Configuration (recommandée) :

- `tools.exec.backgroundMs` (par défaut 10000)
- `tools.exec.timeoutSec` (par défaut 1800)
- `tools.exec.cleanupMs` (par défaut 1800000)
 - `tools.exec.notifyOnExit` (par défaut true) : ajoute un événement système à la file d’attente + demande un signal de vie lorsqu’un exec passé en arrière‑plan se termine.

<div id="process-tool">
  ## outil `process`
</div>

Actions :

- `list` : sessions en cours d’exécution et terminées
- `poll` : récupérer la nouvelle sortie d’une session (et indique aussi le code de sortie)
- `log` : lire la sortie agrégée (prend en charge `offset` + `limit`)
- `write` : envoyer sur stdin (`data`, `eof` facultatif)
- `kill` : terminer une session en arrière-plan
- `clear` : supprimer de la mémoire une session terminée
- `remove` : tuer si en cours d’exécution, sinon effacer si terminée

Remarques :

- Seules les sessions mises en arrière-plan sont affichées et conservées en mémoire.
- Les sessions sont perdues au redémarrage du processus (pas de persistance sur disque).
- Les journaux de session ne sont enregistrés dans l’historique de chat que si vous exécutez `process poll/log` et que le résultat de l’outil est enregistré.
- `process` est limité à chaque agent ; il ne voit que les sessions démarrées par cet agent.
- `process list` inclut un `name` dérivé (verbe de commande + cible) pour des scans rapides.
- `process log` utilise un `offset`/`limit` basé sur les lignes (omettre `offset` pour récupérer les N dernières lignes).

<div id="examples">
  ## Exemples
</div>

Lancez une tâche longue et revenez consulter son état plus tard :

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

Lancer immédiatement en arrière-plan :

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

Envoyer sur stdin :

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
