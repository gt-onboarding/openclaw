---
title: Control UI
summary: "UI de contrôle dans le navigateur pour le Gateway (chat, nœuds, configuration)"
read_when:
  - Vous souhaitez administrer le Gateway via un navigateur
  - Vous souhaitez un accès Tailnet sans tunnels SSH
---

<div id="control-ui-browser">
  # Control UI (navigateur)
</div>

Le Control UI est une petite application mono‑page **Vite + Lit** hébergée par le Gateway :

* par défaut : `http://<host>:18789/`
* préfixe optionnel : définissez `gateway.controlUi.basePath` (par ex. `/openclaw`)

Il communique **directement avec le Gateway via WebSocket** sur le même port.

<div id="quick-open-local">
  ## Ouverture rapide (locale)
</div>

Si le Gateway s’exécute sur le même ordinateur, ouvrez :

* http://127.0.0.1:18789/ (ou http://localhost:18789/)

Si la page ne se charge pas, démarrez d’abord le Gateway : `openclaw gateway`.

L’authentification est fournie pendant le handshake WebSocket via :

* `connect.params.auth.token`
* `connect.params.auth.password`
  Le panneau des paramètres du tableau de bord vous permet de stocker un jeton ; les mots de passe ne sont pas conservés.
  L’assistant de démarrage génère par défaut un jeton de Gateway ; collez-le ici lors de la première connexion.

<div id="what-it-can-do-today">
  ## Ce qu’il peut faire (aujourd’hui)
</div>

* Dialoguer avec le modèle via Gateway en WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
* Diffuser les appels d’outils + cartes de sortie d’outils en direct dans Chat (événements d’agent)
* Canaux : statut WhatsApp/Telegram/Discord/Slack + canaux de plugin (Mattermost, etc.) + connexion par QR + configuration par canal (`channels.status`, `web.login.*`, `config.patch`)
* Instances : liste de présence + rafraîchissement (`system-presence`)
* Sessions : liste + surcharges des modes réflexion/verbeux par session (`sessions.list`, `sessions.patch`)
* Tâches cron : lister/ajouter/exécuter/activer/désactiver + historique d’exécution (`cron.*`)
* Compétences : statut, activer/désactiver, installer, mises à jour de clés API (`skills.*`)
* Nœuds : liste + capacités (`node.list`)
* Approbations d’exécution : modifier les listes d’autorisation du Gateway ou des nœuds + demander la stratégie pour `exec host=gateway/node` (`exec.approvals.*`)
* Config : afficher/modifier `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
* Config : appliquer + redémarrer avec validation (`config.apply`) et réveiller la dernière session active
* Les écritures de configuration incluent une protection par hachage de base pour empêcher d’écraser des modifications concurrentes
* Schéma de configuration + rendu de formulaire (`config.schema`, y compris les schémas de plugins + canaux) ; l’éditeur JSON brut reste disponible
* Debug : instantanés de statut/santé/modèles + journal d’événements + appels RPC manuels (`status`, `health`, `models.list`)
* Journaux : suivi en direct des fichiers journaux du Gateway avec filtrage/export (`logs.tail`)
* Mise à jour : exécuter une mise à jour des paquets/git + redémarrage (`update.run`) avec un rapport de redémarrage

<div id="chat-behavior">
  ## Comportement du chat
</div>

* `chat.send` est **non bloquant** : il accuse immédiatement réception avec `{ runId, status: "started" }` et la réponse est diffusée en continu via des événements `chat`.
* Renvoyer avec le même `idempotencyKey` renvoie `{ status: "in_flight" }` pendant l&#39;exécution, puis `{ status: "ok" }` une fois terminée.
* `chat.inject` ajoute une note d’assistant à la transcription de la session et diffuse un événement `chat` pour des mises à jour côté UI uniquement (aucune exécution d’agent, aucune remise sur un canal).
* Arrêt :
  * Cliquez sur **Stop** (appelle `chat.abort`)
  * Tapez `/stop` (ou `stop|esc|abort|wait|exit|interrupt`) pour interrompre hors bande
  * `chat.abort` prend en charge `{ sessionKey }` (sans `runId`) pour interrompre toutes les exécutions actives pour cette session

<div id="tailnet-access-recommended">
  ## Accès via Tailnet (recommandé)
</div>

<div id="integrated-tailscale-serve-preferred">
  ### Tailscale Serve intégré (recommandé)
</div>

Laissez le Gateway sur l’interface loopback et laissez Tailscale Serve le placer derrière un proxy HTTPS :

```bash
openclaw gateway --tailscale serve
```

Ouvrez :

* `https://<magicdns>/` (ou votre valeur configurée pour `gateway.controlUi.basePath`)

Par défaut, les requêtes Serve peuvent s’authentifier via les en-têtes d’identité Tailscale
(`tailscale-user-login`) lorsque `gateway.auth.allowTailscale` vaut `true`. OpenClaw
vérifie l’identité en résolvant l’adresse `x-forwarded-for` avec
`tailscale whois` et en la faisant correspondre à l’en-tête, et n’accepte ces requêtes que lorsque
la requête arrive sur l’interface loopback locale avec les en-têtes `x-forwarded-*` de Tailscale. Définissez
`gateway.auth.allowTailscale: false` (ou forcez `gateway.auth.mode: "password"`)
afin d’exiger un jeton/mot de passe même pour le trafic généré par Serve.

<div id="bind-to-tailnet-token">
  ### Lier au tailnet + jeton
</div>

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Ensuite ouvrez :

* `http://<tailscale-ip>:18789/` (ou la valeur que vous avez configurée pour `gateway.controlUi.basePath`)

Collez le jeton dans les paramètres de l&#39;UI (envoyé en tant que `connect.params.auth.token`).

<div id="insecure-http">
  ## HTTP non sécurisé
</div>

Si vous ouvrez le tableau de bord en HTTP en clair (`http://<lan-ip>` ou `http://<tailscale-ip>`),
le navigateur s&#39;exécute dans un **contexte non sécurisé** et bloque WebCrypto. Par défaut,
OpenClaw **bloque** les connexions à la Control UI sans identité de l&#39;appareil.

**Correctif recommandé :** utilisez HTTPS (Tailscale Serve) ou ouvrez l&#39;UI localement :

* `https://<magicdns>/` (Serve)
* `http://127.0.0.1:18789/` (sur l&#39;hôte du Gateway)

**Exemple de mode dégradé (jeton uniquement via HTTP) :**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

Cela désactive l’identité de l’appareil et l’appairage pour la Control UI (même via HTTPS). Utilisez-le uniquement si vous faites confiance au réseau.

Voir [Tailscale](/fr/gateway/tailscale) pour des consignes de configuration HTTPS.

<div id="building-the-ui">
  ## Générer l’UI
</div>

Le Gateway sert des fichiers statiques depuis `dist/control-ui`. Générez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances de l'UI lors de la première exécution
```

Base absolue optionnelle (si vous souhaitez des URL de ressources fixes) :

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Pour le développement local (serveur de développement séparé) :

```bash
pnpm ui:dev # installe automatiquement les dépendances de l'UI au premier lancement
```

Ensuite, dans l&#39;UI, renseignez l&#39;URL WS de votre Gateway (par exemple `ws://127.0.0.1:18789`).

<div id="debuggingtesting-dev-server-remote-gateway">
  ## Débogage/tests : serveur de développement + Gateway distante
</div>

La Control UI se compose de fichiers statiques ; la cible WebSocket est configurable et peut
différer de l’origine HTTP. C’est pratique lorsque vous souhaitez exécuter le serveur de développement Vite
en local tandis que la Gateway tourne ailleurs.

1. Démarrez le serveur de développement de l’UI : `pnpm ui:dev`
2. Ouvrez une URL de ce type :

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Authentification facultative, à effectuer une seule fois (si nécessaire) :

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Remarques :

* `gatewayUrl` est stocké dans localStorage après le chargement et supprimé de l’URL.
* `token` est stocké dans localStorage ; `password` est conservé uniquement en mémoire.
* Utilisez `wss://` lorsque le Gateway est derrière TLS (Tailscale Serve, proxy HTTPS, etc.).

Détails de configuration pour l’accès distant : [Accès distant](/fr/gateway/remote).
