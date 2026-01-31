---
title: Dépannage
summary: "Guide de dépannage rapide pour les problèmes courants d'OpenClaw"
read_when:
  - Lorsque vous enquêtez sur des problèmes ou des défaillances à l'exécution
---

<div id="troubleshooting">
  # Dépannage 🔧
</div>

Lorsque OpenClaw se comporte mal, voici comment procéder.

Commencez par la section [60 premières secondes](/fr/help/faq#first-60-seconds-if-somethings-broken) de la FAQ si vous souhaitez simplement une recette de diagnostic rapide. Cette page va plus en profondeur sur les défaillances à l’exécution et les diagnostics.

Raccourcis spécifiques aux fournisseurs : [/channels/troubleshooting](/fr/channels/troubleshooting)

<div id="status-diagnostics">
  ## État et diagnostics
</div>

Commandes de triage rapide (dans l’ordre) :

| Commande | Ce que la commande indique | Quand l’utiliser |
|---|---|---|
| `openclaw status` | Résumé local : OS + mise à jour, accessibilité/mode du Gateway, service, agents/sessions, état de configuration des fournisseurs | Premier contrôle, vue d’ensemble rapide |
| `openclaw status --all` | Diagnostic local complet (en lecture seule, partageable, relativement sûr) incluant la fin des journaux (logs) | Quand vous devez partager un rapport de débogage |
| `openclaw status --deep` | Exécute les contrôles d’intégrité du Gateway (y compris des sondes de fournisseurs ; nécessite un Gateway joignable) | Quand « configuré » ne veut pas dire « fonctionnel » |
| `openclaw gateway probe` | Découverte + accessibilité du Gateway (cibles locales + distantes) | Quand vous soupçonnez d’interroger le mauvais Gateway |
| `openclaw channels status --probe` | Demande au Gateway en cours d’exécution l’état des canaux (et peut éventuellement lancer des sondes) | Quand le Gateway est joignable mais que les canaux se comportent mal |
| `openclaw gateway status` | État du superviseur (launchd/systemd/schtasks), PID/exit du runtime, dernière erreur du Gateway | Quand le service semble démarré mais que rien ne tourne |
| `openclaw logs --follow` | Journaux (logs) en direct (meilleur signal pour les problèmes à l’exécution) | Quand vous avez besoin de la véritable raison de l’échec |

**Partage de sortie :** privilégiez `openclaw status --all` (les jetons y sont masqués). Si vous collez `openclaw status`, pensez à définir d’abord `OPENCLAW_SHOW_SECRETS=0` (pour désactiver les aperçus de jetons).

Voir aussi : [Contrôles d’intégrité](/fr/gateway/health) et [Journalisation](/fr/logging).

<div id="common-issues">
  ## Problèmes fréquents
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### Aucune clé API trouvée pour le fournisseur &quot;anthropic&quot;
</div>

Cela signifie que **le stockage d’authentification de l’agent est vide** ou que les identifiants Anthropic sont manquants.
L’authentification est **par agent**, donc un nouvel agent n’héritera pas des clés de l’agent principal.

Options de correction :

* Relancez l’onboarding et choisissez **Anthropic** pour cet agent.
* Ou collez un setup-token sur l’**hôte du Gateway** :
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* Ou copiez `auth-profiles.json` du répertoire de l’agent principal vers le nouveau répertoire d’agent.

Vérifiez :

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### Échec du rafraîchissement du jeton OAuth (abonnement Anthropic Claude)
</div>

Cela signifie que le jeton OAuth Anthropic stocké a expiré et que son rafraîchissement a échoué.
Si vous avez un abonnement Claude (sans clé API), la solution la plus fiable consiste à
passer à un **setup-token Claude Code** et à le coller sur l’**hôte du Gateway**.

**Recommandé (setup-token) :**

```bash
# Exécuter sur l'hôte du Gateway (coller le setup-token)
openclaw models auth setup-token --provider anthropic
openclaw models status
```

Si vous avez généré le jeton ailleurs :

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

Pour plus de détails : [Anthropic](/fr/providers/anthropic) et [OAuth](/fr/concepts/oauth).

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Échec du Control UI en HTTP (« device identity required » / « connect failed »)
</div>

Si vous ouvrez le tableau de bord en HTTP simple (par ex. `http://<lan-ip>:18789/` ou
`http://<tailscale-ip>:18789/`), le navigateur fonctionne dans un **contexte non sécurisé** et
bloque WebCrypto, ce qui empêche de générer l’identité de l’appareil.

**Correctif :**

* Privilégiez HTTPS via [Tailscale Serve](/fr/gateway/tailscale).
* Ou ouvrez-le localement sur l’hôte du Gateway : `http://127.0.0.1:18789/`.
* Si vous devez absolument rester en HTTP, activez `gateway.controlUi.allowInsecureAuth: true` et
  utilisez un jeton de Gateway (jeton uniquement ; aucune identité d’appareil / aucun appairage). Voir
  [Control UI](/fr/web/control-ui#insecure-http).

<div id="ci-secrets-scan-failed">
  ### L&#39;analyse des secrets CI a échoué
</div>

Cela signifie que `detect-secrets` a trouvé de nouveaux candidats qui ne figurent pas encore dans la base de référence.
Suivez la section [Analyse des secrets](/fr/gateway/security#secret-scanning-detect-secrets).

<div id="service-installed-but-nothing-is-running">
  ### Service installé mais aucun processus n’est en cours d’exécution
</div>

Si le service Gateway est installé mais que le processus se termine immédiatement, le service
peut apparaître comme « chargé » alors que rien n’est en cours d’exécution.

**À vérifier :**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/service affichera l’état d’exécution (PID/dernier arrêt) et des indications issues des logs.

**Logs :**

* Recommandé : `openclaw logs --follow`
* Logs sur fichier (toujours) : `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (ou votre `logging.file` configuré)
* LaunchAgent macOS (si installé) : `$OPENCLAW_STATE_DIR/logs/gateway.log` et `gateway.err.log`
* systemd Linux (si installé) : `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows : `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**Activer une journalisation plus détaillée :**

* Augmenter le niveau de détail des logs sur fichier (JSONL persistant) :
  ```json
  { "logging": { "level": "debug" } }
  ```
* Augmenter la verbosité de la console (sortie TTY uniquement) :
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* Astuce : `--verbose` affecte uniquement la sortie **console**. Les logs sur fichier restent contrôlés par `logging.level`.

Voir [/logging](/fr/logging) pour une présentation complète des formats, de la configuration et de l’accès.

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### « Démarrage du Gateway bloqué : définissez gateway.mode=local »
</div>

Cela signifie que la configuration existe mais que `gateway.mode` n’est pas défini (ou n’est pas `local`), donc le Gateway refuse de démarrer.

**Correction (recommandée) :**

* Exécutez l’assistant de configuration et définissez le mode d’exécution du Gateway sur **Local** :
  ```bash
  openclaw configure
  ```
* Ou définissez-le directement :
  ```bash
  openclaw config set gateway.mode local
  ```

**Si vous souhaitiez plutôt exécuter un Gateway distant :**

* Configurez une URL distante et conservez `gateway.mode=remote` :
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**Ad hoc/dev uniquement :** passez `--allow-unconfigured` pour démarrer le Gateway sans
`gateway.mode=local`.

**Pas encore de fichier de configuration ?** Exécutez `openclaw setup` pour créer une configuration de base, puis relancez
le Gateway.

<div id="service-environment-path-runtime">
  ### Environnement du service (PATH + runtime)
</div>

Le service Gateway s’exécute avec un **PATH minimal** pour éviter le bruit lié au shell et aux gestionnaires de versions :

* macOS : `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
* Linux : `/usr/local/bin`, `/usr/bin`, `/bin`

Cela exclut volontairement les gestionnaires de versions (nvm/fnm/volta/asdf) et les gestionnaires de paquets (pnpm/npm), car le service ne charge pas l’initialisation de votre shell. Les variables d’environnement d’exécution comme `DISPLAY` doivent se trouver dans `~/.openclaw/.env` (chargé très tôt par le Gateway).
Les appels `exec` exécutés sur `host=gateway` fusionnent votre `PATH` de shell de connexion dans l’environnement d’exécution ; par conséquent, si des outils manquent, cela signifie généralement que l’initialisation de votre shell ne les exporte pas (ou que vous devez définir `tools.exec.pathPrepend`). Voir [/tools/exec](/fr/tools/exec).

Les canaux WhatsApp et Telegram nécessitent **Node** ; Bun n’est pas pris en charge. Si votre service a été installé avec Bun ou avec un chemin Node géré par un gestionnaire de versions, exécutez `openclaw doctor` pour migrer vers une installation Node système.

<div id="skill-missing-api-key-in-sandbox">
  ### Clé d’API manquante pour une skill dans le sandbox
</div>

**Symptôme :** La skill fonctionne sur l’hôte, mais échoue dans le sandbox faute de clé d’API.

**Pourquoi :** l’exécution dans le sandbox s’effectue dans Docker et **n’hérite pas** du `process.env` de l’hôte.

**Correctif :**

* définir `agents.defaults.sandbox.docker.env` (ou, par agent, `agents.list[].sandbox.docker.env`)
* ou intégrer la clé dans votre image de sandbox personnalisée
* puis exécuter `openclaw sandbox recreate --agent <id>` (ou `--all`)

<div id="service-running-but-port-not-listening">
  ### Service en cours d’exécution mais port non à l’écoute
</div>

Si le service indique **running** mais que rien n’écoute sur le port du Gateway,
Gateway a probablement refusé de se lier au port.

**Ce que « running » signifie ici**

* `Runtime: running` signifie que votre superviseur (launchd/systemd/schtasks) considère que le processus est vivant.
* `RPC probe` signifie que le CLI a effectivement pu se connecter au WebSocket du Gateway et appeler `status`.
* Fiez-vous toujours à `Probe target:` + `Config (service):` comme lignes « qu’est-ce qu’on a réellement essayé ? ».

**À vérifier :**

* `gateway.mode` doit être `local` pour `openclaw gateway` et le service.
* Si vous définissez `gateway.mode=remote`, le **comportement par défaut du CLI** est d’utiliser une URL distante. Le service peut toujours tourner localement, mais votre CLI peut sonder le mauvais endroit. Utilisez `openclaw gateway status` pour voir le port résolu du service + la cible de la sonde (ou passez `--url`).
* `openclaw gateway status` et `openclaw doctor` exposent la **dernière erreur du Gateway** issue des logs lorsque le service semble en cours d’exécution mais que le port est fermé.
* Les liaisons non loopback (`lan`/`tailnet`/`custom`, ou `auto` lorsque loopback n’est pas disponible) exigent une authentification :
  `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
* `gateway.remote.token` est uniquement destiné aux appels CLI distants ; il n’active **pas** l’auth locale.
* `gateway.token` est ignoré ; utilisez `gateway.auth.token`.

**Si `openclaw gateway status` affiche une incohérence de configuration**

* `Config (cli): ...` et `Config (service): ...` devraient normalement correspondre.
* Si ce n’est pas le cas, vous modifiez presque certainement une config pendant que le service en utilise une autre.
* Correctif : relancez `openclaw gateway install --force` depuis le même `--profile` / `OPENCLAW_STATE_DIR` que celui que vous voulez que le service utilise.

**Si `openclaw gateway status` signale des problèmes de configuration du service**

* La configuration du superviseur (launchd/systemd/schtasks) ne contient pas les valeurs par défaut actuelles.
* Correctif : exécutez `openclaw doctor` pour la mettre à jour (ou `openclaw gateway install --force` pour une réécriture complète).

**Si `Last gateway error:` mentionne « refusing to bind … without auth »**

* Vous avez défini `gateway.bind` sur un mode non loopback (`lan`/`tailnet`/`custom`, ou `auto` lorsque loopback n’est pas disponible) sans configurer l’authentification.
* Correctif : définissez `gateway.auth.mode` + `gateway.auth.token` (ou exportez `OPENCLAW_GATEWAY_TOKEN`) et redémarrez le service.

**Si `openclaw gateway status` indique `bind=tailnet` mais qu’aucune interface Tailnet n’a été trouvée**

* Gateway a tenté de se lier à une IP Tailscale (100.64.0.0/10) mais aucune n’a été détectée sur l’hôte.
* Correctif : démarrez Tailscale sur cette machine (ou changez `gateway.bind` en `loopback`/`lan`).

**Si `Probe note:` indique que la sonde utilise loopback**

* C’est attendu pour `bind=lan` : Gateway écoute sur `0.0.0.0` (toutes les interfaces), et loopback doit toujours permettre une connexion locale.
* Pour les clients distants, utilisez une véritable IP LAN (pas `0.0.0.0`) plus le port, et assurez-vous que l’authentification est configurée.

<div id="address-already-in-use-port-18789">
  ### Adresse déjà utilisée (port 18789)
</div>

Cela signifie qu’un autre processus écoute déjà sur le port utilisé par le Gateway.

**À vérifier :**

```bash
openclaw gateway status
```

Il affichera les processus à l’écoute et les causes probables (Gateway déjà en cours d’exécution, tunnel SSH).
Si nécessaire, arrêtez le service ou choisissez un autre port.

<div id="extra-workspace-folders-detected">
  ### Dossiers d’espace de travail supplémentaires détectés
</div>

Si vous avez effectué une mise à niveau depuis d’anciennes installations, il se peut que `~/openclaw` soit encore présent sur votre disque.
La présence de plusieurs répertoires d’espace de travail peut entraîner des incohérences déroutantes au niveau de l’authentification ou de l’état, car
un seul espace de travail est actif.

**Solution :** conservez un seul espace de travail actif et archivez/supprimez les autres. Voir
[Espace de travail d’agent](/fr/concepts/agent-workspace#extra-workspace-folders).

<div id="main-chat-running-in-a-sandbox-workspace">
  ### Discussion principale exécutée dans un espace de travail sandbox
</div>

Symptômes : `pwd` ou les outils de fichiers indiquent `~/.openclaw/sandboxes/...` alors que vous
vous attendiez à l&#39;espace de travail de l&#39;hôte.

**Pourquoi :** `agents.defaults.sandbox.mode: "non-main"` se base sur `session.mainKey` (par défaut `"main"`).
Les sessions de groupe/canal utilisent leurs propres clés ; elles sont donc traitées comme non-main et
obtiennent des espaces de travail sandbox.

**Solutions possibles :**

* Si vous voulez un espace de travail hôte pour un agent : définissez `agents.list[].sandbox.mode: "off"`.
* Si vous voulez un accès à l&#39;espace de travail de l&#39;hôte à l&#39;intérieur de la sandbox : définissez `workspaceAccess: "rw"` pour cet agent.

<div id="agent-was-aborted">
  ### &quot;L&#39;agent a été interrompu&quot;
</div>

L&#39;agent a été interrompu au milieu de sa réponse.

**Causes :**

* L&#39;utilisateur a envoyé la commande `stop`, `abort`, `esc`, `wait` ou `exit`
* Délai d&#39;attente dépassé
* Le processus a crashé

**Solution :** Envoyez simplement un autre message. La session continue.

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agent failed before reply: Unknown model: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw rejette intentionnellement les **modèles plus anciens/moins sécurisés**
(en particulier les plus vulnérables aux attaques par injection de prompt). Si
vous voyez cette erreur, ce nom de modèle n’est plus pris en charge.

**Solution :**

* Choisissez le **modèle le plus récent** pour le fournisseur et mettez à jour
  votre configuration ou l’alias de modèle.
* Si vous ne savez pas quels modèles sont disponibles, exécutez
  `openclaw models list` ou `openclaw models scan` et choisissez un modèle pris
  en charge.
* Consultez les journaux du Gateway pour connaître la cause détaillée de l’échec.

Voir aussi : [CLI des modèles](/fr/cli/models) et [Fournisseurs de modèles](/fr/concepts/model-providers).

<div id="messages-not-triggering">
  ### Les messages ne déclenchent aucune action
</div>

**Vérification 1 :** L’expéditeur est-il dans la liste d’autorisation ?

```bash
openclaw status
```

Recherchez `AllowFrom: ...` dans la sortie.

**Vérification 2 :** Pour les chats de groupe, la mention est‑elle requise ?

```bash
# Le message doit correspondre aux mentionPatterns ou aux mentions explicites ; les valeurs par défaut se trouvent dans les groupes/guildes de canaux.
# Multi-agent : `agents.list[].groupChat.mentionPatterns` remplace les motifs globaux.
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**Vérification 3 :** Consultez les journaux

```bash
openclaw logs --follow
# ou si vous souhaitez des filtres rapides :
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### Code d’appairage non reçu
</div>

Si `dmPolicy` est défini sur `pairing`, les expéditeurs inconnus doivent recevoir un code et leur message est ignoré tant qu’il n’a pas été approuvé.

**Vérification 1 :** Y a-t-il déjà une demande en attente ?

```bash
openclaw pairing list <channel>
```

Les demandes d’appairage en DM en attente sont limitées à **3 par canal** par défaut. Si la liste est pleine, les nouvelles demandes ne généreront pas de code tant qu’aucune n’a été approuvée ou n’a expiré.

**Vérification 2 :** La demande a-t-elle été créée mais aucune réponse n’a été envoyée ?

```bash
openclaw logs --follow | grep "pairing request"
```

**Vérification 3 :** Vérifiez que `dmPolicy` n’est pas réglé sur `open` ou `allowlist` pour ce canal.

<div id="image-mention-not-working">
  ### Image + mention ne fonctionne pas
</div>

Problème connu : lorsque vous envoyez une image contenant UNIQUEMENT une mention (sans autre texte), WhatsApp n’inclut parfois pas les métadonnées de la mention.

**Solution de contournement :** ajoutez un peu de texte avec la mention :

* ❌ `@openclaw` + image
* ✅ `@openclaw check this` + image

<div id="session-not-resuming">
  ### La session ne reprend pas
</div>

**Vérification 1 :** Le fichier de session est-il présent ?

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**Vérification 2 :** La fenêtre de réinitialisation est-elle trop courte ?

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 jours
    }
  }
}
```

**Vérification 3 :** Quelqu&#39;un a-t-il envoyé la commande `/new`, `/reset` ou déclenché une réinitialisation ?

<div id="agent-timing-out">
  ### Expiration de l’agent pour dépassement de délai
</div>

Le délai d’expiration par défaut est de 30 minutes. Pour les tâches longues :

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 heure
  }
}
```

Ou utilisez l’outil `process` pour exécuter des commandes longues en arrière-plan.

<div id="whatsapp-disconnected">
  ### Déconnexion de WhatsApp
</div>

```bash
# Check local status (creds, sessions, queued events)
openclaw status
# Sonder le Gateway en cours d'exécution + canaux (connexion WA + APIs Telegram + Discord)
openclaw status --deep

# View recent connection events
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**Correctif :** En général, la reconnexion se fait automatiquement une fois que le Gateway est lancé. Si vous êtes bloqué, redémarrez le processus Gateway (quelle que soit la façon dont vous le supervisez), ou exécutez-le manuellement avec une sortie détaillée :

```bash
openclaw gateway --verbose
```

Si vous êtes déconnecté ou non lié :

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # si logout ne peut pas tout supprimer proprement
openclaw channels login --verbose       # scanner à nouveau le code QR
```

<div id="media-send-failing">
  ### Échec de l’envoi de médias
</div>

**Vérification 1 :** Le chemin du fichier est-il valide ?

```bash
ls -la /chemin/vers/votre/image.jpg
```

**Vérification 2 :** Le fichier est-il trop volumineux ?

* Images : 6 Mo max
* Audio/Vidéo : 16 Mo max
* Documents : 100 Mo max

**Vérification 3 :** Consulte les journaux multimédia

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### Utilisation mémoire élevée
</div>

OpenClaw conserve l&#39;historique des conversations en mémoire.

**Solution :** Redémarrez périodiquement ou définissez des limites pour les sessions :

```json
{
  "session": {
    "historyLimit": 100  // Nombre maximum de messages à conserver
  }
}
```

<div id="common-troubleshooting">
  ## Problèmes courants
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### « Gateway ne démarre pas : configuration invalide »
</div>

OpenClaw refuse désormais de démarrer lorsque la configuration contient des clés inconnues, des valeurs mal formées ou des types invalides.
C’est volontaire, pour des raisons de sécurité.

Corrige le problème avec Doctor :

```bash
openclaw doctor
openclaw doctor --fix
```

Remarques :

* `openclaw doctor` signale toute entrée invalide.
* `openclaw doctor --fix` applique les migrations/réparations et réécrit la configuration.
* Les commandes de diagnostic comme `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw gateway status` et `openclaw gateway probe` continuent de s’exécuter même si la configuration est invalide.

<div id="all-models-failed-what-should-i-check-first">
  ### « All models failed » — que devez-vous vérifier en premier ?
</div>

* **Identifiants** présents pour le(s) fournisseur(s) utilisé(s) (profils d’authentification + variables d’environnement).
* **Routage des modèles** : vérifiez que `agents.defaults.model.primary` et les modèles de repli sont des modèles auxquels vous avez accès.
* **Journaux du Gateway** dans `/tmp/openclaw/…` pour connaître l’erreur exacte du fournisseur.
* **Statut des modèles** : utilisez `/model status` (chat) ou `openclaw models status` (CLI).

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### J’utilise mon numéro WhatsApp personnel — pourquoi les conversations avec moi‑même sont‑elles bizarres ?
</div>

Activez le mode self-chat et ajoutez votre propre numéro à la liste d’autorisation :

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

Voir la [configuration de WhatsApp](/fr/channels/whatsapp).

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp m’a déconnecté. Comment me reconnecter ?
</div>

Relancez la commande de connexion et scannez le code QR :

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### Erreurs de build sur `main` — quelle est la procédure standard pour les corriger ?
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. Consultez les issues GitHub ou Discord
4. Contournement temporaire : basculer sur un commit plus ancien

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### L’installation npm échoue (allow-build-scripts / tar ou yargs manquants). Que faire ?
</div>

Si vous exécutez depuis les sources, utilisez le gestionnaire de paquets du dépôt : **pnpm** (recommandé).
Le dépôt déclare `packageManager: "pnpm@…"`.

Procédure de récupération typique :

```bash
git status   # assurez-vous que vous êtes à la racine du dépôt
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

Pourquoi : pnpm est le gestionnaire de paquets configuré pour ce dépôt.

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### Comment basculer entre les installations git et les installations npm ?
</div>

Utilise le **programme d’installation sur le site web** et sélectionne la méthode d’installation avec un indicateur. Cela effectue une mise à niveau sur place et réécrit le service Gateway pour qu’il pointe vers la nouvelle installation.

Basculer **vers une installation git** :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

Passez **en mode global npm** :

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notes :

* Le workflow Git n’effectue un rebase que si le dépôt est propre. Valide (`commit`) ou mets de côté (`stash`) d’abord tes modifications.
* Après le changement, exécute :
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### Le block streaming de Telegram ne découpe pas le texte entre les appels d’outils. Pourquoi ?
</div>

Le block streaming n’envoie que des **blocs de texte terminés**. Raisons courantes pour lesquelles vous ne voyez qu’un seul message :

* `agents.defaults.blockStreamingDefault` est toujours `"off"`.
* `channels.telegram.blockStreaming` est défini à `false`.
* `channels.telegram.streamMode` est `partial` ou `block` **et le draft streaming est actif**
  (conversation privée + sujets). Dans ce cas, le draft streaming désactive le block streaming.
* Vos paramètres `minChars` / coalesce sont trop élevés, donc les chunks sont fusionnés.
* Le modèle émet un seul grand bloc de texte (aucun flush intermédiaire pendant la réponse).

Liste de vérification pour la correction :

1. Placez les paramètres de block streaming sous `agents.defaults`, pas à la racine.
2. Définissez `channels.telegram.streamMode: "off"` si vous voulez de vraies réponses multi‑messages par blocs.
3. Utilisez des seuils de chunk/coalesce plus faibles pendant le débogage.

Voir [Streaming](/fr/concepts/streaming).

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### Discord ne répond pas sur mon serveur même avec `requireMention: false`. Pourquoi ?
</div>

`requireMention` contrôle uniquement la nécessité de mention **après** que le canal a passé les listes d’autorisation.
Par défaut, `channels.discord.groupPolicy` est en mode **allowlist**, donc les guildes doivent être explicitement activées.
Si vous définissez `channels.discord.guilds.<guildId>.channels`, seuls les canaux listés sont autorisés ; omettez‑la pour autoriser tous les canaux de la guilde.

Liste de vérification :

1. Définissez `channels.discord.groupPolicy: "open"` **ou** ajoutez une entrée de liste d’autorisation pour la guilde (et éventuellement une liste d’autorisation pour les canaux).
2. Utilisez des **ID de canaux numériques** dans `channels.discord.guilds.<guildId>.channels`.
3. Placez `requireMention: false` **sous** `channels.discord.guilds` (global ou par canal).
   La clé de haut niveau `channels.discord.requireMention` n’est pas prise en charge.
4. Vérifiez que le bot dispose de **Message Content Intent** et des autorisations sur le canal.
5. Exécutez `openclaw channels status --probe` pour obtenir des indications pour l’audit.

Docs : [Discord](/fr/channels/discord), [Dépannage des canaux](/fr/channels/troubleshooting).

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Erreur Cloud Code Assist API : invalid tool schema (400). Que faire ?
</div>

C’est presque toujours un problème de **compatibilité de schéma d’outil**. L’endpoint
Cloud Code Assist accepte un sous‑ensemble strict de JSON Schema. OpenClaw nettoie/normalise
les schémas d’outils dans la branche `main` actuelle, mais la correction n’est pas encore
incluse dans la dernière version (au 13 janvier 2026).

Liste de contrôle pour la correction :

1. **Mettre à jour OpenClaw** :
   * Si vous pouvez l’exécuter depuis les sources, faites un `git pull` de `main` et redémarrez le Gateway.
   * Sinon, attendez la prochaine version qui inclura le nettoyeur de schémas.
2. Évitez les mots‑clés non pris en charge comme `anyOf/oneOf/allOf`, `patternProperties`,
   `additionalProperties`, `minLength`, `maxLength`, `format`, etc.
3. Si vous définissez des outils personnalisés, gardez le schéma de niveau supérieur défini comme
   `type: "object"` avec `properties` et des enums simples.

Voir [Tools](/fr/tools) et [TypeBox schemas](/fr/concepts/typebox).

<div id="macos-specific-issues">
  ## Problèmes spécifiques à macOS
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### L’application plante lors de l’autorisation des permissions (Dictée/Micro)
</div>

Si l’application disparaît ou affiche &quot;Abort trap 6&quot; lorsque vous cliquez sur &quot;Allow&quot; dans une boîte de dialogue de confidentialité :

**Solution 1 : réinitialiser le cache TCC**

```bash
tccutil reset All bot.molt.mac.debug
```

**Correctif 2 : forcer un nouvel identifiant de bundle**
Si la réinitialisation ne fonctionne pas, modifiez `BUNDLE_ID` dans [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) (par exemple, en ajoutant un suffixe `.test`), puis reconstruisez l’app. Cela force macOS à la considérer comme une nouvelle app.

<div id="gateway-stuck-on-starting">
  ### Gateway bloqué sur « Starting... »
</div>

L&#39;application se connecte à un Gateway local sur le port `18789`. S&#39;il reste bloqué :

**Correctif 1 : arrêter le superviseur (recommandé)**
Si le Gateway est supervisé par launchd, tuer le PID ne fera que le relancer. Arrêtez d&#39;abord le superviseur :

```bash
openclaw gateway status
openclaw gateway stop
# Ou : launchctl bootout gui/$UID/bot.molt.gateway (remplacer par bot.molt.<profile> ; l'ancien com.openclaw.* fonctionne toujours)
```

**Correctif 2 : le port est déjà utilisé (identifier le processus à l&#39;écoute)**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

S’il s’agit d’un processus non supervisé, essayez d’abord un arrêt propre, puis escaladez si nécessaire :

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # en dernier recours
```

**Correctif 3 : vérifiez l&#39;installation de la CLI**
Vérifiez que la CLI globale `openclaw` est installée et qu&#39;elle correspond à la version de l&#39;application :

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## Mode de débogage
</div>

Activez la journalisation détaillée :

```bash
# Activer la journalisation de trace dans la config :
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# Puis exécuter les commandes en mode verbeux pour afficher la sortie de débogage sur stdout :
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## Emplacements des journaux
</div>

| Journal | Emplacement |
|-----|----------|
| Journaux de fichiers du Gateway (structurés) | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (ou `logging.file`) |
| Journaux du service Gateway (superviseur) | macOS : `$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log` (par défaut : `~/.openclaw/logs/...` ; les profils utilisent `~/.openclaw-<profile>/logs/...`)<br />Linux : `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows : `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| Fichiers de session | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| Cache de médias | `$OPENCLAW_STATE_DIR/media/` |
| Identifiants | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## Vérification de l&#39;état du système
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# Inclure les analyses au niveau système (services legacy/supplémentaires, écouteurs de ports)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## Tout réinitialiser
</div>

Mesure radicale :

```bash
openclaw gateway stop
# Si vous avez installé un service et souhaitez une installation propre :
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # re-pair WhatsApp
openclaw gateway restart           # or: openclaw gateway
```

⚠️ Cela entraîne la perte de toutes les sessions et nécessite un nouvel appairage de WhatsApp.

<div id="getting-help">
  ## Obtenir de l&#39;aide
</div>

1. Consultez d&#39;abord les journaux : `/tmp/openclaw/` (par défaut : `openclaw-YYYY-MM-DD.log`, ou votre valeur configurée de `logging.file`)
2. Recherchez les tickets existants sur GitHub
3. Ouvrez un nouveau ticket avec :
   * Version d&#39;OpenClaw
   * Extraits de journaux pertinents
   * Étapes pour reproduire le problème
   * Votre configuration (masquez les secrets !)

***

*« Avez-vous essayé de l&#39;éteindre et de le rallumer ? »* — Tous les informaticiens, un jour ou l&#39;autre

🦞🔧

<div id="browser-not-starting-linux">
  ### Le navigateur ne démarre pas (Linux)
</div>

Si vous voyez `"Failed to start Chrome CDP on port 18800"` :

**Cause la plus probable :** Chromium fourni sous forme de paquet Snap sur Ubuntu.

**Solution rapide :** installez Google Chrome à la place :

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

Puis définissez dans la configuration :

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**Guide complet :** Consultez [browser-linux-troubleshooting](/fr/tools/browser-linux-troubleshooting)
