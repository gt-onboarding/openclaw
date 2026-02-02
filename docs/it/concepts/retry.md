---
title: Retry
summary: "Criteri di retry per le chiamate in uscita verso i provider"
read_when:
  - Aggiornamento del comportamento di retry o dei valori predefiniti dei provider
  - Debug degli errori del metodo send dei provider o dei rate limit
---

<div id="retry-policy">
  # Criteri di ritentativo
</div>

<div id="goals">
  ## Obiettivi
</div>

- Effettuare i retry per singola richiesta HTTP, non per l’intero flusso multi-step.
- Mantenere l’ordinamento ritentando solo lo step corrente.
- Evitare di duplicare le operazioni non idempotenti.

<div id="defaults">
  ## Valori predefiniti
</div>

- Tentativi: 3
- Limite massimo del ritardo: 30000 ms
- Jitter: 0,1 (10 per cento)
- Valori predefiniti del provider:
  - Ritardo minimo per Telegram: 400 ms
  - Ritardo minimo per Discord: 500 ms

<div id="behavior">
  ## Funzionamento
</div>

<div id="discord">
  ### Discord
</div>

- Riprova solo in caso di errori di rate limit (HTTP 429).
- Usa il `retry_after` di Discord quando disponibile, altrimenti applica un backoff esponenziale.

<div id="telegram">
  ### Telegram
</div>

- Ritenta in caso di errori temporanei (429, timeout, connessione reimpostata/chiusa, servizio temporaneamente non disponibile).
- Usa `retry_after` quando disponibile, altrimenti applica un backoff esponenziale.
- Gli errori di parsing del Markdown non vengono ritentati; in questi casi si passa al testo semplice.

<div id="configuration">
  ## Configurazione
</div>

Imposta la retry policy per ogni provider in `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```


<div id="notes">
  ## Note
</div>

- I tentativi di ripetizione si applicano per ogni richiesta (invio di messaggi, caricamento di file multimediali, reazione, sondaggio, sticker).
- I flussi compositi non ritentano i passaggi già completati.