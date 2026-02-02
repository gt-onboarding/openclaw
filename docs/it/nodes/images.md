---
title: Immagini
summary: "Regole di gestione delle immagini e dei contenuti multimediali per send, Gateway e risposte degli agenti"
read_when:
  - Modifica della pipeline multimediale o degli allegati
---

<div id="image-media-support-2025-12-05">
  # Supporto per Immagini e Contenuti Multimediali — 2025-12-05
</div>

Il canale WhatsApp si basa su **Baileys Web**. Questo documento descrive le regole attuali di gestione dei contenuti multimediali per Invia, il Gateway e le risposte degli agenti.

<div id="goals">
  ## Obiettivi
</div>

- Invia contenuti multimediali con didascalie opzionali tramite `openclaw message send --media`.
- Consenti alle risposte automatiche dalla web inbox di includere contenuti multimediali insieme al testo.
- Mantieni i limiti per tipo ragionevoli e prevedibili.

<div id="cli-surface">
  ## Superficie della CLI
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` è opzionale; la didascalia può essere vuota per invii di solo contenuto multimediale.
  - `--dry-run` stampa il payload risultante; `--json` emette `{ channel, to, messageId, mediaUrl, caption }`.

<div id="whatsapp-web-channel-behavior">
  ## Comportamento del canale WhatsApp Web
</div>

- Input: percorso file locale **oppure** URL HTTP(S).
- Flusso: carica in un Buffer, rileva il tipo di media e costruisce il payload corretto:
  - **Immagini:** ridimensiona e ricomprime in JPEG (lato massimo 2048 px) mirando a `agents.defaults.mediaMaxMb` (predefinito 5 MB), con limite massimo di 6 MB.
  - **Audio/Voce/Video:** pass-through fino a 16 MB; l’audio viene inviato come nota vocale (`ptt: true`).
  - **Documenti:** qualsiasi altro tipo, fino a 100 MB, con nome file preservato quando disponibile.
- Riproduzione stile GIF di WhatsApp: invia un MP4 con `gifPlayback: true` (CLI: `--gif-playback`) in modo che i client mobili lo riproducano in loop inline.
- Il rilevamento MIME dà priorità ai magic byte, poi alle intestazioni, poi all’estensione del file.
- La didascalia proviene da `--message` o `reply.text`; è consentita anche una didascalia vuota.
- Logging: in modalità non-verbose mostra `↩️`/`✅`; in modalità verbose include dimensione e percorso/sorgente (file o URL).

<div id="auto-reply-pipeline">
  ## Pipeline di risposta automatica
</div>

- `getReplyFromConfig` restituisce `{ text?, mediaUrl?, mediaUrls? }`.
- Quando è presente contenuto multimediale, il web sender risolve i percorsi locali e gli URL utilizzando la stessa pipeline di `openclaw message send`.
- Più elementi multimediali vengono inviati in sequenza, se forniti.

<div id="inbound-media-to-commands-pi">
  ## Media in ingresso verso i comandi (Pi)
</div>

- Quando i messaggi web in ingresso includono media, OpenClaw scarica il contenuto in un file temporaneo ed espone variabili di template:
  - `{{MediaUrl}}` pseudo-URL per il media in ingresso.
  - `{{MediaPath}}` percorso temporaneo locale scritto prima dell'esecuzione del comando.
- Quando è abilitata una sandbox Docker per sessione, i media in ingresso vengono copiati nello spazio di lavoro della sandbox e `MediaPath`/`MediaUrl` vengono riscritti in un percorso relativo come `media/inbound/<filename>`.
- L’analisi dei media (se configurata tramite `tools.media.*` o `tools.media.models` condiviso) viene eseguita prima del templating e può inserire blocchi `[Image]`, `[Audio]` e `[Video]` in `Body`.
  - L'audio imposta `{{Transcript}}` e usa la trascrizione per l'analisi dei comandi, così i comandi slash continuano a funzionare.
  - Le descrizioni di video e immagini preservano qualsiasi testo di didascalia per l'analisi dei comandi.
- Per impostazione predefinita viene elaborato solo il primo allegato immagine/audio/video corrispondente; imposta `tools.media.<cap>.attachments` per elaborare più allegati.

<div id="limits-errors">
  ## Limiti e errori
</div>

**Limiti di invio (invio WhatsApp Web)**

- Immagini: limite di ~6 MB dopo ricompressione.
- Audio/voce/video: limite di 16 MB; documenti: limite di 100 MB.
- Media troppo grandi o illeggibili → errore esplicito nei log e la risposta viene ignorata.

**Limiti per la comprensione dei media (trascrizione/descrizione)**

- Limite predefinito immagini: 10 MB (`tools.media.image.maxBytes`).
- Limite predefinito audio: 20 MB (`tools.media.audio.maxBytes`).
- Limite predefinito video: 50 MB (`tools.media.video.maxBytes`).
- Per i media troppo grandi la fase di comprensione viene saltata, ma le risposte vengono comunque inviate con il corpo originale.

<div id="notes-for-tests">
  ## Note per i test
</div>

- Copri i flussi di send + reply per i casi di immagini/audio/documenti.
- Verifica la ricompressione per le immagini (limite di dimensione) e il flag di nota vocale per l'audio.
- Assicurati che le risposte multimediali vengano distribuite come send sequenziali.