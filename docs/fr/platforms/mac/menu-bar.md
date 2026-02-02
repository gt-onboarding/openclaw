---
title: Barre de menus
summary: "Logique d'état de la barre de menus et ce qui est affiché aux utilisateurs"
read_when:
  - Ajustement de l'UI du menu macOS ou de la logique d'état
---

<div id="menu-bar-status-logic">
  # Logique d'état de la barre des menus
</div>

<div id="what-is-shown">
  ## Ce qui est affiché
</div>

- Nous affichons l’état de travail actuel de l’agent dans l’icône de la barre de menus et dans la première ligne d’état du menu.
- L’état de santé est masqué pendant qu’un travail est en cours ; il réapparaît lorsque toutes les sessions sont inactives.
- Le bloc « Nodes » dans le menu répertorie uniquement les **appareils** (nœuds appairés via `node.list`), et non les entrées client/présence.
- Une section « Usage » apparaît sous « Context » lorsque des instantanés d’utilisation des fournisseurs sont disponibles.

<div id="state-model">
  ## Modèle d’état
</div>

- Sessions : les événements arrivent avec un `runId` (par exécution) plus un `sessionKey` dans le payload. La session « main » utilise la clé `main` ; si elle est absente, on utilise la session la plus récemment mise à jour.
- Priorité : la session « main » a toujours la priorité. Si main est active, son état est affiché immédiatement. Si main est inactive, la session non‑main la plus récemment active est affichée. Nous ne changeons pas de session en cours d’activité ; nous ne basculons que lorsque la session actuelle devient inactive ou que main devient active.
- Types d’activité :
  - `job` : exécution de commande de haut niveau (`state: started|streaming|done|error`).
  - `tool` : `phase: start|result` avec `toolName` et `meta/args`.

<div id="iconstate-enum-swift">
  ## Énumération IconState (Swift)
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (forçage pour le débogage)

<div id="activitykind-glyph">
  ### ActivityKind → glyphe
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- par défaut → 🛠️

<div id="visual-mapping">
  ### Correspondance visuelle
</div>

- `idle` : créature à l’état normal.
- `workingMain` : badge avec glyphe, teinte complète, animation de patte « en activité ».
- `workingOther` : badge avec glyphe, teinte atténuée, pas d’animation de déplacement.
- `overridden` : utilise le glyphe et la teinte choisis, quelle que soit l’activité.

<div id="status-row-text-menu">
  ## Texte de la ligne d'état (menu)
</div>

- Pendant qu'une tâche est en cours : `<Session role> · <activity label>`
  - Exemples : `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Lorsque inactif : revient au résumé d'état du système.

<div id="event-ingestion">
  ## Ingestion des événements
</div>

- Source : événements `agent` du canal de contrôle (`ControlChannel.handleAgentEvent`).
- Champs extraits :
  - `stream: "job"` avec `data.state` pour le démarrage/l’arrêt.
  - `stream: "tool"` avec `data.phase`, `name`, `meta`/`args` facultatifs.
- Libellés :
  - `exec` : première ligne de `args.command`.
  - `read`/`write` : chemin abrégé.
  - `edit` : chemin plus type de modification déduit à partir de `meta`/nombre de diffs.
  - valeur par défaut : nom de l’outil.

<div id="debug-override">
  ## Substitution pour le débogage
</div>

- Settings ▸ Debug ▸ sélecteur “Icon override” :
  - `System (auto)` (par défaut)
  - `Working: main` (par type d’outil)
  - `Working: other` (par type d’outil)
  - `Idle`
- Stocké via `@AppStorage("iconOverride")`, associé à `IconState.overridden`.

<div id="testing-checklist">
  ## Check‑list de tests
</div>

- Déclencher un job sur la session principale : vérifier que l’icône bascule immédiatement et que la ligne d’état affiche le libellé principal.
- Déclencher un job sur une session non principale pendant que la principale est inactive : l’icône et l’état indiquent la session non principale et restent stables jusqu’à la fin.
- Démarrer la session principale pendant qu’une autre est active : l’icône bascule instantanément vers la principale.
- Exécutions d’outils en rafale : s’assurer que le badge ne clignote pas (délai de grâce TTL sur les résultats d’outils).
- La ligne d’état Health réapparaît une fois que toutes les sessions sont inactives.