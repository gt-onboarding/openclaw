---
title: Fotocamera
summary: "Acquisizione tramite fotocamera (nodo iOS + app macOS) per l'uso da parte dell'agente: foto (jpg) e brevi clip video (mp4)"
read_when:
  - Aggiunta o modifica dell'acquisizione tramite fotocamera su nodi iOS o macOS
  - Estensione dei workflow MEDIA con file temporanei accessibili all'agente
---

<div id="camera-capture-agent">
  # Acquisizione dalla fotocamera (agente)
</div>

OpenClaw supporta l’**acquisizione dalla fotocamera** per i flussi di lavoro dell’agente:

- **nodo iOS** (associato tramite Gateway): permette di acquisire una **foto** (`jpg`) o una **breve clip video** (`mp4`, con audio facoltativo) tramite `node.invoke`.
- **nodo Android** (associato tramite Gateway): permette di acquisire una **foto** (`jpg`) o una **breve clip video** (`mp4`, con audio facoltativo) tramite `node.invoke`.
- **app macOS** (nodo tramite Gateway): permette di acquisire una **foto** (`jpg`) o una **breve clip video** (`mp4`, con audio facoltativo) tramite `node.invoke`.

Tutti gli accessi alla fotocamera sono regolati da **impostazioni controllate dall’utente**.

<div id="ios-node">
  ## Nodo iOS
</div>

<div id="user-setting-default-on">
  ### Impostazione utente (attiva per impostazione predefinita)
</div>

- Scheda Impostazioni iOS → **Camera** → **Allow Camera** (`camera.enabled`)
  - Valore predefinito: **on** (una chiave mancante viene considerata come abilitata).
  - Quando è disattivata: i comandi `camera.*` restituiscono `CAMERA_DISABLED`.

<div id="commands-via-gateway-nodeinvoke">
  ### Comandi (tramite Gateway `node.invoke`)
</div>

- `camera.list`
  - Payload di risposta:
    - `devices`: array di `{ id, name, position, deviceType }`

- `camera.snap`
  - Parametri:
    - `facing`: `front|back` (predefinito: `front`)
    - `maxWidth`: number (opzionale; predefinito `1600` sul nodo iOS)
    - `quality`: `0..1` (opzionale; predefinito `0.9`)
    - `format`: attualmente `jpg`
    - `delayMs`: number (opzionale; predefinito `0`)
    - `deviceId`: string (opzionale; da `camera.list`)
  - Payload di risposta:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Controllo sul payload: le foto vengono ricomprese per mantenere il payload base64 sotto i 5 MB.

- `camera.clip`
  - Parametri:
    - `facing`: `front|back` (predefinito: `front`)
    - `durationMs`: number (predefinito `3000`, limitato a un massimo di `60000`)
    - `includeAudio`: boolean (predefinito `true`)
    - `format`: attualmente `mp4`
    - `deviceId`: string (opzionale; da `camera.list`)
  - Payload di risposta:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### Requisito di esecuzione in primo piano
</div>

Come per `canvas.*`, il nodo iOS consente i comandi `camera.*` solo in **primo piano**. Le chiamate in background restituiscono `NODE_BACKGROUND_UNAVAILABLE`.

<div id="cli-helper-temp-files-media">
  ### Helper CLI (file temporanei + MEDIA)
</div>

Il modo più semplice per recuperare gli allegati è tramite l&#39;helper CLI, che scrive i contenuti multimediali decodificati in un file temporaneo e stampa `MEDIA:<path>`.

Esempi:

```bash
openclaw nodes camera snap --node <id>               # predefinito: sia frontale che posteriore (2 righe MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Note:

* `nodes camera snap` per impostazione predefinita usa **entrambe** le fotocamere (frontale e posteriore) per dare all’agente entrambe le visuali.
* I file di output sono temporanei (nella directory temporanea del sistema operativo) a meno che tu non implementi un tuo wrapper personalizzato.


<div id="android-node">
  ## Nodo Android
</div>

<div id="user-setting-default-on">
  ### Impostazione utente (attiva per impostazione predefinita)
</div>

- Scheda Impostazioni Android → **Camera** → **Allow Camera** (`camera.enabled`)
  - Impostazione predefinita: **on** (se la chiave è assente viene considerata come abilitata).
  - Quando è **off**: i comandi `camera.*` restituiscono `CAMERA_DISABLED`.

<div id="permissions">
  ### Autorizzazioni
</div>

- Android richiede autorizzazioni di runtime:
  - `CAMERA` sia per `camera.snap` che per `camera.clip`.
  - `RECORD_AUDIO` per `camera.clip` quando `includeAudio=true`.

Se le autorizzazioni mancano, l'app mostrerà una richiesta di autorizzazione quando possibile; se vengono negate, le richieste `camera.*` falliscono con un errore
`*_PERMISSION_REQUIRED`.

### Requisito per il primo piano

Come per `canvas.*`, il nodo Android consente i comandi `camera.*` solo quando è in **primo piano**. Le invocazioni eseguite in background restituiscono `NODE_BACKGROUND_UNAVAILABLE`.

<div id="payload-guard">
  ### Protezione payload
</div>

Le foto vengono ricomprese per mantenere il payload base64 inferiore a 5 MB.

<div id="macos-app">
  ## app per macOS
</div>

<div id="user-setting-default-off">
  ### Impostazione utente (disattivata per impostazione predefinita)
</div>

L'app companion macOS espone una casella di controllo:

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - Impostazione predefinita: **off**
  - Quando è disattivata: le richieste alla fotocamera restituiscono “Fotocamera disattivata dall'utente”.

<div id="cli-helper-node-invoke">
  ### Helper CLI (invocazione node)
</div>

Usa la CLI principale `openclaw` per eseguire i comandi della fotocamera sul node macOS.

Esempi:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # stampa MEDIA:<path> (flag legacy)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Note:

* `openclaw nodes camera snap` usa come valore predefinito `maxWidth=1600`, a meno che non venga modificato.
* Su macOS, `camera.snap` attende `delayMs` (predefinito 2000 ms) dopo il riscaldamento e la stabilizzazione dell’esposizione, prima di scattare la foto.
* I payload delle foto vengono ricompressi per mantenere la stringa base64 entro i 5 MB.


<div id="safety-practical-limits">
  ## Limiti di sicurezza e pratici
</div>

- L’accesso a fotocamera e microfono attiva le consuete richieste di autorizzazione del sistema operativo (e richiede apposite stringhe di utilizzo nel file Info.plist).
- Le clip video sono limitate (attualmente `<= 60s`) per evitare payload del nodo di dimensioni eccessive (overhead base64 + limiti sui messaggi).

<div id="macos-screen-video-os-level">
  ## Video dello schermo macOS (a livello di sistema operativo)
</div>

Per la registrazione video dello *schermo* (non della fotocamera), usa l&#39;app companion per macOS:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # stampa MEDIA:<percorso>
```

Note:

* Richiede l’autorizzazione macOS **Registrazione schermo** (TCC).
