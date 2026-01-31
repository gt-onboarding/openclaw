---
title: Compaction
summary: "Fenêtre de contexte et compaction : comment OpenClaw maintient les sessions sous les limites des modèles"
read_when:
  - Vous voulez comprendre la compaction automatique et /compact
  - Vous déboguez de longues sessions qui atteignent les limites de la fenêtre de contexte
---

<div id="context-window-compaction">
  # Fenêtre de contexte &amp; Compactage
</div>

Chaque modèle possède une **fenêtre de contexte** (nombre maximal de jetons qu’il peut prendre en compte). Les conversations de longue durée accumulent des messages et des résultats d’outils ; une fois que la fenêtre se rapproche de sa limite, OpenClaw **compacte** l’historique plus ancien pour rester dans les limites.

<div id="what-compaction-is">
  ## Ce qu’est la compaction
</div>

La compaction **résume les anciennes parties de la conversation** en une entrée de résumé concise et conserve les messages récents intacts. Le résumé est stocké dans l’historique de la session, de sorte que les requêtes futures utilisent :

* Le résumé de compaction
* Les messages récents après le point de compaction

La compaction est **conservée** dans l’historique JSONL de la session.

<div id="configuration">
  ## Configuration
</div>

Voir [Configuration et modes de compaction](/fr/concepts/compaction) pour les paramètres `agents.defaults.compaction`.

<div id="auto-compaction-default-on">
  ## Compactage automatique (activé par défaut)
</div>

Lorsqu’une session se rapproche ou dépasse la fenêtre de contexte du modèle, OpenClaw déclenche le compactage automatique et peut retenter la requête initiale en utilisant le contexte compacté.

Vous verrez :

* `🧹 Auto-compaction complete` en mode détaillé
* `/status` affichant `🧹 Compactions: <count>`

Avant le compactage, OpenClaw peut exécuter une étape de **vidage silencieux de la mémoire** pour stocker
des notes persistantes sur le disque. Voir [Mémoire](/fr/concepts/memory) pour plus de détails et de paramètres de configuration.

<div id="manual-compaction">
  ## Compactage manuel
</div>

Utilisez `/compact` (éventuellement avec des instructions) pour forcer une passe de compactage :

```
/compact Focus on decisions and open questions
```

<div id="context-window-source">
  ## Source de la fenêtre de contexte
</div>

La taille de la fenêtre de contexte est spécifique au modèle. OpenClaw utilise la définition du modèle issue du catalogue du fournisseur configuré pour déterminer les limites.

<div id="compaction-vs-pruning">
  ## Compaction vs élagage
</div>

* **Compaction** : résume et **stocke de manière persistante** en JSONL.
* **Élagage de session** : supprime uniquement les anciens **résultats d’outils**, **en mémoire**, par requête.

Voir [/concepts/session-pruning](/fr/concepts/session-pruning) pour plus de détails sur l’élagage.

<div id="tips">
  ## Conseils
</div>

* Utilisez `/compact` lorsque les sessions vous semblent obsolètes ou que le contexte est trop volumineux.
* Les sorties volumineuses des outils sont déjà tronquées ; un élagage supplémentaire peut encore réduire l&#39;accumulation de résultats d&#39;outils.
* Si vous avez besoin de repartir de zéro, `/new` ou `/reset` démarre un nouvel identifiant de session.