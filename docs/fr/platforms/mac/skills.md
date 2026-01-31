---
title: Compétences
summary: "UI des préférences des compétences macOS et état géré par le Gateway"
read_when:
  - Mise à jour de l'UI des préférences des compétences macOS
  - Modification des mécanismes de contrôle ou du comportement d'installation des compétences
---

<div id="skills-macos">
  # Compétences (macOS)
</div>

L'app macOS met à disposition les compétences OpenClaw via le Gateway ; elle n’interprète pas les compétences en local.

<div id="data-source">
  ## Source de données
</div>

- `skills.status` (Gateway) renvoie toutes les compétences ainsi que leur éligibilité et les exigences manquantes
  (y compris les blocages liés à la liste d’autorisation pour les compétences intégrées).
- Les exigences proviennent de `metadata.openclaw.requires` dans chaque `SKILL.md`.

<div id="install-actions">
  ## Actions d'installation
</div>

- `metadata.openclaw.install` définit les options d'installation (brew/node/go/uv).
- L'application appelle `skills.install` pour exécuter les programmes d'installation sur l'hôte du Gateway.
- Le Gateway ne propose qu'un seul programme d'installation privilégié lorsque plusieurs sont fournis
  (brew s'il est disponible, sinon le gestionnaire Node.js provenant de `skills.install`, avec npm par défaut).

<div id="envapi-keys">
  ## Clés d’environnement/API
</div>

- L’application stocke les clés dans `~/.openclaw/openclaw.json` sous `skills.entries.<skillKey>`.
- `skills.update` met à jour `enabled`, `apiKey` et `env`.

<div id="remote-mode">
  ## Mode distant
</div>

- Les opérations d’installation et les mises à jour de configuration s’effectuent sur l’hôte du Gateway (et non sur le Mac local).