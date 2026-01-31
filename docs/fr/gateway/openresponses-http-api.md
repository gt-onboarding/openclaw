---
title: API HTTP OpenResponses
summary: "Exposer un point de terminaison HTTP /v1/responses compatible OpenResponses depuis le Gateway"
read_when:
  - Intégrer des clients qui utilisent l’API OpenResponses
  - Vous avez besoin d’entrées de type « item », d’appels d’outils côté client ou d’événements SSE
---

<div id="openresponses-api-http">
  # API OpenResponses (HTTP)
</div>

Le Gateway d’OpenClaw peut exposer un point de terminaison `POST /v1/responses` compatible OpenResponses.

Ce point de terminaison est **désactivé par défaut**. Activez-le d’abord dans la configuration.

- `POST /v1/responses`
- Même port que le Gateway (multiplexage WS + HTTP)&nbsp;: `http://<gateway-host>:<port>/v1/responses`

Sous le capot, les requêtes sont exécutées comme une exécution normale d’un agent du Gateway (même chemin de code que
`openclaw agent`), de sorte que le routage, les autorisations et la configuration correspondent à ceux de votre Gateway.

<div id="authentication">
  ## Authentification
</div>

Utilise la configuration d’authentification du Gateway. Envoie un jeton Bearer :

- `Authorization: Bearer <token>`

Remarques :

- Quand `gateway.auth.mode="token"`, utilise `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Quand `gateway.auth.mode="password"`, utilise `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).

<div id="choosing-an-agent">
  ## Choisir un agent
</div>

Aucun en-tête personnalisé n’est requis : encodez l’ID de l’agent dans le champ `model` d’OpenResponses :

- `model: "openclaw:&lt;agentId&gt;"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:&lt;agentId&gt;"` (alias)

Ou ciblez un agent OpenClaw spécifique via un en-tête :

- `x-openclaw-agent-id: &lt;agentId&gt;` (valeur par défaut : `main`)

Avancé :

- `x-openclaw-session-key: &lt;sessionKey&gt;` pour contrôler entièrement le routage de la session.

<div id="enabling-the-endpoint">
  ## Activation du point de terminaison
</div>

Définissez `gateway.http.endpoints.responses.enabled` sur `true` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## Désactivation du point de terminaison
</div>

Définissez `gateway.http.endpoints.responses.enabled` sur `false` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## Comportement de session
</div>

Par défaut, l’endpoint est **sans état, requête par requête** (une nouvelle clé de session est générée à chaque appel).

Si la requête inclut une chaîne OpenResponses `user`, le Gateway en déduit une clé de session stable, afin que les appels répétés puissent partager une session d’agent.

<div id="request-shape-supported">
  ## Format de la requête (pris en charge)
</div>

La requête suit l’API OpenResponses avec une entrée basée sur des éléments. Prise en charge actuelle :

- `input` : chaîne de caractères ou tableau d’objets « item ».
- `instructions` : intégrées au prompt système.
- `tools` : définitions d’outils côté client (outils de type fonction).
- `tool_choice` : filtrer ou imposer les outils côté client.
- `stream` : active le streaming SSE.
- `max_output_tokens` : limite de sortie en mode « best effort » (dépend du fournisseur).
- `user` : routage de session stable.

Acceptés mais **actuellement ignorés** :

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## Éléments (entrée)
</div>

<div id="message">
  ### `message`
</div>

Rôles : `system`, `developer`, `user`, `assistant`.

- `system` et `developer` sont ajoutés à l’invite système.
- L’élément `user` ou `function_call_output` le plus récent devient le « message actuel ».
- Les messages antérieurs user/assistant sont inclus dans l’historique de contexte.

<div id="function_call_output-turn-based-tools">
  ### `function_call_output` (outils tour par tour)
</div>

Renvoyez les résultats des outils au modèle :

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```


<div id="reasoning-and-item_reference">
  ### `reasoning` et `item_reference`
</div>

Acceptés pour la compatibilité avec le schéma mais ignorés lors de la génération du prompt.

<div id="tools-client-side-function-tools">
  ## Outils (outils de fonction côté client)
</div>

Fournissez les outils sous la forme `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Si l'agent décide d'appeler un outil, la réponse renvoie un élément de sortie `function_call`.
Vous envoyez ensuite une requête de suivi avec `function_call_output` pour poursuivre le tour de conversation.

<div id="images-input_image">
  ## Images (`input_image`)
</div>

Prend en charge les sources au format base64 ou par URL :

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Types MIME actuellement autorisés : `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Taille maximale actuelle : 10 Mo.


<div id="files-input_file">
  ## Fichiers (`input_file`)
</div>

Prend en charge les sources au format base64 ou par URL :

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Types MIME autorisés (actuels) : `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Taille maximale (actuelle) : 5 Mo.

Comportement actuel :

* Le contenu du fichier est décodé et ajouté au **system prompt**, et non au message utilisateur,
  de sorte qu’il reste éphémère (non conservé dans l’historique de la session).
* Les PDF sont analysés pour en extraire le texte. Si peu de texte est trouvé, les premières pages sont converties
  en images matricielles et transmises au modèle.

L’analyse PDF utilise la version legacy de `pdfjs-dist` compatible Node (sans worker). La version moderne
de PDF.js nécessite des workers de navigateur et des globales DOM, elle n’est donc pas utilisée dans le Gateway.

Valeurs par défaut pour la récupération d’URL :

* `files.allowUrl` : `true`
* `images.allowUrl` : `true`
* Les requêtes sont protégées (résolution DNS, blocage des IP privées, limites de redirection, délais d’expiration/timeouts).


<div id="file-image-limits-config">
  ## Limites de fichiers et d’images (config)
</div>

Les valeurs par défaut peuvent être ajustées dans `gateway.http.endpoints.responses` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: ["text/plain", "text/markdown", "text/html", "text/csv", "application/json", "application/pdf"],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200
            }
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000
          }
        }
      }
    }
  }
}
```

Valeurs par défaut en l’absence de configuration explicite :

* `maxBodyBytes`: 20MB
* `files.maxBytes`: 5MB
* `files.maxChars`: 200k
* `files.maxRedirects`: 3
* `files.timeoutMs`: 10s
* `files.pdf.maxPages`: 4
* `files.pdf.maxPixels`: 4,000,000
* `files.pdf.minTextChars`: 200
* `images.maxBytes`: 10MB
* `images.maxRedirects`: 3
* `images.timeoutMs`: 10s


<div id="streaming-sse">
  ## Diffusion en continu (SSE)
</div>

Définissez `stream: true` pour recevoir des Server-Sent Events (SSE) :

- `Content-Type: text/event-stream`
- Chaque ligne d’événement a la forme `event: <type>` et `data: <json>`
- Le flux se termine par `data: [DONE]`

Types d’événements actuellement émis :

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (en cas d’erreur)

<div id="usage">
  ## Utilisation
</div>

`usage` est renseigné lorsque le fournisseur sous-jacent renvoie les comptes de jetons.

<div id="errors">
  ## Erreurs
</div>

Les erreurs sont représentées par un objet JSON de la forme :

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Cas courants :

* `401` authentification manquante/non valide
* `400` corps de requête non valide
* `405` méthode HTTP incorrecte


<div id="examples">
  ## Exemples
</div>

Sans streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
