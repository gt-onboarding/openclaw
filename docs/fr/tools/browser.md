---
title: Navigateur
summary: "Service intégré de contrôle du navigateur + commandes d’action"
read_when:
  - Ajout d’une automatisation du navigateur contrôlée par un agent
  - Débogage de la raison pour laquelle openclaw interfère avec votre propre Chrome
  - Implémentation des paramètres du navigateur et de son cycle de vie dans l’app macOS
---

<div id="browser-openclaw-managed">
  # Navigateur (géré par openclaw)
</div>

OpenClaw peut exécuter un **profil Chrome/Brave/Edge/Chromium dédié** contrôlé par l’agent.
Il est isolé de votre navigateur personnel et est géré via un petit service de
contrôle local au sein du Gateway (boucle locale/loopback uniquement).

Vue débutant :

* Pensez-y comme à un **navigateur distinct, réservé à l’agent**.
* Le profil `openclaw` **n’interfère pas** avec le profil de votre navigateur personnel.
* L’agent peut **ouvrir des onglets, lire des pages, cliquer et saisir du texte** dans un environnement isolé.
* Le profil `chrome` par défaut utilise le **navigateur Chromium par défaut du système** via le
  relais d’extension ; basculez sur `openclaw` pour utiliser le navigateur géré et isolé.

<div id="what-you-get">
  ## Ce que vous obtenez
</div>

* Un profil de navigateur distinct nommé **openclaw** (accent orange par défaut).
* Un contrôle déterministe des onglets (lister/ouvrir/mettre au premier plan/fermer).
* Des actions d’agent (clic/saisie/glisser-déposer/sélection), des instantanés, des captures d’écran, des PDF.
* Prise en charge optionnelle de plusieurs profils (`openclaw`, `work`, `remote`, ...).

Ce navigateur n’est **pas** votre navigateur principal. Il constitue une surface sûre et isolée pour
l’automatisation et la vérification par les agents.

<div id="quick-start">
  ## Démarrage rapide
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Si le message « Browser disabled » s’affiche, activez-le dans la config (voir ci-dessous) et redémarrez le
Gateway.

<div id="profiles-openclaw-vs-chrome">
  ## Profils : `openclaw` vs `chrome`
</div>

* `openclaw` : navigateur géré et isolé (aucune extension requise).
* `chrome` : relais via extension vers votre **navigateur système** (nécessite que l’extension OpenClaw soit attachée à un onglet).

Définissez `browser.defaultProfile: "openclaw"` si vous voulez le mode géré par défaut.

<div id="configuration">
  ## Configuration
</div>

Les paramètres du navigateur sont définis dans `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true,                    // default: true
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    remoteCdpTimeoutMs: 1500,         // remote CDP HTTP timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // délai d'expiration de l'établissement de connexion WebSocket CDP distante (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

Notes :

* Le service de contrôle du navigateur écoute sur l’interface loopback, sur un port dérivé de `gateway.port`
  (par défaut : `18791`, soit gateway + 2). Le relais utilise le port suivant (`18792`).
* Si vous modifiez le port du Gateway (`gateway.port` ou `OPENCLAW_GATEWAY_PORT`),
  les ports dérivés du navigateur s’ajustent pour rester dans la même « famille ».
* `cdpUrl` pointe par défaut vers le port du relais s’il n’est pas renseigné.
* `remoteCdpTimeoutMs` s’applique aux vérifications d’accessibilité CDP distantes (non-loopback).
* `remoteCdpHandshakeTimeoutMs` s’applique aux vérifications d’accessibilité CDP WebSocket distantes.
* `attachOnly: true` signifie « ne jamais lancer un navigateur local ; uniquement s’y connecter s’il est déjà en cours d’exécution. »
* `color` + `color` par profil teintent l’UI du navigateur afin que vous puissiez voir quel profil est actif.
* Le profil par défaut est `chrome` (relais d’extension). Utilisez `defaultProfile: "openclaw"` pour le navigateur géré.
* Ordre de détection automatique : navigateur par défaut du système s’il est basé sur Chromium ; sinon Chrome → Brave → Edge → Chromium → Chrome Canary.
* Les profils `openclaw` locaux attribuent automatiquement `cdpPort`/`cdpUrl` — ne définissez ces valeurs que pour un CDP distant.

<div id="use-brave-or-another-chromium-based-browser">
  ## Utiliser Brave (ou un autre navigateur basé sur Chromium)
</div>

Si votre navigateur **par défaut du système** est basé sur Chromium (Chrome/Brave/Edge/etc),
OpenClaw l&#39;utilise automatiquement. Définissez `browser.executablePath` pour remplacer
la détection automatique :

Exemple de commande CLI :

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## Contrôle local vs distant
</div>

* **Contrôle local (par défaut):** le Gateway démarre le service de contrôle loopback et peut lancer un navigateur local.
* **Contrôle distant (hôte nœud):** exécutez un hôte de nœud sur la machine sur laquelle s’exécute le navigateur ; le Gateway y relaie les actions du navigateur.
* **CDP distant :** définissez `browser.profiles.<name>.cdpUrl` (ou `browser.cdpUrl`) pour
  vous connecter à un navigateur distant basé sur Chromium. Dans ce cas, OpenClaw ne lancera pas de navigateur local.

Les URL de CDP distant peuvent inclure une authentification :

* Jetons de requête (par ex. `https://provider.example?token=<token>`)
* Authentification HTTP Basic (par ex. `https://user:pass@provider.example`)

OpenClaw préserve l’authentification lors des appels aux endpoints `/json/*` et lors de la connexion
au WebSocket CDP. Privilégiez les variables d’environnement ou les gestionnaires de secrets pour
les jetons plutôt que de les enregistrer dans des fichiers de configuration.

<div id="node-browser-proxy-zero-config-default">
  ## Proxy de navigateur du nœud (valeur par défaut sans configuration)
</div>

Si vous exécutez un **hôte de nœud** sur la machine où se trouve votre navigateur, OpenClaw peut
acheminer automatiquement les appels d’outils de navigateur vers ce nœud sans aucune configuration supplémentaire du navigateur.
C’est le chemin par défaut pour les Gateways distants.

Remarques :

* L’hôte de nœud expose son serveur local de contrôle du navigateur via une **commande de proxy**.
* Les profils proviennent de la configuration `browser.profiles` propre au nœud (comme en local).
* Désactivez cette fonction si vous n’en voulez pas :
  * Sur le nœud : `nodeHost.browserProxy.enabled=false`
  * Sur la Gateway : `gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless (CDP hébergé à distance)
</div>

[Browserless](https://browserless.io) est un service Chromium hébergé qui expose
des points de terminaison CDP via HTTPS. Vous pouvez configurer un profil de navigateur OpenClaw pour utiliser un
point de terminaison régional Browserless et vous authentifier avec votre clé d&#39;API.

Exemple :

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

Remarques :

* Remplacez `<BROWSERLESS_API_KEY>` par votre jeton Browserless réel.
* Choisissez l’endpoint régional qui correspond à votre compte Browserless (voir leur documentation).

<div id="security">
  ## Sécurité
</div>

Principes clés :

* Le contrôle du navigateur est limité à la boucle locale ; les accès passent via l’authentification du Gateway ou l’appairage d’un nœud.
* Gardez le Gateway et tout hôte de nœud sur un réseau privé (Tailscale) ; évitez toute exposition publique.
* Considérez les URL/jetons CDP distants comme des secrets ; privilégiez les variables d’environnement ou un gestionnaire de secrets.

Conseils pour le CDP distant :

* Privilégiez les points de terminaison HTTPS et les jetons de courte durée lorsque c’est possible.
* Évitez d’intégrer directement des jetons de longue durée dans les fichiers de configuration.

<div id="profiles-multi-browser">
  ## Profils (multi-navigateurs)
</div>

OpenClaw prend en charge plusieurs profils nommés (configurations de routage). Les profils peuvent être :

* **openclaw-managed** : une instance dédiée de navigateur basé sur Chromium avec son propre répertoire de données utilisateur + port CDP
* **remote** : une URL CDP explicite (navigateur basé sur Chromium s’exécutant ailleurs)
* **extension relay** : vos onglets Chrome existants via le relais local + l’extension Chrome

Valeurs par défaut :

* Le profil `openclaw` est créé automatiquement s’il n’existe pas.
* Le profil `chrome` est intégré pour le relais d’extension Chrome (pointe par défaut vers `http://127.0.0.1:18792`).
* Les ports CDP locaux sont alloués de **18800 à 18899** par défaut.
* La suppression d’un profil déplace son répertoire de données local vers la Corbeille.

Tous les points de terminaison de contrôle acceptent `?profile=<name>` ; la CLI utilise `--browser-profile`.

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Relais via extension Chrome (utilisez votre Chrome existant)
</div>

OpenClaw peut aussi piloter **vos onglets Chrome existants** (aucune instance Chrome « openclaw » séparée) via un relais CDP local + une extension Chrome.

Guide complet : [Chrome extension](/fr/tools/chrome-extension)

Fonctionnement :

* Le Gateway s’exécute en local (même machine) ou un hôte de nœud s’exécute sur la machine du navigateur.
* Un **serveur de relais** local écoute sur une `cdpUrl` de loopback (par défaut : `http://127.0.0.1:18792`).
* Vous cliquez sur l’icône d’extension **OpenClaw Browser Relay** dans un onglet pour le raccorder (aucun rattachement automatique n’est effectué).
* L’agent contrôle cet onglet via l’outil `browser` standard, en sélectionnant le bon profil.

Si le Gateway s’exécute ailleurs, exécutez un hôte de nœud sur la machine du navigateur pour que le Gateway puisse relayer les actions du navigateur.

<div id="sandboxed-sessions">
  ### Sessions en sandbox
</div>

Si la session de l’agent est sandboxée, l’outil `browser` peut utiliser par défaut `target="sandbox"` (navigateur en sandbox).
La prise de contrôle via le relais de l’extension Chrome nécessite le contrôle du navigateur hôte, vous devez donc soit :

* exécuter la session hors sandbox, soit
* définir `agents.defaults.sandbox.browser.allowHostControl: true` et utiliser `target="host"` lors de l’appel de l’outil.

<div id="setup">
  ### Configuration
</div>

1. Chargez l’extension (mode dev / non empaquetée) :

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → activer le « Developer mode »
* « Load unpacked » → sélectionner le répertoire affiché par `openclaw browser extension path`
* Épinglez l’extension, puis cliquez dessus sur l’onglet que vous voulez contrôler (le badge indique `ON`).

2. Pour l’utiliser :

* CLI : `openclaw browser --browser-profile chrome tabs`
* Outil d’agent : `browser` avec `profile="chrome"`

Facultatif : si vous souhaitez un autre nom ou un port de relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Remarques :

* Ce mode s’appuie sur Playwright-on-CDP pour la plupart des opérations (captures d’écran/snapshots/actions).
* Pour détacher, cliquez de nouveau sur l’icône de l’extension.

<div id="isolation-guarantees">
  ## Garanties d’isolation
</div>

* **Répertoire de données utilisateur dédié** : n’interagit jamais avec votre profil de navigateur personnel.
* **Ports dédiés** : évite `9222` pour éviter les collisions avec vos workflows de développement.
* **Contrôle déterministe des onglets** : cible les onglets via leur `targetId`, et non le « dernier onglet ».

<div id="browser-selection">
  ## Sélection du navigateur
</div>

Lors d’un lancement local, OpenClaw choisit le premier disponible :

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Vous pouvez surcharger ce comportement avec `browser.executablePath`.

Plateformes :

* macOS : vérifie `/Applications` et `~/Applications`.
* Linux : recherche `google-chrome`, `brave`, `microsoft-edge`, `chromium`, etc.
* Windows : vérifie les emplacements d’installation courants.

<div id="control-api-optional">
  ## API de contrôle (facultative)
</div>

Uniquement pour les intégrations locales, le Gateway expose une petite API HTTP de loopback locale :

* Statut/démarrer/arrêter : `GET /`, `POST /start`, `POST /stop`
* Onglets : `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* Instantané/capture d’écran : `GET /snapshot`, `POST /screenshot`
* Actions : `POST /navigate`, `POST /act`
* Hooks : `POST /hooks/file-chooser`, `POST /hooks/dialog`
* Téléchargements : `POST /download`, `POST /wait/download`
* Débogage : `GET /console`, `POST /pdf`
* Débogage : `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* Réseau : `POST /response/body`
* État : `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* État : `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* Paramètres : `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Tous les endpoints acceptent le paramètre `?profile=<name>`.

<div id="playwright-requirement">
  ### Prérequis Playwright
</div>

Certaines fonctionnalités (navigation/action/instantané IA/instantané de rôle, captures d’éléments, PDF) nécessitent
Playwright. Si Playwright n’est pas installé, ces points de terminaison renvoient une erreur 501
explicite. Les instantanés ARIA et les captures d’écran élémentaires continuent de fonctionner pour Chrome géré par openclaw.
Pour le driver de relais de l’extension Chrome, les instantanés ARIA et les captures d’écran nécessitent Playwright.

Si vous voyez `Playwright is not available in this gateway build`, installez le package
Playwright complet (et non `playwright-core`), puis redémarrez le Gateway, ou réinstallez
OpenClaw avec la prise en charge du navigateur.

<div id="how-it-works-internal">
  ## Fonctionnement (interne)
</div>

Flux global :

* Un petit **serveur de contrôle** accepte les requêtes HTTP.
* Il se connecte aux navigateurs basés sur Chromium (Chrome/Brave/Edge/Chromium) via **CDP**.
* Pour les actions avancées (clic/saisie/capture/PDF), il utilise **Playwright** par-dessus
  CDP.
* Lorsque Playwright n’est pas disponible, seules les opérations ne dépendant pas de Playwright sont proposées.

Cette conception maintient l’agent sur une interface stable et déterministe tout en vous permettant
de basculer entre des navigateurs et profils locaux ou distants.

<div id="cli-quick-reference">
  ## Référence rapide de la CLI
</div>

Toutes les commandes acceptent `--browser-profile <name>` pour cibler un profil spécifique.
Toutes les commandes acceptent également `--json` pour une sortie lisible par machine (payloads stables).

Principes de base :

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

Inspection :

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

Actions :

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

État :

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

Remarques :

* `upload` et `dialog` sont des appels d’**armement** ; les exécuter avant le clic ou l’appui
  qui déclenche le sélecteur/la boîte de dialogue.
* `upload` peut aussi renseigner directement les champs de fichier via `--input-ref` ou `--element`.
* `snapshot` :
  * `--format ai` (valeur par défaut quand Playwright est installé) : renvoie un snapshot IA avec des refs numériques (`aria-ref="<n>"`).
  * `--format aria` : renvoie l’arbre d’accessibilité (aucune ref ; inspection uniquement).
  * `--efficient` (ou `--mode efficient`) : préréglage de snapshot basé sur les rôles, compact (interactif + compact + profondeur + maxChars plus faible).
  * Valeur par défaut de configuration (outil/CLI uniquement) : définir `browser.snapshotDefaults.mode: "efficient"` pour utiliser des snapshots efficients lorsque l’appelant ne fournit pas de mode (voir [Gateway configuration](/fr/gateway/configuration#browser-openclaw-managed-browser)).
  * Les options de snapshot basé sur les rôles (`--interactive`, `--compact`, `--depth`, `--selector`) imposent un snapshot basé sur les rôles avec des refs comme `ref=e12`.
  * `--frame "<iframe selector>"` limite la portée des snapshots basés sur les rôles à un iframe (apparié à des refs de rôle comme `e12`).
  * `--interactive` produit une liste plate, facile à parcourir, d’éléments interactifs (idéal pour piloter des actions).
  * `--labels` ajoute une capture d’écran limitée au viewport avec des étiquettes de refs superposées (affiche `MEDIA:<path>`).
* `click`/`type`/etc nécessitent une `ref` issue de `snapshot` (soit une ref numérique `12`, soit une ref de rôle `e12`).
  Les sélecteurs CSS ne sont volontairement pas pris en charge pour les actions.

<div id="snapshots-and-refs">
  ## Instantanés et références
</div>

OpenClaw prend en charge deux styles d’« instantanés » :

* **Instantané IA (références numériques)** : `openclaw browser snapshot` (par défaut ; `--format ai`)
  * Résultat : un instantané textuel qui inclut des références numériques.
  * Actions : `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  * En interne, la référence est résolue via `aria-ref` de Playwright.

* **Instantané par rôle (références de rôle comme `e12`)** : `openclaw browser snapshot --interactive` (ou `--compact`, `--depth`, `--selector`, `--frame`)
  * Résultat : une liste/un arbre basé sur les rôles avec `[ref=e12]` (et éventuellement `[nth=1]`).
  * Actions : `openclaw browser click e12`, `openclaw browser highlight e12`.
  * En interne, la référence est résolue via `getByRole(...)` (plus `nth()` pour les doublons).
  * Ajoutez `--labels` pour inclure une capture d’écran du viewport avec les étiquettes `e12` superposées.

Comportement des références :

* Les références **ne sont pas stables d’une navigation à l’autre** ; si quelque chose échoue, relancez `snapshot` et utilisez une nouvelle référence.
* Si l’instantané par rôle a été effectué avec `--frame`, les références de rôle sont limitées à cet iframe jusqu’au prochain instantané par rôle.

<div id="wait-power-ups">
  ## Fonctionnalités avancées d&#39;attente
</div>

Vous pouvez attendre sur d&#39;autres types de conditions que le simple temps ou le texte :

* Attendre une URL (globs pris en charge par Playwright) :
  * `openclaw browser wait --url "**/dash"`
* Attendre un état de chargement :
  * `openclaw browser wait --load networkidle`
* Attendre un prédicat JS :
  * `openclaw browser wait --fn "window.ready===true"`
* Attendre qu&#39;un sélecteur devienne visible :
  * `openclaw browser wait "#main"`

Ces options peuvent être combinées :

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## Déboguer les workflows
</div>

Lorsqu’une action échoue (par exemple « not visible », « strict mode violation », « covered ») :

1. `openclaw browser snapshot --interactive`
2. Utilisez `click <ref>` / `type <ref>` (privilégiez les refs de rôle en mode interactif)
3. Si cela échoue encore : `openclaw browser highlight <ref>` pour voir ce que Playwright cible
4. Si la page se comporte de manière anormale :
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. Pour un débogage approfondi : enregistrez une trace :
   * `openclaw browser trace start`
   * reproduisez le problème
   * `openclaw browser trace stop` (affiche `TRACE:<path>`)

<div id="json-output">
  ## Sortie JSON
</div>

`--json` est destiné au scripting et aux outils structurés.

Exemples :

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Les instantanés de rôle au format JSON incluent `refs` ainsi qu’un petit bloc `stats` (lines/chars/refs/interactive), afin que les outils puissent raisonner sur la taille et la densité du payload.

<div id="state-and-environment-knobs">
  ## Réglages d’état et d’environnement
</div>

Ces commandes sont utiles pour des workflows du type « faire en sorte que le site se comporte comme X » :

* Cookies : `cookies`, `cookies set`, `cookies clear`
* Stockage : `storage local|session get|set|clear`
* Mode hors ligne : `set offline on|off`
* En-têtes HTTP : `set headers --json '{"X-Debug":"1"}'` (ou `--clear`)
* Authentification HTTP basique : `set credentials user pass` (ou `--clear`)
* Géolocalisation : `set geo <lat> <lon> --origin "https://example.com"` (ou `--clear`)
* Média : `set media dark|light|no-preference|none`
* Fuseau horaire / paramètres régionaux : `set timezone ...`, `set locale ...`
* Appareil / viewport :
  * `set device "iPhone 14"` (préréglages d’appareils Playwright)
  * `set viewport 1280 720`

<div id="security-privacy">
  ## Sécurité &amp; vie privée
</div>

* Le profil de navigateur openclaw peut contenir des sessions authentifiées ; considérez-le comme une donnée sensible.
* `browser act kind=evaluate` / `openclaw browser evaluate` et `wait --fn`
  exécutent du JavaScript arbitraire dans le contexte de la page. Une injection de prompt peut en prendre le contrôle. Désactivez cette fonctionnalité avec `browser.evaluateEnabled=false` si vous n’en avez pas besoin.
* Pour les connexions et les mécanismes anti-bot (X/Twitter, etc.), voir [Connexion navigateur + publication X/Twitter](/fr/tools/browser-login).
* Gardez l’hôte Gateway/nœud privé (boucle locale ou uniquement via tailnet).
* Les endpoints CDP distants sont puissants ; faites-les passer par un tunnel et protégez-les.

<div id="troubleshooting">
  ## Dépannage
</div>

Pour les problèmes spécifiques à Linux (en particulier avec Chromium installé via snap), consultez
[Dépannage du navigateur](/fr/tools/browser-linux-troubleshooting).

<div id="agent-tools-how-control-works">
  ## Outils d’agent + fonctionnement du contrôle
</div>

L’agent dispose d’**un seul outil** pour l’automatisation du navigateur :

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

Correspondance :

* `browser snapshot` renvoie un arbre d’UI stable (AI ou ARIA).
* `browser act` utilise les identifiants `ref` du snapshot pour cliquer/saisir/faire glisser/sélectionner.
* `browser screenshot` capture les pixels (page complète ou élément).
* `browser` accepte :
  * `profile` pour choisir un profil de navigateur nommé (openclaw, chrome ou CDP distant).
  * `target` (`sandbox` | `host` | `node`) pour sélectionner où le navigateur s’exécute.
  * Dans les sessions sandboxées, `target: "host"` nécessite `agents.defaults.sandbox.browser.allowHostControl=true`.
  * Si `target` est omis : les sessions sandboxées utilisent par défaut `sandbox`, les sessions non sandboxées utilisent par défaut `host`.
  * Si un nœud doté de capacités de navigateur est connecté, l’outil peut y être automatiquement routé, sauf si vous forcez `target="host"` ou `target="node"`.

Cela rend l’agent déterministe et évite les sélecteurs fragiles.