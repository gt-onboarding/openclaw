---
title: Bilder
summary: "Regeln zur Bild- und Medienverarbeitung für send-, Gateway- und Agent-Antworten"
read_when:
  - Beim Ändern der Medien-Pipeline oder von Anhängen
---

<div id="image-media-support-2025-12-05">
  # Bild- & Medienunterstützung — 2025-12-05
</div>

Der WhatsApp-Kanal verwendet **Baileys Web**. Dieses Dokument fasst die aktuellen Regeln zur Medienverarbeitung für Senden-, Gateway- und agent-Antworten zusammen.

<div id="goals">
  ## Ziele
</div>

- Medien mit optionalen Beschriftungen über `openclaw message send --media` senden.
- Ermöglichen, dass automatische Antworten aus dem Web-Posteingang Medien zusammen mit Text enthalten.
- Grenzwerte pro Typ sinnvoll und vorhersagbar halten.

<div id="cli-surface">
  ## CLI-Oberfläche
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` optional; die Bildunterschrift kann für reine Medien-Sendungen leer bleiben.
  - `--dry-run` gibt die aufgelöste Nutzlast aus; `--json` gibt `{ channel, to, messageId, mediaUrl, caption }` aus.

<div id="whatsapp-web-channel-behavior">
  ## Verhalten des WhatsApp-Web-Kanals
</div>

- Eingabe: lokaler Dateipfad **oder** HTTP(S)-URL.
- Ablauf: in einen Buffer laden, Medientyp erkennen und den passenden Payload erzeugen:
  - **Bilder:** Größenanpassung & erneute Komprimierung zu JPEG (maximale Kantenlänge 2048 px) mit Zielwert `agents.defaults.mediaMaxMb` (Standard 5 MB), begrenzt auf 6 MB.
  - **Audio/Sprachnachrichten/Video:** Durchleitung bis 16 MB; Audio wird als Sprachnachricht gesendet (`ptt: true`).
  - **Dokumente:** alles andere, bis 100 MB, Dateiname wird nach Möglichkeit beibehalten.
- GIF-ähnliche Wiedergabe in WhatsApp: Sende ein MP4 mit `gifPlayback: true` (CLI: `--gif-playback`), damit es auf Mobilgeräten inline in einer Schleife wiedergegeben wird.
- MIME-Erkennung bevorzugt Magic Bytes, dann Header, dann Dateiendung.
- Beschriftung stammt aus `--message` oder `reply.text`; eine leere Beschriftung ist erlaubt.
- Logging: im nicht-verbose-Modus wird `↩️`/`✅` angezeigt; im verbose-Modus zusätzlich Größe und Quellpfad/URL.

<div id="auto-reply-pipeline">
  ## Pipeline für automatische Antworten
</div>

- `getReplyFromConfig` gibt `{ text?, mediaUrl?, mediaUrls? }` zurück.
- Wenn Medien vorhanden sind, löst der Web-Sender lokale Pfade oder URLs mit derselben Pipeline wie `openclaw message send` auf.
- Mehrere Medienelemente werden, falls vorhanden, nacheinander gesendet.

<div id="inbound-media-to-commands-pi">
  ## Eingehende Medien zu Befehlen (Pi)
</div>

- Wenn eingehende Web-Nachrichten Medien enthalten, lädt OpenClaw diese in eine temporäre Datei herunter und stellt Templating-Variablen bereit:
  - `{{MediaUrl}}` Pseudo-URL für die eingehenden Medien.
  - `{{MediaPath}}` lokaler temporärer Pfad, der vor der Ausführung des Befehls angelegt wird.
- Wenn eine Docker-sandbox pro Sitzung aktiviert ist, werden eingehende Medien in den Sandbox-Arbeitsbereich kopiert und `MediaPath`/`MediaUrl` auf einen relativen Pfad wie `media/inbound/<filename>` umgeschrieben.
- Die Medienerkennung (falls über `tools.media.*` oder gemeinsame `tools.media.models` konfiguriert) läuft vor dem Templating und kann `[Image]`-, `[Audio]`- und `[Video]`-Blöcke in `Body` einfügen.
  - Audio setzt `{{Transcript}}` und verwendet dieses Transkript für die Befehlsanalyse, sodass Slash-Befehle weiterhin funktionieren.
  - Video- und Bildbeschreibungen erhalten vorhandenen Beschriftungstext für die Befehlsanalyse.
- Standardmäßig wird nur der erste passende Bild-/Audio-/Video-Anhang verarbeitet; setze `tools.media.<cap>.attachments`, um mehrere Anhänge zu verarbeiten.

<div id="limits-errors">
  ## Begrenzungen & Fehler
</div>

**Ausgehende Sendelimits (WhatsApp Web send)**

- Bilder: ca. 6 MB Limit nach Neukomprimierung.
- Audio/Sprachaufnahmen/Video: 16 MB Limit; Dokumente: 100 MB Limit.
- Zu große oder nicht lesbare Medien → eindeutiger Fehler in den Logs, und die Antwort wird übersprungen.

**Limits für Medienverarbeitung (Transkription/Beschreibung)**

- Standard für Bilder: 10 MB (`tools.media.image.maxBytes`).
- Standard für Audio: 20 MB (`tools.media.audio.maxBytes`).
- Standard für Video: 50 MB (`tools.media.video.maxBytes`).
- Zu große Medien werden bei der Verarbeitung übersprungen, aber Antworten werden weiterhin mit dem ursprünglichen Nachrichteninhalt gesendet.

<div id="notes-for-tests">
  ## Hinweise für Tests
</div>

- Decke Sende- und Antwortflüsse für Bild-, Audio- und Dokumentfälle ab.
- Überprüfe die Neukomprimierung für Bilder (Größenbegrenzung) und das Sprachnotiz-Flag für Audio.
- Stelle sicher, dass Multimedia-Antworten in sequentielle Sendevorgänge aufgeteilt werden.