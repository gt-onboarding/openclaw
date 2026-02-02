---
title: Proxy d'API Claude Max
summary: "Utiliser votre abonnement Claude Max/Pro comme point de terminaison d'API compatible OpenAI"
read_when:
  - Vous souhaitez utiliser un abonnement Claude Max avec des outils compatibles OpenAI
  - Vous souhaitez un serveur API local qui encapsule la CLI Claude Code
  - Vous souhaitez économiser de l'argent en utilisant un abonnement plutôt que des clés d'API
---

<div id="claude-max-api-proxy">
  # Proxy d’API Claude Max
</div>

**claude-max-api-proxy** est un outil communautaire qui expose votre abonnement Claude Max/Pro sous la forme d’un point de terminaison d’API compatible avec OpenAI. Cela vous permet d’utiliser votre abonnement avec n’importe quel outil qui prend en charge le format d’API OpenAI.

<div id="why-use-this">
  ## Pourquoi utiliser ceci ?
</div>

| Approche | Coût | Idéal pour |
|----------|------|------------|
| Anthropic API | Paiement à l&#39;utilisation (~15 $ / M en entrée, 75 $ / M en sortie pour Opus) | Applications de production, volume élevé |
| Abonnement Claude Max | Forfait de 200 $/mois | Usage personnel, développement, utilisation illimitée |

Si vous avez un abonnement Claude Max et que vous souhaitez l&#39;utiliser avec des outils compatibles OpenAI, ce proxy peut vous permettre d&#39;économiser beaucoup d&#39;argent.

<div id="how-it-works">
  ## Fonctionnement
</div>

```
Votre application → claude-max-api-proxy → Claude Code CLI → Anthropic (via abonnement)
     (format OpenAI)                (convertit le format)      (utilise vos identifiants)
```

Le proxy :

1. Accepte les requêtes au format OpenAI à l’adresse `http://localhost:3456/v1/chat/completions`
2. Les convertit en commandes CLI Claude Code
3. Renvoie les réponses au format OpenAI (streaming pris en charge)

<div id="installation">
  ## Installation
</div>

```bash
# Nécessite Node.js 20+ et l'interface CLI Claude Code
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

<div id="usage">
  ## Utilisation
</div>

<div id="start-the-server">
  ### Démarrez le serveur
</div>

```bash
claude-max-api
# Le serveur s'exécute à l'adresse http://localhost:3456
```

<div id="test-it">
  ### Tester
</div>

```bash
# Vérification de l'état de santé
curl http://localhost:3456/health

# Lister les modèles
curl http://localhost:3456/v1/models

# Complétion de conversation
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

<div id="with-openclaw">
  ### Avec OpenClaw
</div>

Vous pouvez configurer OpenClaw pour utiliser le proxy en tant que point de terminaison personnalisé compatible avec OpenAI :

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1"
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" }
    }
  }
}
```

<div id="available-models">
  ## Modèles disponibles
</div>

| Identifiant du modèle | Correspond au modèle |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## Démarrage automatique sous macOS
</div>

Créez un LaunchAgent pour exécuter le proxy automatiquement :

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

<div id="links">
  ## Liens
</div>

* **npm :** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub :** https://github.com/atalovesyou/claude-max-api-proxy
* **Tickets :** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## Remarques
</div>

* Il s’agit d’un **outil développé par la communauté**, non officiellement pris en charge par Anthropic ou OpenClaw
* Nécessite un abonnement actif Claude Max/Pro avec Claude Code CLI déjà authentifié
* Le proxy s’exécute localement et n’envoie aucune donnée à des serveurs tiers
* Les réponses en streaming sont entièrement prises en charge

<div id="see-also">
  ## Voir aussi
</div>

* [fournisseur Anthropic](/fr/providers/anthropic) - Intégration native OpenClaw pour Claude via setup-token ou clés api
* [fournisseur OpenAI](/fr/providers/openai) - Pour les abonnements OpenAI/Codex