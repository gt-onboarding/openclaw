---
title: Overlay vocale
summary: "Ciclo di vita dell'overlay vocale quando la wake word e il push-to-talk si sovrappongono"
read_when:
  - Modificare il comportamento dell'overlay vocale
---

<div id="voice-overlay-lifecycle-macos">
  # Ciclo di vita del Voice Overlay (macOS)
</div>

Pubblico: collaboratori dell’app macOS. Obiettivo: mantenere il comportamento del voice overlay prevedibile quando la parola di attivazione (wake word) e il push-to-talk si sovrappongono.

<div id="current-intent">
  ### Intent attuale
</div>

- Se l'overlay è già visibile tramite wake-word e l'utente preme la hotkey, la sessione della hotkey *adotta* il testo esistente invece di reimpostarlo. L'overlay rimane visibile finché la hotkey è tenuta premuta. Quando l'utente rilascia: invia se è presente testo ripulito, altrimenti chiudi.
- La sola wake-word continua a inviare automaticamente in caso di silenzio; il push-to-talk invia immediatamente al rilascio.

<div id="implemented-dec-9-2025">
  ### Implementato (9 dic 2025)
</div>

- Le sessioni overlay ora includono un token per ogni acquisizione (wake word o push-to-talk). Gli aggiornamenti di tipo partial/final/send/dismiss/level vengono scartati quando il token non corrisponde, evitando callback obsoleti.
- Il push-to-talk adotta qualsiasi testo overlay visibile come prefisso (quindi premere la hotkey mentre l’overlay di wake è attivo mantiene il testo e vi aggiunge il nuovo parlato). Attende fino a 1,5 s una trascrizione finale prima di tornare al testo corrente.
- Il logging relativo a chime/overlay viene emesso a livello `info` nelle categorie `voicewake.overlay`, `voicewake.ptt` e `voicewake.chime` (avvio della sessione, partial, final, send, dismiss, motivo del chime).

<div id="next-steps">
  ### Prossimi passi
</div>

1. **VoiceSessionCoordinator (actor)**
   - Gestisce esattamente una `VoiceSession` alla volta.
   - API (basata su token): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Scarta le callback che contengono token obsoleti (impedisce ai vecchi riconoscitori di riaprire l'overlay).
2. **VoiceSession (model)**
   - Campi: `token`, `source` (wakeWord|pushToTalk), testo confermato/volatile, flag per i chime (segnali acustici), timer (invio automatico, inattività), `overlayMode` (display|editing|sending), scadenza del cooldown.
3. **Binding dell'overlay**
   - `VoiceSessionPublisher` (`ObservableObject`) riflette la sessione attiva in SwiftUI.
   - `VoiceWakeOverlayView` esegue il rendering solo tramite il publisher; non modifica mai direttamente singleton globali.
   - Le azioni utente sull'overlay (`sendNow`, `dismiss`, `edit`) richiamano il coordinator passando il token della sessione.
4. **Percorso di invio unificato**
   - Su `endCapture`: se il testo ripulito è vuoto → chiudi; altrimenti `performSend(session:)` (riproduce una volta il chime di invio, inoltra, chiude).
   - Push-to-talk: nessun ritardo; wake-word: ritardo opzionale per l'invio automatico.
   - Applica un breve cooldown al runtime di wake-word dopo che il push-to-talk è terminato, così che la wake-word non si riattivi immediatamente.
5. **Logging**
   - Il coordinator emette log `.info` nel subsystem `bot.molt`, categorie `voicewake.overlay` e `voicewake.chime`.
   - Eventi chiave: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

<div id="debugging-checklist">
  ### Checklist di debug
</div>

- Metti in streaming i log mentre riproduci un overlay bloccato:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- Verifica che ci sia un solo token di sessione attivo; le callback obsolete devono essere scartate dal coordinatore.
- Assicurati che il rilascio del pulsante push-to-talk chiami sempre `endCapture` con il token attivo; se il testo è vuoto, aspettati `dismiss` senza suono o Invia.

<div id="migration-steps-suggested">
  ### Passaggi di migrazione (suggeriti)
</div>

1. Aggiungi `VoiceSessionCoordinator`, `VoiceSession` e `VoiceSessionPublisher`.
2. Effettua il refactoring di `VoiceWakeRuntime` per creare/aggiornare/terminare le sessioni invece di accedere direttamente a `VoiceWakeOverlayController`.
3. Effettua il refactoring di `VoicePushToTalk` per adottare le sessioni esistenti e chiamare `endCapture` al rilascio; applica un cooldown a livello di runtime.
4. Collega `VoiceWakeOverlayController` al publisher; rimuovi le chiamate dirette dal runtime/PTT.
5. Aggiungi test di integrazione per l'adozione delle sessioni, il cooldown e la chiusura in caso di testo vuoto.