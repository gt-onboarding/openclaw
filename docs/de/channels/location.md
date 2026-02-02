---
title: Standort
summary: "Parsing von Standortdaten eingehender Kanäle (Telegram + WhatsApp) und zugehöriger Kontextfelder"
read_when:
  - Hinzufügen oder Ändern des Standort-Parsings für Kanäle
  - Verwenden von Standort-Kontextfeldern in Agent-Prompts oder Tools
---

<div id="channel-location-parsing">
  # Parsing von Kanal-Standorten
</div>

OpenClaw normalisiert geteilte Standortdaten aus Chat-Kanälen in:

- menschenlesbaren Text, der an den eingehenden Nachrichten-Body angehängt wird, und
- strukturierte Felder in der Kontext-Payload für automatische Antworten.

Derzeit unterstützt:

- **Telegram** (Standort-Pins + Orte (Venues) + Live-Standorte)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` mit `geo_uri`)

<div id="text-formatting">
  ## Textformatierung
</div>

Standortangaben werden als gut lesbare Zeilen ohne Klammern dargestellt:

* Pin:
  * `📍 48.858844, 2.294351 ±12m`
* Benannter Ort:
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* Live-Standortfreigabe:
  * `🛰 Live-Standort: 48.858844, 2.294351 ±12m`

Wenn der Kanal eine Bildunterschrift oder einen Kommentar enthält, wird dieser in der nächsten Zeile ergänzt:

```
📍 48.858844, 2.294351 ±12m
Treffen hier
```


<div id="context-fields">
  ## Kontextfelder
</div>

Wenn ein Standort vorhanden ist, werden diese Felder zu `ctx` hinzugefügt:

- `LocationLat` (Zahl)
- `LocationLon` (Zahl)
- `LocationAccuracy` (Zahl, in Metern; optional)
- `LocationName` (String; optional)
- `LocationAddress` (String; optional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolescher Wert)

<div id="channel-notes">
  ## Kanalhinweise
</div>

- **Telegram**: Venues werden auf `LocationName/LocationAddress` abgebildet; Live-Standorte verwenden `live_period`.
- **WhatsApp**: `locationMessage.comment` und `liveLocationMessage.caption` werden an die Beschriftungszeile angehängt.
- **Matrix**: `geo_uri` wird als Pin-Position interpretiert; die Höhe wird ignoriert und `LocationIsLive` ist immer `false`.