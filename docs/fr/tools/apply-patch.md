---
title: Appliquer un patch
summary: "Appliquer des patchs multi-fichiers avec l’outil apply_patch"
read_when:
  - Vous avez besoin de modifications structurées impliquant plusieurs fichiers
  - Vous souhaitez documenter ou déboguer des modifications effectuées via des patchs
---

<div id="apply_patch-tool">
  # outil apply_patch
</div>

Applique des modifications de fichiers à l&#39;aide d&#39;un format de patch structuré. C&#39;est idéal pour les modifications multi-fichiers
ou multi-hunks, où un appel unique à `edit` serait fragile.

L&#39;outil accepte une unique chaîne de caractères `input` qui encapsule une ou plusieurs opérations sur des fichiers :

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```


<div id="parameters">
  ## Paramètres
</div>

- `input` (obligatoire) : contenu complet du patch incluant `*** Begin Patch` et `*** End Patch`.

<div id="notes">
  ## Notes
</div>

- Les chemins sont résolus par rapport à la racine de l’espace de travail.
- Utilisez `*** Move to:` dans un bloc `*** Update File:` pour renommer des fichiers.
- `*** End of File` marque, lorsque nécessaire, une insertion uniquement en fin de fichier.
- Fonctionnalité expérimentale et désactivée par défaut. Activez-la avec `tools.exec.applyPatch.enabled`.
- Spécifique à OpenAI (y compris OpenAI Codex). Peut éventuellement être restreinte par modèle via
  `tools.exec.applyPatch.allowModels`.
- La configuration se trouve uniquement sous `tools.exec`.

<div id="example">
  ## Exemple
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
