---
title: Navigateur
summary: "Référence CLI pour `openclaw browser` (profils, onglets, actions, relais d’extension)"
read_when:
  - Vous utilisez `openclaw browser` et souhaitez des exemples pour les tâches courantes
  - Vous voulez contrôler un navigateur qui s’exécute sur une autre machine via un nœud hôte
  - Vous voulez utiliser le relais via l’extension Chrome (attacher/détacher via le bouton de la barre d’outils)
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

Gérez le serveur de contrôle du navigateur d’OpenClaw et exécutez des actions dans le navigateur (onglets, instantanés, captures d’écran, navigation, clics, saisie).

Liens associés :

* Outil de navigation + API : [Browser tool](/fr/tools/browser)
* Relais via extension Chrome : [Chrome extension](/fr/tools/chrome-extension)

<div id="common-flags">
  ## Options courantes
</div>

* `--url <gatewayWsUrl>`: URL WebSocket du Gateway (par défaut depuis la configuration).
* `--token <token>`: jeton du Gateway (si nécessaire).
* `--timeout <ms>`: délai d&#39;expiration de la requête (en ms).
* `--browser-profile <name>`: choisir un profil de navigateur (par défaut depuis la configuration).
* `--json`: sortie exploitable par une machine (lorsque pris en charge).

<div id="quick-start-local">
  ## Démarrage rapide (local)
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## Profils
</div>

Les profils sont des configurations nommées de routage du navigateur. En pratique :

* `openclaw` : lance/s’attache à une instance Chrome dédiée gérée par OpenClaw (répertoire de données utilisateur isolé).
* `chrome` : contrôle vos onglets Chrome existants via le relais de l’extension Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Utiliser un profil spécifique :

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## Onglets
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## Instantané / capture d’écran / actions
</div>

Instantané :

```bash
openclaw browser snapshot
```

Capture d’écran :

```bash
openclaw browser screenshot
```

Naviguer / cliquer / saisir (automatisation de l’UI basée sur des références) :

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Relais par extension Chrome (via le bouton de la barre d’outils)
</div>

Ce mode permet à l’agent de contrôler un onglet Chrome existant que vous associez manuellement (il ne s’associe pas automatiquement).

Installez l’extension non empaquetée dans un chemin stable :

```bash
openclaw browser extension install
openclaw browser extension path
```

Ensuite, dans Chrome → `chrome://extensions` → activez le « Developer mode » → « Load unpacked » → sélectionnez le dossier affiché.

Guide complet : [Extension Chrome](/fr/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## Contrôle à distance du navigateur (proxy de nœud hôte)
</div>

Si le Gateway s’exécute sur une machine différente de celle du navigateur, exécutez un **nœud hôte** sur la machine qui dispose de Chrome/Brave/Edge/Chromium. Le Gateway fera transiter les actions du navigateur vers ce nœud (aucun serveur de contrôle de navigateur séparé n’est requis).

Utilisez `gateway.nodes.browser.mode` pour contrôler le routage automatique et `gateway.nodes.browser.node` pour épingler un nœud spécifique si plusieurs sont connectés.

Sécurité et configuration à distance : [Outil navigateur](/fr/tools/browser), [Accès à distance](/fr/gateway/remote), [Tailscale](/fr/gateway/tailscale), [Sécurité](/fr/gateway/security)