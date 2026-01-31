---
title: Premiers pas
summary: "Guide pour débuter : de zéro au premier message (assistant de configuration, authentification, canaux, appairage)"
read_when:
  - Configuration initiale à partir de zéro
  - Vous cherchez la méthode la plus rapide pour passer de l'installation → la mise en route → au premier message
---

<div id="getting-started">
  # Bien démarrer
</div>

Objectif : passer de **zéro** → **premier chat fonctionnel** (avec des valeurs par défaut raisonnables) aussi vite que possible.

Pour obtenir un chat le plus rapidement possible : ouvrez le Control UI (aucune configuration de canal nécessaire). Exécutez `openclaw dashboard`
et discutez dans le navigateur, ou ouvrez `http://127.0.0.1:18789/` sur l’hôte du Gateway.
Docs : [Dashboard](/fr/web/dashboard) et [Control UI](/fr/web/control-ui).

Parcours recommandé : utilisez l’**assistant d’onboarding en CLI** (`openclaw onboard`). Il configure :

* modèle/auth (OAuth recommandé)
* paramètres du Gateway
* canaux (WhatsApp/Telegram/Discord/Mattermost (plugin)/...)
* valeurs par défaut d’appairage (DM sécurisés)
* bootstrap de l’espace de travail + compétences
* service optionnel en arrière-plan

Si vous voulez des pages de référence plus détaillées, allez directement à : [Wizard](/fr/start/wizard), [Setup](/fr/start/setup), [Pairing](/fr/start/pairing), [Security](/fr/gateway/security).

Remarque concernant le sandbox : `agents.defaults.sandbox.mode: "non-main"` utilise `session.mainKey` (par défaut `"main"`),
donc les sessions de groupe/canal sont isolées dans un sandbox. Si vous voulez que l’agent principal s’exécute toujours
sur l’hôte, définissez une surcharge explicite par agent :

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

<div id="0-prereqs">
  ## 0) Prérequis
</div>

* Node `>=22`
* `pnpm` (facultatif ; recommandé si vous compilez depuis les sources)
* **Recommandé :** clé d’API Brave Search pour la recherche web. Chemin le plus simple :
  `openclaw configure --section web` (stocke `tools.web.search.apiKey`).
  Voir [Outils web](/fr/tools/web).

macOS : si vous prévoyez de compiler les apps, installez Xcode / CLT. Pour la CLI + Gateway uniquement, Node suffit.
Windows : utilisez **WSL2** (Ubuntu recommandé). WSL2 est fortement recommandé : Windows natif n’est pas testé, plus problématique et offre une moins bonne compatibilité avec les outils. Installez d’abord WSL2, puis exécutez les étapes Linux à l’intérieur de WSL2. Voir [Windows (WSL2)](/fr/platforms/windows).

<div id="1-install-the-cli-recommended">
  ## 1) Installer la CLI (recommandée)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Options d’installation (méthode, mode non interactif, depuis GitHub) : [Installation](/fr/install).

Windows (PowerShell) :

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

Alternative (installation globale) :

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2) Lancer l’assistant de configuration (et installer le service)
</div>

```bash
openclaw onboard --install-daemon
```

Ce que vous allez choisir :

* **Gateway locale ou distante**
* **Auth** : abonnement OpenAI Code (Codex) (OAuth) ou clés API. Pour Anthropic, nous recommandons une clé API ; `claude setup-token` est également pris en charge.
* **Fournisseurs** : connexion WhatsApp via QR, jetons de bot Telegram/Discord, jetons de plugin Mattermost, etc.
* **Daemon** : installation en arrière-plan (launchd/systemd ; WSL2 utilise systemd)
  * **Runtime** : Node (recommandé ; requis pour WhatsApp/Telegram). Bun n’est **pas recommandé**.
* **Jeton Gateway** : l’assistant en génère un par défaut (même sur l’interface loopback) et le stocke dans `gateway.auth.token`.

Doc de l’assistant : [Wizard](/fr/start/wizard)

<div id="auth-where-it-lives-important">
  ### Auth : où ça se trouve (important)
</div>

* **Chemin Anthropic recommandé :** définissez une clé API (l’assistant de configuration peut la stocker pour que le service puisse l’utiliser). `claude setup-token` est également pris en charge si vous voulez réutiliser les identifiants Claude Code.

* Identifiants OAuth (importation héritée) : `~/.openclaw/credentials/oauth.json`

* Profils d’authentification (OAuth + clés API) : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Astuce pour utilisation en mode headless / serveur : effectuez d’abord l’authentification OAuth sur une machine classique, puis copiez `oauth.json` sur la machine hébergeant le Gateway.

<div id="3-start-the-gateway">
  ## 3) Démarrer le Gateway
</div>

Si vous avez installé le service durant la phase d’onboarding, le Gateway devrait déjà être en cours d’exécution :

```bash
openclaw gateway status
```

Exécution manuelle au premier plan :

```bash
openclaw gateway --port 18789 --verbose
```

Tableau de bord (loopback local) : `http://127.0.0.1:18789/`
Si un jeton est configuré, collez-le dans les paramètres de Control UI (enregistré sous `connect.params.auth.token`).

⚠️ **Avertissement Bun (WhatsApp + Telegram) :** Bun présente des problèmes connus avec ces canaux. Si vous utilisez WhatsApp ou Telegram, exécutez le Gateway avec **Node**.

<div id="35-quick-verify-2-min">
  ## 3.5) Vérification rapide (2 min)
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4) Associez et connectez votre premier canal de discussion
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp (connexion par code QR)
</div>

```bash
openclaw channels login
```

Scannez depuis WhatsApp → Settings → Linked Devices.

Documentation WhatsApp : [WhatsApp](/fr/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / autres
</div>

L’assistant de configuration peut renseigner les tokens et la configuration pour vous. Si vous préférez une configuration manuelle, commencez par :

* Telegram : [Telegram](/fr/channels/telegram)
* Discord : [Discord](/fr/channels/discord)
* Mattermost (plugin) : [Mattermost](/fr/channels/mattermost)

**Astuce Telegram en DM (message privé) :** lors de votre premier DM, vous recevrez un code d’appairage. Approuvez-le (voir l’étape suivante), sinon le bot ne répondra pas.

<div id="5-dm-safety-pairing-approvals">
  ## 5) Sécurité des MP (approbations d&#39;appairage)
</div>

Comportement par défaut : les MP inconnus reçoivent un code court et les messages ne sont pas traités tant qu&#39;ils n&#39;ont pas été approuvés.
Si votre premier MP ne reçoit pas de réponse, approuvez l&#39;appairage :

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

Doc d’appairage : [Appairage](/fr/start/pairing)

<div id="from-source-development">
  ## À partir des sources (développement)
</div>

Si vous travaillez directement sur OpenClaw, exécutez‑le à partir des sources :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'UI à la première exécution
pnpm build
openclaw onboard --install-daemon
```

Si vous n&#39;avez pas encore d&#39;installation globale, exécutez l&#39;étape d’onboarding via `pnpm openclaw ...` depuis le dépôt.
`pnpm build` regroupe également les ressources A2UI ; si vous avez uniquement besoin d&#39;exécuter cette étape, utilisez `pnpm canvas:a2ui:bundle`.

Gateway (depuis ce dépôt) :

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) Vérifiez de bout en bout
</div>

Dans un nouveau terminal, envoyez un message de test :

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

Si `openclaw health` affiche « no auth configured », revenez à l’assistant de configuration et définissez l’authentification OAuth/clé — l’agent ne pourra pas répondre sans authentification.

Astuce : `openclaw status --all` est le meilleur rapport de débogage en lecture seule, prêt à être copié-collé.
Sondes de santé : `openclaw health` (ou `openclaw status --deep`) interroge le Gateway en cours d’exécution pour obtenir un instantané de son état de santé.

<div id="next-steps-optional-but-great">
  ## Prochaines étapes (facultatif, mais recommandé)
</div>

* Application macOS pour la barre de menus + réveil vocal : [macOS app](/fr/platforms/macos)
* Nœuds iOS/Android (Canvas/appareil photo/voix) : [Nodes](/fr/nodes)
* Accès à distance (tunnel SSH / Tailscale Serve) : [Remote access](/fr/gateway/remote) et [Tailscale](/fr/gateway/tailscale)
* Configurations always-on / VPN : [Remote access](/fr/gateway/remote), [exe.dev](/fr/platforms/exe-dev), [Hetzner](/fr/platforms/hetzner), [macOS remote](/fr/platforms/mac/remote)