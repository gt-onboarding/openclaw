---
title: Canaux de développement
summary: "Canaux stable, bêta et dev : sémantique, changement de canal et étiquetage"
read_when:
  - Vous souhaitez passer d'un canal stable/bêta/dev à un autre
  - Vous taguez ou publiez des préversions
---

<div id="development-channels">
  # Canaux de développement
</div>

Dernière mise à jour : 2026-01-21

OpenClaw propose trois canaux de mise à jour :

- **stable** : npm dist-tag `latest`.
- **beta** : npm dist-tag `beta` (builds en cours de test).
- **dev** : pointe actuelle de la branche `main` (git). npm dist-tag : `dev` (lorsqu'il est publié).

Nous publions des builds sur **beta**, nous les testons, puis **nous promouvons un build validé vers `latest`**
sans changer le numéro de version — les dist-tags sont la source de vérité pour les installations npm.

<div id="switching-channels">
  ## Changer de canal
</div>

Git checkout :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` récupèrent le dernier tag correspondant (souvent le même tag).
* `dev` bascule sur `main` et effectue un rebase sur le dépôt amont.

Installation globale npm/pnpm :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Cette opération se met à jour via le dist-tag npm correspondant (`latest`, `beta`, `dev`).

Lorsque vous changez **explicitement** de canal avec `--channel`, OpenClaw aligne aussi
la méthode d&#39;installation :

* `dev` garantit un checkout git (par défaut `~/openclaw`, modifiable avec `OPENCLAW_GIT_DIR`),
  le met à jour et installe la CLI globale à partir de ce checkout.
* `stable`/`beta` installe depuis npm en utilisant le dist-tag correspondant.

Astuce : si vous voulez stable et dev en parallèle, conservez deux clones et faites pointer votre Gateway vers celui en stable.


<div id="plugins-and-channels">
  ## Plugins et canaux
</div>

Lorsque vous changez de canal avec `openclaw update`, OpenClaw synchronise également les sources de plugins&nbsp;:

- `dev` privilégie les plugins intégrés issus du checkout Git.
- `stable` et `beta` restaurent les packages de plugins installés avec npm.

<div id="tagging-best-practices">
  ## Bonnes pratiques de gestion des tags
</div>

- Marquez les versions que vous souhaitez cibler avec vos `git checkout` (`vYYYY.M.D` ou `vYYYY.M.D-<patch>`).
- Gardez les tags immuables : ne déplacez jamais un tag et ne le réutilisez jamais.
- Les `npm dist-tags` restent la source de référence pour les installations npm :
  - `latest` → stable
  - `beta` → version candidate
  - `dev` → snapshot de la branche `main` (optionnel)

<div id="macos-app-availability">
  ## Disponibilité de l’app macOS
</div>

Les versions bêta et de développement peuvent **ne pas** inclure de publication de l’app macOS. Ce n’est pas un problème :

- Le tag git et le dist-tag npm peuvent toujours être publiés.
- Indiquez « pas de build macOS pour cette bêta » dans les notes de version ou le changelog.