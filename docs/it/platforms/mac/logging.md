---
title: Registrazione dei log
summary: "Registrazione dei log di OpenClaw: file di log diagnostici a rotazione + flag di privacy dell’Unified Log"
read_when:
  - Raccolta dei log di macOS o analisi del logging di dati privati
  - Debug di problemi relativi all’attivazione vocale o al ciclo di vita della sessione
---

<div id="logging-macos">
  # Log (macOS)
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## File di log diagnostici a rotazione (riquadro Debug)
</div>

OpenClaw instrada i log dell’app macOS tramite swift-log (logging unificato per impostazione predefinita) e può scrivere su disco un file di log locale a rotazione quando ti serve una registrazione persistente.

- Verbosità: **Debug pane → Logs → App logging → Verbosity**
- Abilita: **Debug pane → Logs → App logging → “Write rolling diagnostics log (JSONL)”**
- Posizione: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (ruota automaticamente; i file vecchi hanno suffissi `.1`, `.2`, …)
- Cancella: **Debug pane → Logs → App logging → “Clear”**

Note:

- Questa opzione è **disattivata per impostazione predefinita**. Abilitala solo durante il debug attivo.
- Tratta il file come sensibile; non condividerlo senza una revisione preventiva.

<div id="unified-logging-private-data-on-macos">
  ## Dati privati del unified logging su macOS
</div>

Il sistema di unified logging oscura la maggior parte dei payload, a meno che un sottosistema non abiliti `privacy -off`. Come descritto da Peter nel suo articolo sulle [logging privacy shenanigans](https://steipete.me/posts/2025/logging-privacy-shenanigans) su macOS (2025), questo comportamento è controllato da un plist in `/Library/Preferences/Logging/Subsystems/` associato al nome del sottosistema. Solo le nuove voci di log recepiscono il flag, quindi abilitalo prima di riprodurre un problema.

<div id="enable-for-openclaw-botmolt">
  ## Abilitare in OpenClaw (`bot.molt`)
</div>

* Scrivi prima il plist in un file temporaneo, quindi installalo in modo atomico come root:

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

* Non è necessario alcun riavvio; `logd` rileva rapidamente il file, ma solo le nuove righe di log includeranno i payload privati.
* Visualizza l&#39;output più dettagliato con l&#39;helper esistente, ad esempio `./scripts/clawlog.sh --category WebChat --last 5m`.


<div id="disable-after-debugging">
  ## Disattiva dopo il debug
</div>

- Rimuovi l'override: `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`.
- Facoltativamente esegui `sudo log config --reload` per forzare logd a eliminare immediatamente l'override.
- Ricorda che questo livello di log può includere numeri di telefono e contenuti dei messaggi; mantieni questo plist solo finché hai effettivamente bisogno dei dettagli aggiuntivi.