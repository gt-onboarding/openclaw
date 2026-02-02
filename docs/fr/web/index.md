---
title: Web
summary: "Interfaces Web du Gateway : Control UI, modes de liaison et sécurité"
read_when:
  - Vous souhaitez accéder au Gateway via Tailscale
  - Vous souhaitez utiliser le Control UI dans le navigateur et modifier la configuration
---

<div id="web-gateway">
  # Web (Gateway)
</div>

Le Gateway sert une petite **Control UI pour navigateur** (Vite + Lit) depuis le même port que le WebSocket du Gateway :

* par défaut : `http://<host>:18789/`
* préfixe optionnel : définir `gateway.controlUi.basePath` (par ex. `/openclaw`)

Les fonctionnalités sont accessibles dans [Control UI](/fr/web/control-ui).
Cette page se concentre sur les modes de bind, la sécurité et les surfaces exposées sur le web.

<div id="webhooks">
  ## Webhooks
</div>

Lorsque `hooks.enabled=true`, le Gateway expose également un petit point de terminaison webhook sur le même serveur HTTP.
Voir [Gateway configuration](/fr/gateway/configuration) → `hooks` pour l’authentification et les payloads.

<div id="config-default-on">
  ## Config (activée par défaut)
</div>

La Control UI est **activée par défaut** lorsque les ressources statiques sont présentes (`dist/control-ui`).
Vous pouvez la gérer via la configuration :

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" } // basePath facultatif
  }
}
```

<div id="tailscale-access">
  ## Accès via Tailscale
</div>

<div id="integrated-serve-recommended">
  ### Serve intégré (recommandé)
</div>

Laissez le Gateway sur l’interface loopback et laissez Tailscale Serve agir comme proxy :

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Ensuite, démarrez Gateway :

```bash
openclaw gateway
```

Ouvrez :

* `https://<magicdns>/` (ou le `gateway.controlUi.basePath` que vous avez configuré)

<div id="tailnet-bind-token">
  ### Liaison au tailnet + jeton
</div>

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

Ensuite, démarrez le Gateway (token requis pour les liaisons réseau non-loopback) :

```bash
openclaw gateway
```

Ouvrez :

* `http://<tailscale-ip>:18789/` (ou le `gateway.controlUi.basePath` que vous avez configuré)

<div id="public-internet-funnel">
  ### Internet public (Funnel)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // ou OPENCLAW_GATEWAY_PASSWORD
  }
}
```

<div id="security-notes">
  ## Notes de sécurité
</div>

* L’authentification du Gateway est requise par défaut (jeton/mot de passe ou en-têtes d’identité Tailscale).
* Les liaisons non loopback **nécessitent** toujours un jeton/mot de passe partagé (`gateway.auth` ou variables d’environnement).
* L’assistant génère un jeton du Gateway par défaut (même sur loopback).
* L’UI envoie `connect.params.auth.token` ou `connect.params.auth.password`.
* Avec Serve, les en-têtes d’identité Tailscale peuvent tenir lieu d’authentification lorsque
  `gateway.auth.allowTailscale` est `true` (aucun jeton/mot de passe requis). Définissez
  `gateway.auth.allowTailscale: false` pour exiger des identifiants explicites. Voir
  [Tailscale](/fr/gateway/tailscale) et [Sécurité](/fr/gateway/security).
* `gateway.tailscale.mode: "funnel"` nécessite `gateway.auth.mode: "password"` (mot de passe partagé).

<div id="building-the-ui">
  ## Générer l’UI
</div>

Gateway sert des fichiers statiques à partir de `dist/control-ui`. Générez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances de l'UI lors de la première exécution
```
