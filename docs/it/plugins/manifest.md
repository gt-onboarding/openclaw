---
title: Manifest
summary: "Manifest del plugin + requisiti dello schema JSON (rigorosa convalida della configurazione)"
read_when:
  - Stai sviluppando un plugin OpenClaw
  - Devi fornire uno schema di configurazione del plugin o diagnosticare errori di convalida del plugin
---

<div id="plugin-manifest-openclawpluginjson">
  # Manifest del plugin (openclaw.plugin.json)
</div>

Ogni plugin **deve** includere un file `openclaw.plugin.json` nella **directory principale del plugin**.
OpenClaw utilizza questo manifest per convalidare la configurazione **senza eseguire il codice del plugin**.
I manifest mancanti o non validi vengono considerati errori del plugin e impediscono
la convalida della configurazione.

Consulta la guida completa al sistema dei plugin: [Plugins](/it/plugin).

<div id="required-fields">
  ## Campi obbligatori
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Chiavi obbligatorie:

* `id` (string): id canonico del plugin.
* `configSchema` (object): JSON Schema per la configurazione del plugin (inline).

Chiavi opzionali:

* `kind` (string): tipo di plugin (esempio: `"memory"`).
* `channels` (array): id dei canali registrati da questo plugin (esempio: `["matrix"]`).
* `providers` (array): id dei provider registrati da questo plugin.
* `skills` (array): directory delle abilità da caricare (relative alla directory root del plugin).
* `name` (string): nome visualizzato per il plugin.
* `description` (string): breve descrizione del plugin.
* `uiHints` (object): etichette dei campi/segnaposto/flag di riservatezza per il rendering della UI.
* `version` (string): versione del plugin (solo informativa).

<div id="json-schema-requirements">
  ## Requisiti dello schema JSON
</div>

* **Ogni plugin deve includere uno schema JSON**, anche se non accetta alcuna configurazione.
* Uno schema vuoto è accettabile (ad esempio, `{ "type": "object", "additionalProperties": false }`).
* Gli schemi vengono validati in fase di lettura/scrittura della configurazione, non a runtime.

<div id="validation-behavior">
  ## Comportamento di validazione
</div>

* Le chiavi `channels.*` sconosciute sono **errori**, a meno che l&#39;ID del canale non sia dichiarato in un manifest di plugin.
* `plugins.entries.<id>`, `plugins.allow`, `plugins.deny` e `plugins.slots.*`
  devono fare riferimento a ID di plugin **individuabili**. Gli ID sconosciuti sono **errori**.
* Se un plugin è installato ma ha un manifest o uno schema mancanti o non validi,
  la validazione non va a buon fine e Doctor segnala l&#39;errore del plugin.
* Se esiste una configurazione del plugin ma il plugin è **disabilitato**, la configurazione viene mantenuta e
  viene generato un **avviso** in Doctor e nei log.

<div id="notes">
  ## Note
</div>

* Il manifest è **obbligatorio per tutti i plugin**, inclusi i caricamenti dal filesystem locale.
* Il runtime carica comunque il modulo del plugin separatamente; il manifest serve solo per
  rilevamento + validazione.
* Se il tuo plugin dipende da moduli nativi, documenta i passaggi di build e
  eventuali requisiti di lista di autorizzati del gestore di pacchetti (per esempio, pnpm `allow-build-scripts`
  * `pnpm rebuild <package>`).