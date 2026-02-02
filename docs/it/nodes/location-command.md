---
title: Comando posizione
summary: "Comando posizione per i nodi (location.get), modalità di autorizzazione e comportamento in background"
read_when:
  - Aggiunta del supporto per nodi di localizzazione o UI delle autorizzazioni
  - Progettazione di flussi di localizzazione in background e push
---

<div id="location-command-nodes">
  # Comando di localizzazione (nodi)
</div>

<div id="tldr">
  ## TL;DR
</div>

- `location.get` è un comando del nodo (via `node.invoke`).
- Disattivato per impostazione predefinita.
- Le impostazioni utilizzano un selettore: Disattivato / Durante l'utilizzo / Sempre.
- Interruttore separato: Posizione precisa.

<div id="why-a-selector-not-just-a-switch">
  ## Perché un selettore (non solo un interruttore)
</div>

Le autorizzazioni del sistema operativo sono multilivello. Possiamo esporre un selettore nell'app, ma è comunque il sistema operativo a decidere l'autorizzazione effettiva.

- iOS/macOS: l'utente può scegliere **While Using** o **Always** nei prompt di sistema/in Impostazioni. L'app può richiedere un upgrade dell'autorizzazione, ma il sistema operativo può imporre il passaggio dalle Impostazioni.
- Android: la posizione in background è un'autorizzazione separata; su Android 10+ spesso richiede una procedura nelle Impostazioni.
- La posizione precisa è un'autorizzazione separata (iOS 14+ “Precise”, Android “fine” vs “coarse”).

Il selettore nella UI determina la modalità che richiediamo; l'autorizzazione effettiva è gestita nelle impostazioni del sistema operativo.

<div id="settings-model">
  ## Modello di impostazioni
</div>

Per ogni nodo/dispositivo:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

Comportamento della UI:

- Se selezioni `whileUsing`, viene richiesta l’autorizzazione per l’uso in primo piano.
- Se selezioni `always`, viene prima garantita `whileUsing`, quindi viene richiesta l’autorizzazione per l’uso in background (o l’utente viene inviato a Impostazioni se necessario).
- Se il sistema operativo nega il livello richiesto, viene ripristinato il livello più alto concesso e lo stato viene mostrato all’utente.

<div id="permissions-mapping-nodepermissions">
  ## Mappatura delle autorizzazioni (node.permissions)
</div>

Facoltativo. Il nodo macOS espone `location` tramite la mappa delle autorizzazioni; iOS/Android possono ometterlo.

<div id="command-locationget">
  ## Comando: `location.get`
</div>

Richiamato tramite `node.invoke`.

Parametri (consigliati):

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Payload della risposta:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Errors (codici stabili):

* `LOCATION_DISABLED`: selettore disattivato.
* `LOCATION_PERMISSION_REQUIRED`: autorizzazione mancante per la modalità richiesta.
* `LOCATION_BACKGROUND_UNAVAILABLE`: l&#39;app è in background ma è consentito solo &quot;Durante l&#39;uso&quot;.
* `LOCATION_TIMEOUT`: impossibile ottenere la posizione entro il tempo previsto.
* `LOCATION_UNAVAILABLE`: errore di sistema / nessun provider disponibile.


<div id="background-behavior-future">
  ## Comportamento in background (futuro)
</div>

Obiettivo: il modello possa richiedere la posizione anche quando il nodo è in background, ma solo quando:

- L’utente ha selezionato **Sempre**.
- Il sistema operativo concede l’accesso alla posizione in background.
- All’app è consentita l’esecuzione in background per la posizione (modalità background iOS / servizio in foreground Android o autorizzazione speciale).

Flusso attivato da push (futuro):

1) Il Gateway invia un push al nodo (push silenzioso o dati FCM).
2) Il nodo si riattiva brevemente e richiede la posizione al dispositivo.
3) Il nodo inoltra il payload al Gateway.

Note:

- iOS: sono richieste l’autorizzazione Sempre e la modalità posizione in background. I push silenziosi possono essere limitati; prevedi errori intermittenti.
- Android: la posizione in background può richiedere un servizio in foreground; in caso contrario, è probabile che la richiesta venga negata.

<div id="modeltooling-integration">
  ## Integrazione modelli/strumenti
</div>

- Superficie dello strumento: il tool `nodes` aggiunge l'azione `location_get` (nodo richiesto).
- CLI: `openclaw nodes location get --node <id>`.
- Linee guida per l'agente: effettua la chiamata solo quando l'utente ha abilitato la condivisione della posizione e comprende lo scope.

<div id="ux-copy-suggested">
  ## Testo UX (suggerito)
</div>

- Disattivato: “La condivisione della posizione è disattivata.”
- Durante l'uso: “Solo quando OpenClaw è in esecuzione.”
- Sempre: “Consenti la posizione in background. Richiede un'autorizzazione di sistema.”
- Preciso: “Usa la posizione GPS precisa. Disattiva per condividere solo una posizione approssimativa.”