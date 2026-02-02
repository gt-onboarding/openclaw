---
title: Retry
summary: "Retry-Richtlinie für ausgehende Aufrufe an Anbieter"
read_when:
  - Beim Anpassen des Retry-Verhaltens oder der Standardwerte für Anbieter
  - Beim Debuggen von `send`-Fehlern von Anbietern oder von Rate-Limits
---

<div id="retry-policy">
  # Wiederholungsrichtlinie
</div>

<div id="goals">
  ## Ziele
</div>

- Pro HTTP-Anfrage wiederholen, nicht pro mehrstufigem Ablauf.
- Reihenfolge beibehalten, indem nur der aktuelle Schritt erneut ausgeführt wird.
- Duplizierung nicht-idempotenter Operationen vermeiden.

<div id="defaults">
  ## Standardwerte
</div>

- Versuche: 3
- Obergrenze für Verzögerung: 30000 ms
- Jitter: 0,1 (10 Prozent)
- Anbieter-Standardwerte:
  - Minimale Verzögerung für Telegram: 400 ms
  - Minimale Verzögerung für Discord: 500 ms

<div id="behavior">
  ## Verhalten
</div>

<div id="discord">
  ### Discord
</div>

- Wiederholungsversuche nur bei Rate-Limit-Fehlern (HTTP 429).
- Verwendet Discords `retry_after`-Wert, wenn verfügbar, andernfalls ein exponentielles Backoff.

<div id="telegram">
  ### Telegram
</div>

- Erneute Versuche bei vorübergehenden Fehlern (429, Timeout, Verbindungsaufbau/-zurücksetzung/-abbruch, vorübergehend nicht verfügbar).
- Verwendet `retry_after`, falls verfügbar, sonst exponentielles Backoff.
- Markdown-Parsefehler werden nicht erneut versucht; stattdessen wird auf reinen Text zurückgegriffen.

<div id="configuration">
  ## Konfiguration
</div>

Setze die Retry-Policy pro Anbieter in `~/.openclaw/openclaw.json` fest:

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
  ## Hinweise
</div>

- Retries gelten pro Request (Senden von Nachrichten, Medien-Upload, Reaktionen, Umfragen, Sticker).
- Zusammengesetzte Abläufe führen bereits abgeschlossene Schritte nicht erneut aus.