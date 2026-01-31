---
title: Accès distant
summary: "Flux de l’app macOS permettant de contrôler un Gateway OpenClaw distant via SSH"
read_when:
  - Pour configurer ou déboguer le contrôle à distance depuis macOS
---

<div id="remote-openclaw-macos-remote-host">
  # OpenClaw distant (macOS ⇄ hôte distant)
</div>

Ce mode permet à l’application macOS d’agir comme une télécommande complète pour un Gateway OpenClaw qui s’exécute sur un autre hôte (poste de travail/serveur). Il s’agit de la fonctionnalité **Remote over SSH** (exécution distante) de l’application. Toutes les fonctionnalités — contrôles d’intégrité, transfert Voice Wake et Web Chat — réutilisent la même configuration SSH distante définie dans *Settings → General*.

<div id="modes">
  ## Modes
</div>

* **Local (ce Mac)** : Tout s’exécute sur cet ordinateur. Aucune connexion SSH.
* **Distant via SSH (par défaut)** : Les commandes OpenClaw sont exécutées sur l’hôte distant. L’app macOS ouvre une connexion SSH avec `-o BatchMode`, l’identité/la clé de votre choix et un transfert de port local.
* **Distant direct (ws/wss)** : Aucun tunnel SSH. L’app macOS se connecte directement à l’URL du Gateway (par exemple via Tailscale Serve ou un reverse proxy HTTPS public).

<div id="remote-transports">
  ## Transports distants
</div>

Le mode distant prend en charge deux types de transport :

* **Tunnel SSH** (par défaut) : utilise `ssh -N -L ...` pour transférer le port du Gateway vers localhost. Le Gateway verra l’adresse IP du nœud comme `127.0.0.1`, car le tunnel passe par l’interface de loopback.
* **Direct (ws/wss)** : se connecte directement à l’URL du Gateway. Le Gateway voit la véritable adresse IP du client.

<div id="prereqs-on-the-remote-host">
  ## Prérequis sur l&#39;hôte distant
</div>

1. Installez Node.js et pnpm, puis générez/installez l’outil CLI OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2. Vérifiez que `openclaw` est présent dans le PATH pour les shells non interactifs (créez un lien symbolique vers `/usr/local/bin` ou `/opt/homebrew/bin` si nécessaire).
3. Configurez SSH avec authentification par clé. Nous recommandons d’utiliser des IP **Tailscale** pour une connectivité stable en dehors du LAN.

<div id="macos-app-setup">
  ## Configuration de l’app macOS
</div>

1. Ouvrez *Settings → General*.
2. Sous **OpenClaw runs**, choisissez **Remote over SSH** et configurez :
   * **Transport** : **SSH tunnel** ou **Direct (ws/wss)**.
   * **SSH target** : `user@host` (`:port` facultatif).
     * Si le Gateway est sur le même LAN et s’annonce via Bonjour, sélectionnez-le dans la liste des services découverts pour renseigner automatiquement ce champ.
   * **Gateway URL** (Direct uniquement) : `wss://gateway.example.ts.net` (ou `ws://...` pour le réseau local/LAN).
   * **Identity file** (avancé) : chemin vers votre clé.
   * **Project root** (avancé) : chemin du dépôt distant utilisé pour les commandes.
   * **CLI path** (avancé) : chemin optionnel vers un binaire/point d’entrée `openclaw` exécutable (renseigné automatiquement lorsqu’il est annoncé).
3. Cliquez sur **Test remote**. En cas de succès, cela indique que `openclaw status --json` s’exécute correctement à distance. Les échecs signalent généralement des problèmes de PATH/CLI ; le code de sortie 127 indique que la CLI n’est pas trouvée sur l’hôte distant.
4. Les contrôles d’intégrité (health checks) et le Web Chat passeront désormais automatiquement par ce tunnel SSH.

<div id="web-chat">
  ## Web Chat
</div>

* **Tunnel SSH** : Web Chat se connecte au Gateway via le port de contrôle WS (WebSocket) redirigé (18789 par défaut).
* **Direct (ws/wss)** : Web Chat se connecte directement à l’URL du Gateway configuré.
* Il n’existe plus de serveur HTTP WebChat distinct.

<div id="permissions">
  ## Autorisations
</div>

* L’hôte distant doit disposer des mêmes autorisations TCC que l’hôte local (Automatisation, Accessibilité, Enregistrement de l’écran, Microphone, Reconnaissance vocale, Notifications). Exécute la procédure d’onboarding sur cette machine pour les accorder une fois.
* Les nœuds exposent leur état d’autorisation via `node.list` / `node.describe` afin que les agents sachent ce qui est disponible.

<div id="security-notes">
  ## Notes de sécurité
</div>

* Privilégiez l’écoute sur l’interface loopback de l’hôte distant et connectez-vous via SSH ou Tailscale.
* Si vous faites écouter le Gateway sur une interface non-loopback, imposez une authentification par jeton ou mot de passe.
* Voir [Sécurité](/fr/gateway/security) et [Tailscale](/fr/gateway/tailscale).

<div id="whatsapp-login-flow-remote">
  ## Processus de connexion WhatsApp (distant)
</div>

* Exécutez `openclaw channels login --verbose` **sur l&#39;hôte distant**. Scannez le code QR avec WhatsApp sur votre téléphone.
* Relancez la connexion sur cet hôte si l’authentification expire. La vérification d’intégrité mettra en évidence les problèmes de lien.

<div id="troubleshooting">
  ## Dépannage
</div>

* **exit 127 / not found** : `openclaw` n’est pas dans le PATH pour les shells non-login. Ajoutez-le à `/etc/paths`, à votre fichier rc de shell, ou créez un lien symbolique dans `/usr/local/bin` ou `/opt/homebrew/bin`.
* **Health probe failed** : vérifiez l’accessibilité SSH, le PATH et que Baileys est connecté (`openclaw status --json`).
* **Web Chat stuck** : vérifiez que le Gateway s’exécute sur l’hôte distant et que le port transféré correspond au port WS du Gateway ; l’UI requiert une connexion WS saine.
* **Node IP shows 127.0.0.1** : comportement attendu avec le tunnel SSH. Basculez **Transport** sur **Direct (ws/wss)** si vous voulez que le Gateway voie la véritable IP cliente.
* **Voice Wake** : les phrases de déclenchement sont transmises automatiquement en mode distant ; aucun service de transfert séparé n’est nécessaire.

<div id="notification-sounds">
  ## Sons de notification
</div>

Définissez les sons de notification dans des scripts utilisant `openclaw` et `node.invoke`, par exemple :

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Gateway distant prêt" --sound Glass
```

Il n’existe plus de paramètre global de « son par défaut » dans l’app ; le son (ou son absence) est choisi requête par requête.
