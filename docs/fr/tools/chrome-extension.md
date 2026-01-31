---
title: Extension pour Chrome
summary: "Extension Chrome : laissez OpenClaw piloter votre onglet Chrome existant"
read_when:
  - Vous souhaitez que l'agent pilote un onglet Chrome existant (bouton de la barre d’outils)
  - Vous avez besoin d'un Gateway distant + d’une automatisation locale du navigateur via Tailscale
  - Vous souhaitez comprendre les implications de sécurité de la prise de contrôle du navigateur
---

<div id="chrome-extension-browser-relay">
  # Extension Chrome (relais via le navigateur)
</div>

L’extension Chrome OpenClaw permet à l’agent de contrôler vos **onglets Chrome existants** (votre fenêtre Chrome habituelle) au lieu de lancer un profil Chrome distinct géré par openclaw.

L’attache/détache se fait via un **unique bouton dans la barre d’outils de Chrome**.

<div id="what-it-is-concept">
  ## De quoi s’agit-il (concept)
</div>

Il y a trois éléments :

* **Service de contrôle du navigateur** (Gateway ou nœud) : l’API que l’agent/l’outil appelle (via la Gateway)
* **Serveur de relais local** (CDP en loopback) : fait le pont entre le serveur de contrôle et l’extension (`http://127.0.0.1:18792` par défaut)
* **Extension Chrome MV3** : se connecte à l’onglet actif via `chrome.debugger` et achemine les messages CDP vers le relais

OpenClaw contrôle ensuite l’onglet connecté via l’interface d’outil `browser` habituelle (en sélectionnant le bon profil).

<div id="install-load-unpacked">
  ## Installation / chargement (non empaquetée)
</div>

1. Installez l’extension dans un chemin local stable :

```bash
openclaw browser extension install
```

2. Affichez le chemin du répertoire de l’extension installée :

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* Activez le « mode développeur »
* « Load unpacked » → sélectionnez le répertoire indiqué ci-dessus

4. Épinglez l’extension.

<div id="updates-no-build-step">
  ## Mises à jour (sans étape de build)
</div>

L’extension est fournie avec la version OpenClaw (package npm) sous forme de fichiers statiques. Il n’y a pas d’étape de « build » séparée.

Après la mise à jour d’OpenClaw :

* Réexécutez `openclaw browser extension install` pour actualiser les fichiers installés dans votre répertoire d’état OpenClaw.
* Chrome → `chrome://extensions` → cliquez sur « Reload » sur l’extension.

<div id="use-it-no-extra-config">
  ## L’utiliser (sans configuration supplémentaire)
</div>

OpenClaw est livré avec un profil de navigateur intégré nommé `chrome` qui cible le relais d’extension sur le port par défaut.

Utilisez-le :

* CLI : `openclaw browser --browser-profile chrome tabs`
* Outil d’agent : `browser` avec `profile="chrome"`

Si vous souhaitez un nom différent ou un port de relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## Attacher / détacher (bouton de barre d’outils)
</div>

* Ouvrez l’onglet qu’OpenClaw doit contrôler.
* Cliquez sur l’icône de l’extension.
  * Le badge affiche `ON` lorsqu’il est attaché.
* Cliquez à nouveau pour détacher.

<div id="which-tab-does-it-control">
  ## Quel onglet contrôle-t-elle ?
</div>

* Elle ne contrôle **pas** automatiquement « l’onglet que vous êtes en train de consulter ».
* Elle contrôle **uniquement l’onglet (ou les onglets) que vous avez explicitement associés** en cliquant sur le bouton de la barre d’outils.
* Pour basculer : ouvrez l’autre onglet et cliquez sur l’icône de l’extension dans cet onglet.

<div id="badge-common-errors">
  ## Badge + erreurs courantes
</div>

* `ON` : connecté ; OpenClaw peut piloter cet onglet.
* `…` : connexion au relais local en cours.
* `!` : relais inaccessible (le cas le plus fréquent : le serveur de relais du navigateur n’est pas en cours d’exécution sur cette machine).

Si vous voyez `!` :

* Assurez-vous que le Gateway s’exécute en local (configuration par défaut), ou exécutez un hôte de nœud sur cette machine si le Gateway tourne ailleurs.
* Ouvrez la page d’options de l’extension ; elle indique si le relais est accessible.

<div id="remote-gateway-use-a-node-host">
  ## Gateway à distance (avec un hôte nœud)
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### Gateway local (même machine que Chrome) — en général **aucune étape supplémentaire**
</div>

Si le Gateway s’exécute sur la même machine que Chrome, il démarre le service de contrôle du navigateur sur l’interface loopback
et lance automatiquement le serveur relais. L’extension communique avec le relais local ; les appels CLI/outil sont envoyés au Gateway.

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### Gateway distant (Gateway s’exécute ailleurs) — **exécuter un hôte de nœud**
</div>

Si votre Gateway s’exécute sur une autre machine, démarrez un hôte de nœud sur la machine qui exécute Chrome.
La Gateway achemine les actions du navigateur vers ce nœud ; l’extension et le relais restent locaux à la machine du navigateur.

Si plusieurs nœuds sont connectés, épinglez-en un avec `gateway.nodes.browser.node` ou définissez `gateway.nodes.browser.mode`.

<div id="sandboxing-tool-containers">
  ## Mise en sandbox (conteneurs d’outils)
</div>

Si la session de votre agent est en sandbox (`agents.defaults.sandbox.mode != "off"`), l’outil `browser` peut être restreint :

* Par défaut, les sessions en sandbox ciblent souvent le **navigateur en sandbox** (`target="sandbox"`), et non votre Chrome hôte.
* La prise de contrôle via le relais de l’extension Chrome nécessite de contrôler le serveur de contrôle du navigateur **hôte**.

Options :

* Le plus simple : utiliser l’extension depuis une session/un agent **sans sandbox**.
* Ou autoriser le contrôle du navigateur hôte pour les sessions en sandbox :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Assurez-vous ensuite que l’outil n’est pas bloqué par la politique des outils et, si nécessaire, appelez `browser` avec `target="host"`.

Débogage : `openclaw sandbox explain`

<div id="remote-access-tips">
  ## Conseils pour l’accès à distance
</div>

* Gardez le Gateway et l’hôte du nœud sur le même tailnet ; évitez d’exposer les ports de relais au LAN ou à l’Internet public.
* Associez les nœuds de manière explicite ; désactivez le routage via le proxy du navigateur si vous ne voulez pas permettre le contrôle à distance (`gateway.nodes.browser.mode="off"`).

<div id="how-extension-path-works">
  ## Fonctionnement de « extension path »
</div>

`openclaw browser extension path` affiche le répertoire **installé** sur le disque qui contient les fichiers de l’extension.

La CLI n’affiche **pas** délibérément un chemin `node_modules`. Exécutez toujours d’abord `openclaw browser extension install` pour copier l’extension dans un emplacement stable sous votre répertoire d’état OpenClaw.

Si vous déplacez ou supprimez ce répertoire d’installation, Chrome marquera l’extension comme corrompue jusqu’à ce que vous la rechargiez depuis un chemin valide.

<div id="security-implications-read-this">
  ## Implications de sécurité (à lire)
</div>

C’est puissant et risqué. Considérez cela comme si vous donniez au modèle le contrôle direct de votre navigateur.

* L’extension utilise l’API de débogage de Chrome (`chrome.debugger`). Lorsqu’elle est attachée, le modèle peut :
  * cliquer/taper/naviguer dans cet onglet
  * lire le contenu de la page
  * accéder à tout ce à quoi la session connectée de l’onglet a accès
* **Ce n’est pas isolé** comme le profil dédié géré par openclaw.
  * Si vous l’attachez à votre profil/onglet principal au quotidien, vous accordez l’accès à l’état de ce compte.

Recommandations :

* Préférez un profil Chrome dédié (séparé de votre navigation personnelle) pour l’utilisation du relais d’extension.
* Gardez la Gateway et tout hôte de nœud accessibles uniquement via votre Tailnet ; comptez sur l’authentification de la Gateway + l’appairage des nœuds.
* Évitez d’exposer les ports de relais sur le LAN (`0.0.0.0`) et évitez Funnel (public).

Liens associés :

* Vue d’ensemble de l’outil de navigation : [Browser](/fr/tools/browser)
* Audit de sécurité : [Security](/fr/gateway/security)
* Configuration Tailscale : [Tailscale](/fr/gateway/tailscale)