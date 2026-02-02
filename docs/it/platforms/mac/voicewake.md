---
title: Attivazione vocale
summary: "Modalità di attivazione vocale e push-to-talk e dettagli di instradamento nell'app macOS"
read_when:
  - Quando lavori sui flussi di attivazione vocale o PTT
---

<div id="voice-wake-push-to-talk">
  # Attivazione vocale e push-to-talk
</div>

<div id="modes">
  ## Modalità
</div>

- **Modalità wake-word** (predefinita): il riconoscitore vocale sempre attivo attende i token di attivazione (`swabbleTriggerWords`). Al rilevamento avvia la cattura, mostra l'overlay con il testo parziale e invia automaticamente dopo un momento di silenzio.
- **Push-to-talk (tasto Opzione destro tenuto premuto)**: tieni premuto il tasto Opzione destro per iniziare subito la cattura, senza bisogno di trigger. L'overlay viene mostrato mentre il tasto è premuto; al rilascio la cattura viene finalizzata e inoltrata dopo un breve ritardo, così puoi ritoccare il testo.

<div id="runtime-behavior-wake-word">
  ## Comportamento in runtime (wake-word)
</div>

- Il riconoscitore vocale viene eseguito in `VoiceWakeRuntime`.
- Il trigger scatta solo quando c’è una **pausa significativa** tra la wake word e la parola successiva (intervallo di ~0,55 s). L’overlay/il suono di avviso può iniziare durante la pausa, anche prima che il comando inizi.
- Finestre di silenzio: 2,0 s quando il parlato è continuo, 5,0 s se è stato rilevato solo il trigger.
- Interruzione forzata: 120 s per prevenire sessioni fuori controllo.
- Debounce tra le sessioni: 350 ms.
- L’overlay è gestito da `VoiceWakeOverlayController` con colorazione per gli stati committed/volatile.
- Dopo l’azione di `send`, il riconoscitore si riavvia in modo pulito per ascoltare il trigger successivo.

<div id="lifecycle-invariants">
  ## Invarianti di ciclo di vita
</div>

- Se Voice Wake è abilitato e le autorizzazioni sono concesse, il riconoscitore della parola chiave di attivazione deve rimanere in ascolto (tranne durante un’acquisizione esplicita tramite push-to-talk).
- La visibilità dell’overlay (inclusa la chiusura manuale tramite il pulsante X) non deve mai impedire al riconoscitore di riprendere l’ascolto.

<div id="sticky-overlay-failure-mode-previous">
  ## Modalità di errore con overlay bloccato (precedente)
</div>

In precedenza, se l’overlay rimaneva visibile e bloccato e lo chiudevi manualmente, Voice Wake poteva sembrare “morto” perché il tentativo di riavvio del runtime poteva essere bloccato dalla visibilità dell’overlay e nessun riavvio successivo veniva pianificato.

Rafforzamento:

- Il riavvio del runtime di Wake non è più bloccato dalla visibilità dell’overlay.
- Il completamento della chiusura dell’overlay attiva un `VoiceWakeRuntime.refresh(...)` tramite `VoiceSessionCoordinator`, quindi la chiusura manuale con la X riprende sempre l’ascolto.

<div id="push-to-talk-specifics">
  ## Dettagli sul push-to-talk
</div>

- Il rilevamento del tasto rapido usa un monitor globale `.flagsChanged` per **Option destro** (`keyCode 61` + `.option`). Osserviamo soltanto gli eventi (non li intercettiamo/consumiamo).
- La pipeline di cattura risiede in `VoicePushToTalk`: avvia immediatamente Speech, invia in streaming i parziali all’overlay e chiama `VoiceWakeForwarder` al rilascio.
- Quando il push-to-talk si avvia sospendiamo il runtime della wake word per evitare conflitti tra tap audio; viene riavviato automaticamente dopo il rilascio.
- Autorizzazioni: richiede Microphone + Speech; per poter osservare gli eventi è necessaria l’autorizzazione Accessibilità/Monitoraggio input.
- Tastiere esterne: alcune potrebbero non esporre Option destro come previsto—offri una scorciatoia alternativa di fallback se gli utenti segnalano mancate attivazioni.

<div id="user-facing-settings">
  ## Impostazioni utente
</div>

- Interruttore **Voice Wake**: abilita il runtime della wake word.
- **Tieni premuti Cmd+Fn per parlare**: abilita il monitor push-to-talk. Disabilitato su macOS < 26.
- Selettori per lingua e microfono, indicatore di livello in tempo reale, tabella delle parole di attivazione, tester (solo locale; non inoltra).
- Il selettore del microfono conserva l'ultima selezione se un dispositivo si disconnette, mostra un avviso di disconnessione e passa temporaneamente al valore predefinito di sistema finché il dispositivo non torna disponibile.
- **Suoni**: segnali acustici al rilevamento del trigger e all'invio; per impostazione predefinita usa il suono di sistema macOS "Glass". Puoi scegliere qualsiasi file caricabile da `NSSound` (ad es. MP3/WAV/AIFF) per ciascun evento oppure selezionare **Nessun suono**.

<div id="forwarding-behavior">
  ## Comportamento di inoltro
</div>

- Quando Voice Wake è abilitato, le trascrizioni vengono inoltrate al Gateway/agente attivo (nella stessa modalità locale vs remota utilizzata dal resto dell’app macOS).
- Le risposte vengono recapitate all’**ultimo provider principale utilizzato** (WhatsApp/Telegram/Discord/WebChat). Se il recapito non riesce, l’errore viene registrato e l’esecuzione rimane comunque visibile tramite WebChat/log della sessione.

<div id="forwarding-payload">
  ## Payload di inoltro
</div>

- `VoiceWakeForwarder.prefixedTranscript(_:)` antepone l'hint per la macchina prima dell'invio. Utilizzato sia nel percorso con parola di attivazione sia in quello push-to-talk.

<div id="quick-verification">
  ## Verifica rapida
</div>

- Attiva il push-to-talk, tieni premuti Cmd+Fn, parla, poi rilascia: l’overlay dovrebbe mostrare le trascrizioni parziali e poi inviarle.
- Mentre tieni premuti i tasti, le orecchie nella barra dei menu dovrebbero rimanere ingrandite (utilizza `triggerVoiceEars(ttl:nil)`); tornano normali dopo il rilascio dei tasti.