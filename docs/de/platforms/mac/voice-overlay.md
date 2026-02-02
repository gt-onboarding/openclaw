---
title: Sprachoverlay
summary: "Lebenszyklus des Sprachoverlays, wenn Aktivierungswort und Push-to-Talk überlappen"
read_when:
  - Verhalten des Sprachoverlays anpassen
---

<div id="voice-overlay-lifecycle-macos">
  # Voice-Overlay-Lebenszyklus (macOS)
</div>

Zielgruppe: Mitwirkende an der macOS-App. Ziel: Das Voice-Overlay vorhersagbar halten, wenn sich Wake-Word- und Push-to-Talk-Ereignisse überschneiden.

<div id="current-intent">
  ### Aktuelles Verhalten
</div>

- Wenn das Overlay bereits durch das Wake-Word sichtbar ist und der Benutzer den Hotkey drückt, übernimmt die Hotkey-Sitzung den bestehenden Text, anstatt ihn zurückzusetzen. Das Overlay bleibt sichtbar, solange der Hotkey gehalten wird. Wenn der Benutzer loslässt: senden, wenn bereinigter Text vorhanden ist, andernfalls ausblenden.
- Das Wake-Word allein sendet bei Stille weiterhin automatisch; Push-to-Talk sendet sofort beim Loslassen.

<div id="implemented-dec-9-2025">
  ### Implementiert (9. Dez. 2025)
</div>

- Overlay-Sitzungen führen jetzt ein Token pro Aufnahme (Weckwort oder Push-to-Talk). Partial-/Final-/Senden-/Verwerfen-/Level-Updates werden verworfen, wenn das Token nicht übereinstimmt, um veraltete Callbacks zu vermeiden.
- Push-to-Talk übernimmt jeden sichtbaren Overlay-Text als Präfix (d. h. das Drücken des Hotkeys, während das Weck-Overlay eingeblendet ist, behält den Text bei und hängt neu erkannte Sprache an). Es wartet bis zu 1,5 s auf ein endgültiges Transkript, bevor es auf den aktuellen Text zurückfällt.
- Chime-/Overlay-Logging wird auf Log-Level `info` in den Kategorien `voicewake.overlay`, `voicewake.ptt` und `voicewake.chime` ausgegeben (Sitzungsstart, partiell, final, Senden, Verwerfen, Chime-Grund).

<div id="next-steps">
  ### Nächste Schritte
</div>

1. **VoiceSessionCoordinator (Actor)**
   - Besitzt zu jedem Zeitpunkt genau eine `VoiceSession`.
   - API (tokenbasiert): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Verwirft Callbacks, die veraltete Token enthalten (verhindert, dass alte Recognizer das Overlay erneut öffnen).
2. **VoiceSession (Model)**
   - Felder: `token`, `source` (wakeWord|pushToTalk), finaler/volatiler Text, Signalton-Flags, Timer (Auto-Send, Inaktivität), `overlayMode` (display|editing|sending), Ablaufzeitpunkt der Cooldown-Phase.
3. **Overlay-Bindung**
   - `VoiceSessionPublisher` (`ObservableObject`) spiegelt die aktive Sitzung in SwiftUI.
   - `VoiceWakeOverlayView` rendert ausschließlich über den Publisher; globale Singletons werden niemals direkt verändert.
   - Overlay-Benutzeraktionen (`sendNow`, `dismiss`, `edit`) rufen den Koordinator mit dem Sitzungs-Token wieder auf.
4. **Einheitlicher Sendeweg**
   - Bei `endCapture`: Wenn getrimmter Text leer ist → Overlay schließen; andernfalls `performSend(session:)` (spielt einmal den Sende-Signalton ab, leitet weiter, schließt).
   - Push-to-Talk: keine Verzögerung; Wake-Word: optionale Verzögerung für Auto-Send.
   - Wende nach Abschluss von Push-to-Talk eine kurze Cooldown-Phase auf die Wake-Runtime an, damit das Wake-Word nicht sofort erneut ausgelöst wird.
5. **Logging**
   - Der Koordinator gibt `.info`-Logs im Subsystem `bot.molt` aus, Kategorien `voicewake.overlay` und `voicewake.chime`.
   - Schlüsselereignisse: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

<div id="debugging-checklist">
  ### Checkliste zur Fehlerdiagnose
</div>

- Logs streamen, während du ein festhängendes Overlay reproduzierst:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- Prüfe, dass nur ein aktives Sitzungstoken existiert; veraltete Callbacks sollten vom Koordinator verworfen werden.
- Stelle sicher, dass das Loslassen von Push-to-Talk immer `endCapture` mit dem aktiven Token aufruft; wenn der Text leer ist, sollte `dismiss` ohne Signalton und ohne Senden erfolgen.

<div id="migration-steps-suggested">
  ### Migrationsschritte (empfohlen)
</div>

1. Füge `VoiceSessionCoordinator`, `VoiceSession` und `VoiceSessionPublisher` hinzu.
2. Refaktoriere `VoiceWakeRuntime`, sodass Sitzungen erstellt/aktualisiert/beendet werden, anstatt `VoiceWakeOverlayController` direkt zu verwenden.
3. Refaktoriere `VoicePushToTalk`, um bestehende Sitzungen zu übernehmen und beim Loslassen `endCapture` aufzurufen; wende einen Cooldown zur Laufzeit an.
4. Verbinde `VoiceWakeOverlayController` mit dem Publisher; entferne direkte Aufrufe aus Runtime/PTT.
5. Füge Integrationstests für Sitzungsübernahme, Cooldown und das Verwerfen leerer Texte hinzu.