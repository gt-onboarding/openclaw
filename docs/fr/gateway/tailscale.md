---
title: Tailscale
summary: "Intégration Tailscale Serve/Funnel pour le tableau de bord du Gateway"
read_when:
  - Exposer l'interface Control UI du Gateway au-delà de localhost
  - Automatiser l'accès au tailnet ou au tableau de bord public
---

<div id="tailscale-gateway-dashboard">
  # Tailscale (tableau de bord du Gateway)
</div>

OpenClaw peut configurer automatiquement Tailscale **Serve** (tailnet) ou **Funnel** (public) pour le
tableau de bord du Gateway et le port WebSocket. Cela permet de garder le Gateway restreint à l'interface de bouclage tout en laissant
Tailscale fournir HTTPS, le routage et, pour Serve, les en-têtes d'identité.

<div id="modes">
  ## Modes
</div>

- `serve` : Serve uniquement sur le Tailnet via `tailscale serve`. Le Gateway reste sur `127.0.0.1`.
- `funnel` : HTTPS public via `tailscale funnel`. OpenClaw nécessite un mot de passe partagé.
- `off` : Valeur par défaut (aucune automatisation Tailscale).

<div id="auth">
  ## Authentification
</div>

Définissez `gateway.auth.mode` pour contrôler le handshake :

- `token` (valeur par défaut lorsque `OPENCLAW_GATEWAY_TOKEN` est défini)
- `password` (secret partagé via `OPENCLAW_GATEWAY_PASSWORD` ou la configuration)

Quand `tailscale.mode = "serve"` et que `gateway.auth.allowTailscale` est `true`,
les requêtes proxy Serve valides peuvent s’authentifier via les en-têtes
d’identité Tailscale (`tailscale-user-login`) sans fournir de token ni de mot de passe.
OpenClaw vérifie l’identité en résolvant l’adresse `x-forwarded-for` via le
daemon Tailscale local (`tailscale whois`) et en la faisant correspondre à
l’en-tête avant de l’accepter. OpenClaw ne considère une requête comme Serve
que lorsqu’elle arrive depuis l’interface loopback avec les en-têtes
`x-forwarded-for`, `x-forwarded-proto` et `x-forwarded-host` de Tailscale.
Pour exiger des identifiants explicites, définissez
`gateway.auth.allowTailscale: false` ou forcez `gateway.auth.mode: "password"`.

<div id="config-examples">
  ## Exemples de configuration
</div>

<div id="tailnet-only-serve">
  ### Tailnet uniquement (Serve)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Ouvrez : `https://<magicdns>/` (ou le `gateway.controlUi.basePath` que vous avez configuré)


<div id="tailnet-only-bind-to-tailnet-ip">
  ### Uniquement Tailnet (liaison à l’IP Tailnet)
</div>

Utilisez cette option lorsque vous voulez que le Gateway écoute directement sur l’adresse IP Tailnet (sans Serve/Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

Connectez-vous à partir d’un autre appareil sur le Tailnet :

* Control UI : `http://<tailscale-ip>:18789/`
* WebSocket : `ws://<tailscale-ip>:18789`

Remarque : le loopback (`http://127.0.0.1:18789`) **ne** fonctionnera **pas** dans ce mode.


<div id="public-internet-funnel-shared-password">
  ### Internet public (Funnel + mot de passe partagé)
</div>

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

Préférez la variable d’environnement `OPENCLAW_GATEWAY_PASSWORD` plutôt qu’enregistrer un mot de passe sur le disque.


<div id="cli-examples">
  ## Exemples de commandes CLI
</div>

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```


<div id="notes">
  ## Notes
</div>

- Tailscale Serve/Funnel nécessite que la CLI `tailscale` soit installée et connectée.
- `tailscale.mode: "funnel"` refuse de démarrer sauf si le mode d’authentification est `password`, afin d’éviter une exposition publique.
- Paramétrez `gateway.tailscale.resetOnExit` si vous voulez qu’OpenClaw annule la configuration
  `tailscale serve` ou `tailscale funnel` à l’arrêt.
- `gateway.bind: "tailnet"` est une liaison Tailnet directe (pas de HTTPS, pas de Serve/Funnel).
- `gateway.bind: "auto"` privilégie l’interface loopback ; utilisez `tailnet` si vous voulez un accès limité à Tailnet.
- Serve/Funnel n’exposent que la **Control UI du Gateway + WS**. Les nœuds se connectent via
  le même point de terminaison WS du Gateway, donc Serve peut fonctionner pour l’accès aux nœuds.

<div id="browser-control-remote-gateway-local-browser">
  ## Contrôle du navigateur (Gateway distant + navigateur local)
</div>

Si vous exécutez le Gateway sur une machine mais que vous souhaitez contrôler un navigateur sur une autre machine,
exécutez un **hôte de nœud** sur la machine du navigateur et gardez les deux sur le même tailnet.
Le Gateway servira de proxy pour les actions du navigateur vers le nœud ; aucun serveur de contrôle séparé ni URL Serve n’est nécessaire.

Évitez d’utiliser Funnel pour le contrôle du navigateur ; traitez l’appairage du nœud comme un accès d’opérateur.

<div id="tailscale-prerequisites-limits">
  ## Prérequis et limites Tailscale
</div>

- Serve nécessite l’activation de HTTPS pour votre tailnet ; la CLI vous le signalera s’il n’est pas activé.
- Serve injecte des en-têtes d’identité Tailscale ; Funnel ne le fait pas.
- Funnel nécessite Tailscale v1.38.3+, MagicDNS, HTTPS activé et un attribut de nœud « funnel ».
- Funnel ne prend en charge que les ports `443`, `8443` et `10000` sur TLS.
- Funnel sur macOS nécessite la variante open source de l’application Tailscale.

<div id="learn-more">
  ## En savoir plus
</div>

- Présentation de Tailscale Serve : https://tailscale.com/kb/1312/serve
- Commande `tailscale serve` : https://tailscale.com/kb/1242/tailscale-serve
- Présentation de Tailscale Funnel : https://tailscale.com/kb/1223/tailscale-funnel
- Commande `tailscale funnel` : https://tailscale.com/kb/1311/tailscale-funnel