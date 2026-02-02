---
title: Google Chat
summary: "Statut de la prise en charge, des fonctionnalités et de la configuration de l'application Google Chat"
read_when:
  - Travail sur les fonctionnalités du canal Google Chat
---

<div id="google-chat-chat-api">
  # Google Chat (Chat API)
</div>

Statut : prêt pour les DMs et les espaces via les webhooks de l’API Google Chat (HTTP uniquement).

<div id="quick-setup-beginner">
  ## Configuration rapide (débutant)
</div>

1. Créez un projet Google Cloud et activez l’**API Google Chat**.
   * Allez sur : [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * Activez l’API si elle n’est pas déjà activée.
2. Créez un **compte de service** :
   * Cliquez sur **Create Credentials** &gt; **Service Account**.
   * Nommez‑le comme vous voulez (par ex. `openclaw-chat`).
   * Laissez les autorisations vides (cliquez sur **Continue**).
   * Laissez les principaux avec accès vides (cliquez sur **Done**).
3. Créez et téléchargez la **clé JSON** :
   * Dans la liste des comptes de service, cliquez sur celui que vous venez de créer.
   * Allez dans l’onglet **Keys**.
   * Cliquez sur **Add Key** &gt; **Create new key**.
   * Sélectionnez **JSON** et cliquez sur **Create**.
4. Stockez le fichier JSON téléchargé sur l’hôte de votre Gateway (par ex. `~/.openclaw/googlechat-service-account.json`).
5. Créez une application Google Chat dans la [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) :
   * Renseignez les **Application info** :
     * **App name** : (par ex. `OpenClaw`)
     * **Avatar URL** : (par ex. `https://openclaw.ai/logo.png`)
     * **Description** : (par ex. `Personal AI Assistant`)
   * Activez les **Interactive features**.
   * Sous **Functionality**, cochez **Join spaces and group conversations**.
   * Sous **Connection settings**, sélectionnez **HTTP endpoint URL**.
   * Sous **Triggers**, sélectionnez **Use a common HTTP endpoint URL for all triggers** et définissez‑la sur l’URL publique de votre Gateway suivie de `/googlechat`.
     * *Astuce : exécutez `openclaw status` pour trouver l’URL publique de votre Gateway.*
   * Sous **Visibility**, cochez **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   * Saisissez votre adresse e‑mail (par ex. `user@example.com`) dans la zone de texte.
   * Cliquez sur **Save** en bas.
6. **Activez le statut de l’app** :
   * Après avoir enregistré, **rafraîchissez la page**.
   * Recherchez la section **App status** (généralement en haut ou en bas après l’enregistrement).
   * Modifiez le statut en **Live - available to users**.
   * Cliquez de nouveau sur **Save**.
7. Configurez OpenClaw avec le chemin du compte de service + l’audience du webhook :
   * Variable d’env : `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * Ou config : `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Définissez le type et la valeur de l’audience du webhook (doit correspondre à la configuration de votre app Chat).
9. Démarrez la Gateway. Google Chat enverra des requêtes POST vers votre chemin de webhook.

<div id="add-to-google-chat">
  ## Ajouter à Google Chat
</div>

Une fois que le Gateway est en cours d’exécution et que votre adresse e-mail a été ajoutée à la liste de visibilité :

1. Allez sur [Google Chat](https://chat.google.com/).
2. Cliquez sur l’icône **+** (plus) à côté de **Messages directs**.
3. Dans la barre de recherche (là où vous ajoutez habituellement des personnes), saisissez le **nom de l’application** que vous avez configurée dans Google Cloud Console.
   * **Remarque** : le bot *n’apparaîtra pas* dans la liste de navigation du « Marketplace » car il s’agit d’une application privée. Vous devez le rechercher par son nom.
4. Sélectionnez votre bot dans les résultats.
5. Cliquez sur **Ajouter** ou **Discuter** pour démarrer une conversation individuelle.
6. Envoyez « Hello » pour déclencher l’assistant !

<div id="public-url-webhook-only">
  ## URL publique (webhook uniquement)
</div>

Les webhooks Google Chat nécessitent un point de terminaison HTTPS public. Pour des raisons de sécurité, **n’exposez sur Internet que le chemin `/googlechat`**. Conservez le tableau de bord OpenClaw et les autres points de terminaison sensibles sur votre réseau privé.

<div id="option-a-tailscale-funnel-recommended">
  ### Option A : Tailscale Funnel (recommandé)
</div>

Utilisez Tailscale Serve pour le tableau de bord privé et Funnel pour le chemin de webhook public. Cela garde `/` privé tout en exposant uniquement `/googlechat`.

1. **Vérifiez sur quelle adresse votre Gateway est à l&#39;écoute :**
   ```bash
   ss -tlnp | grep 18789
   ```
   Notez l&#39;adresse IP (par exemple `127.0.0.1`, `0.0.0.0` ou votre IP Tailscale comme `100.x.x.x`).

2. **Exposez le tableau de bord uniquement au tailnet (port 8443) :**
   ```bash
   # Si à l'écoute sur localhost (127.0.0.1 ou 0.0.0.0) :
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Si à l'écoute uniquement sur l'IP Tailscale (par ex. 100.106.161.80) :
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Exposez uniquement le chemin du webhook publiquement :**
   ```bash
   # Si à l'écoute sur localhost (127.0.0.1 ou 0.0.0.0) :
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Si à l'écoute uniquement sur l'IP Tailscale (par ex. 100.106.161.80) :
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Autorisez le nœud pour l&#39;accès Funnel :**
   Le cas échéant, ouvrez l&#39;URL d&#39;autorisation affichée dans la sortie pour activer Funnel pour ce nœud dans la politique de votre tailnet.

5. **Vérifiez la configuration :**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Votre URL de webhook publique sera :
`https://<node-name>.<tailnet>.ts.net/googlechat`

Votre tableau de bord privé reste accessible uniquement via le tailnet :
`https://<node-name>.<tailnet>.ts.net:8443/`

Utilisez l&#39;URL publique (sans `:8443`) dans la configuration de l&#39;application Google Chat.

> Remarque : cette configuration persiste après les redémarrages. Pour la supprimer plus tard, exécutez `tailscale funnel reset` et `tailscale serve reset`.

<div id="option-b-reverse-proxy-caddy">
  ### Option B : proxy inverse (Caddy)
</div>

Si vous utilisez un proxy inverse comme Caddy, ne configurez le proxy que pour le chemin spécifique :

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Avec cette configuration, toute requête vers `your-domain.com/` sera ignorée ou renverra une réponse 404, tandis que `your-domain.com/googlechat` sera correctement routé vers OpenClaw.

<div id="option-c-cloudflare-tunnel">
  ### Option C : Cloudflare Tunnel
</div>

Configurez les règles d’ingress de votre tunnel pour n’acheminer que le chemin du webhook :

* **Chemin** : `/googlechat` -&gt; `http://localhost:18789/googlechat`
* **Règle par défaut** : HTTP 404 (Not Found)

<div id="how-it-works">
  ## Fonctionnement
</div>

1. Google Chat envoie des requêtes POST de webhook au Gateway. Chaque requête inclut un en-tête `Authorization: Bearer <token>`.
2. OpenClaw vérifie le jeton par rapport aux valeurs configurées `audienceType` + `audience` :
   * `audienceType: "app-url"` → l’audience est votre URL de webhook HTTPS.
   * `audienceType: "project-number"` → l’audience est le numéro de projet Cloud.
3. Les messages sont routés par espace :
   * Les MP utilisent la clé de session `agent:<agentId>:googlechat:dm:<spaceId>`.
   * Les espaces utilisent la clé de session `agent:<agentId>:googlechat:group:<spaceId>`.
4. L’accès aux MP se fait par appairage par défaut. Les expéditeurs inconnus reçoivent un code d’appairage ; approuvez avec :
   * `openclaw pairing approve googlechat <code>`
5. Les espaces de groupe nécessitent une @-mention par défaut. Utilisez `botUser` si la détection des mentions nécessite le nom d’utilisateur de l’application.

<div id="targets">
  ## Cibles
</div>

Utilisez ces identifiants pour l’envoi et les listes d’autorisation :

* Messages privés : `users/<userId>` ou `users/<email>` (les adresses e-mail sont acceptées).
* Espaces : `spaces/<spaceId>`.

<div id="config-highlights">
  ## Points de configuration importants
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optionnel ; aide à la détection des mentions
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Remarques :

* Les identifiants du compte de service peuvent aussi être fournis en ligne via `serviceAccount` (chaîne JSON).
* Le chemin de webhook par défaut est `/googlechat` si `webhookPath` n’est pas défini.
* Les réactions sont disponibles via l’outil `reactions` et `channels action` lorsque `actions.reactions` est activé.
* `typingIndicator` prend en charge `none`, `message` (valeur par défaut) et `reaction` (la réaction nécessite l’OAuth côté utilisateur).
* Les pièces jointes sont téléchargées via l’API Chat et stockées dans le pipeline multimédia (taille limitée par `mediaMaxMb`).

<div id="troubleshooting">
  ## Résolution des problèmes
</div>

<div id="405-method-not-allowed">
  ### 405 Method Not Allowed
</div>

Si Google Cloud Logs Explorer affiche des erreurs comme :

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Cela signifie que le gestionnaire de webhook n&#39;est pas enregistré. Causes fréquentes :

1. **Canal non configuré** : La section `channels.googlechat` est manquante dans votre configuration. Vérifiez avec :
   ```bash
   openclaw config get channels.googlechat
   ```
   Si la commande renvoie &quot;Config path not found&quot;, ajoutez la configuration (voir [Config highlights](#config-highlights)).

2. **Plugin non activé** : Vérifiez l&#39;état du plugin :
   ```bash
   openclaw plugins list | grep googlechat
   ```
   Si la sortie indique &quot;disabled&quot;, ajoutez `plugins.entries.googlechat.enabled: true` à votre configuration.

3. **Gateway non redémarré** : Après avoir ajouté la configuration, redémarrez Gateway :
   ```bash
   openclaw gateway restart
   ```

Vérifiez que le canal est en cours d&#39;exécution :

```bash
openclaw channels status
# Devrait afficher : Google Chat default: enabled, configured, ...
```

<div id="other-issues">
  ### Autres problèmes
</div>

* Vérifiez `openclaw channels status --probe` pour détecter les erreurs d’authentification ou une configuration d’audience manquante.
* Si aucun message ne parvient, vérifiez l’URL du webhook de l’application Chat ainsi que les abonnements aux événements.
* Si la restriction par mention bloque les réponses, définissez `botUser` sur le nom de ressource utilisateur de l’application et vérifiez `requireMention`.
* Utilisez `openclaw logs --follow` pendant que vous envoyez un message de test pour voir si les requêtes atteignent le Gateway.

Documentation associée :

* [Configuration du Gateway](/fr/gateway/configuration)
* [Sécurité](/fr/gateway/security)
* [Réactions](/fr/tools/reactions)