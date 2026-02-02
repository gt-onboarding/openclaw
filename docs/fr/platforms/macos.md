---
title: macOS
summary: "Application compagnon OpenClaw pour macOS (barre des menus + broker du Gateway)"
read_when:
  - Implémentation de fonctionnalités de l'application macOS
  - Modification du cycle de vie du Gateway ou du relais de nœuds sur macOS
---

<div id="openclaw-macos-companion-menu-bar-gateway-broker">
  # OpenClaw macOS Companion (barre des menus + courtier du Gateway)
</div>

L’application macOS est le **compagnon de barre des menus** d’OpenClaw. Elle gère les autorisations,
se connecte au Gateway localement (via launchd ou manuellement) et expose les
capacités macOS à l’agent sous forme de nœud.

<div id="what-it-does">
  ## Ce qu’il fait
</div>

* Affiche des notifications natives et l’état dans la barre de menus.
* Gère les invites TCC (Notifications, Accessibilité, Enregistrement de l’écran, Microphone,
  Reconnaissance vocale, Automatisation/AppleScript).
* Exécute le Gateway ou s’y connecte (en local ou à distance).
* Expose des outils spécifiques à macOS (Canvas, Camera, Enregistrement de l’écran, `system.run`).
* Démarre le service hôte du nœud local en mode **remote** (launchd) et l’arrête en mode **local**.
* Héberge éventuellement **PeekabooBridge** pour l’automatisation de l’UI.
* Installe sur demande la CLI globale (`openclaw`) via npm/pnpm (bun n’est pas recommandé pour l’exécution du Gateway).

<div id="local-vs-remote-mode">
  ## Modes local et distant
</div>

* **Local** (par défaut) : l’app se connecte à un Gateway local déjà en cours d’exécution s’il est présent ;
  sinon, elle active le service launchd via `openclaw gateway install`.
* **Distant** : l’app se connecte à un Gateway via SSH/Tailscale et ne démarre jamais
  de processus Gateway local.
  L’app démarre le **service hôte de nœud** local pour que le Gateway distant puisse atteindre ce Mac.
  L’app ne lance pas le Gateway en tant que processus enfant.

<div id="launchd-control">
  ## Contrôle via launchd
</div>

L’application gère un LaunchAgent par utilisateur intitulé `bot.molt.gateway`
(ou `bot.molt.<profile>` lorsque vous utilisez `--profile`/`OPENCLAW_PROFILE` ; les anciens `com.openclaw.*` sont toujours déchargés).

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Remplacez le libellé par `bot.molt.&lt;profile&gt;` lorsque vous exécutez un profil nommé.

Si le LaunchAgent n’est pas installé, activez-le depuis l’application ou exécutez
`openclaw gateway install`.

<div id="node-capabilities-mac">
  ## Capacités du nœud (mac)
</div>

L’app macOS se présente comme un nœud. Commandes courantes :

* Canvas : `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
* Camera : `camera.snap`, `camera.clip`
* Screen : `screen.record`
* System : `system.run`, `system.notify`

Le nœud signale une map `permissions` pour que les agents puissent décider ce qui est autorisé.

Service de nœud + IPC de l’app :

* Quand le service hôte de nœud headless est en cours d’exécution (mode distant), il se connecte au WS du Gateway en tant que nœud.
* `system.run` s’exécute dans l’app macOS (contexte UI/TCC) via un socket Unix local ; les invites et la sortie restent dans l’app.

Schéma (SCI) :

```
Gateway -> Service Nœud (WS)
                 |  IPC (UDS + jeton + HMAC + TTL)
                 v
             App Mac (UI + TCC + system.run)
```

<div id="exec-approvals-systemrun">
  ## Approbations d’exécution (system.run)
</div>

`system.run` est contrôlé par les **Approbations d’exécution** dans l’app macOS (Settings → Exec approvals).
Les paramètres Security, Ask et l’allowlist sont stockés localement sur le Mac dans :

```
~/.openclaw/exec-approvals.json
```

Exemple :

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

Notes :

* Les entrées de `allowlist` sont des modèles glob appliqués aux chemins binaires résolus.
* Choisir « Toujours autoriser » dans la boîte de dialogue ajoute cette commande à la liste d’autorisation.
* Les surcharges d’environnement `system.run` sont filtrées (suppression de `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`), puis fusionnées avec l’environnement de l’application.

<div id="deep-links">
  ## Liens profonds
</div>

L’application enregistre le schéma d’URL `openclaw://` pour des actions locales.

<div id="openclawagent">
  ### `openclaw://agent`
</div>

Déclenche une requête `agent` auprès du Gateway.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Paramètres de requête :

* `message` (obligatoire)
* `sessionKey` (facultatif)
* `thinking` (facultatif)
* `deliver` / `to` / `channel` (facultatif)
* `timeoutSeconds` (facultatif)
* `key` (clé pour le mode non supervisé, facultative)

Sécurité :

* Sans `key`, l’application demande une confirmation.
* Avec une `key` valide, l’exécution est non supervisée (prévue pour les automatisations personnelles).

<div id="onboarding-flow-typical">
  ## Processus d’onboarding (typique)
</div>

1. Installez et lancez **OpenClaw.app**.
2. Terminez la liste de contrôle des autorisations (prompts TCC).
3. Vérifiez que le mode **Local** est actif et que le Gateway est en cours d’exécution.
4. Installez la CLI si vous voulez un accès au terminal.

<div id="build-dev-workflow-native">
  ## Workflow de build et de développement (natif)
</div>

* `cd apps/macos && swift build`
* `swift run OpenClaw` (ou Xcode)
* Créer le package de l’app : `scripts/package-mac-app.sh`

<div id="debug-gateway-connectivity-macos-cli">
  ## Déboguer la connectivité du Gateway (CLI macOS)
</div>

Utilisez la CLI de débogage pour tester le même processus de handshake WebSocket et la même logique de découverte du Gateway que ceux utilisés par l’application macOS, sans lancer cette dernière.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Options de connexion :

* `--url <ws://host:port>` : remplace la configuration
* `--mode <local|remote>` : détermine le mode à partir de la configuration (par défaut : config ou local)
* `--probe` : force un nouveau test d’intégrité
* `--timeout <ms>` : délai d’expiration de la requête (par défaut : `15000`)
* `--json` : sortie structurée pour effectuer des diff

Options de découverte :

* `--include-local` : inclut les Gateway qui seraient filtrées comme « local »
* `--timeout <ms>` : fenêtre globale de découverte (par défaut : `2000`)
* `--json` : sortie structurée pour effectuer des diff

Astuce : comparez avec `openclaw gateway discover --json` pour voir si le
pipeline de découverte de l’app macOS (NWBrowser + solution de repli DNS‑SD tailnet) diffère de
la découverte basée sur `dns-sd` du CLI Node.

<div id="remote-connection-plumbing-ssh-tunnels">
  ## Infrastructure de connexion distante (tunnels SSH)
</div>

Lorsque l’app macOS s’exécute en mode **Remote**, elle ouvre un tunnel SSH afin que les composants de l’UI locale puissent communiquer avec le Gateway distant comme s’il était sur localhost.

<div id="control-tunnel-gateway-websocket-port">
  ### Tunnel de contrôle (port WebSocket du Gateway)
</div>

* **Objectif :** vérifications de santé, statut, Web Chat, configuration et autres appels du plan de contrôle.
* **Port local :** le port du Gateway (par défaut `18789`), toujours stable.
* **Port distant :** le même port du Gateway sur l&#39;hôte distant.
* **Comportement :** aucun port local aléatoire ; l’app réutilise un tunnel sain existant
  ou le redémarre si nécessaire.
* **Commande SSH :** `ssh -N -L <local>:127.0.0.1:<remote>` avec les options BatchMode +
  ExitOnForwardFailure + keepalive.
* **Remontée de l&#39;IP :** le tunnel SSH utilise l’interface de boucle locale, donc le Gateway verra l’IP du nœud
  comme `127.0.0.1`. Utilisez le transport **Direct (ws/wss)** si vous voulez que la véritable IP
  du client apparaisse (voir [accès distant macOS](/fr/platforms/mac/remote)).

Pour les étapes de configuration, voir [accès distant macOS](/fr/platforms/mac/remote). Pour les détails
du protocole, voir [protocole du Gateway](/fr/gateway/protocol).

<div id="related-docs">
  ## Documentation associée
</div>

* [Runbook du Gateway](/fr/gateway)
* [Gateway (macOS)](/fr/platforms/mac/bundled-gateway)
* [Autorisations macOS](/fr/platforms/mac/permissions)
* [Canvas](/fr/platforms/mac/canvas)