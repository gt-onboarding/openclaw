---
title: Authentification
summary: "Authentification des modèles : OAuth, clés d'API et setup-token"
read_when:
  - Débogage de l'authentification des modèles ou de l'expiration OAuth
  - Documentation de l'authentification ou du stockage des identifiants
---

<div id="authentication">
  # Authentification
</div>

OpenClaw prend en charge OAuth et les clés API pour les fournisseurs de modèles. Pour les comptes Anthropic, nous recommandons d’utiliser une **clé API**. Pour l’accès à Claude via un abonnement, utilisez le jeton de longue durée créé par `claude setup-token`.

Consultez [/concepts/oauth](/fr/concepts/oauth) pour le flux OAuth complet et la structure de stockage.

<div id="recommended-anthropic-setup-api-key">
  ## Configuration recommandée pour Anthropic (clé API)
</div>

Si vous utilisez directement Anthropic, utilisez une clé API.

1. Créez une clé API dans l’Anthropic Console.
2. Placez-la sur l’**hôte du Gateway** (la machine sur laquelle tourne `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Si le Gateway s’exécute sous systemd/launchd, il est préférable de placer la clé dans
   `~/.openclaw/.env` afin que le démon puisse la lire :

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Ensuite, redémarrez le démon (ou relancez votre processus Gateway) et vérifiez à nouveau :

```bash
openclaw models status
openclaw doctor
```

Si vous préférez ne pas gérer vous‑même les variables d&#39;environnement, l’assistant de configuration initiale peut stocker
les clés d’API pour que le démon puisse les utiliser : `openclaw onboard`.

Voir l’[aide](/fr/help) pour plus de détails sur l’héritage des variables d’environnement (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic : setup-token (authentification par abonnement)
</div>

Pour Anthropic, la méthode recommandée est une **clé d’API**. Si vous utilisez un abonnement Claude, le flux setup-token est également pris en charge. Exécutez-le sur la **machine hôte du Gateway** :

```bash
claude setup-token
```

Puis collez-le dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si le jeton a été créé sur une autre machine, collez-le manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Si vous rencontrez une erreur Anthropic de ce type :

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…utilisez plutôt une clé d’API Anthropic.

Saisie manuelle du jeton (n’importe quel fournisseur ; écrit `auth-profiles.json` et met à jour la configuration) :

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Vérification adaptée à l’automatisation (renvoie le code de sortie `1` si expiré/manquant, `2` si en cours d’expiration) :

```bash
openclaw models status --check
```

Les scripts d&#39;ops facultatifs (systemd/Termux) sont documentés ici :
[/automation/auth-monitoring](/fr/automation/auth-monitoring)

> `claude setup-token` nécessite un TTY interactif.

<div id="checking-model-auth-status">
  ## Vérifier le statut d&#39;authentification du modèle
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## Contrôler l’identifiant d’authentification utilisé
</div>

<div id="per-session-chat-command">
  ### Par session (commande de chat)
</div>

Utilisez `/model <alias-or-id>@<profileId>` pour verrouiller un identifiant d’authentification de fournisseur spécifique pour la session en cours (exemples d’ID de profil : `anthropic:default`, `anthropic:work`).

Utilisez `/model` (ou `/model list`) pour un sélecteur compact ; utilisez `/model status` pour la vue complète (candidats + prochain profil d’authentification, ainsi que les détails de l’endpoint du fournisseur lorsqu’il est configuré).

<div id="per-agent-cli-override">
  ### Par agent (surcharge CLI)
</div>

Définissez un ordre explicite des profils d’authentification pour un agent (enregistré dans le fichier `auth-profiles.json` de cet agent) :

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Utilisez `--agent <id>` pour cibler un agent précis ; omettez-le pour utiliser l’agent par défaut défini dans la configuration.

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="no-credentials-found">
  ### « Aucune information d’identification trouvée »
</div>

Si le profil de jeton Anthropic est manquant, exécute `claude setup-token` sur l’**hôte Gateway**, puis vérifie de nouveau :

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### Jeton arrivant à expiration/expiré
</div>

Exécutez `openclaw models status` pour identifier quel profil arrive à expiration. Si le profil
n’apparaît pas, réexécutez `claude setup-token` et collez de nouveau le jeton.

<div id="requirements">
  ## Prérequis
</div>

* Abonnement à Claude Max ou Pro (pour `claude setup-token`)
* CLI Claude Code installée (commande `claude` disponible)