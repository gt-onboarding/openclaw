---
title: "Invocation d’outils via l’API HTTP"
summary: "Appeler directement un seul outil via le point de terminaison HTTP du Gateway"
read_when:
  - Appeler des outils sans exécuter un tour complet d’agent
  - Créer des automatisations qui nécessitent le respect des politiques des outils
---

<div id="tools-invoke-http">
  # Invocation d’outils (HTTP)
</div>

Le Gateway OpenClaw expose un endpoint HTTP simple permettant d’appeler directement un seul outil. Il est toujours activé, mais protégé par l’authentification du Gateway et la politique des outils.

- `POST /tools/invoke`
- Même port que le Gateway (multiplexage WS + HTTP) : `http://<gateway-host>:<port>/tools/invoke`

La taille maximale par défaut du corps de la requête est de 2 Mo.

<div id="authentication">
  ## Authentification
</div>

Utilise la configuration d'authentification du Gateway. Envoyez un jeton Bearer :

- `Authorization: Bearer <token>`

Remarques :

- Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).

<div id="request-body">
  ## Corps de la requête
</div>

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Champs :

* `tool` (string, requis) : nom de l’outil à invoquer.
* `action` (string, optionnel) : recopié dans `args` si le schéma de l’outil prend en charge `action` et que la payload `args` l’a omis.
* `args` (object, optionnel) : arguments spécifiques à l’outil.
* `sessionKey` (string, optionnel) : clé de session cible. Si elle est omise ou vaut `"main"`, le Gateway utilise la clé de session principale configurée (respecte `session.mainKey` et l’agent par défaut, ou `global` dans la portée globale).
* `dryRun` (boolean, optionnel) : réservé pour une utilisation future ; actuellement ignoré.


<div id="policy-routing-behavior">
  ## Comportement des politiques et du routage
</div>

La disponibilité des outils est filtrée par la même chaîne de politiques utilisée par les agents du Gateway :

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- politiques de groupe (si la clé de session correspond à un groupe ou à un canal)
- politique de sous-agent (lors de l’invocation avec une clé de session de sous-agent)

Si un outil n’est pas autorisé par les politiques, le point de terminaison renvoie **404**.

Pour aider les politiques de groupe à déterminer le contexte, vous pouvez définir en option :

- `x-openclaw-message-channel: <channel>` (exemple : `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (lorsque plusieurs comptes existent)

<div id="responses">
  ## Réponses
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (requête invalide ou erreur d’outil)
- `401` → non autorisé
- `404` → outil non disponible (introuvable ou non présent dans la liste d’autorisation)
- `405` → méthode non autorisée

<div id="example">
  ## Exemple
</div>

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
