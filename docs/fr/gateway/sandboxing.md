---
summary: "Fonctionnement de la sandbox OpenClaw : modes, portées, accès à l’espace de travail et images"
title: Sandboxing
read_when: "Vous avez besoin d’une explication dédiée de la sandbox ou devez ajuster agents.defaults.sandbox."
status: active
---

<div id="sandboxing">
  # Sandboxing
</div>

OpenClaw peut exécuter des **outils dans des conteneurs Docker** pour réduire l’étendue des dégâts.
C’est **optionnel** et contrôlé par la configuration (`agents.defaults.sandbox` ou
`agents.list[].sandbox`). Si la fonctionnalité de sandbox est désactivée, les outils s’exécutent sur l’hôte.
Le Gateway reste sur l’hôte ; l’exécution des outils se fait dans un sandbox isolé
lorsque celui-ci est activé.

Ce n’est pas une frontière de sécurité parfaite, mais cela limite de manière significative l’accès au système de fichiers
et aux processus lorsque le modèle fait n’importe quoi.

<div id="what-gets-sandboxed">
  ## Ce qui est exécuté en sandbox
</div>

* Exécution d’outils (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, etc.).
* Navigateur optionnel en sandbox (`agents.defaults.sandbox.browser`).
  * Par défaut, le navigateur en sandbox se lance automatiquement (garantit que CDP est joignable) lorsque l’outil de navigateur en a besoin.
    Configurez ce comportement via `agents.defaults.sandbox.browser.autoStart` et `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  * `agents.defaults.sandbox.browser.allowHostControl` permet aux sessions en sandbox de cibler explicitement le navigateur de l’hôte.
  * Des listes d’autorisation optionnelles encadrent `target: "custom"` : `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Non exécuté en sandbox :

* Le processus Gateway lui-même.
* Tout outil explicitement autorisé à s’exécuter sur l’hôte (par exemple `tools.elevated`).
  * **L’exécution `exec` en mode élevé s’effectue sur l’hôte et contourne la sandbox.**
  * Si la sandbox est désactivée, `tools.elevated` ne change rien à l’exécution (déjà sur l’hôte). Voir [Elevated Mode](/fr/tools/elevated).

<div id="modes">
  ## Modes
</div>

`agents.defaults.sandbox.mode` contrôle **quand** le sandbox est utilisé :

* `"off"` : aucun sandbox.
* `"non-main"` : ne met en sandbox que les sessions **non-main** (valeur par défaut si vous voulez des discussions normales sur l’hôte).
* `"all"` : chaque session s’exécute dans un sandbox.
  Note : `"non-main"` est basé sur `session.mainKey` (par défaut `"main"`), et non sur l’identifiant de l’agent.
  Les sessions de groupe/canal utilisent leurs propres clés ; elles sont donc considérées comme non-main et seront mises en sandbox.

<div id="scope">
  ## Portée
</div>

`agents.defaults.sandbox.scope` contrôle **combien de conteneurs** sont créés :

* `"session"` (par défaut) : un conteneur par session.
* `"agent"` : un conteneur par agent.
* `"shared"` : un conteneur partagé par toutes les sessions sandboxées.

<div id="workspace-access">
  ## Accès à l’espace de travail
</div>

`agents.defaults.sandbox.workspaceAccess` contrôle **ce à quoi le sandbox peut accéder** :

* `"none"` (valeur par défaut) : les outils voient un espace de travail du sandbox sous `~/.openclaw/sandboxes`.
* `"ro"` : monte l’espace de travail de l’agent en lecture seule sur `/agent` (désactive `write`/`edit`/`apply_patch`).
* `"rw"` : monte l’espace de travail de l’agent en lecture/écriture sur `/workspace`.

Les contenus entrants sont copiés dans l’espace de travail du sandbox actif (`media/inbound/*`).
Remarque sur les compétences : l’outil `read` est ancré à la racine du sandbox. Avec `workspaceAccess: "none"`,
OpenClaw recopie les compétences éligibles dans l’espace de travail du sandbox (`.../skills`) afin
qu’elles puissent être lues. Avec `"rw"`, les compétences de l’espace de travail sont lisibles depuis
`/workspace/skills`.

<div id="custom-bind-mounts">
  ## Points de montage personnalisés
</div>

`agents.defaults.sandbox.docker.binds` monte des répertoires hôte supplémentaires dans le conteneur.
Format : `host:container:mode` (par exemple, `"/home/user/source:/source:rw"`).

Les montages globaux et ceux spécifiques à un agent sont **fusionnés** (et non remplacés). Avec `scope: "shared"`, les montages spécifiques à un agent sont ignorés.

Exemple (source en lecture seule + socket Docker) :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

Notes de sécurité :

* Les binds contournent le système de fichiers de la sandbox : ils exposent des chemins de l’hôte avec le mode que vous définissez (`:ro` ou `:rw`).
* Les montages sensibles (par exemple, `docker.sock`, secrets, clés SSH) devraient être en `:ro` sauf si un accès en écriture est absolument requis.
* Combinez avec `workspaceAccess: "ro"` si vous n’avez besoin que d’un accès en lecture seule à l’espace de travail ; les modes des binds restent indépendants.
* Voir [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated) pour comprendre comment les binds interagissent avec la politique des outils et l’exécution avec privilèges élevés.

<div id="images-setup">
  ## Images + mise en place
</div>

Image par défaut : `openclaw-sandbox:bookworm-slim`

Construire l’image une seule fois :

```bash
scripts/sandbox-setup.sh
```

Remarque : l’image par défaut **n’inclut pas** Node. Si une compétence a besoin de Node (ou
d’autres runtimes), vous pouvez soit créer une image personnalisée, soit l’installer via
`sandbox.docker.setupCommand` (nécessite un accès réseau sortant + une racine inscriptible +
l’utilisateur root).

Image de navigateur sandboxée :

```bash
scripts/sandbox-browser-setup.sh
```

Par défaut, les conteneurs de sandbox s&#39;exécutent **sans accès au réseau**.
Vous pouvez modifier ce comportement avec `agents.defaults.sandbox.docker.network`.

L&#39;installation de Docker et le Gateway conteneurisé sont présentés ici :
[Docker](/fr/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand (configuration unique du conteneur)
</div>

`setupCommand` s’exécute **une seule fois** après la création du conteneur sandbox (pas à chaque exécution).
Il s’exécute à l’intérieur du conteneur via `sh -lc`.

Chemins :

* Global : `agents.defaults.sandbox.docker.setupCommand`
* Par agent : `agents.list[].sandbox.docker.setupCommand`

Pièges courants :

* La valeur par défaut de `docker.network` est `"none"` (aucune sortie réseau), donc les installations de paquets échoueront.
* `readOnlyRoot: true` empêche les écritures ; définissez `readOnlyRoot: false` ou intégrez cela dans une image personnalisée.
* `user` doit être root pour les installations de paquets (omettez `user` ou définissez `user: "0:0"`).
* L’exécution dans la sandbox **n’hérite pas** du `process.env` de l’hôte. Utilisez
  `agents.defaults.sandbox.docker.env` (ou une image personnalisée) pour les clés API des skills.

<div id="tool-policy-escape-hatches">
  ## Politiques d’outils + échappatoires
</div>

Les politiques d’autorisation/interdiction des outils s’appliquent toujours avant les règles de sandbox. Si un outil est refusé
globalement ou par agent, la sandbox ne le réactivera pas.

`tools.elevated` est une échappatoire explicite qui exécute `exec` sur l’hôte.
Les directives `/exec` ne s’appliquent qu’aux expéditeurs autorisés et persistent par session ; pour désactiver complètement
`exec`, utilisez un refus dans la politique d’outils (voir [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated)).

Débogage :

* Utilisez `openclaw sandbox explain` pour inspecter le mode de sandbox effectivement appliqué, la politique d’outils et les clés de configuration de correction (« fix-it »).
* Voir [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated) pour le modèle mental « pourquoi est-ce bloqué ? ».
  Maintenez une configuration bien verrouillée.

<div id="multi-agent-overrides">
  ## Surcharges multi-agents
</div>

Chaque agent peut surcharger la sandbox et les outils :
`agents.list[].sandbox` et `agents.list[].tools` (ainsi que `agents.list[].tools.sandbox.tools` pour la politique des outils dans la sandbox).
Voir [Sandbox et outils multi-agents](/fr/multi-agent-sandbox-tools) pour l’ordre de priorité.

<div id="minimal-enable-example">
  ## Exemple minimal d’activation
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## Documentation associée
</div>

* [Configuration de la sandbox](/fr/gateway/configuration#agentsdefaults-sandbox)
* [Sandbox multi‑agents et outils](/fr/multi-agent-sandbox-tools)
* [Sécurité](/fr/gateway/security)