---
title: Accès à distance
summary: "Accès à distance via des tunnels SSH (WS du Gateway) et des tailnets"
read_when:
  - Utilisation ou dépannage de configurations de Gateway distantes
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # Accès à distance (SSH, tunnels et tailnets)
</div>

Ce dépôt prend en charge le « remote over SSH » en maintenant un seul Gateway (le principal) en fonctionnement sur un hôte dédié (ordinateur de bureau / serveur) et en y connectant les clients.

* Pour **les opérateurs (vous / l’app macOS)** : le tunneling SSH est la solution de repli universelle.
* Pour **les nœuds (iOS/Android et futurs appareils)** : connectez-vous au **WebSocket** du Gateway (LAN/tailnet ou tunnel SSH selon les besoins).

<div id="the-core-idea">
  ## L’idée centrale
</div>

* La connexion WebSocket du Gateway est liée à l’interface **loopback** sur le port que vous avez configuré (18789 par défaut).
* Pour une utilisation à distance, vous redirigez ce port loopback via SSH (ou vous utilisez un tailnet/VPN et réduisez le besoin de tunneling).

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## Configurations VPN/tailnet courantes (où l’agent s’exécute)
</div>

Considérez l’**hôte Gateway** comme « l’endroit où l’agent vit ». Il détient les sessions, les profils d’authentification, les canaux et l’état.
Votre ordinateur portable/ordinateur de bureau (et les nœuds) se connectent à cet hôte.

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) Gateway toujours actif dans votre tailnet (VPS ou serveur domestique)
</div>

Exécutez Gateway sur un hôte persistant et accédez-y via **Tailscale** ou SSH.

* **Meilleure UX :** conservez `gateway.bind: "loopback"` et utilisez **Tailscale Serve** pour le Control UI.
* **Solution de repli :** conservez loopback + un tunnel SSH depuis toute machine qui a besoin d’un accès.
* **Exemples :** [exe.dev](/fr/platforms/exe-dev) (VM simple) ou [Hetzner](/fr/platforms/hetzner) (VPS de production).

C’est idéal si votre ordinateur portable se met souvent en veille mais que vous voulez que l’agent reste toujours actif.

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2) L’ordinateur de bureau à la maison exécute le Gateway, l’ordinateur portable sert de télécommande
</div>

L’ordinateur portable n’exécute **pas** l’agent. Il se connecte à distance :

* Utilisez le mode **Remote over SSH** de l’app macOS (Réglages → Général → « OpenClaw runs »).
* L’app ouvre et gère le tunnel, donc WebChat et les contrôles d’intégrité fonctionnent simplement.

Runbook : [Accès distant macOS](/fr/platforms/mac/remote).

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) L’ordinateur portable exécute le Gateway, accès distant depuis d’autres machines
</div>

Conserve le Gateway en local mais expose-le de façon sécurisée :

* Tunnel SSH vers l’ordinateur portable depuis d’autres machines, ou
* Utilise Tailscale Serve pour exposer le Control UI et limite le Gateway à l’interface loopback uniquement.

Guide : [Tailscale](/fr/gateway/tailscale) et [Vue d’ensemble du Web](/fr/web).

<div id="command-flow-what-runs-where">
  ## Flux de commandes (où s’exécute quoi)
</div>

Un service Gateway détient l’état et les canaux. Les nœuds sont des périphériques.

Exemple de flux (Telegram → nœud) :

* Un message Telegram parvient au **Gateway**.
* Gateway exécute l’**agent** et décide s’il doit appeler un outil du nœud.
* Gateway appelle le **nœud** via le WebSocket du Gateway (`node.*` RPC).
* Le nœud renvoie le résultat ; Gateway répond ensuite à Telegram.

Remarques :

* **Les nœuds n’exécutent pas le service Gateway.** Un seul Gateway doit s’exécuter par hôte, sauf si vous lancez intentionnellement des profils isolés (voir [Multiple gateways](/fr/gateway/multiple-gateways)).
* Le mode « nœud » de l’app macOS est simplement un client nœud via le WebSocket du Gateway.

<div id="ssh-tunnel-cli-tools">
  ## Tunnel SSH (CLI + outils)
</div>

Créez un tunnel local vers le WS du Gateway distant :

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Avec le tunnel actif :

* `openclaw health` et `openclaw status --deep` atteignent désormais le Gateway distant via `ws://127.0.0.1:18789`.
* `openclaw gateway {status,health,send,agent,call}` peut également cibler l’URL redirigée via `--url` si nécessaire.

Remarque : remplacez `18789` par votre `gateway.port` configuré (ou `--port`/`OPENCLAW_GATEWAY_PORT`).

<div id="cli-remote-defaults">
  ## Valeurs par défaut du CLI distant
</div>

Vous pouvez enregistrer une cible distante pour que les commandes CLI l&#39;utilisent par défaut :

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

Lorsque le Gateway est limité à la boucle locale, conservez l’URL `ws://127.0.0.1:18789` et ouvrez d’abord le tunnel SSH.

<div id="chat-ui-over-ssh">
  ## UI de chat via SSH
</div>

WebChat n’utilise plus de port HTTP distinct. L’UI de chat SwiftUI se connecte directement au WebSocket du Gateway.

* Faites passer le port `18789` en tunnel SSH (voir ci-dessus), puis connectez les clients à `ws://127.0.0.1:18789`.
* Sous macOS, privilégiez le mode « Remote over SSH » de l’application, qui gère automatiquement le tunnel.

<div id="macos-app-remote-over-ssh">
  ## App macOS « Remote over SSH »
</div>

L’app macOS de barre de menus peut piloter la même configuration de bout en bout (contrôles d’état à distance, WebChat et redirection de Voice Wake).

Runbook : [Accès distant macOS](/fr/platforms/mac/remote).

<div id="security-rules-remotevpn">
  ## Règles de sécurité (accès distant/VPN)
</div>

Version courte : **gardez le Gateway accessible uniquement en loopback** sauf si vous êtes sûr d’avoir besoin d’un bind.

* **Loopback + SSH/Tailscale Serve** est la configuration par défaut la plus sûre (aucune exposition publique).
* Les **binds non-loopback** (`lan`/`tailnet`/`custom`, ou `auto` quand le loopback n’est pas disponible) doivent obligatoirement utiliser des jetons/mots de passe d’authentification.
* `gateway.remote.token` est **uniquement** destiné aux appels CLI distants — il **n’active pas** l’authentification locale.
* `gateway.remote.tlsFingerprint` épingle le certificat TLS distant lors de l’utilisation de `wss://`.
* **Tailscale Serve** peut assurer l’authentification via des en-têtes d’identité quand `gateway.auth.allowTailscale: true`.
  Mettez ce paramètre à `false` si vous préférez les jetons/mots de passe.
* Traitez le contrôle depuis le navigateur comme un accès opérateur : tailnet uniquement + appairage explicite de nœud.

Analyse détaillée : [Sécurité](/fr/gateway/security).