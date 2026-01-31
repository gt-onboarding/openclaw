---
title: Signal de vie
summary: "Messages de sondage du signal de vie et règles de notification"
read_when:
  - Ajuster la fréquence ou la configuration des messages du signal de vie
  - Choisir entre le signal de vie et cron pour des tâches programmées
---

<div id="heartbeat-gateway">
  # Heartbeat (Gateway)
</div>

> **Heartbeat vs Cron ?** Voir [Cron vs Heartbeat](/fr/automation/cron-vs-heartbeat) pour savoir quand utiliser l’un ou l’autre.

Heartbeat exécute périodiquement des itérations d’agent dans la session principale afin que le modèle puisse faire remonter tout ce qui nécessite votre attention sans vous inonder de messages.

<div id="quick-start-beginner">
  ## Démarrage rapide (débutant)
</div>

1. Laissez les signaux de vie activés (valeur par défaut : `30m`, ou `1h` pour Anthropic OAuth/setup-token) ou définissez votre propre cadence.
2. Créez une petite checklist `HEARTBEAT.md` dans l’espace de travail de l’agent (facultatif mais recommandé).
3. Décidez où les messages de signal de vie doivent être envoyés (`target: "last"` est la valeur par défaut).
4. Facultatif : activez la diffusion du raisonnement des signaux de vie pour plus de transparence.
5. Facultatif : limitez les signaux de vie aux heures actives (heure locale).

Exemple de configuration :

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // optionnel : envoyer aussi un message `Reasoning:` séparé
      }
    }
  }
}
```

<div id="defaults">
  ## Valeurs par défaut
</div>

* Intervalle : `30m` (ou `1h` lorsque Anthropic OAuth/setup-token est le mode d&#39;authentification détecté). Configurez `agents.defaults.heartbeat.every` ou, par agent, `agents.list[].heartbeat.every` ; utilisez `0m` pour désactiver.
* Corps de l&#39;invite (configurable via `agents.defaults.heartbeat.prompt`) :
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
* L&#39;invite de signal de vie est envoyée **telle quelle** comme message utilisateur. L&#39;invite
  système inclut une section « Heartbeat » et l&#39;exécution est marquée en interne.
* Les heures actives (`heartbeat.activeHours`) sont vérifiées dans le fuseau horaire configuré.
  En dehors de cette plage horaire, les signaux de vie sont ignorés jusqu&#39;au prochain déclenchement à l&#39;intérieur de cette plage.

<div id="what-the-heartbeat-prompt-is-for">
  ## À quoi sert l’invite de signal de vie
</div>

L’invite par défaut est volontairement générale :

* **Tâches d’arrière-plan** : « Consider outstanding tasks » incite l’agent à examiner
  les suivis (boîte de réception, agenda, rappels, travail en file d’attente) et à faire remonter tout ce qui est urgent.
* **Point de contact humain** : « Checkup sometimes on your human during day time » incite
  l’envoi occasionnel d’un message léger du type « besoin de quelque chose ? », tout en évitant le spam nocturne
  grâce à l’utilisation de votre fuseau horaire local configuré (voir [/concepts/timezone](/fr/concepts/timezone)).

Si vous voulez qu’un signal de vie fasse quelque chose de très spécifique (par ex. « check Gmail PubSub
stats » ou « verify gateway health »), définissez `agents.defaults.heartbeat.prompt` (ou
`agents.list[].heartbeat.prompt`) sur un texte personnalisé (envoyé tel quel).

<div id="response-contract">
  ## Contrat de réponse
</div>

* Si rien ne nécessite d&#39;attention, répondez avec **`HEARTBEAT_OK`**.
* Pendant les exécutions du signal de vie, OpenClaw traite `HEARTBEAT_OK` comme un accusé de réception lorsqu&#39;il apparaît
  **au début ou à la fin** de la réponse. Le jeton est supprimé et la réponse est
  ignorée si le contenu restant est **≤ `ackMaxChars`** (valeur par défaut : 300).
* Si `HEARTBEAT_OK` apparaît **au milieu** d&#39;une réponse, il n&#39;est pas traité
  de manière particulière.
* Pour les alertes, **n&#39;incluez pas** `HEARTBEAT_OK` ; renvoyez uniquement le texte de l&#39;alerte.

En dehors des signaux de vie, un `HEARTBEAT_OK` isolé au début ou à la fin d&#39;un message est supprimé
et consigné dans les journaux ; un message qui contient uniquement `HEARTBEAT_OK` est ignoré.

<div id="config">
  ## Configuration
</div>

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last",         // last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567",     // optional channel-specific override
        prompt: "Lis HEARTBEAT.md s'il existe (contexte de l'espace de travail). Suis-le strictement. N'infère pas et ne répète pas d'anciennes tâches issues de conversations précédentes. Si rien ne nécessite d'attention, réponds HEARTBEAT_OK.",
        ackMaxChars: 300         // max chars allowed after HEARTBEAT_OK
      }
    }
  }
}
```

<div id="scope-and-precedence">
  ### Portée et priorité
</div>

* `agents.defaults.heartbeat` définit le comportement global du signal de vie.
* `agents.list[].heartbeat` est fusionné par‑dessus ; si un agent possède un bloc `heartbeat`, **seuls ces agents** exécutent le signal de vie.
* `channels.defaults.heartbeat` définit les paramètres de visibilité par défaut pour tous les canaux.
* `channels.<channel>.heartbeat` remplace les valeurs par défaut du canal.
* `channels.<channel>.accounts.<id>.heartbeat` (canaux multi‑comptes) remplace les paramètres propres à chaque canal.

<div id="per-agent-heartbeats">
  ### Signaux de vie par agent
</div>

Si une entrée `agents.list[]` contient un bloc `heartbeat`, **seuls ces agents**
exécutent des signaux de vie. Le bloc par agent est fusionné par‑dessus `agents.defaults.heartbeat`
(vous pouvez ainsi définir des valeurs par défaut communes une seule fois et les surcharger agent par agent).

Exemple : deux agents, seul le deuxième agent exécute des signaux de vie.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md s'il existe (contexte de l'espace de travail). Suis-le strictement. N'infère pas et ne répète pas d'anciennes tâches issues de discussions précédentes. Si rien ne nécessite d'attention, réponds HEARTBEAT_OK."
        }
      }
    ]
  }
}
```

<div id="field-notes">
  ### Notes pratiques
</div>

* `every` : intervalle du signal de vie (chaîne de durée ; unité par défaut = minutes).
* `model` : surcharge facultative du modèle pour les exécutions du signal de vie (`provider/model`).
* `includeReasoning` : lorsqu’il est activé, transmet aussi le message séparé `Reasoning:` lorsqu’il est disponible (même forme que `/reasoning on`).
* `session` : clé de session facultative pour les exécutions du signal de vie.
  * `main` (par défaut) : session principale de l’agent.
  * Clé de session explicite (copiée depuis `openclaw sessions --json` ou la [CLI sessions](/fr/cli/sessions)).
  * Formats de clé de session : voir [Sessions](/fr/concepts/session) et [Groups](/fr/concepts/groups).
* `target` :
  * `last` (par défaut) : envoyer vers le dernier canal externe utilisé.
  * canal explicite : `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  * `none` : exécuter le signal de vie mais **ne pas l’envoyer** en externe.
* `to` : remplacement facultatif du destinataire (identifiant spécifique au canal, par ex. E.164 pour WhatsApp ou un identifiant de conversation Telegram).
* `prompt` : remplace le corps du prompt par défaut (non fusionné).
* `ackMaxChars` : nombre maximal de caractères autorisés après `HEARTBEAT_OK` avant l’envoi.

<div id="delivery-behavior">
  ## Comportement d’acheminement
</div>

* Les signaux de vie s’exécutent par défaut dans la session principale de l’agent (`agent:<id>:<mainKey>`),
  ou `global` lorsque `session.scope = "global"`. Définissez `session` pour la remplacer par une
  session de canal spécifique (Discord/WhatsApp/etc.).
* `session` affecte uniquement le contexte d’exécution ; l’acheminement des messages est contrôlé par `target` et `to`.
* Pour envoyer vers un canal/destinataire spécifique, définissez `target` + `to`. Avec
  `target: "last"`, l’acheminement utilise le dernier canal externe pour cette session.
* Si la file principale est saturée, le signal de vie est ignoré et sera réessayé ultérieurement.
* Si `target` ne correspond à aucune destination externe, l’exécution a tout de même lieu mais
  aucun message sortant n’est envoyé.
* Les réponses déclenchées uniquement par le signal de vie ne **maintiennent pas** la session active ; la dernière valeur de `updatedAt`
  est restaurée afin que l’expiration pour inactivité fonctionne normalement.

<div id="visibility-controls">
  ## Paramètres de visibilité
</div>

Par défaut, les accusés de réception `HEARTBEAT_OK` sont supprimés tant que le contenu d’alerte est
envoyé. Vous pouvez ajuster ce comportement par canal ou par compte :

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # Masquer HEARTBEAT_OK (par défaut)
      showAlerts: true   # Afficher les messages d'alerte (par défaut)
      useIndicator: true # Émettre des événements d'indicateur (par défaut)
  telegram:
    heartbeat:
      showOk: true       # Afficher les accusés de réception OK sur Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Supprimer la distribution d'alertes pour ce compte
```

Ordre de priorité : par compte → par canal → valeurs par défaut du canal → valeurs par défaut intégrées.

<div id="what-each-flag-does">
  ### Fonction de chaque indicateur
</div>

* `showOk` : envoie un accusé de réception `HEARTBEAT_OK` lorsque le modèle renvoie une réponse contenant uniquement OK.
* `showAlerts` : envoie le contenu de l’alerte lorsque le modèle renvoie une réponse non OK.
* `useIndicator` : émet des événements d’indicateur pour les surfaces d’état de l’UI.

Si **tous les trois** sont à `false`, OpenClaw ignore complètement l’exécution du signal de vie (aucun appel au modèle n’est effectué).

<div id="per-channel-vs-per-account-examples">
  ### Exemples : par canal ou par compte
</div>

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # tous les comptes Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # supprimer les alertes pour le compte ops uniquement
  telegram:
    heartbeat:
      showOk: true
```

<div id="common-patterns">
  ### Modèles courants
</div>

| Objectif | Config |
| --- | --- |
| Comportement par défaut (OK silencieux, alertes activées) | *(aucune configuration nécessaire)* |
| Entièrement silencieux (aucun message, aucun indicateur) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Indicateur uniquement (aucun message) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| OK dans un seul canal uniquement | `channels.telegram.heartbeat: { showOk: true }` |

<div id="heartbeatmd-optional">
  ## HEARTBEAT.md (facultatif)
</div>

Si un fichier `HEARTBEAT.md` existe dans l’espace de travail, l’invite par défaut demande à l’agent de le read (le lire). Considérez-le comme votre « checklist de signal de vie » : petite, stable et suffisamment sûre pour être incluse toutes les 30 minutes.

Si `HEARTBEAT.md` existe mais est effectivement vide (uniquement des lignes vides et des en-têtes markdown comme `# Heading`), OpenClaw ignore l’exécution du signal de vie pour économiser des appels API. Si le fichier est absent, le signal de vie s’exécute quand même et le modèle décide de ce qu’il faut faire.

Gardez-le très court (checklist ou rappels succincts) pour éviter le gonflement de l’invite.

Exemple de `HEARTBEAT.md` :

```md
# Liste de contrôle du signal de vie

- Scan rapide : quelque chose d'urgent dans les boîtes de réception ?
- Si c'est en journée, faire un point léger si rien d'autre n'est en attente.
- Si une tâche est bloquée, noter *ce qui manque* et demander à Peter la prochaine fois.
```

<div id="can-the-agent-update-heartbeatmd">
  ### L’agent peut-il mettre à jour HEARTBEAT.md ?
</div>

Oui — si vous le lui demandez.

`HEARTBEAT.md` n’est qu’un fichier ordinaire dans l’espace de travail de l’agent, vous pouvez donc dire à l’agent (dans une conversation normale) quelque chose comme :

* « Mets à jour `HEARTBEAT.md` pour ajouter une vérification quotidienne du calendrier. »
* « Réécris `HEARTBEAT.md` pour qu’il soit plus court et centré sur les relances de la boîte de réception. »

Si vous voulez que cela se produise de manière proactive, vous pouvez aussi inclure une ligne explicite dans votre invite de signal de vie, par exemple : « Si la liste de contrôle devient obsolète, mets à jour HEARTBEAT.md avec une meilleure version. »

Note de sécurité : ne mettez pas de secrets (clés API, numéros de téléphone, jetons privés) dans `HEARTBEAT.md` — il fait partie du contexte de l’invite.

<div id="manual-wake-on-demand">
  ## Réveil manuel (à la demande)
</div>

Vous pouvez mettre en file d&#39;attente un événement système et déclencher immédiatement un signal de vie avec :

```bash
openclaw system event --text "Vérifier les suivis urgents" --mode now
```

Si plusieurs agents ont `heartbeat` configuré, un réveil manuel exécute immédiatement
le signal de vie de chacun de ces agents.

Utilisez `--mode next-heartbeat` pour attendre le prochain tick programmé.

<div id="reasoning-delivery-optional">
  ## Diffusion du raisonnement (optionnel)
</div>

Par défaut, les signaux de vie ne transmettent que la charge utile de la « réponse » finale.

Si vous souhaitez plus de transparence, activez :

* `agents.defaults.heartbeat.includeReasoning: true`

Une fois activés, les signaux de vie transmettront également un message séparé préfixé
par `Reasoning:` (même structure que `/reasoning on`). Cela peut être utile lorsque l’agent
gère plusieurs sessions/codex et que vous voulez voir pourquoi il a décidé de vous envoyer
un ping — mais cela peut aussi exposer plus de détails internes que vous ne le souhaiteriez.
Il est préférable de le laisser désactivé dans les conversations de groupe.

<div id="cost-awareness">
  ## Maîtrise des coûts
</div>

Les signaux de vie exécutent des cycles d’agent complets. Des intervalles plus courts consomment davantage de jetons. Gardez `HEARTBEAT.md` aussi court que possible et envisagez un `model` moins coûteux ou `target: "none"` si vous ne voulez que des mises à jour de l’état interne.