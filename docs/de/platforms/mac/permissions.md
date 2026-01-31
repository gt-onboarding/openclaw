---
title: Berechtigungen
summary: "Persistenz von macOS-Berechtigungen (TCC) und Signaturanforderungen"
read_when:
  - Debuggen fehlender oder hängenbleibender macOS-Berechtigungsdialoge
  - Paketierung oder Signieren der macOS-App
  - Ändern von Bundle-IDs oder App-Installationspfaden
---

<div id="macos-permissions-tcc">
  # macOS-Berechtigungen (TCC)
</div>

macOS-Berechtigungen sind anfällig. TCC verknüpft eine erteilte Berechtigung mit der
Code-Signatur der App, dem Bundle-Identifier und dem Pfad auf dem Datenträger. Wenn sich einer dieser Werte ändert,
behandelt macOS die App als neu und kann Nachfragen verwerfen oder ausblenden.

<div id="requirements-for-stable-permissions">
  ## Anforderungen an stabile Berechtigungen
</div>

- Gleicher Pfad: Führe die App immer vom gleichen Speicherort aus (für OpenClaw `dist/OpenClaw.app`).
- Gleicher Bundle-Identifier: Das Ändern der Bundle-ID erzeugt eine neue Berechtigungsidentität.
- Signierte App: Nicht signierte oder ad-hoc-signierte Builds behalten Berechtigungen nicht bei.
- Konsistente Signatur: Verwende ein echtes Apple Development- oder Developer-ID-Zertifikat,
  damit die Signatur über erneute Builds hinweg stabil bleibt.

Ad-hoc-Signaturen erzeugen bei jedem Build eine neue Identität. macOS „vergisst“ frühere
Berechtigungen, und Berechtigungsabfragen können vollständig verschwinden, bis die veralteten Einträge bereinigt sind.

<div id="recovery-checklist-when-prompts-disappear">
  ## Checkliste zur Wiederherstellung, wenn Berechtigungsabfragen nicht mehr erscheinen
</div>

1. Beende die App.
2. Entferne den App-Eintrag in Systemeinstellungen -&gt; Datenschutz &amp; Sicherheit.
3. Starte die App vom selben Pfad neu und erteile die Berechtigungen erneut.
4. Wenn die Abfrage immer noch nicht erscheint, setze die TCC-Einträge mit `tccutil` zurück und versuche es erneut.
5. Einige Berechtigungen erscheinen erst nach einem vollständigen Neustart von macOS erneut.

Beispielbefehle zum Zurücksetzen (Bundle-ID nach Bedarf ersetzen):

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

Wenn du Berechtigungen testest, signiere immer mit einem echten Zertifikat. Ad-hoc-Builds sind nur für schnelle lokale Testläufe geeignet, bei denen Berechtigungen keine Rolle spielen.
