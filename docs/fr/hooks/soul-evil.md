---
title: SOUL Evil
summary: "Hook SOUL Evil (remplacez SOUL.md par SOUL_EVIL.md)"
read_when:
  - Vous souhaitez activer ou ajuster le hook SOUL Evil
  - Vous souhaitez une fenêtre de purge ou un basculement de persona aléatoire
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

Le hook SOUL Evil remplace le contenu **injecté** de `SOUL.md` par `SOUL_EVIL.md` au cours
d’une fenêtre de purge ou de façon aléatoire. Il ne modifie **pas** les fichiers sur le disque.

<div id="how-it-works">
  ## Fonctionnement
</div>

Quand `agent:bootstrap` s&#39;exécute, le hook peut remplacer le contenu de `SOUL.md` en mémoire
avant que l&#39;invite système ne soit assemblée. Si `SOUL_EVIL.md` est manquant ou vide,
OpenClaw journalise un avertissement et conserve le `SOUL.md` normal.

Les exécutions des sous-agents **n&#39;incluent pas** `SOUL.md` dans leurs fichiers de bootstrap, ce hook
n&#39;a donc aucun effet sur les sous-agents.

<div id="enable">
  ## Activation
</div>

```bash
openclaw hooks enable soul-evil
```

Ensuite, définissez la configuration :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

Créez `SOUL_EVIL.md` à la racine de l&#39;espace de travail de l&#39;agent (à côté de `SOUL.md`).

<div id="options">
  ## Options
</div>

* `file` (string) : nom de fichier SOUL alternatif (par défaut : `SOUL_EVIL.md`)
* `chance` (number 0–1) : probabilité aléatoire (entre 0 et 1) à chaque exécution d’utiliser `SOUL_EVIL.md`
* `purge.at` (HH:mm) : début quotidien de la purge (format 24 heures)
* `purge.duration` (duration) : durée de la fenêtre (par ex. `30s`, `10m`, `1h`)

**Priorité :** la fenêtre de purge a priorité sur la probabilité.

**Fuseau horaire :** utilise `agents.defaults.userTimezone` lorsqu’il est défini ; sinon le fuseau horaire de l’hôte.

<div id="notes">
  ## Notes
</div>

* Aucun fichier n’est écrit ni modifié sur le disque.
* Si `SOUL.md` ne figure pas dans la liste de bootstrap, le hook ne fait rien.

<div id="see-also">
  ## Voir aussi
</div>

* [Hooks](/fr/hooks)