---
title: Indicatori di digitazione
summary: "Quando OpenClaw mostra gli indicatori di digitazione e come personalizzarli"
read_when:
  - Modifica del comportamento o delle impostazioni predefinite degli indicatori di digitazione
---

<div id="typing-indicators">
  # Indicatori di digitazione
</div>

Gli indicatori di digitazione vengono inviati al canale di chat mentre un'esecuzione è in corso. Usa
`agents.defaults.typingMode` per controllare **quando** inizia la digitazione e `typingIntervalSeconds`
per controllare **ogni quanto** viene aggiornata.

<div id="defaults">
  ## Valori predefiniti
</div>

Quando `agents.defaults.typingMode` **non è impostato**, OpenClaw mantiene il comportamento legacy:

- **Chat dirette**: la digitazione inizia immediatamente non appena ha inizio il loop del modello.
- **Chat di gruppo con menzione**: la digitazione inizia immediatamente.
- **Chat di gruppo senza menzione**: la digitazione inizia solo quando il testo del messaggio inizia a essere trasmesso in streaming.
- **Esecuzioni heartbeat**: la digitazione è disabilitata.

<div id="modes">
  ## Modalità
</div>

Imposta `agents.defaults.typingMode` su uno dei seguenti valori:

- `never` — nessun indicatore di digitazione, mai.
- `instant` — inizia a digitare **non appena inizia il loop del modello**, anche se
  l'esecuzione in seguito restituisce solo il token di risposta silenziosa.
- `thinking` — inizia a digitare al **primo delta di ragionamento** (richiede
  `reasoningLevel: "stream"` per l'esecuzione).
- `message` — inizia a digitare al **primo delta di testo non silenzioso** (ignora
  il token silenzioso `NO_REPLY`).

Ordine di “quanto presto si attiva”:
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## Configurazione
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

Puoi sovrascrivere la modalità o la cadenza per singola sessione:

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## Note
</div>

- La modalità `message` non mostrerà la digitazione per risposte esclusivamente silenziose (ad es. il token `NO_REPLY`
  usato per sopprimere l’output).
- `thinking` si attiva solo se la run esegue lo streaming del ragionamento (`reasoningLevel: "stream"`).
  Se il modello non emette delta di ragionamento, la digitazione non partirà.
- Gli heartbeat non mostrano mai la digitazione, indipendentemente dalla modalità.
- `typingIntervalSeconds` controlla la **cadenza di aggiornamento**, non il momento di avvio.
  Il valore predefinito è 6 secondi.