---
title: Webhook
summary: "Point d'entrée webhook pour le réveil et les exécutions d'agents isolées"
read_when:
  - Ajout ou modification de points de terminaison webhook
  - Connexion de systèmes externes à OpenClaw
---

<div id="webhooks">
  # Webhooks
</div>

Gateway peut exposer un petit endpoint HTTP de type webhook afin de recevoir des déclenchements externes.

<div id="enable">
  ## Activation
</div>

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

Notes :

* `hooks.token` est obligatoire lorsque `hooks.enabled=true`.
* `hooks.path` a pour valeur par défaut `/hooks`.

<div id="auth">
  ## Authentification
</div>

Chaque requête doit inclure le jeton du webhook. Privilégiez les en‑têtes :

* `Authorization: Bearer <token>` (recommandé)
* `x-openclaw-token: <token>`
* `?token=<token>` (déprécié ; génère un avertissement dans les journaux et sera supprimé dans une prochaine version majeure)

<div id="endpoints">
  ## Points de terminaison
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

Corps de la requête :

```json
{ "text": "System line", "mode": "now" }
```

* `text` **requis** (string) : La description de l’événement (par exemple, &quot;Nouvel e-mail reçu&quot;).
* `mode` optionnel (`now` | `next-heartbeat`) : Indique s’il faut déclencher un signal de vie immédiat (valeur par défaut `now`) ou attendre le prochain contrôle périodique.

Effet :

* Met en file d’attente un événement système pour la session **principale**
* Si `mode=now`, déclenche un signal de vie immédiat

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

Corps de requête :

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

* `message` **required** (string) : Le prompt ou message que l’agent doit traiter.
* `name` facultatif (string) : Nom lisible par un humain pour le hook (par exemple, &quot;GitHub&quot;), utilisé comme préfixe dans les résumés de session.
* `sessionKey` facultatif (string) : Clé utilisée pour identifier la session de l’agent. Par défaut, une valeur aléatoire `hook:<uuid>`. Utiliser une clé cohérente permet une conversation multi‑tours dans le contexte du hook.
* `wakeMode` facultatif (`now` | `next-heartbeat`) : Indique s’il faut déclencher immédiatement un signal de vie (par défaut `now`) ou attendre la prochaine vérification périodique.
* `deliver` facultatif (boolean) : Si `true`, la réponse de l’agent sera envoyée au canal de messagerie. Par défaut `true`. Les réponses qui ne sont que des accusés de réception de signal de vie sont automatiquement ignorées.
* `channel` facultatif (string) : Canal de messagerie pour l’envoi. L’un de : `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Par défaut `last`.
* `to` facultatif (string) : Identifiant du destinataire pour le canal (par exemple, numéro de téléphone pour WhatsApp/Signal, ID de chat pour Telegram, ID de canal pour Discord/Slack/Mattermost (plugin), ID de conversation pour MS Teams). Par défaut, le dernier destinataire de la session principale.
* `model` facultatif (string) : Remplacement du modèle (par exemple, `anthropic/claude-3-5-sonnet` ou un alias). Doit figurer dans la liste des modèles autorisés si des restrictions s’appliquent.
* `thinking` facultatif (string) : Remplacement du niveau de réflexion (par exemple, `low`, `medium`, `high`).
* `timeoutSeconds` facultatif (number) : Durée maximale d’exécution de l’agent, en secondes.

Effet :

* Exécute un tour d’agent **isolé** (sa propre clé de session)
* Publie toujours un résumé dans la session **principale**
* Si `wakeMode=now`, déclenche immédiatement un signal de vie

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>` (mappé)
</div>

Les noms de hooks personnalisés sont résolus via `hooks.mappings` (voir configuration). Un mapping peut
convertir des payloads arbitraires en actions `wake` ou `agent`, avec des templates ou
des transformations de code optionnels.

Options de mapping (résumé) :

* `hooks.presets: ["gmail"]` active le mapping Gmail intégré.
* `hooks.mappings` vous permet de définir `match`, `action` et des templates dans la config.
* `hooks.transformsDir` + `transform.module` charge un module JS/TS pour une logique personnalisée.
* Utilisez `match.source` pour conserver un endpoint d’ingestion générique (routage piloté par le payload).
* Les transformations TS nécessitent un chargeur TS (par ex. `bun` ou `tsx`) ou un `.js` précompilé à l’exécution.
* Configurez `deliver: true` + `channel`/`to` sur les mappings pour router les réponses vers une surface de chat
  (`channel` vaut par défaut `last` et se rabat sur WhatsApp).
* `allowUnsafeExternalContent: true` désactive l’enveloppe de sécurité des contenus externes pour ce hook
  (dangereux ; uniquement pour des sources internes de confiance).
* `openclaw webhooks gmail setup` écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`.
  Voir [Gmail Pub/Sub](/fr/automation/gmail-pubsub) pour le flux complet de surveillance Gmail.

<div id="responses">
  ## Réponses
</div>

* `200` pour `/hooks/wake`
* `202` pour `/hooks/agent` (exécution asynchrone démarrée)
* `401` en cas d&#39;échec d&#39;authentification
* `400` en cas de corps de requête invalide
* `413` en cas de corps de requête trop volumineux

<div id="examples">
  ## Exemples
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### Utiliser un modèle différent
</div>

Ajoutez `model` au payload de l’agent (ou au mapping) pour remplacer le modèle utilisé pour cette exécution :

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si vous forcez `agents.defaults.models`, assurez-vous que le modèle de remplacement y est inclus.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## Sécurité
</div>

* Maintenez les endpoints de hook derrière l’interface loopback, un tailnet ou un reverse proxy de confiance.
* Utilisez un jeton dédié aux hooks ; ne réutilisez pas les jetons d’authentification du Gateway.
* Évitez d’inclure des payloads bruts contenant des données sensibles dans les journaux de webhook.
* Les payloads de hook sont considérées comme non fiables et sont, par défaut, encapsulées dans des mécanismes de sécurité.
  Si vous devez désactiver cela pour un hook spécifique, définissez `allowUnsafeExternalContent: true`
  dans le mappage de ce hook (dangereux).