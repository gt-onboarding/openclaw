---
title: Posizione
summary: "Analisi dei dati di posizione dei canali in ingresso (Telegram + WhatsApp) e campi di contesto"
read_when:
  - Aggiunta o modifica dell'analisi dei dati di posizione dei canali
  - Utilizzo dei campi di contesto relativi alla posizione nei prompt o negli strumenti degli agenti
---

<div id="channel-location-parsing">
  # Analisi della posizione del canale
</div>

OpenClaw normalizza le posizioni condivise dai canali di chat in:

- testo leggibile dalle persone aggiunto al body in ingresso e
- campi strutturati nel payload di contesto per le risposte automatiche.

Attualmente supportati:

- **Telegram** (pin di posizione + luoghi (venue) + posizioni in tempo reale)
- **WhatsApp** (`locationMessage` + `liveLocationMessage`)
- **Matrix** (`m.location` con `geo_uri`)

<div id="text-formatting">
  ## Formattazione del testo
</div>

Le posizioni vengono visualizzate come righe leggibili senza parentesi:

* Pin:
  * `📍 48.858844, 2.294351 ±12m`
* Luogo con nome:
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* Condivisione in tempo reale:
  * `🛰 Posizione in tempo reale: 48.858844, 2.294351 ±12m`

Se il canale include una didascalia o un commento, questo viene aggiunto sulla riga successiva:

```
📍 48.858844, 2.294351 ±12m
Ci vediamo qui
```


<div id="context-fields">
  ## Campi di contesto
</div>

Quando è disponibile una posizione, questi campi vengono aggiunti a `ctx`:

- `LocationLat` (numero)
- `LocationLon` (numero)
- `LocationAccuracy` (numero, metri; opzionale)
- `LocationName` (stringa; opzionale)
- `LocationAddress` (stringa; opzionale)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (booleano)

<div id="channel-notes">
  ## Note sui canali
</div>

- **Telegram**: i venue vengono mappati a `LocationName/LocationAddress`; le posizioni in tempo reale usano `live_period`.
- **WhatsApp**: `locationMessage.comment` e `liveLocationMessage.caption` vengono aggiunti come riga di didascalia.
- **Matrix**: `geo_uri` viene interpretato come un punto sulla mappa; l'altitudine viene ignorata e `LocationIsLive` è sempre false.