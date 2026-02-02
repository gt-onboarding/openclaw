---
title: Mémoire
summary: "Référence CLI pour `openclaw memory` (status/index/search)"
read_when:
  - Vous souhaitez indexer ou rechercher la mémoire sémantique
  - Vous diagnostiquez la disponibilité ou l’indexation de la mémoire
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

Gérer l’indexation et la recherche dans la mémoire sémantique.
Fourni par le plugin mémoire actif (par défaut : `memory-core` ; définissez `plugins.slots.memory = "none"` pour le désactiver).

Voir aussi :

* Concept de mémoire : [Mémoire](/fr/concepts/memory)
* Plugins : [Plugins](/fr/plugins)

<div id="examples">
  ## Exemples
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## Options
</div>

Général :

* `--agent <id>` : limite l’exécution à un seul agent (par défaut : tous les agents configurés).
* `--verbose` : produit une journalisation détaillée pendant les vérifications et l’indexation.

Notes :

* `memory status --deep` vérifie la disponibilité des vecteurs et des embeddings.
* `memory status --deep --index` exécute un réindexage si le store est marqué comme « dirty ».
* `memory index --verbose` affiche les détails par phase (fournisseur, modèle, sources, activité par lot).
* `memory status` inclut tous les chemins supplémentaires configurés via `memorySearch.extraPaths`.