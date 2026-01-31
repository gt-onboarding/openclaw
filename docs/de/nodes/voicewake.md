---
title: Voicewake
summary: "Globale, Gateway-eigene Sprachaktivierungswörter (Wake Words) und wie sie über Knoten hinweg synchronisiert werden"
read_when:
  - Wenn du das Verhalten oder die Standardwerte der Sprachaktivierungswörter änderst
  - Wenn du neue Knoten-Plattformen hinzufügst, die eine Synchronisierung der Sprachaktivierungswörter benötigen
---

<div id="voice-wake-global-wake-words">
  # Voice Wake (Globale Weckwörter)
</div>

OpenClaw behandelt **Weckwörter als eine einzige globale Liste**, die vom **Gateway** verwaltet wird.

- Es gibt **keine knotenspezifischen benutzerdefinierten Weckwörter**.
- **Jede Knoten- bzw. App-UI kann** die Liste bearbeiten; Änderungen werden vom Gateway gespeichert und an alle verteilt.
- Jedes Gerät behält weiterhin eine eigene Umschaltoption **Voice Wake aktiviert/deaktiviert** (lokale UX und Berechtigungen können sich unterscheiden).

<div id="storage-gateway-host">
  ## Speicherort (Gateway-Host)
</div>

Wake-Wörter werden auf dem Gateway-Host unter folgendem Pfad gespeichert:

* `~/.openclaw/settings/voicewake.json`

Struktur:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## Protokoll
</div>

<div id="methods">
  ### Methoden
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` mit Parametern `{ triggers: string[] }` → `{ triggers: string[] }`

Hinweise:

- Trigger werden normalisiert (getrimmt, leere Einträge entfernt). Leere Listen fallen auf Standardwerte zurück.
- Zur Sicherheit werden Grenzwerte (für Anzahl/Länge) durchgesetzt.

<div id="events">
  ### Events
</div>

- `voicewake.changed` Payload `{ triggers: string[] }`

Wer erhält sie:

- Alle WebSocket-Clients (macOS-App, WebChat usw.)
- Alle verbundenen Knoten (iOS/Android) sowie beim Verbinden eines Knotens als anfänglicher Push des „aktuellen Zustands“.

<div id="client-behavior">
  ## Clientverhalten
</div>

<div id="macos-app">
  ### macOS-App
</div>

- Verwendet die globale Liste, um `VoiceWakeRuntime`-Trigger zu steuern.
- Das Bearbeiten von „Triggerwörtern“ in den Voice-Wake-Einstellungen ruft `voicewake.set` auf und nutzt anschließend den Broadcast, um andere Clients synchron zu halten.

<div id="ios-node">
  ### iOS-Knoten
</div>

- Verwendet die globale Liste für die Trigger-Erkennung in `VoiceWakeManager`.
- Das Bearbeiten von Weckwörtern in den Einstellungen ruft `voicewake.set` (über den Gateway-WS) auf und hält zugleich die lokale Weckwort-Erkennung reaktionsfähig.

<div id="android-node">
  ### Android-Knoten
</div>

- Stellt einen Wake-Word-Editor in den Einstellungen bereit.
- Ruft `voicewake.set` über die Gateway-WS auf, damit Änderungen überall synchronisiert werden.