---
title: OpenClaw
summary: "Guide de bout en bout pour utiliser OpenClaw comme assistant personnel, avec mises en garde de sécurité"
read_when:
  - Mise en service d'une nouvelle instance d'assistant
  - Revue des implications en matière de sécurité et d'autorisations
---

<div id="building-a-personal-assistant-with-openclaw">
  # Créer un assistant personnel avec OpenClaw
</div>

OpenClaw est une passerelle WhatsApp + Telegram + Discord + iMessage pour des agents **Pi**. Des plugins ajoutent la prise en charge de Mattermost. Ce guide présente la configuration de « l’assistant personnel » : un numéro WhatsApp dédié qui se comporte comme votre agent toujours actif.

<div id="safety-first">
  ## ⚠️ Sécurité avant tout
</div>

Vous mettez un agent dans une position où il peut :

* exécuter des commandes sur votre ordinateur (selon la configuration de vos outils Pi)
* lire/écrire des fichiers dans votre espace de travail
* envoyer des messages via WhatsApp/Telegram/Discord/Mattermost (plugin)

Commencez prudemment :

* Configurez toujours `channels.whatsapp.allowFrom` (ne lancez jamais une configuration accessible depuis tout Internet sur votre Mac personnel).
* Utilisez un numéro WhatsApp dédié pour l’agent.
* Les signaux de vie sont désormais configurés par défaut toutes les 30 minutes. Désactivez-les tant que vous n’avez pas confiance dans la configuration en définissant `agents.defaults.heartbeat.every: "0m"`.

<div id="prerequisites">
  ## Prérequis
</div>

* Node **22+**
* OpenClaw disponible dans le PATH (recommandé : installation globale)
* Un deuxième numéro de téléphone (SIM/eSIM/prépayé) pour l&#39;assistant

```bash
npm install -g openclaw@latest
# ou : pnpm add -g openclaw@latest
```

À partir du code source (développement) :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## Configuration à deux téléphones (recommandée)
</div>

Voici ce que vous voulez :

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

Si vous connectez votre compte WhatsApp personnel à OpenClaw, chaque message que vous recevez devient une « entrée d’agent ». Ce n’est quasiment jamais ce que vous voulez.

<div id="5-minute-quick-start">
  ## Démarrage rapide en 5 minutes
</div>

1. Associez WhatsApp Web (un code QR s’affiche ; scannez-le avec le téléphone de l’assistant) :

```bash
openclaw channels login
```

2. Démarrez Gateway (laissez-le en cours d’exécution) :

```bash
openclaw gateway --port 18789
```

3. Ajoutez une configuration minimale dans `~/.openclaw/openclaw.json` :

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

Envoyez maintenant un message au numéro de l’assistant depuis votre téléphone autorisé dans la liste d’autorisation.

Une fois l’onboarding terminé, nous ouvrons automatiquement le tableau de bord avec votre jeton du Gateway et nous affichons le lien contenant ce jeton. Pour le rouvrir plus tard : `openclaw dashboard`.

<div id="give-the-agent-a-workspace-agents">
  ## Donner un espace de travail à l’agent (AGENTS)
</div>

OpenClaw lit les instructions de fonctionnement et la « mémoire » depuis son répertoire d’espace de travail.

Par défaut, OpenClaw utilise `~/.openclaw/workspace` comme espace de travail de l’agent et le crée automatiquement (ainsi que les fichiers initiaux `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`) lors de la configuration ou de la première exécution de l’agent. `BOOTSTRAP.md` n’est créé que lorsque l’espace de travail est entièrement nouveau (il ne devrait pas réapparaître après que vous l’avez supprimé).

Conseil : traitez ce dossier comme la « mémoire » d’OpenClaw et faites-en un dépôt git (idéalement privé) afin que vos fichiers `AGENTS.md` et de mémoire soient sauvegardés. Si git est installé, les espaces de travail entièrement nouveaux sont initialisés automatiquement.

```bash
openclaw setup
```

Organisation complète de l’espace de travail + guide de sauvegarde : [Agent workspace](/fr/concepts/agent-workspace)
Workflow de la mémoire : [Memory](/fr/concepts/memory)

Facultatif : choisissez un autre espace de travail avec `agents.defaults.workspace` (prend en charge `~`).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

Si vous livrez déjà vos propres fichiers d’espace de travail à partir d’un dépôt, vous pouvez désactiver entièrement la création des fichiers de bootstrap :

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## La configuration qui le transforme en « assistant »
</div>

OpenClaw utilise par défaut une bonne configuration d’assistant, mais vous voudrez généralement ajuster :

* la persona et les instructions dans `SOUL.md`
* les paramètres de raisonnement par défaut (si vous le souhaitez)
* les signaux de vie (une fois que vous lui faites confiance)

Exemple :

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Commencer à 0 ; activer plus tard.
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## Sessions et mémoire
</div>

* Fichiers de session : `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* Métadonnées de session (utilisation des jetons, dernière route, etc.) : `~/.openclaw/agents/<agentId>/sessions/sessions.json` (ancien emplacement : `~/.openclaw/sessions/sessions.json`)
* `/new` ou `/reset` démarre une nouvelle session pour cette conversation (paramétrable via `resetTriggers`). S&#39;il est envoyé seul, l&#39;agent répond avec un bref message de bienvenue pour confirmer la réinitialisation.
* `/compact [instructions]` compacte le contexte de la session et indique le budget de contexte restant.

<div id="heartbeats-proactive-mode">
  ## Signaux de vie (mode proactif)
</div>

Par défaut, OpenClaw exécute un signal de vie toutes les 30 minutes avec la consigne :
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
Définissez `agents.defaults.heartbeat.every: "0m"` pour le désactiver.

* Si `HEARTBEAT.md` existe mais est effectivement vide (uniquement des lignes vides et des en-têtes markdown comme `# Heading`), OpenClaw ignore l’exécution du signal de vie pour économiser des appels API.
* Si le fichier est absent, le signal de vie s’exécute quand même et le modèle décide quoi faire.
* Si l’agent répond avec `HEARTBEAT_OK` (éventuellement avec un court texte d’accompagnement ; voir `agents.defaults.heartbeat.ackMaxChars`), OpenClaw supprime l’envoi sortant pour ce signal de vie.
* Les signaux de vie exécutent des tours d’agent complets — des intervalles plus courts consomment davantage de jetons.

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## Médias en entrée et en sortie
</div>

Les pièces jointes entrantes (images/audio/documents) peuvent être mises à disposition de votre commande via des modèles :

* `{{MediaPath}}` (chemin de fichier temporaire local)
* `{{MediaUrl}}` (pseudo-URL)
* `{{Transcript}}` (si la transcription audio est activée)

Pour les pièces jointes sortantes générées par l&#39;agent : incluez `MEDIA:<path-or-url>` sur une ligne dédiée (sans espaces). Exemple :

```
Voici la capture d'écran.
MEDIA:/tmp/screenshot.png
```

OpenClaw les extrait et les envoie sous forme de médias, en complément du texte.

<div id="operations-checklist">
  ## Liste de vérification des opérations
</div>

```bash
openclaw status          # statut local (identifiants, sessions, événements en file d'attente)
openclaw status --all    # diagnostic complet (lecture seule, copiable-collable)
openclaw status --deep   # ajoute les sondes de santé du Gateway (Telegram + Discord)
openclaw health --json   # instantané de santé du Gateway (WS)
```

Les fichiers journaux se trouvent dans le répertoire `/tmp/openclaw/` (par défaut : `openclaw-YYYY-MM-DD.log`).

<div id="next-steps">
  ## Prochaines étapes
</div>

* WebChat : [WebChat](/fr/web/webchat)
* Exploitation du Gateway : [Runbook du Gateway](/fr/gateway)
* Cron + réveils : [Tâches Cron](/fr/automation/cron-jobs)
* Compagnon dans la barre de menus macOS : [App macOS OpenClaw](/fr/platforms/macos)
* Application de nœud iOS : [Application iOS](/fr/platforms/ios)
* Application de nœud Android : [Application Android](/fr/platforms/android)
* Statut Windows : [Windows (WSL2)](/fr/platforms/windows)
* Statut Linux : [Application Linux](/fr/platforms/linux)
* Sécurité : [Sécurité](/fr/gateway/security)