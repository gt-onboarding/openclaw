---
title: Denken
summary: "Direktivsyntax für /think + /verbose und wie sie das Reasoning des Modells beeinflusst"
read_when:
  - Anpassen des Parsings oder der Standardwerte für /think- oder /verbose-Direktiven
---

<div id="thinking-levels-think-directives">
  # Denkebenen (/think-Direktiven)
</div>

<div id="what-it-does">
  ## Was es macht
</div>

* Inline-Direktive im Body jeder eingehenden Anfrage: `/t <level>`, `/think:<level>` oder `/thinking <level>`.
* Level (Aliasse): `off | minimal | low | medium | high | xhigh` (nur GPT-5.2 + Codex-Modelle)
  * minimal → „think“
  * low → „think hard“
  * medium → „think harder“
  * high → „ultrathink“ (maximales Budget)
  * xhigh → „ultrathink+“ (nur GPT-5.2 + Codex-Modelle)
  * `highest`, `max` werden `high` zugeordnet.
* Anbieter-Hinweise:
  * Z.AI (`zai/*`) unterstützt nur binäres „Thinking“ (`on`/`off`). Jeder nicht-`off`-Level wird als `on` behandelt (zugeordnet zu `low`).

<div id="resolution-order">
  ## Auflösungsreihenfolge
</div>

1. Inline-Direktive an der Nachricht (gilt nur für diese Nachricht).
2. Sitzungs-Override (gesetzt durch Senden einer reinen Direktive-Nachricht).
3. Globaler Standardwert (`agents.defaults.thinkingDefault` in der Konfiguration).
4. Fallback: niedrig für Modelle mit Reasoning-Fähigkeiten, sonst aus.

<div id="setting-a-session-default">
  ## Standard für eine Sitzung festlegen
</div>

* Sende eine Nachricht, die **nur** aus der Direktive besteht (Leerzeichen erlaubt), z. B. `/think:medium` oder `/t high`.
* Diese gilt für die aktuelle Sitzung (standardmäßig pro Absender) und wird durch `/think:off` oder ein Zurücksetzen bei Sitzungs-Leerlauf aufgehoben.
* Es wird eine Bestätigungsantwort gesendet (`Denklevel auf hoch gesetzt.` / `Denken deaktiviert.`). Wenn die Stufe ungültig ist (z. B. `/thinking big`), wird der Befehl mit einem Hinweis abgelehnt und der Sitzungszustand bleibt unverändert.
* Sende `/think` (oder `/think:`) ohne Argumente, um das aktuelle Denklevel anzuzeigen.

<div id="application-by-agent">
  ## Anwendung pro agent
</div>

* **Embedded Pi**: Das ermittelte Level wird an die im Prozess laufende Pi-agent-Laufzeitumgebung übergeben.

<div id="verbose-directives-verbose-or-v">
  ## Ausführliche Direktiven (/verbose oder /v)
</div>

* Stufen: `on` (minimal) | `full` | `off` (Standard).
* Eine Nachricht, die nur aus der Direktive besteht, schaltet die ausführliche Sitzungsprotokollierung um und antwortet mit `Verbose logging enabled.` / `Verbose logging disabled.`; ungültige Stufen liefern einen Hinweis, ohne den Zustand zu ändern.
* `/verbose off` speichert eine explizite Sitzungsüberschreibung; entferne sie über die Sessions UI, indem du `inherit` wählst.
* Eine Inline-Direktive wirkt nur auf diese Nachricht; ansonsten gelten die Standardwerte der Sitzung bzw. global.
* Sende `/verbose` (oder `/verbose:`) ohne Argument, um die aktuelle Ausführlichkeitsstufe zu sehen.
* Wenn Ausführlichkeit aktiviert ist (`on`), senden Agenten, die strukturierte Tool-Ergebnisse ausgeben (Pi, andere JSON-Agenten), jeden Tool-Aufruf als eigene, reine Metadaten-Nachricht zurück, mit Präfix `<emoji> <tool-name>: <arg>` (falls verfügbar, Pfad/Befehl). Diese Tool-Zusammenfassungen werden gesendet, sobald jedes Tool startet (separate Bubbles), nicht als Streaming-Deltas.
* Wenn Ausführlichkeit auf `full` steht, werden Tool-Ausgaben nach Abschluss ebenfalls weitergeleitet (separate Bubble, auf eine sichere Länge gekürzt). Wenn du `/verbose on|full|off` umschaltest, während eine Ausführung noch läuft, beachten nachfolgende Tool-Bubbles die neue Einstellung.

<div id="reasoning-visibility-reasoning">
  ## Sichtbarkeit des Reasonings (/reasoning)
</div>

* Stufen: `on|off|stream`.
* Eine Nachricht, die nur aus Direktiven besteht, steuert, ob Thinking‑Blöcke in Antworten angezeigt werden.
* Wenn aktiviert, wird das Reasoning als **separate Nachricht** mit dem Präfix `Reasoning:` gesendet.
* `stream` (nur Telegram): streamt das Reasoning in das Telegram‑Entwurfsfeld, während die Antwort generiert wird, und sendet dann die endgültige Antwort ohne Reasoning.
* Alias: `/reason`.
* Sende `/reasoning` (oder `/reasoning:`) ohne Argument, um die aktuelle Reasoning‑Stufe anzuzeigen.

<div id="related">
  ## Verwandt
</div>

* Die Dokumentation zum Elevated-Modus findest du unter [Elevated mode](/de/tools/elevated).

<div id="heartbeats">
  ## Herzschläge
</div>

* Der Inhalt der Herzschlag-Abfrage ist der konfigurierte Herzschlag-Prompt (Standard: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Inline-Direktiven in einer Herzschlag-Nachricht gelten wie üblich (vermeide es aber, Sitzungs-Defaults durch Herzschläge zu ändern).
* Die Herzschlag-Zustellung umfasst standardmäßig nur die finale Nutzlast. Um zusätzlich die separate `Reasoning:`-Nachricht zu senden (falls vorhanden), setze `agents.defaults.heartbeat.includeReasoning: true` oder pro agent `agents.list[].heartbeat.includeReasoning: true`.

<div id="web-chat-ui">
  ## Web-Chat-UI
</div>

* Der Denkstufen-Selektor im Web-Chat spiegelt beim Laden der Seite die in der eingehenden Sitzungs-Store/-Konfiguration gespeicherte Stufe der Sitzung wider.
* Wenn du eine andere Stufe auswählst, gilt diese nur für die nächste Nachricht (`thinkingOnce`); nach dem Senden springt der Selektor auf die gespeicherte Sitzungsstufe zurück.
* Um den Sitzungsstandard zu ändern, sende wie bisher eine `/think:<level>`-Direktive; der Selektor spiegelt dies nach dem nächsten Laden der Seite wider.