---
title: Nœud
summary: "Référence CLI pour `openclaw node` (hôte de nœud headless)"
read_when:
  - Lors de l'exécution de l'hôte de nœud headless
  - Lors de l'appairage d'un nœud non macOS pour system.run
---

<div id="openclaw-node">
  # `openclaw node`
</div>

Exécute un **hôte de nœud sans interface graphique** qui se connecte au WebSocket du Gateway et y expose
`system.run` / `system.which` pour cette machine.

<div id="why-use-a-node-host">
  ## Pourquoi utiliser un hôte de nœud ?
</div>

Utilisez un hôte de nœud lorsque vous voulez que des agents **exécutent des commandes sur d’autres machines** de votre
réseau sans y installer une application compagnon macOS complète.

Cas d’usage courants :

* Exécuter des commandes sur des machines Linux/Windows distantes (serveurs de build, machines de labo, NAS).
* Garder l’exécution **dans un sandbox** sur le Gateway, mais déléguer les exécutions approuvées à d’autres hôtes.
* Fournir une cible d’exécution légère et sans interface (headless) pour l’automatisation ou des nœuds CI.

L’exécution reste protégée par des **approbations exec** et des listes d’autorisation par agent sur l’hôte de nœud, ce qui vous permet de garder l’accès aux commandes bien délimité et explicite.

<div id="browser-proxy-zero-config">
  ## Proxy navigateur (zéro configuration)
</div>

Les hôtes exécutant un nœud annoncent automatiquement un proxy navigateur si `browser.enabled` n’est pas désactivé sur le nœud. Cela permet à l’agent d’utiliser l’automatisation du navigateur sur ce nœud sans configuration supplémentaire.

Désactivez-le sur le nœud si nécessaire :

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## Exécution (au premier plan)
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options :

* `--host <host>` : hôte WebSocket du Gateway (par défaut : `127.0.0.1`)
* `--port <port>` : port WebSocket du Gateway (par défaut : `18789`)
* `--tls` : utiliser TLS pour la connexion au Gateway
* `--tls-fingerprint <sha256>` : empreinte attendue du certificat TLS (sha256)
* `--node-id <id>` : remplacer l’identifiant du nœud (efface le jeton d’appairage)
* `--display-name <name>` : remplacer le nom d’affichage du nœud

<div id="service-background">
  ## Service (en arrière-plan)
</div>

Installez un hôte de nœud sans interface graphique en tant que service utilisateur.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Options :

* `--host <host>` : Hôte WebSocket du Gateway (par défaut : `127.0.0.1`)
* `--port <port>` : Port WebSocket du Gateway (par défaut : `18789`)
* `--tls` : Utiliser TLS pour la connexion au Gateway
* `--tls-fingerprint <sha256>` : Empreinte attendue du certificat TLS (sha256)
* `--node-id <id>` : Remplacer l’ID du node (efface le jeton d’appairage)
* `--display-name <name>` : Remplacer le nom d’affichage du node
* `--runtime <runtime>` : Environnement d’exécution du service (`node` ou `bun`)
* `--force` : Réinstaller/écraser si déjà installé

Gérer le service :

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Utilisez `openclaw node run` pour exécuter un hôte de nœud au premier plan (sans service).

Les commandes de service acceptent l’option `--json` pour une sortie lisible par des programmes.

<div id="pairing">
  ## Appairage
</div>

La première connexion crée, dans le Gateway, une demande d’appairage de nœud en attente.
Approuvez-la via :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

L’hôte du nœud enregistre son identifiant de nœud, son jeton, son nom d’affichage et ses informations de connexion au Gateway dans
`~/.openclaw/node.json`.

<div id="exec-approvals">
  ## Autorisations d&#39;exécution
</div>

`system.run` est soumis à des autorisations d&#39;exécution locales :

* `~/.openclaw/exec-approvals.json`
* [Autorisations d&#39;exécution](/fr/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>` (à modifier depuis le Gateway)