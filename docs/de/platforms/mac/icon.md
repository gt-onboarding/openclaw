---
title: Symbol
summary: "Status und Animationen des Menüleistensymbols von OpenClaw unter macOS"
read_when:
  - Verhalten des Menüleistensymbols ändern
---

<div id="menu-bar-icon-states">
  # Menüleisten-Icon-Zustände
</div>

Author: steipete · Aktualisiert: 2025-12-06 · Scope: macOS app (`apps/macos`)

- **Leerlauf:** Normale Icon-Animation (Blinken, gelegentliches Wackeln).
- **Pausiert:** Status-Item verwendet `appearsDisabled`; keine Bewegung.
- **Sprachtrigger (große Ohren):** Der Sprachaktivierungsdetektor ruft `AppState.triggerVoiceEars(ttl: nil)` auf, wenn das Aktivierungswort gehört wird, und hält `earBoostActive=true`, während die Äußerung aufgezeichnet wird. Die Ohren vergrößern sich (1,9x), erhalten runde Ohrlöcher für bessere Lesbarkeit und werden dann über `stopVoiceEars()` nach 1 s Stille wieder zurückgesetzt. Wird nur aus der In-App-Sprachpipeline ausgelöst.
- **Arbeitend (agent läuft):** `AppState.isWorking=true` steuert eine „Schwanz-/Bein‑Scurry“-Mikrobewegung: schnellere Beinbewegung und leichte Verschiebung, während Arbeit läuft. Wird derzeit rund um WebChat-agent-Läufe getoggelt; füge denselben Toggle um andere lange Tasks, wenn du sie anbindest.

Wiring-Punkte

- Sprachaktivierung: zur Laufzeit/im Test `AppState.triggerVoiceEars(ttl: nil)` beim Trigger und `stopVoiceEars()` nach 1 s Stille aufrufen, um das Capture-Fenster nachzubilden.
- Agent-Aktivität: `AppStateStore.shared.setWorking(true/false)` um Arbeitsspannen herum setzen (bereits im WebChat-agent-Aufruf erledigt). Halte Spannen kurz und setze in `defer`-Blöcken zurück, um festhängende Animationen zu vermeiden.

Formen & Größen

- Basis-Icon wird in `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` gezeichnet.
- Ohr-Skalierung ist standardmäßig `1.0`; der Sprach-Boost setzt `earScale=1.9` und toggelt `earHoles=true`, ohne den Gesamtrahmen zu ändern (18×18 pt Template-Bild, gerendert in einen 36×36 px Retina-Backing-Store).
- Scurry verwendet Beinbewegung bis ~1.0 mit einem kleinen horizontalen Wackeln; sie ist additiv zu jeglichem vorhandenen Leerlaufwackeln.

Verhaltenshinweise

- Kein externer CLI-/Broker-Toggle für Ohren/Arbeitszustand; halte das intern zu den eigenen Signalen der App, um unbeabsichtigtes Flapping zu vermeiden.
- Halte TTLs kurz (&lt;10 s), damit das Icon schnell zum Ausgangszustand zurückkehrt, falls ein Job hängen bleibt.