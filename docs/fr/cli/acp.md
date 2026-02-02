---
title: Acp
summary: "Exécuter le bridge ACP pour les intégrations IDE"
read_when:
  - Configurer des intégrations IDE basées sur ACP
  - Déboguer le routage des sessions ACP vers le Gateway
---

<div id="acp">
  # acp
</div>

Exécute le pont ACP (Agent Client Protocol) qui communique avec la Gateway OpenClaw.

Cette commande utilise ACP via stdio pour les IDE et transfère les invites à la Gateway
via WebSocket. Elle maintient la correspondance entre les sessions ACP et les clés de session de la Gateway.

<div id="usage">
  ## Utilisation
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# Réinitialiser la clé de session avant le premier prompt
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## Client ACP (debug)
</div>

Utilisez le client ACP intégré pour vérifier rapidement le bridge sans IDE.
Il démarre le bridge ACP et vous permet de saisir des requêtes de manière interactive.

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# Remplacer la commande serveur (par défaut : openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="how-to-use-this">
  ## Comment l&#39;utiliser
</div>

Utilisez ACP lorsqu&#39;un IDE (ou un autre client) prend en charge Agent Client Protocol et que vous voulez
qu&#39;il pilote une session du Gateway OpenClaw.

1. Assurez-vous que le Gateway est en cours d&#39;exécution (en local ou à distance).
2. Configurez la cible du Gateway (fichier de config ou options).
3. Configurez votre IDE pour exécuter `openclaw acp` via stdio.

Exemple de configuration (persistante) :

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Exemple d’exécution directe (sans modification de la configuration) :

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## Sélection des agents
</div>

ACP ne sélectionne pas directement les agents. Il effectue le routage via la clé de session du Gateway.

Utilisez des clés de session à portée d’agent pour cibler un agent spécifique :

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Chaque session ACP est associée à une seule clé de session Gateway. Un agent peut avoir de nombreuses
sessions ; par défaut, ACP utilise une session isolée `acp:<uuid>` sauf si vous modifiez
la clé ou le libellé.

<div id="zed-editor-setup">
  ## Configuration de l’éditeur Zed
</div>

Ajoutez un agent ACP personnalisé dans `~/.config/zed/settings.json` (ou utilisez l’UI de configuration de Zed) :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Pour cibler un Gateway ou un agent en particulier :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Dans Zed, ouvrez le panneau Agent et sélectionnez « OpenClaw ACP » pour créer un fil de discussion.

<div id="session-mapping">
  ## Correspondance des sessions
</div>

Par défaut, les sessions ACP obtiennent une clé de session Gateway isolée avec un préfixe `acp:`.
Pour réutiliser une session connue, passez une clé ou un libellé de session :

* `--session <key>` : utiliser une clé de session Gateway spécifique.
* `--session-label <label>` : retrouver une session existante par son libellé.
* `--reset-session` : générer un nouvel identifiant de session pour cette clé (même clé, nouvel historique).

Si votre client ACP prend en charge les métadonnées, vous pouvez les surcharger session par session :

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "boîte de réception du support",
    "resetSession": true
  }
}
```

Pour en savoir plus sur les clés de session, consultez [/concepts/session](/fr/concepts/session).

<div id="options">
  ## Options
</div>

* `--url <url>`: URL WebSocket du Gateway (par défaut `gateway.remote.url` lorsqu&#39;il est configuré).
* `--token <token>`: jeton d&#39;authentification du Gateway.
* `--password <password>`: mot de passe d&#39;authentification du Gateway.
* `--session <key>`: clé de session par défaut.
* `--session-label <label>`: libellé de session par défaut à utiliser pour la résolution.
* `--require-existing`: renvoie une erreur si la clé ou le libellé de session n&#39;existe pas.
* `--reset-session`: réinitialise la clé de session avant la première utilisation.
* `--no-prefix-cwd`: ne préfixe pas les prompts par le répertoire de travail.
* `--verbose, -v`: journalisation détaillée sur stderr.

<div id="acp-client-options">
  ### Options de `acp client`
</div>

* `--cwd <dir>`: répertoire de travail pour la session ACP.
* `--server <command>`: commande du serveur ACP (par défaut : `openclaw`).
* `--server-args <args...>`: arguments supplémentaires transmis au serveur ACP.
* `--server-verbose`: activer le mode verbeux sur le serveur ACP.
* `--verbose, -v`: activer le mode verbeux côté client.