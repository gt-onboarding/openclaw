---
title: TUI
summary: "Terminal-UI (TUI): von jedem beliebigen Rechner aus mit dem Gateway verbinden"
read_when:
  - Du möchtest eine einsteigerfreundliche Einführung in die TUI
  - Du benötigst die komplette Liste der TUI-Funktionen, -Befehle und -Tastenkürzel
---

<div id="tui-terminal-ui">
  # TUI (Terminal UI)
</div>

<div id="quick-start">
  ## Schnellstart
</div>

1. Starte das Gateway.

```bash
openclaw gateway
```

2. Öffnen Sie die TUI.

```bash
openclaw tui
```

3. Gib eine Nachricht ein und drücke die Eingabetaste.

Remote Gateway:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Verwenden Sie `--password`, wenn Ihr Gateway eine Passwortauthentifizierung nutzt.

<div id="what-you-see">
  ## Was du siehst
</div>

* Kopfzeile: Verbindungs-URL, aktueller Agent, aktuelle Sitzung.
* Chatprotokoll: Benutzernachrichten, Assistentenantworten, Systemhinweise, Tool-Karten.
* Statuszeile: Verbindungs-/Ausführungsstatus (Verbindungsaufbau, läuft, Streaming, Leerlauf, Fehler).
* Fußzeile: Verbindungsstatus + Agent + Sitzung + Modell + think/verbose/reasoning + Token-Zähler + Senden.
* Eingabe: Text-Editor mit Autovervollständigung.

<div id="mental-model-agents-sessions">
  ## Mentales Modell: Agenten + Sitzungen
</div>

* Agenten sind eindeutige Slugs (z. B. `main`, `research`). Das Gateway stellt diese Liste bereit.
* Sitzungen gehören zum aktuellen agent.
* Sitzungsschlüssel werden als `agent:<agentId>:<sessionKey>` gespeichert.
  * Wenn du `/session main` eingibst, erweitert die TUI dies zu `agent:<currentAgent>:main`.
  * Wenn du `/session agent:other:main` eingibst, wechselst du explizit zu der Sitzung dieses agents.
* Sitzungs-Scope:
  * `per-sender` (Standard): Jeder agent kann mehrere Sitzungen haben.
  * `global`: Die TUI verwendet immer die `global`-Sitzung (der Picker kann leer sein).
* Der aktuelle agent + die aktuelle Sitzung sind immer in der Fußzeile sichtbar.

<div id="sending-delivery">
  ## Senden + Zustellung
</div>

* Nachrichten werden an das Gateway gesendet; die Zustellung an Anbieter ist standardmäßig deaktiviert.
* Aktiviere die Zustellung:
  * `/deliver on`
  * oder über den Einstellungsbereich
  * oder starte mit `openclaw tui --deliver`

<div id="pickers-overlays">
  ## Auswahlfelder und Overlays
</div>

* Modell-Auswahl: listet verfügbare Modelle auf und setzt das Sitzungs-Override.
* Agent-Auswahl: Wähle einen anderen agent.
* Sitzungs-Auswahl: zeigt nur Sitzungen für den aktuellen agent.
* Einstellungen: Schalte Zustellung, Erweiterung der Werkzeugausgabe und Sichtbarkeit des Denkprozesses um.

<div id="keyboard-shortcuts">
  ## Tastenkürzel
</div>

* Enter: Nachricht senden
* Esc: aktuellen Lauf abbrechen
* Ctrl+C: Eingabe löschen (zweimal drücken zum Beenden)
* Ctrl+D: Beenden
* Ctrl+L: Modellauswahl
* Ctrl+G: Agent-Auswahl
* Ctrl+P: Sitzungsauswahl
* Ctrl+O: Werkzeugausgabe ein-/ausklappen
* Ctrl+T: Denkschritte ein-/ausblenden (lädt Verlauf neu)

<div id="slash-commands">
  ## Slash-Befehle
</div>

Kern:

* `/help`
* `/status`
* `/agent <id>` (oder `/agents`)
* `/session <key>` (oder `/sessions`)
* `/model <provider/model>` (oder `/models`)

Sitzungssteuerung:

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>` (Alias: `/elev`)
* `/activation <mention|always>`
* `/deliver <on|off>`

Sitzungslebenszyklus:

* `/new` oder `/reset` (setzt die Sitzung zurück)
* `/abort` (bricht den aktiven Durchlauf ab)
* `/settings`
* `/exit`

Andere Slash-Befehle des Gateways (zum Beispiel `/context`) werden an das Gateway weitergeleitet und als Systemausgabe angezeigt. Siehe [Slash-Befehle](/de/tools/slash-commands).

<div id="local-shell-commands">
  ## Lokale Shell-Befehle
</div>

* Stelle einer Zeile ein `!` voran, um einen lokalen Shell-Befehl auf dem TUI-Host auszuführen.
* Die TUI fragt einmal pro Sitzung nach der Erlaubnis für lokale Ausführung; bei Ablehnung bleibt `!` für die Sitzung deaktiviert.
* Befehle werden in einer frischen, nicht-interaktiven Shell im Arbeitsverzeichnis der TUI ausgeführt (kein persistentes `cd` und keine persistenten Umgebungsvariablen).
* Ein einzelnes `!` wird als normale Nachricht gesendet; führende Leerzeichen lösen keine lokale Ausführung aus.

<div id="tool-output">
  ## Toolausgabe
</div>

* Toolaufrufe werden als Karten mit Argumenten und Ergebnissen dargestellt.
* Strg+O wechselt zwischen eingeklappter und ausgeklappter Ansicht.
* Während Tools laufen, werden Zwischenergebnisse fortlaufend in derselben Karte aktualisiert.

<div id="history-streaming">
  ## Verlauf + Streaming
</div>

* Beim Herstellen der Verbindung lädt die TUI den neuesten Verlauf (standardmäßig 200 Nachrichten).
* Streaming-Antworten werden inline aktualisiert, bis sie abgeschlossen sind.
* Die TUI hört zudem auf agent-Tool-Ereignisse, um aussagekräftigere Tool-Karten anzuzeigen.

<div id="connection-details">
  ## Verbindungsdetails
</div>

* Die TUI registriert sich beim Gateway mit `mode: "tui"`.
* Bei erneuten Verbindungen wird eine Systemnachricht angezeigt; Ereignislücken werden im Log ausgewiesen.

<div id="options">
  ## Optionen
</div>

* `--url <url>`: Gateway-WebSocket-URL (Standardwert: aus Konfiguration oder `ws://127.0.0.1:<port>`)
* `--token <token>`: Gateway-Token (falls erforderlich)
* `--password <password>`: Gateway-Passwort (falls erforderlich)
* `--session <key>`: Sitzungsschlüssel (Standard: `main` oder `global`, wenn scope global ist)
* `--deliver`: Liefert Antworten des Assistenten an den anbieter (standardmäßig aus)
* `--thinking <level>`: Thinking-Level für Sendevorgänge überschreiben
* `--timeout-ms <ms>`: Agent-Timeout in ms (Standard ist `agents.defaults.timeoutSeconds`)

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

Keine Ausgabe nach dem Senden einer Nachricht:

* Führe `/status` in der TUI aus, um zu bestätigen, dass der Gateway verbunden und im Leerlauf/ausgelastet ist.
* Prüfe die Gateway-Logs: `openclaw logs --follow`.
* Stelle sicher, dass der agent lauffähig ist: `openclaw status` und `openclaw models status`.
* Wenn du Nachrichten in einem Chat-Kanal erwartest, aktiviere die Zustellung (`/deliver on` oder `--deliver`).
* `--history-limit &lt;n&gt;`: Anzahl der Verlaufseinträge, die geladen werden (Standard: 200)

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

* `disconnected`: Stell sicher, dass das Gateway läuft und deine `--url/--token/--password` korrekt sind.
* Keine Agenten in der Auswahlliste: Prüfe `openclaw agents list` und deine Routing-Konfiguration.
* Leere Sitzungsauswahl: Du bist möglicherweise im globalen Scope oder hast noch keine Sitzungen.