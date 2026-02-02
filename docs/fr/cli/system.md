---
title: Système
summary: "Référence CLI pour `openclaw system` (événements système, signal de vie, présence)"
read_when:
  - Vous souhaitez mettre en file d’attente un événement système sans configurer de tâche cron
  - Vous devez activer ou désactiver le signal de vie
  - Vous voulez inspecter les enregistrements de présence système
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Outils système pour le Gateway : mettre en file d’attente des événements système, contrôler les signaux de vie et afficher la présence.

<div id="common-commands">
  ## Commandes courantes
</div>

```bash
openclaw system event --text "Vérifier les suivis urgents" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

Met en file d’attente un événement système sur la session **principale**. Le prochain signal de vie l’injectera comme une ligne `System:` dans le prompt. Utilisez `--mode now` pour déclencher immédiatement le signal de vie ; `next-heartbeat` attend le prochain déclenchement prévu.

Flags :

- `--text <text>` : texte obligatoire de l’événement système.
- `--mode <mode>` : `now` ou `next-heartbeat` (valeur par défaut).
- `--json` : sortie lisible par machine.

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

Commandes du signal de vie :

- `last` : afficher le dernier événement de signal de vie.
- `enable` : réactiver les signaux de vie (à utiliser s’ils ont été désactivés).
- `disable` : mettre les signaux de vie en pause.

Options :

- `--json` : sortie lisible par une machine.

<div id="system-presence">
  ## `system presence`
</div>

Liste les entrées de présence système que le Gateway connaît actuellement (nœuds,
instances et lignes d’état similaires).

Options :

- `--json` : sortie lisible par une machine.

<div id="notes">
  ## Remarques
</div>

- Nécessite un Gateway en cours d’exécution, accessible depuis votre configuration actuelle (en local ou à distance).
- Les événements système sont éphémères et ne sont pas conservés entre les redémarrages.