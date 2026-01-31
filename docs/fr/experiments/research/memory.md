---
title: Mémoire
summary: "Notes de recherche : système de mémoire hors connexion pour les espaces de travail Clawd (Markdown comme source de vérité + index dérivé)"
read_when:
  - Conception de la mémoire de l’espace de travail (~/.openclaw/workspace) au-delà des journaux quotidiens en Markdown
  - Décider : CLI autonome ou intégration étroite à OpenClaw
  - Ajout de capacités de rappel et de réflexion hors connexion (retain/recall/reflect)
---

<div id="workspace-memory-v2-offline-research-notes">
  # Mémoire de l’espace de travail v2 (hors ligne) : notes de recherche
</div>

Cible : espace de travail de type Clawd (`agents.defaults.workspace`, par défaut `~/.openclaw/workspace`) où la « mémoire » est stockée sous la forme d’un fichier Markdown par jour (`memory/YYYY-MM-DD.md`), plus un petit ensemble de fichiers stables (par exemple `memory.md`, `SOUL.md`).

Ce document propose une architecture de mémoire **offline-first** qui conserve Markdown comme source de vérité canonique et révisable, mais ajoute un **rappel structuré** (recherche, résumés d’entités, mises à jour du niveau de confiance) via un index dérivé.

<div id="why-change">
  ## Pourquoi changer ?
</div>

La configuration actuelle (un fichier par jour) est excellente pour :

- la journalisation « append-only »
- l’édition manuelle
- la durabilité + l’auditabilité via git
- une capture à faible friction (« il suffit de l’écrire quelque part »)

Elle est moins adaptée pour :

- la recherche à fort rappel (« qu’avons-nous décidé concernant X ? », « quelles étaient nos conclusions la dernière fois que nous avons essayé Y ? »)
- les réponses centrées sur les entités (« parle-moi d’Alice / The Castle / warelay ») sans avoir à relire de nombreux fichiers
- la stabilité des opinions/préférences (et les éléments à l’appui lorsqu’elles évoluent)
- les contraintes temporelles (« qu’est-ce qui était vrai en novembre 2025 ? ») et la résolution de conflits

<div id="design-goals">
  ## Objectifs de conception
</div>

- **Hors ligne** : fonctionne sans réseau ; peut tourner sur un laptop/Castle ; aucune dépendance au cloud.
- **Traçable** : les éléments récupérés doivent être attribuables (fichier + emplacement) et séparables de l’inférence.
- **Sans formalisme excessif** : la journalisation quotidienne reste en Markdown, sans gros travail de schéma.
- **Incrémentiel** : la v1 est déjà utile avec la seule recherche plein texte (FTS) ; le sémantique/vectoriel et les graphes sont des améliorations optionnelles.
- **Adapté aux agents** : facilite le « rappel dans les budgets de jetons » (renvoyer de petits paquets de faits).

<div id="north-star-model-hindsight-letta">
  ## Modèle North Star (Hindsight × Letta)
</div>

Deux éléments à combiner :

1) **Boucle de contrôle de type Letta/MemGPT**

- garder un petit « noyau » toujours en contexte (persona + faits clés sur l’utilisateur)
- tout le reste est hors contexte et récupéré via des outils
- les écritures en mémoire sont des appels d’outils explicites (append/replace/insert), persistées, puis réinjectées au tour suivant

2) **Substrat mémoire de type Hindsight**

- séparer ce qui est observé, ce qui est cru et ce qui est résumé
- prendre en charge retain/recall/reflect
- opinions assorties d’un niveau de confiance pouvant évoluer avec les preuves
- récupération sensible aux entités + requêtes temporelles (même sans graphes de connaissances complets)

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## Architecture proposée (Markdown comme source de vérité + index dérivé)
</div>

<div id="canonical-store-git-friendly">
  ### Stockage canonique (compatible avec Git)
</div>

Conservez `~/.openclaw/workspace` comme mémoire canonique lisible par un humain.

Disposition de l&#39;espace de travail suggérée :

```
~/.openclaw/workspace/
  memory.md                    # petit : faits durables + préférences (essentiel)
  memory/
    YYYY-MM-DD.md              # journal quotidien (ajout ; narratif)
  bank/                        # pages mémoire « typées » (stables, révisables)
    world.md                   # faits objectifs sur le monde
    experience.md              # ce qu'a fait l'agent (première personne)
    opinions.md                # préférences/jugements subjectifs + confiance + pointeurs de preuves
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes :

* **Le log quotidien reste un log quotidien**. Pas besoin de le transformer en JSON.
* Les fichiers `bank/` sont **curés/sélectionnés**, produits par des jobs de réflexion, et peuvent toujours être modifiés à la main.
* `memory.md` reste « petit + central » : les éléments que tu veux que Clawd voie à chaque session.


<div id="derived-store-machine-recall">
  ### Magasin dérivé (rappel machine)
</div>

Ajoutez un index dérivé dans l’espace de travail (pas nécessairement suivi dans Git) :

```
~/.openclaw/workspace/.memory/index.sqlite
```

Appuyez-le sur :

* un schéma SQLite pour les faits + liens d&#39;entités + métadonnées d&#39;opinion
* SQLite **FTS5** pour le rappel lexical (rapide, léger, hors ligne)
* une table d&#39;embeddings optionnelle pour le rappel sémantique (toujours hors ligne)

L&#39;index peut toujours être **reconstruit à partir de Markdown**.


<div id="retain-recall-reflect-operational-loop">
  ## Retenir / Rappeler / Réfléchir (boucle opérationnelle)
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### Retain : normaliser les journaux quotidiens en « faits »
</div>

L’intuition clé de Hindsight qui importe ici : stocker des **faits narratifs et autonomes**, pas des micro-extraits.

Règle pratique pour `memory/YYYY-MM-DD.md` :

* en fin de journée (ou pendant), ajoute une section `## Retain` avec 2 à 5 éléments qui sont :
  * narratifs (contexte multi-échanges préservé)
  * autonomes (compréhensibles de manière autonome plus tard)
  * étiquetés avec un type + les entités mentionnées

Exemple :

```
## Conserver
- W @Peter : Actuellement à Marrakech (27 nov.–1er déc. 2025) pour l'anniversaire d'Andy.
- B @warelay : J'ai corrigé le crash WS de Baileys en enveloppant les gestionnaires connection.update dans try/catch (voir memory/2025-11-27.md).
- O(c=0.95) @Peter : Préfère les réponses concises (&lt;1500 caractères) sur WhatsApp ; le contenu long va dans des fichiers.
```

Analyse minimale :

* Préfixe de type : `W` (monde), `B` (expérience/biographique), `O` (opinion), `S` (observation/résumé ; généralement généré)
* Entités : `@Peter`, `@warelay`, etc. (les slugs correspondent aux fichiers `bank/entities/*.md`)
* Degré de confiance de l’opinion : `O(c=0.0..1.0)` optionnel

Si vous ne voulez pas que les auteurs et autrices aient à y penser, le job de réflexion peut déduire ces éléments à partir du reste du journal, mais avoir une section explicite `## Retain` est le « levier de qualité » le plus simple.


<div id="recall-queries-over-the-derived-index">
  ### Recall : requêtes sur l’index dérivé
</div>

Recall doit prendre en charge :

- **lexical** : « trouver des termes / noms / commandes exacts » (FTS5)
- **entity** : « parle-moi de X » (pages d’entités + faits liés aux entités)
- **temporal** : « que s’est-il passé autour du 27 novembre » / « depuis la semaine dernière »
- **opinion** : « quelles sont les préférences de Peter ? » (avec niveau de confiance + éléments de preuve)

Le format de retour doit être adapté aux agents et citer les sources :

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (jour d’origine, ou intervalle temporel extrait si présent)
- `entities` (`["Peter","warelay"]`)
- `content` (le fait, sous forme narrative)
- `source` (`memory/2025-11-27.md#L12` etc)

<div id="reflect-produce-stable-pages-update-beliefs">
  ### Réflexion : produire des pages stables + mettre à jour les croyances
</div>

La réflexion est une tâche planifiée (quotidienne ou signal de vie `ultrathink`) qui :

- met à jour `bank/entities/*.md` à partir des faits récents (résumés d’entités)
- met à jour la confiance dans `bank/opinions.md` en fonction du renforcement/de la contradiction
- propose éventuellement des modifications à `memory.md` (faits durables « quasi centraux »)

Évolution des opinions (simple, explicable) :

- chaque opinion a :
  - un énoncé
  - une confiance `c ∈ [0,1]`
  - `last_updated`
  - des liens vers des éléments de preuve (ID de faits à l’appui + contradictoires)
- quand de nouveaux faits arrivent :
  - identifier les opinions candidates par recouvrement d’entités + similarité (FTS d’abord, embeddings ensuite)
  - mettre à jour la confiance par petits incréments ; les grands sauts nécessitent une forte contradiction + des preuves répétées

<div id="cli-integration-standalone-vs-deep-integration">
  ## Intégration de la CLI : mode autonome vs intégration poussée
</div>

Recommandation : **intégration poussée dans OpenClaw**, tout en conservant une bibliothèque cœur distincte.

<div id="why-integrate-into-openclaw">
  ### Pourquoi s’intégrer à OpenClaw ?
</div>

- OpenClaw connaît déjà :
  - le chemin de l’espace de travail (`agents.defaults.workspace`)
  - le modèle de session + les signaux de vie
  - les modèles de journalisation et de dépannage
- Vous voulez que l’agent appelle lui‑même les outils :
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### Pourquoi continuer à scinder une bibliothèque ?
</div>

- maintenir la logique de gestion de la mémoire testable sans Gateway ni runtime
- pouvoir la réutiliser dans d'autres contextes (scripts locaux, future application de bureau, etc.)

Structure :
Les outils de gestion de la mémoire sont pensés comme une fine couche CLI + bibliothèque, mais cela reste pour l’instant exploratoire.

<div id="s-collide-suco-when-to-use-it-research">
  ## « S-Collide » / SuCo : quand l’utiliser (recherche)
</div>

Si « S-Collide » fait référence à **SuCo (Subspace Collision)** : c’est une approche de retrieval ANN qui vise de bons compromis rappel/latence en utilisant des collisions apprises/structurées dans des sous‑espaces (publication : arXiv 2411.14754, 2024).

Recommandation pragmatique pour `~/.openclaw/workspace` :

- **ne commencez pas** avec SuCo.
- commencez avec SQLite FTS + (facultatif) des embeddings simples ; vous obtiendrez immédiatement la majorité des gains UX.
- n’envisagez des solutions de la classe SuCo/HNSW/ScaNN que lorsque :
  - le corpus est volumineux (dizaines/centaines de milliers de chunks)
  - la recherche exhaustive par embeddings devient trop lente
  - la qualité de rappel est réellement limitée par la recherche lexicale

Alternatives adaptées au mode hors ligne (par complexité croissante) :

- SQLite FTS5 + filtres de métadonnées (zéro ML)
- Embeddings + recherche exhaustive (fonctionne étonnamment bien si le nombre de chunks est faible)
- Index HNSW (classique, robuste ; nécessite une liaison vers une bibliothèque)
- SuCo (de niveau recherche ; intéressant s’il existe une implémentation solide que vous pouvez embarquer)

Question ouverte :

- quel est le **meilleur** modèle d’embedding hors ligne pour la « mémoire d’assistant personnel » sur vos machines (ordinateur portable + desktop) ?
  - si vous avez déjà Ollama : générez les embeddings avec un modèle local ; sinon, embarquez un petit modèle d’embedding dans la chaîne d’outils.

<div id="smallest-useful-pilot">
  ## Pilote minimal utile
</div>

Si vous voulez une version minimale mais déjà exploitable :

- Ajoutez des pages d’entité `bank/` et une section `## Retain` dans les journaux quotidiens.
- Utilisez SQLite FTS pour la recherche avec références (chemin + numéros de ligne).
- Ajoutez des embeddings uniquement si la qualité du rappel ou les besoins de mise à l’échelle l’exigent.

<div id="references">
  ## Références
</div>

- Concepts Letta / MemGPT : « core memory blocks » + « archival memory » + mémoire auto-éditable pilotée par des outils.
- Rapport technique Hindsight : « retain / recall / reflect », mémoire à quatre réseaux, extraction de faits narratifs, évolution de la confiance associée aux opinions.
- SuCo : arXiv 2411.14754 (2024) : « Subspace Collision » et recherche de plus proches voisins approximative.