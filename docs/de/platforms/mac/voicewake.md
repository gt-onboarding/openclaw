---
title: Voicewake
summary: "Sprachaktivierung und Push-to-Talk-Modi sowie Routingdetails in der macOS-App"
read_when:
  - Bei der Arbeit an Voice-Wake- oder PTT-Abläufen
---

<div id="voice-wake-push-to-talk">
  # Sprachaktivierung & Push-to-Talk
</div>

<div id="modes">
  ## Modi
</div>

- **Wake-Word-Modus** (Standard): Der dauerhaft aktive Spracherkenner wartet auf Trigger-Wörter (`swabbleTriggerWords`). Bei Treffer startet er die Aufnahme, zeigt das Overlay mit Teiltranskription an und sendet nach erkannter Stille automatisch.
- **Push-to-talk (rechte Option-Taste gedrückt halten)**: Halte die rechte Option-Taste, um sofort aufzunehmen – kein Trigger erforderlich. Das Overlay erscheint, solange sie gehalten wird; beim Loslassen wird nach einer kurzen Verzögerung abgeschlossen und gesendet, damit du den Text noch anpassen kannst.

<div id="runtime-behavior-wake-word">
  ## Laufzeitverhalten (Wake-Word)
</div>

- Der Spracherkenner ist in `VoiceWakeRuntime` implementiert.
- Der Trigger löst nur aus, wenn eine **deutliche Pause** zwischen dem Wake-Word und dem nächsten Wort liegt (~0,55 s Lücke). Das Overlay/der Signalton kann bereits während dieser Pause starten, noch bevor der Befehl beginnt.
- Zeitfenster für Stille: 2,0 s, wenn Sprache fortläuft, 5,0 s, wenn nur der Trigger erkannt wurde.
- Hartes Limit: 120 s, um aus dem Ruder laufende Sitzungen zu verhindern.
- Entprellintervall (Debounce) zwischen Sitzungen: 350 ms.
- Das Overlay wird über `VoiceWakeOverlayController` mit committed/volatile-Farbkodierung gesteuert.
- Nach dem Senden startet der Spracherkenner sauber neu, um auf den nächsten Trigger zu warten.

<div id="lifecycle-invariants">
  ## Lebenszyklus-Invarianten
</div>

- Wenn Voice Wake aktiviert ist und die Berechtigungen erteilt wurden, sollte der Wake-Word-Recognizer auf Eingaben lauschen (außer während einer expliziten Push-to-Talk-Aufnahme).
- Die Sichtbarkeit des Overlays (einschließlich des manuellen Schließens über die X-Schaltfläche) darf niemals verhindern, dass der Wake-Word-Recognizer das Zuhören wieder aufnimmt.

<div id="sticky-overlay-failure-mode-previous">
  ## Fehlermodus bei hängenbleibendem Overlay (früher)
</div>

Früher konnte Voice Wake „tot“ wirken, wenn das Overlay sichtbar hängen blieb und du es manuell geschlossen hast, weil der Neustartversuch der Runtime durch die Overlay-Sichtbarkeit blockiert werden konnte und kein nachfolgender Neustart geplant wurde.

Härtung:

- Neustarts der Wake-Runtime werden nicht mehr durch die Overlay-Sichtbarkeit blockiert.
- Der Abschluss des Schließens des Overlays löst ein `VoiceWakeRuntime.refresh(...)` über `VoiceSessionCoordinator` aus, sodass ein manuelles Schließen per X das Zuhören immer wieder aufnimmt.

<div id="push-to-talk-specifics">
  ## Push-to-talk-Details
</div>

- Die Hotkey-Erkennung verwendet einen globalen `.flagsChanged`-Monitor für die **rechte Option-Taste** (`keyCode 61` + `.option`). Wir beobachten nur Ereignisse (ohne sie abzufangen/zu unterdrücken).
- Die Capture-Pipeline sitzt in `VoicePushToTalk`: startet die Spracherkennung sofort, streamt Teilergebnisse ins Overlay und ruft `VoiceWakeForwarder` beim Loslassen auf.
- Wenn Push-to-Talk startet, pausieren wir die Wake-Word-Runtime, um konkurrierende Audio-Zugriffe zu vermeiden; sie startet nach dem Loslassen automatisch neu.
- Berechtigungen: erfordert Mikrofon + Spracherkennung; für das Erfassen von Ereignissen ist eine Freigabe für Bedienungshilfen/Eingabeüberwachung nötig.
- Externe Tastaturen: manche stellen die rechte Option-Taste nicht wie erwartet bereit – biete eine alternative Tastenkombination als Fallback an, wenn Nutzer verpasste Auslösungen melden.

<div id="user-facing-settings">
  ## Benutzereinstellungen
</div>

- **Voice Wake**-Schalter: aktiviert die Wake-Word-Erkennung zur Laufzeit.
- **Cmd+Fn gedrückt halten zum Sprechen**: aktiviert den Push-to-Talk-Monitor. Deaktiviert auf macOS < 26.
- Sprach- & Mikrofon-Auswahl, Live-Pegelanzeige, Trigger-Wort-Tabelle, Tester (nur lokal; leitet nicht weiter).
- Die Mikrofon-Auswahl merkt sich die letzte Auswahl, wenn ein Gerät getrennt wird, zeigt einen Hinweis auf die getrennte Verbindung an und greift vorübergehend auf die Systemvorgabe zurück, bis das Gerät wieder verfügbar ist.
- **Sounds**: Signaltöne bei Trigger-Erkennung und beim Senden; standardmäßig ist der macOS-Systemton „Glass“ eingestellt. Du kannst für jedes Ereignis jede von `NSSound` ladbare Datei (z. B. MP3/WAV/AIFF) auswählen oder **Kein Sound** wählen.

<div id="forwarding-behavior">
  ## Weiterleitungsverhalten
</div>

- Wenn Voice Wake aktiviert ist, werden Transkripte an den aktiven Gateway/agenten weitergeleitet (im gleichen lokalen bzw. entfernten Modus wie im Rest der Mac-App).
- Antworten werden an den **zuletzt verwendeten primären Anbieter** (WhatsApp/Telegram/Discord/WebChat) zugestellt. Wenn die Zustellung fehlschlägt, wird der Fehler protokolliert und der Run ist weiterhin über WebChat-/Sitzungsprotokolle sichtbar.

<div id="forwarding-payload">
  ## Weiterleitungs-Payload
</div>

- `VoiceWakeForwarder.prefixedTranscript(_:)` fügt den maschinellen Hinweis an den Anfang des Transkripts, bevor es gesendet wird. Wird sowohl im Wake-Word- als auch im Push-to-Talk-Pfad verwendet.

<div id="quick-verification">
  ## Schnelltest
</div>

- Push-to-Talk aktivieren, Cmd+Fn gedrückt halten, sprechen, loslassen: Das Overlay sollte zuerst Teiltranskripte anzeigen und dann senden.
- Während du die Tasten gedrückt hältst, sollten die Ohren in der Menüleiste vergrößert bleiben (via `triggerVoiceEars(ttl:nil)`); nach dem Loslassen werden sie wieder kleiner.