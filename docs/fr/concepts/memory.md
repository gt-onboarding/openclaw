---
title: Mémoire
summary: "Fonctionnement de la mémoire d’OpenClaw (fichiers d’espace de travail + vidage automatique de la mémoire)"
read_when:
  - Vous souhaitez connaître la structure des fichiers de mémoire et le flux de travail associé
  - Vous souhaitez ajuster le vidage automatique de la mémoire avant pré‑compactage
---

<div id="memory">
  # Mémoire
</div>

La mémoire d’OpenClaw est du **Markdown brut dans l’espace de travail de l’agent**. Les fichiers sont la
source de vérité ; le modèle ne « se souvient » que de ce qui est écrit sur le disque.

Les outils de recherche dans la mémoire sont fournis par le plugin de mémoire actif (par défaut :
`memory-core`). Pour désactiver les plugins de mémoire, définissez `plugins.slots.memory = "none"`.

<div id="memory-files-markdown">
  ## Fichiers de mémoire (Markdown)
</div>

La mise en page par défaut de l’espace de travail utilise deux niveaux de mémoire :

* `memory/YYYY-MM-DD.md`
  * Journal quotidien (écriture uniquement en ajout).
  * Lu pour aujourd’hui et hier au démarrage de la session.
* `MEMORY.md` (optionnel)
  * Mémoire à long terme organisée.
  * **À charger uniquement dans la session principale et privée** (jamais dans des contextes de groupe).

Ces fichiers se trouvent dans l’espace de travail (`agents.defaults.workspace`, par défaut
`~/.openclaw/workspace`). Voir [Espace de travail de l’Agent](/fr/concepts/agent-workspace) pour la structure complète.

<div id="when-to-write-memory">
  ## Quand écrire en mémoire
</div>

* Les décisions, préférences et informations factuelles persistantes vont dans `MEMORY.md`.
* Les notes quotidiennes et le contexte courant vont dans `memory/YYYY-MM-DD.md`.
* Si quelqu’un dit « garde ça en mémoire », écrivez-le (ne le laissez pas en RAM).
* Cet aspect est encore en évolution. Il est utile de rappeler régulièrement au modèle de stocker des éléments en mémoire ; il saura quoi faire.
* Si vous voulez que quelque chose reste, **demandez au bot de l’écrire** en mémoire.

<div id="automatic-memory-flush-pre-compaction-ping">
  ## Vidage automatique de la mémoire (ping pré-compaction)
</div>

Lorsqu&#39;une session est **sur le point d&#39;être auto-compactée**, OpenClaw déclenche un
**tour d&#39;agent silencieux** qui rappelle au modèle d&#39;écrire une mémoire
persistante **avant** que le contexte ne soit compacté. Les prompts par défaut
indiquent explicitement que le modèle *peut répondre*, mais en général
`NO_REPLY` est la bonne réponse afin que l&#39;utilisateur ne voie jamais ce tour.

Ce comportement est contrôlé par `agents.defaults.compaction.memoryFlush` :

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Écrivez toutes les notes durables dans memory/YYYY-MM-DD.md ; répondez avec NO_REPLY s'il n'y a rien à enregistrer."
        }
      }
    }
  }
}
```

Détails :

* **Seuil souple** : le vidage est déclenché lorsque l’estimation de jetons de la session dépasse
  `contextWindow - reserveTokensFloor - softThresholdTokens`.
* **Silencieux** par défaut : les prompts incluent `NO_REPLY` afin que rien ne soit envoyé.
* **Deux prompts** : un prompt utilisateur plus un prompt système qui ajoute le rappel.
* **Un vidage par cycle de compactage** (suivi dans `sessions.json`).
* **L’espace de travail doit être accessible en écriture** : si la session s’exécute dans un sandbox avec
  `workspaceAccess: "ro"` ou `"none"`, le vidage est ignoré.

Pour le cycle de vie complet du compactage, voir
[Gestion des sessions + compactage](/fr/reference/session-management-compaction).

<div id="vector-memory-search">
  ## Recherche dans la mémoire vectorielle
</div>

OpenClaw peut construire un petit index vectoriel sur `MEMORY.md` et `memory/*.md` (plus
tout répertoire ou fichier supplémentaire que vous activez explicitement) afin que des requêtes
sémantiques puissent trouver des notes liées même lorsque la formulation diffère.

Valeurs par défaut :

* Activée par défaut.
* Surveille les fichiers de mémoire pour détecter les modifications (avec temporisation / debounce).
* Utilise des embeddings distants par défaut. Si `memorySearch.provider` n’est pas défini, OpenClaw sélectionne automatiquement :
  1. `local` si un `memorySearch.local.modelPath` est configuré et que le fichier existe.
  2. `openai` si une clé OpenAI peut être résolue.
  3. `gemini` si une clé Gemini peut être résolue.
  4. Sinon, la recherche dans la mémoire reste désactivée tant qu’elle n’est pas configurée.
* Le mode local utilise node-llama-cpp et peut nécessiter `pnpm approve-builds`.
* Utilise sqlite-vec (lorsqu’il est disponible) pour accélérer la recherche vectorielle dans SQLite.

Les embeddings distants **requièrent** une clé d’API pour le fournisseur d’embeddings. OpenClaw
résout les clés à partir des profils d’authentification, de `models.providers.*.apiKey` ou des variables
d’environnement. Codex OAuth ne couvre que le chat/les complétions et **ne** satisfait **pas**
les embeddings pour la recherche dans la mémoire. Pour Gemini, utilisez `GEMINI_API_KEY` ou
`models.providers.google.apiKey`. Lorsque vous utilisez un endpoint personnalisé compatible OpenAI,
définissez `memorySearch.remote.apiKey` (et éventuellement `memorySearch.remote.headers`).

<div id="additional-memory-paths">
  ### Chemins de mémoire supplémentaires
</div>

Si vous souhaitez indexer des fichiers Markdown en dehors de la structure par défaut de l’espace de travail, ajoutez
des chemins explicites :

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Remarques :

* Les chemins peuvent être absolus ou relatifs à l’espace de travail.
* Les répertoires sont analysés récursivement à la recherche de fichiers `.md`.
* Seuls les fichiers Markdown sont indexés.
* Les liens symboliques sont ignorés (fichiers ou répertoires).

<div id="gemini-embeddings-native">
  ### Embeddings Gemini (natifs)
</div>

Configurez le fournisseur sur `gemini` pour utiliser directement l&#39;api d&#39;embeddings Gemini :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Remarques :

* `remote.baseUrl` est facultatif (par défaut, l’URL de base de l’API Gemini est utilisée).
* `remote.headers` vous permet d’ajouter des en-têtes supplémentaires si nécessaire.
* Modèle par défaut : `gemini-embedding-001`.

Si vous voulez utiliser un **endpoint personnalisé compatible OpenAI** (OpenRouter, vLLM ou un proxy),
vous pouvez utiliser la configuration `remote` avec le fournisseur OpenAI :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Si vous ne voulez pas définir de clé API, utilisez `memorySearch.provider = "local"` ou définissez
`memorySearch.fallback = "none"`.

Mécanismes de repli :

* `memorySearch.fallback` peut être `openai`, `gemini`, `local` ou `none`.
* Le fournisseur de repli n’est utilisé que lorsque le fournisseur d’embeddings principal échoue.

Indexation par lots (OpenAI + Gemini) :

* Activée par défaut pour les embeddings OpenAI et Gemini. Définissez `agents.defaults.memorySearch.remote.batch.enabled = false` pour la désactiver.
* Le comportement par défaut attend la fin du traitement par lots ; ajustez `remote.batch.wait`, `remote.batch.pollIntervalMs` et `remote.batch.timeoutMinutes` si nécessaire.
* Définissez `remote.batch.concurrency` pour contrôler combien de traitements par lots sont soumis en parallèle (valeur par défaut : 2).
* Le mode batch s’applique lorsque `memorySearch.provider = "openai"` ou `"gemini"` et utilise la clé API correspondante.
* Les traitements par lots Gemini utilisent l’endpoint asynchrone d’embeddings par lots et nécessitent la disponibilité de l’API Gemini Batch.

Pourquoi le traitement par lots OpenAI est rapide et peu coûteux :

* Pour les gros backfills, OpenAI est généralement l’option la plus rapide que nous prenons en charge, car nous pouvons soumettre de nombreuses requêtes d’embedding dans un seul traitement par lots et laisser OpenAI les traiter de manière asynchrone.
* OpenAI propose des tarifs réduits pour les charges de travail utilisant la Batch API, donc les grosses exécutions d’indexation sont généralement moins chères que d’envoyer les mêmes requêtes de façon synchrone.
* Consultez la documentation et la tarification de la Batch API OpenAI pour plus de détails :
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

Exemple de configuration :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Outils :

* `memory_search` — renvoie des extraits avec le fichier et les plages de lignes.
* `memory_get` — lit le contenu d’un fichier de mémoire par chemin.

Mode local :

* Définissez `agents.defaults.memorySearch.provider = "local"`.
* Indiquez `agents.defaults.memorySearch.local.modelPath` (GGUF ou URI `hf:`).
* Facultatif : définissez `agents.defaults.memorySearch.fallback = "none"` pour éviter le repli sur un service distant.

<div id="how-the-memory-tools-work">
  ### Fonctionnement des outils de mémoire
</div>

* `memory_search` effectue une recherche sémantique dans des blocs Markdown (~400 jetons visés, avec 80 jetons de chevauchement) issus de `MEMORY.md` + `memory/**/*.md`. Il renvoie un extrait de texte (limité à ~700 caractères), le chemin du fichier, l’intervalle de lignes, le score, le fournisseur/modèle, ainsi que l’indication d’un éventuel repli des embeddings locaux vers des embeddings distants. Aucun fichier complet n’est renvoyé.
* `memory_get` lit un fichier Markdown de mémoire spécifique (relatif à l’espace de travail), éventuellement à partir d’une ligne de départ et pour N lignes. Les chemins en dehors de `MEMORY.md` / `memory/` ne sont autorisés que lorsqu’ils sont explicitement listés dans `memorySearch.extraPaths`.
* Les deux outils ne sont activés que lorsque `memorySearch.enabled` est évalué à `true` pour l’agent.

<div id="what-gets-indexed-and-when">
  ### Ce qui est indexé (et quand)
</div>

* Type de fichier : uniquement Markdown (`MEMORY.md`, `memory/**/*.md`, ainsi que tous les fichiers `.md` sous `memorySearch.extraPaths`).
* Stockage de l’index : SQLite par agent à `~/.openclaw/memory/<agentId>.sqlite` (configurable via `agents.defaults.memorySearch.store.path`, prend en charge le jeton `{agentId}`).
* Fraîcheur : un watcher sur `MEMORY.md`, `memory/` et `memorySearch.extraPaths` marque l’index comme obsolète (délai d’anti-rebond de 1,5 s). La synchronisation est planifiée au démarrage de la session, lors d’une recherche ou à intervalle régulier, et s’exécute de manière asynchrone. Les transcriptions de session utilisent des seuils de delta pour déclencher une synchronisation en arrière-plan.
* Déclencheurs de réindexation : l’index stocke le **fournisseur/modèle + empreinte du point de terminaison + paramètres de découpage (chunking)**. Si l’un de ces éléments change, OpenClaw réinitialise et réindexe automatiquement l’intégralité du magasin d’index.

<div id="hybrid-search-bm25-vector">
  ### Recherche hybride (BM25 + vecteur)
</div>

Lorsqu’elle est activée, OpenClaw combine :

* **Similarité vectorielle** (correspondance sémantique, la formulation peut différer)
* **Pertinence des mots-clés BM25** (jetons exacts comme les IDs, variables d’environnement, symboles de code)

Si la recherche en texte intégral n’est pas disponible sur votre plateforme, OpenClaw revient à une recherche uniquement vectorielle.

<div id="why-hybrid">
  #### Pourquoi hybride ?
</div>

La recherche vectorielle est excellente pour retrouver « ça signifie la même chose » :

* « Mac Studio gateway host » vs « la machine qui exécute le Gateway »
* « debounce file updates » vs « éviter l’indexation à chaque écriture »

Mais elle peut être faible sur les tokens/tokens exacts à fort signal :

* IDs (`a828e60`, `b3b9895a…`)
* symboles de code (`memorySearch.query.hybrid`)
* messages d’erreur (« sqlite-vec unavailable »)

BM25 (recherche en plein texte) est l’inverse : très bon sur les tokens exacts, plus faible sur les reformulations.
La recherche hybride est le compromis pragmatique : **exploitant les deux signaux de recherche**, tu obtiens
de bons résultats aussi bien pour les requêtes en « langage naturel » que pour les requêtes « aiguille dans une botte de foin ».

<div id="how-we-merge-results-the-current-design">
  #### Comment nous fusionnons les résultats (conception actuelle)
</div>

Esquisse d’implémentation :

1. Récupérer un ensemble de candidats des deux côtés :

* **Vector** : les `maxResults * candidateMultiplier` premiers par similarité cosinus.
* **BM25** : les `maxResults * candidateMultiplier` premiers par rang FTS5 BM25 (plus le rang est bas, mieux c’est).

2. Convertir le rang BM25 en un score approximativement entre 0 et 1 :

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. Fusionner les candidats par identifiant de chunk et calculer un score pondéré :

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Notes :

* `vectorWeight` + `textWeight` est normalisé à 1,0 lors de la résolution de la config, donc les poids se comportent comme des pourcentages.
* Si les embeddings ne sont pas disponibles (ou si le fournisseur renvoie un vecteur nul), nous exécutons quand même BM25 et renvoyons les correspondances par mots-clés.
* Si FTS5 ne peut pas être créé, nous conservons la recherche uniquement vectorielle (pas d’échec bloquant).

Ce n’est pas « parfait du point de vue de la théorie de la recherche d’information », mais c’est simple, rapide, et cela a tendance à améliorer le rappel et la précision sur des notes réelles.
Si l’on veut faire plus sophistiqué plus tard, les étapes classiques sont la Reciprocal Rank Fusion (RRF) ou une normalisation des scores
(min/max ou z-score) avant le mélange.

Config :

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4
        }
      }
    }
  }
}
```

<div id="embedding-cache">
  ### Cache des embeddings
</div>

OpenClaw peut mettre en cache les **embeddings de segments** dans SQLite afin que le réindexage et les mises à jour fréquentes (en particulier les transcriptions de session) ne recalculent pas les embeddings du texte inchangé.

Config :

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

<div id="session-memory-search-experimental">
  ### Recherche dans la mémoire de session (expérimental)
</div>

Vous pouvez optionnellement indexer les **transcriptions de session** et les rendre accessibles via `memory_search`.
Cette fonctionnalité est activée par un indicateur expérimental.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Notes :

* L’indexation de session est **à activer explicitement** (désactivée par défaut).
* Les mises à jour de session sont regroupées (debounced) et **indexées de façon asynchrone** une fois qu’elles dépassent les seuils de delta (*best-effort*).
* `memory_search` ne se bloque jamais sur l’indexation ; les résultats peuvent être légèrement obsolètes tant que la synchronisation en arrière-plan n’est pas terminée.
* Les résultats ne contiennent toujours que des extraits ; `memory_get` reste limité aux fichiers de mémoire.
* L’indexation de session est isolée par agent (seuls les journaux de session de cet agent sont indexés).
* Les journaux de session sont stockés sur le disque (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Tout processus ou utilisateur disposant d’un accès au système de fichiers peut les read ; considérez donc l’accès disque comme la frontière de confiance. Pour une isolation plus stricte, exécutez les agents sous des utilisateurs ou des hôtes OS distincts.

Seuils de delta (valeurs par défaut indiquées) :

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // lignes JSONL
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### Accélération vectorielle SQLite (sqlite-vec)
</div>

Lorsque l’extension sqlite-vec est disponible, OpenClaw stocke les embeddings dans une
table virtuelle SQLite (`vec0`) et exécute les requêtes de distance vectorielle directement dans la
base de données. Cela permet de conserver une recherche rapide sans charger chaque embedding dans JS.

Configuration (optionnelle) :

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Notes :

* `enabled` est défini sur true par défaut ; lorsqu’il est désactivé, la recherche revient à une similarité cosinus intra‑processus sur les embeddings stockés.
* Si l’extension sqlite-vec est manquante ou ne parvient pas à se charger, OpenClaw consigne l’erreur et continue avec la solution de repli en JS (aucune table de vecteurs).
* `extensionPath` remplace le chemin sqlite-vec fourni (utile pour les compilations personnalisées ou les emplacements d’installation non standard).

<div id="local-embedding-auto-download">
  ### Téléchargement automatique des embeddings locaux
</div>

* Modèle d&#39;embedding local par défaut : `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf` (~0,6 Go).
* Lorsque `memorySearch.provider = "local"`, `node-llama-cpp` résout `modelPath` ; si le fichier GGUF manque, il le **télécharge automatiquement** dans le cache (ou dans `local.modelCacheDir` si défini), puis le charge. Les téléchargements reprennent automatiquement en cas de nouvelle tentative.
* Exigence pour la compilation native : exécutez `pnpm approve-builds`, sélectionnez `node-llama-cpp`, puis exécutez `pnpm rebuild node-llama-cpp`.
* Repli : si la configuration locale échoue et que `memorySearch.fallback = "openai"`, nous basculons automatiquement vers des embeddings distants (`openai/text-embedding-3-small` sauf remplacement explicite) et enregistrons la raison.

<div id="custom-openai-compatible-endpoint-example">
  ### Exemple d’endpoint personnalisé compatible avec OpenAI
</div>

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Notes :

* `remote.*` a la priorité sur `models.providers.openai.*`.
* `remote.headers` est fusionné avec les en-têtes OpenAI ; les valeurs de `remote.headers` l’emportent en cas de conflit de clés. Omettez `remote.headers` pour utiliser les valeurs par défaut d’OpenAI.
