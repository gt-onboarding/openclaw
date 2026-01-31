---
title: Appairage
summary: "Appairage des nœuds gérés par le Gateway (Option B) pour iOS et autres nœuds distants"
read_when:
  - Mettre en œuvre l’approbation d’appairage de nœuds sans UI macOS
  - Ajouter des flux CLI pour approuver des nœuds distants
  - Étendre le protocole du Gateway pour la gestion des nœuds
---

<div id="gateway-owned-pairing-option-b">
  # Appairage géré par le Gateway (Option B)
</div>

Dans l’appairage géré par le **Gateway**, le **Gateway** est la source de vérité
pour déterminer quels nœuds sont autorisés à se connecter. Les UI (app macOS, futurs
clients) ne sont que des frontends qui approuvent ou rejettent les requêtes en
attente.

**Important :** les nœuds WS utilisent l’**appairage d’appareil** (rôle `node`)
pendant `connect`. `node.pair.*` est un store d’appairage distinct et ne
conditionne **pas** le handshake WS. Seuls les clients qui appellent
explicitement `node.pair.*` utilisent ce flux.

<div id="concepts">
  ## Concepts
</div>

- **Demande en attente** : un nœud a demandé à rejoindre ; nécessite une approbation.
- **Nœud appairé** : nœud approuvé muni d’un jeton d’authentification.
- **Transport** : le point de terminaison WS du Gateway relaie les requêtes mais ne décide pas
  de l’appartenance. (La prise en charge de l’ancien pont TCP est obsolète et a été supprimée.)

<div id="how-pairing-works">
  ## Fonctionnement de l’appairage
</div>

1. Un nœud se connecte au WS du Gateway et demande un appairage.
2. Le Gateway enregistre une **demande en attente** et émet `node.pair.requested`.
3. Vous approuvez ou rejetez la demande (CLI ou UI).
4. En cas d’approbation, le Gateway délivre un **nouveau jeton** (les jetons sont renouvelés lors d’un nouvel appairage).
5. Le nœud se reconnecte à l’aide du jeton et est désormais « appairé ».

Les demandes en attente expirent automatiquement après **5 minutes**.

<div id="cli-workflow-headless-friendly">
  ## Flux de travail CLI (compatible avec les environnements sans interface graphique)
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` affiche la liste des nœuds appairés ou connectés et leurs capacités.


<div id="api-surface-gateway-protocol">
  ## Surface de l'API (protocole du Gateway)
</div>

Événements :

- `node.pair.requested` — émis lorsqu'une nouvelle demande en attente est créée.
- `node.pair.resolved` — émis lorsqu'une demande est approuvée/rejetée/expirée.

Méthodes :

- `node.pair.request` — crée ou réutilise une demande en attente.
- `node.pair.list` — répertorie les nœuds en attente et appariés.
- `node.pair.approve` — approuve une demande en attente (délivre un jeton).
- `node.pair.reject` — rejette une demande en attente.
- `node.pair.verify` — vérifie `{ nodeId, token }`.

Remarques :

- `node.pair.request` est idempotente pour un nœud donné : des appels répétés renvoient la même
  demande en attente.
- L'approbation génère **toujours** un nouveau jeton ; aucun jeton n'est jamais renvoyé par
  `node.pair.request`.
- Les demandes peuvent inclure `silent: true` comme indication pour les flux d'approbation automatique.

<div id="auto-approval-macos-app">
  ## Approbation automatique (app macOS)
</div>

L’app macOS peut, le cas échéant, tenter une **approbation silencieuse** lorsque :

- la requête est marquée `silent` ; et
- l’app peut vérifier une connexion SSH à l’hôte du Gateway en utilisant le même utilisateur.

Si l’approbation silencieuse échoue, elle repasse à l’invite normale « Approuver/Refuser ».

<div id="storage-local-private">
  ## Stockage (local, privé)
</div>

L'état d'appairage est stocké dans le répertoire d'état du Gateway (par défaut `~/.openclaw`) :

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Si vous redéfinissez `OPENCLAW_STATE_DIR`, le dossier `nodes/` est déplacé en même temps.

Notes de sécurité :

- Les jetons sont des secrets ; traitez `paired.json` comme un fichier sensible.
- Le renouvellement d'un jeton nécessite une nouvelle approbation (ou la suppression de l'entrée de nœud).

<div id="transport-behavior">
  ## Comportement du transport
</div>

- Le transport est **sans état** ; il ne conserve aucune information de membres.
- Si le Gateway est hors ligne ou si l’appairage est désactivé, les nœuds ne peuvent pas être appairés.
- Si le Gateway est en mode distant, l’appairage continue de s’effectuer sur le magasin de données du Gateway distant.