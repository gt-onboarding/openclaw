---
title: Messages
summary: "Flux de messages, sessions, mise en file d'attente et visibilité du raisonnement"
read_when:
  - Expliquer comment les messages entrants deviennent des réponses
  - Clarifier les sessions, les modes de mise en file d'attente ou le comportement de streaming
  - Documenter la visibilité du raisonnement et les implications pour l'utilisation
---

<div id="messages">
  # Messages
</div>

Cette page présente la manière dont OpenClaw gère les messages entrants, les sessions, la mise en file d&#39;attente, le streaming et la visibilité du raisonnement.

<div id="message-flow-high-level">
  ## Flux des messages (vue d’ensemble)
</div>

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Les principaux réglages se trouvent dans la configuration :

* `messages.*` pour les préfixes, la mise en file d&#39;attente et le comportement de groupe.
* `agents.defaults.*` pour les valeurs par défaut de streaming par blocs et de découpage en fragments.
* Surcharges de canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) pour les limites et les options d’activation du streaming.

Consultez [Configuration](/fr/gateway/configuration) pour le schéma complet.

<div id="inbound-dedupe">
  ## Déduplication des messages entrants
</div>

Les canaux peuvent réémettre le même message après une reconnexion. OpenClaw conserve
un cache de courte durée de vie indexé par canal/compte/correspondant/session/id de message, afin que les doublons
ne déclenchent pas une nouvelle exécution d’agent.

<div id="inbound-debouncing">
  ## Anti-rebond des messages entrants
</div>

Des messages consécutifs rapides provenant du **même expéditeur** peuvent être regroupés en un seul
tour d’agent via `messages.inbound`. L’anti-rebond s’applique par canal et par conversation
et utilise le message le plus récent pour le chaînage des réponses et les IDs.

Configuration (valeur globale par défaut + surcharges par canal) :

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notes :

* Le « debounce » s’applique uniquement aux messages **texte** ; les médias et pièces jointes sont transmis immédiatement.
* Les commandes de contrôle contournent le « debounce » afin de rester indépendantes.

<div id="sessions-and-devices">
  ## Sessions et appareils
</div>

Les sessions appartiennent au Gateway, pas aux clients.

* Les discussions directes sont regroupées dans la clé de session principale de l’agent.
* Les groupes/canaux ont leurs propres clés de session.
* Le stockage des sessions et les transcriptions résident sur l’hôte du Gateway.

Plusieurs appareils/canaux peuvent être associés à la même session, mais l’historique n’est pas entièrement
synchronisé pour chaque client. Recommandation : utilisez un appareil principal pour les longues
conversations afin d’éviter une divergence de contexte. Le Control UI et le TUI affichent toujours la
transcription de session maintenue par le Gateway ; ils font donc foi.

Détails : [Gestion des sessions](/fr/concepts/session).

<div id="inbound-bodies-and-history-context">
  ## Corps des messages entrants et contexte d’historique
</div>

OpenClaw sépare le **corps de prompt** du **corps de commande** :

* `Body` : texte de prompt envoyé à l’agent. Cela peut inclure des enveloppes de canal et
  des wrappers d’historique optionnels.
* `CommandBody` : texte brut de l’utilisateur pour l’analyse des directives/commandes.
* `RawBody` : alias hérité de `CommandBody` (conservé pour la compatibilité).

Lorsqu’un canal fournit de l’historique, il utilise un wrapper partagé :

* `[Chat messages since your last reply - for context]`
* `[Current message - respond to this]`

Pour les **conversations non directes** (groupes/canaux/salons), le **corps du message courant** est préfixé par le
libellé de l’expéditeur (même style que celui utilisé pour les entrées d’historique). Cela permet de garder les messages
en temps réel et les messages en file d’attente/historique cohérents dans le prompt de l’agent.

Les buffers d’historique sont **uniquement en attente** : ils incluent les messages de groupe qui *n’ont pas*
déclenché de run (par exemple, des messages soumis à condition de mention) et **excluent** les messages
déjà présents dans la transcription de la session.

La suppression des directives ne s’applique qu’à la section **message courant** afin que l’historique
reste intact. Les canaux qui encapsulent l’historique doivent définir `CommandBody` (ou
`RawBody`) sur le texte original du message et conserver `Body` comme prompt combiné.
Les buffers d’historique sont configurables via `messages.groupChat.historyLimit` (valeur
par défaut globale) et via des surcharges par canal comme `channels.slack.historyLimit` ou
`channels.telegram.accounts.<id>.historyLimit` (définir `0` pour désactiver).

<div id="queueing-and-followups">
  ## Mise en file d&#39;attente et suivis
</div>

Si une exécution est déjà active, les messages entrants peuvent être mis en file
d&#39;attente, dirigés vers l&#39;exécution en cours ou collectés pour un tour ultérieur.

* Configurez via `messages.queue` (et `messages.queue.byChannel`).
* Modes : `interrupt`, `steer`, `followup`, `collect`, plus des variantes pour le backlog.

Détails : [Mise en file d&#39;attente](/fr/concepts/queue).

<div id="streaming-chunking-and-batching">
  ## Diffusion en continu, découpage et traitement par lots
</div>

La diffusion en continu par blocs envoie des réponses partielles au fur et à mesure que le modèle produit des blocs de texte.
Le découpage respecte les limites de texte des canaux et évite de couper les blocs de code délimités.

Paramètres clés :

* `agents.defaults.blockStreamingDefault` (`on|off`, désactivé par défaut)
* `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
* `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
* `agents.defaults.blockStreamingCoalesce` (regroupement par lots basé sur l’inactivité)
* `agents.defaults.humanDelay` (pause simulant un délai humain entre les réponses par bloc)
* Surcharges au niveau des canaux : `*.blockStreaming` et `*.blockStreamingCoalesce` (les canaux non-Telegram nécessitent explicitement `*.blockStreaming: true`)

Détails : [Streaming et découpage](/fr/concepts/streaming).

<div id="reasoning-visibility-and-tokens">
  ## Visibilité du raisonnement et jetons
</div>

OpenClaw peut exposer ou masquer le raisonnement du modèle :

* `/reasoning on|off|stream` contrôle la visibilité.
* Le contenu de raisonnement est tout de même pris en compte dans l’utilisation de jetons lorsqu’il est produit par le modèle.
* Telegram prend en charge le streaming du raisonnement dans la bulle de brouillon.

Détails : [Directives de réflexion et de raisonnement](/fr/tools/thinking) et [Utilisation des jetons](/fr/token-use).

<div id="prefixes-threading-and-replies">
  ## Préfixes, fils de discussion et réponses
</div>

Le formatage des messages sortants est centralisé dans `messages` :

* `messages.responsePrefix` (préfixe sortant) et `channels.whatsapp.messagePrefix` (préfixe entrant WhatsApp)
* Gestion des fils de réponse via `replyToMode` et des valeurs par défaut propres à chaque canal

Détails : [Configuration](/fr/gateway/configuration#messages) et documentation des canaux.