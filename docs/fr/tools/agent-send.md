---
title: Envoi d’Agent
summary: "Exécutions directes de `openclaw agent` via la CLI (avec envoi facultatif)"
read_when:
  - Ajout ou modification du point d’entrée CLI de l’agent
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent` (exécutions directes d’agent)
</div>

`openclaw agent` exécute un seul tour d’agent sans nécessiter de message de chat entrant.
Par défaut, il passe **via le Gateway** ; ajoutez `--local` pour forcer l’exécution embarquée
sur la machine locale.

<div id="behavior">
  ## Comportement
</div>

- Obligatoire : `--message <text>`
- Sélection de la session :
  - `--to <dest>` dérive la clé de session (les cibles de groupe/canal préservent l'isolation ; les discussions directes sont regroupées dans `main`), **ou**
  - `--session-id <id>` réutilise une session existante par identifiant, **ou**
  - `--agent <id>` cible directement un agent configuré (utilise la clé de session `main` de cet agent)
- Exécute le même runtime d'agent embarqué que pour les réponses entrantes normales.
- Les indicateurs de réflexion / verbosité sont conservés dans le stockage de session.
- Sortie :
  - par défaut : affiche le texte de la réponse (plus les lignes `MEDIA:<url>`)
  - `--json` : affiche un payload structuré + des métadonnées
- Envoi optionnel vers un canal avec `--deliver` + `--channel` (les formats cibles correspondent à `openclaw message --target`).
- Utilisez `--reply-channel`/`--reply-to`/`--reply-account` pour modifier la livraison sans changer la session.

Si le Gateway est indisponible, la CLI **bascule** vers l'exécution locale embarquée.

<div id="examples">
  ## Exemples
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## Flags
</div>

- `--local`: exécute localement (nécessite des clés API de fournisseur de modèle dans votre shell)
- `--deliver`: envoie la réponse vers le canal choisi
- `--channel`: canal d'envoi (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, par défaut : `whatsapp`)
- `--reply-to`: remplace la cible d'envoi
- `--reply-channel`: remplace le canal d'envoi
- `--reply-account`: remplace l'identifiant de compte d'envoi
- `--thinking <off|minimal|low|medium|high|xhigh>`: enregistre le niveau de réflexion (modèles GPT-5.2 + Codex uniquement)
- `--verbose <on|full|off>`: enregistre le niveau de verbosité
- `--timeout <seconds>`: remplace le délai d'expiration de l'agent
- `--json`: produit du JSON structuré