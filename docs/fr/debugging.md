---
title: Débogage
summary: "Outils de débogage : mode watch, flux bruts du modèle et traçage des fuites de raisonnement"
read_when:
  - Vous devez inspecter la sortie brute du modèle pour détecter des fuites de raisonnement
  - Vous voulez exécuter Gateway en mode watch pendant vos itérations
  - Vous avez besoin d’un processus de débogage reproductible
---

<div id="debugging">
  # Débogage
</div>

Cette page présente les outils de débogage pour la sortie en streaming, en particulier lorsqu'un fournisseur mêle des éléments de raisonnement au texte normal.

<div id="runtime-debug-overrides">
  ## Forçages de débogage à l&#39;exécution
</div>

Utilisez `/debug` dans le chat pour définir des forçages de configuration **valables uniquement à l&#39;exécution** (en mémoire, pas sur disque).
`/debug` est désactivé par défaut ; activez-le avec `commands.debug: true`.
C&#39;est pratique lorsque vous devez basculer des réglages obscurs sans modifier `openclaw.json`.

Exemples :

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` efface toutes les surcharges et revient à la configuration stockée sur le disque.


<div id="gateway-watch-mode">
  ## Mode watch du Gateway
</div>

Pour itérer rapidement, exécute le Gateway avec le watcher de fichiers :

```bash
pnpm gateway:watch --force
```

Cela correspond à :

```bash
tsx watch src/entry.ts gateway --force
```

Ajoutez toutes les options CLI du Gateway après `gateway:watch` ; elles seront transmises à chaque redémarrage.


<div id="dev-profile-dev-gateway-dev">
  ## Profil de dev + Gateway de dev (--dev)
</div>

Utilisez le profil de dev pour isoler l’état et lancer rapidement une configuration sans risque et jetable pour
le débogage. Il existe **deux** options `--dev` :

* **`--dev` global (profil) :** isole l’état sous `~/.openclaw-dev` et
  définit par défaut le port du Gateway à `19001` (les ports dérivés sont ajustés en conséquence).
* **`gateway --dev` : indique au Gateway de créer automatiquement une configuration par défaut +
  espace de travail** lorsqu’ils sont absents (et d’ignorer BOOTSTRAP.md).

Flux recommandé (profil de dev + bootstrap de dev) :

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Si vous n’avez pas encore d’installation globale, exécutez la CLI via `pnpm openclaw ...`.

Ce que cela fait :

1. **Isolation du profil** (global `--dev`)
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001` (le navigateur et le canvas s’ajustent en conséquence)

2. **Bootstrap de développement** (`gateway --dev`)
   * Écrit une configuration minimale si elle est absente (`gateway.mode=local`, liaison sur l’interface loopback).
   * Définit `agent.workspace` sur l’espace de travail de développement.
   * Définit `agent.skipBootstrap=true` (pas de BOOTSTRAP.md).
   * Initialise les fichiers de l’espace de travail s’ils sont manquants :
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   * Identité par défaut : **C3‑PO** (droïde de protocole).
   * Ignore les fournisseurs de canaux en mode développement (`OPENCLAW_SKIP_CHANNELS=1`).

Flux de réinitialisation (repartir de zéro) :

```bash
pnpm gateway:dev:reset
```

Remarque : `--dev` est un indicateur de profil **global** et est consommé par certains runners.
Si vous devez le spécifier explicitement, utilisez la forme sous variable d&#39;environnement :

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` efface la configuration, les identifiants, les sessions et l’espace de travail de dev (en utilisant
`trash`, pas `rm`), puis recrée la configuration de dev par défaut.

Conseil : si un Gateway non dev est déjà en cours d’exécution (launchd/systemd), arrête‑le d’abord :

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## Journalisation du flux brut (OpenClaw)
</div>

OpenClaw peut journaliser le **flux brut de l’assistant** avant tout filtrage/formatage.
C’est le meilleur moyen de voir si le raisonnement arrive sous forme de deltas de texte brut
(ou en blocs de réflexion séparés).

Activez-la via la CLI :

```bash
pnpm gateway:watch --force --raw-stream
```

Redéfinition facultative du chemin :

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variables d’environnement équivalentes :

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Fichier par défaut :

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## Journalisation des chunks bruts (pi-mono)
</div>

Pour capturer les **raw OpenAI-compat chunks** avant qu&#39;ils ne soient analysés en blocs,
pi-mono fournit un logger distinct :

```bash
PI_RAW_STREAM=1
```

Chemin facultatif :

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Fichier par défaut :

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Remarque : ceci n&#39;est généré que par les processus qui utilisent le fournisseur
> `openai-completions`.


<div id="safety-notes">
  ## Notes de sécurité
</div>

- Les journaux de flux bruts peuvent inclure les prompts complets, les sorties d’outils et des données d’utilisateurs.
- Conservez les journaux en local et supprimez-les après le débogage.
- Si vous partagez des journaux, supprimez d’abord tous les secrets et les données personnelles identifiables (PII).