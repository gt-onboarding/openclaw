---
title: Journalisation
summary: "Journalisation d’OpenClaw : fichier journal de diagnostic rotatif + indicateurs de confidentialité pour le journal unifié"
read_when:
  - Capture de journaux macOS ou analyse de la journalisation de données privées
  - Débogage des problèmes de réveil vocal / du cycle de vie de session
---

<div id="logging-macos">
  # Journalisation (macOS)
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## Fichier journal de diagnostic rotatif (panneau Debug)
</div>

OpenClaw achemine les logs de l’app macOS via swift-log (journalisation unifiée par défaut) et peut écrire sur le disque un fichier journal local rotatif lorsque vous avez besoin d’une capture persistante.

- Verbosité : **Debug pane → Logs → App logging → Verbosity**
- Activation : **Debug pane → Logs → App logging → “Write rolling diagnostics log (JSONL)”**
- Emplacement : `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rotation automatique ; les anciens fichiers sont suffixés avec `.1`, `.2`, …)
- Effacer : **Debug pane → Logs → App logging → “Clear”**

Remarques :

- Cette option est **désactivée par défaut**. Activez-la uniquement pendant un débogage actif.
- Considérez ce fichier comme sensible ; ne le partagez pas sans relecture préalable.

<div id="unified-logging-private-data-on-macos">
  ## Données privées de la journalisation unifiée sur macOS
</div>

La journalisation unifiée masque la plupart des données des messages, sauf si un sous-système active `privacy -off`. Comme expliqué dans l’article de Peter sur les [bizarreries de confidentialité de la journalisation](https://steipete.me/posts/2025/logging-privacy-shenanigans) sur macOS (2025), ce comportement est contrôlé par un fichier plist dans `/Library/Preferences/Logging/Subsystems/`, identifié par le nom du sous-système. Seules les nouvelles entrées de journal prennent en compte ce paramètre ; active-le donc avant de reproduire un problème.

<div id="enable-for-openclaw-botmolt">
  ## Activer pour OpenClaw (`bot.molt`)
</div>

* Écrire d’abord le plist dans un fichier temporaire, puis l’installer de manière atomique en tant que root :

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

* Aucun redémarrage n’est nécessaire : `logd` détecte rapidement le fichier, mais seules les nouvelles lignes de log incluront les données privées.
* Affichez une sortie plus détaillée avec l’utilitaire existant, par exemple : `./scripts/clawlog.sh --category WebChat --last 5m`.


<div id="disable-after-debugging">
  ## Désactiver une fois le débogage terminé
</div>

- Supprimez la surcharge : `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`.
- Vous pouvez également exécuter `sudo log config --reload` pour forcer `logd` à abandonner immédiatement cette surcharge.
- Gardez en tête que cette surface peut inclure des numéros de téléphone et du contenu de messages ; ne laissez ce fichier plist en place que lorsque vous avez réellement besoin de ces détails supplémentaires.