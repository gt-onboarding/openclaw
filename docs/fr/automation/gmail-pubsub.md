---
title: Gmail Pub/Sub
summary: "Push Gmail Pub/Sub relié aux webhooks OpenClaw via gogcli"
read_when:
  - Raccorder les déclencheurs de la boîte de réception Gmail à OpenClaw
  - Configurer le push Pub/Sub pour le réveil des agents
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

Objectif : Gmail watch -&gt; Pub/Sub push -&gt; `gog gmail watch serve` -&gt; webhook vers OpenClaw.

<div id="prereqs">
  ## Prérequis
</div>

* `gcloud` installé et connecté ([guide d’installation](https://docs.cloud.google.com/sdk/docs/install-sdk)).
* `gog` (gogcli) installé et autorisé pour le compte Gmail ([gogcli.sh](https://gogcli.sh/)).
* Hooks OpenClaw activés (voir [Webhooks](/fr/automation/webhook)).
* `tailscale` connecté ([tailscale.com](https://tailscale.com/)). La configuration prise en charge utilise Tailscale Funnel pour le point de terminaison HTTPS public.
  D’autres services de tunnel peuvent fonctionner, mais relèvent du bricolage, ne sont pas pris en charge et nécessitent un câblage manuel.
  À ce jour, seul Tailscale est officiellement pris en charge.

Exemple de configuration de hook (activer le mappage prédéfini Gmail) :

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

Pour envoyer le récapitulatif Gmail vers une interface de chat, remplacez le préréglage par un mappage qui définit `deliver` et éventuellement `channel`/`to` :

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567"
      }
    ]
  }
}
```

Si vous voulez un canal fixe, définissez `channel` + `to`. Sinon, `channel: "last"`
utilise le dernier routage de livraison (avec repli sur WhatsApp).

Pour imposer un modèle moins coûteux pour les exécutions Gmail, définissez `model` dans le mapping
(`provider/model` ou alias). Si vous imposez `agents.defaults.models`, incluez‑le là.

Pour définir un modèle et un niveau de réflexion par défaut spécifiques aux hooks Gmail, ajoutez
`hooks.gmail.model` / `hooks.gmail.thinking` dans votre configuration :

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off"
    }
  }
}
```

Remarques :

* Le `model`/`thinking` propre à chaque hook dans le mappage continue de prévaloir sur ces valeurs par défaut.
* Ordre de repli : `hooks.gmail.model` → `agents.defaults.model.fallbacks` → modèle principal (auth/limitation de débit/délais d’expiration).
* Si `agents.defaults.models` est défini, le modèle Gmail doit figurer dans la liste d’autorisation.
* Le contenu du hook Gmail est encapsulé par défaut dans des garde-fous de sécurité pour contenu externe.
  Pour désactiver cela (dangereux), définissez `hooks.gmail.allowUnsafeExternalContent: true`.

Pour personnaliser davantage la gestion des payloads, ajoutez `hooks.mappings` ou un module de transformation JS/TS
dans `hooks.transformsDir` (voir [Webhooks](/fr/automation/webhook)).

<div id="wizard-recommended">
  ## Assistant de configuration (recommandé)
</div>

Utilisez l’assistant de configuration OpenClaw pour tout relier (installe les dépendances sur macOS via brew) :

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Valeurs par défaut :

* Utilise Tailscale Funnel pour le point de terminaison public de push.
* Écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`.
* Active le préréglage de hook Gmail (`hooks.presets: ["gmail"]`).

Remarque sur le chemin : lorsque `tailscale.mode` est activé, OpenClaw définit automatiquement
`hooks.gmail.serve.path` sur `/` et conserve le chemin public sur
`hooks.gmail.tailscale.path` (par défaut `/gmail-pubsub`), car Tailscale
supprime le préfixe de chemin défini avant de relayer la requête en proxy.
Si vous avez besoin que le backend reçoive le chemin préfixé, définissez
`hooks.gmail.tailscale.target` (ou `--tailscale-target`) sur une URL complète comme
`http://127.0.0.1:8788/gmail-pubsub` et alignez `hooks.gmail.serve.path` sur celle-ci.

Vous voulez un point de terminaison personnalisé ? Utilisez `--push-endpoint <url>` ou `--tailscale off`.

Remarque sur la plateforme : sur macOS, l’assistant installe `gcloud`, `gogcli` et `tailscale`
via Homebrew ; sur Linux, installez-les d’abord manuellement.

Démarrage automatique du Gateway (recommandé) :

* Lorsque `hooks.enabled=true` et que `hooks.gmail.account` est défini, le Gateway lance
  `gog gmail watch serve` au démarrage et renouvelle automatiquement la surveillance.
* Définissez `OPENCLAW_SKIP_GMAIL_WATCHER=1` pour désactiver cette fonctionnalité (utile si vous exécutez vous-même le démon).
* N’exécutez pas le démon manuel en même temps, sinon vous rencontrerez
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Démon manuel (démarre `gog gmail watch serve` + renouvellement automatique) :

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## Configuration initiale
</div>

1. Sélectionnez le projet GCP **qui possède le client OAuth** utilisé par `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Remarque : la surveillance de Gmail requiert que le sujet Pub/Sub réside dans le même projet que le client OAuth.

2. Activez les API :

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Créez un sujet :

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Autoriser Gmail (push) à publier :

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## Démarrer le watch
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Enregistrez le `history_id` figurant dans la sortie (pour le débogage).

<div id="run-the-push-handler">
  ## Exécuter le gestionnaire de push
</div>

Exemple en local (authentification par jeton partagé) :

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notes :

* `--token` protège le point de terminaison de push (`x-gog-token` ou `?token=`).
* `--hook-url` pointe vers OpenClaw `/hooks/gmail` (mappé ; exécution isolée + résumé vers l’instance principale).
* `--include-body` et `--max-bytes` contrôlent l’extrait du contenu envoyé à OpenClaw.

Recommandé : `openclaw webhooks gmail run` encapsule le même flux et renouvelle automatiquement le watch.

<div id="expose-the-handler-advanced-unsupported">
  ## Exposer le handler (avancé, non pris en charge)
</div>

Si vous avez besoin d’un tunnel sans Tailscale, configurez-le manuellement et utilisez l’URL publique dans l’abonnement push (non pris en charge, sans garde-fous) :

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Utilisez l’URL générée comme endpoint de push :

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

En production : utilisez un endpoint HTTPS stable et configurez le JWT OIDC Pub/Sub, puis exécutez :

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## Test
</div>

Envoyez un message dans la boîte de réception surveillée :

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Vérifier l’état et l’historique de la surveillance :

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## Dépannage
</div>

* `Invalid topicName` : incohérence de projet (topic ne faisant pas partie du projet du client OAuth).
* `User not authorized` : rôle `roles/pubsub.publisher` manquant sur le topic.
* Messages vides : les notifications push Gmail ne fournissent que `historyId` ; récupérez-les via `gog gmail history`.

<div id="cleanup">
  ## Nettoyage
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
