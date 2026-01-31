---
title: Cron-Add-Härtung
summary: "Härtung der Eingabeverarbeitung von cron.add, Angleichung der Schemas und Verbesserung des Cron-UI-/Agent-Toolings"
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Härtung von cron add &amp; Schema-Angleichung
</div>

<div id="context">
  ## Kontext
</div>

Aktuelle Gateway-Logs zeigen wiederholte `cron.add`-Fehlermeldungen mit ungültigen Parametern (fehlende `sessionTarget`, `wakeMode`, `payload` und fehlerhaftes `schedule`). Dies deutet darauf hin, dass mindestens ein Client (vermutlich der Aufrufpfad des agent-Tools) gekapselte oder nur teilweise spezifizierte Job-Payloads sendet. Außerdem gibt es Abweichungen zwischen den Cron-Anbieter-Enums in TypeScript, dem Gateway-Schema, CLI-Flags und UI-Formulartypen, sowie eine UI-Unstimmigkeit für `cron.status` (erwartet `jobCount`, während das Gateway `jobs` zurückgibt).

<div id="goals">
  ## Ziele
</div>

* Den `cron.add`-INVALID&#95;REQUEST-Spam stoppen, indem gängige Wrapper-Payloads normalisiert und fehlende `kind`-Felder abgeleitet werden.
* Cron-Anbieter-Listen über Gateway-Schema, Cron-Typen, CLI-Dokumentation und UI-Formulare hinweg angleichen.
* Das Agent-Cron-Tool-Schema explizit machen, damit das LLM korrekte Job-Payloads erzeugt.
* Die Anzeige der Jobanzahl im Cron-Status der Control UI korrigieren.
* Tests hinzufügen, die Normalisierung und Tool-Verhalten abdecken.

<div id="non-goals">
  ## Nichtziele
</div>

* Nicht die Semantik der cron-Zeitplanung oder das Ausführungsverhalten von Jobs ändern.
* Keine neuen Zeitplantypen hinzufügen oder das Parsen von cron-Ausdrücken erweitern.
* Die UI/UX für cron nicht über die notwendigen Feldkorrekturen hinaus überarbeiten.

<div id="findings-current-gaps">
  ## Feststellungen (aktuelle Lücken)
</div>

* `CronPayloadSchema` im Gateway schließt `signal` + `imessage` aus, während die TypeScript-Typen sie enthalten.
* Die CronStatus-Ansicht in der Control UI erwartet `jobCount`, das Gateway liefert jedoch `jobs`.
* Das Cron-Tool-Schema des Agents erlaubt beliebige `job`-Objekte und ermöglicht dadurch fehlerhafte Eingaben.
* Das Gateway validiert `cron.add` strikt ohne Normalisierung, sodass gewrappte Payloads fehlschlagen.

<div id="what-changed">
  ## Was sich geändert hat
</div>

* `cron.add` und `cron.update` normalisieren jetzt gängige Wrapper-Strukturen und leiten fehlende `kind`-Felder ab.
* Das Agent-Cron-Tool-Schema entspricht dem Gateway-Schema, was ungültige Payloads reduziert.
* Anbieter-Enums sind über Gateway, CLI, UI und den macOS-Picker hinweg angeglichen.
* Die Control UI verwendet für den Status das `jobs`-Zählerfeld des Gateways.

<div id="current-behavior">
  ## Aktuelles Verhalten
</div>

* **Normalisierung:** eingebettete `data`/`job`-Payloads werden ausgepackt; `schedule.kind` und `payload.kind` werden abgeleitet, wenn dies sicher möglich ist.
* **Defaults:** sichere Standardwerte werden für `wakeMode` und `sessionTarget` gesetzt, wenn sie fehlen.
* **Anbieter:** Discord/Slack/Signal/iMessage werden jetzt konsistent in CLI und UI angezeigt.

Siehe [Cron-Jobs](/de/automation/cron-jobs) für das normalisierte Format und Beispiele.

<div id="verification">
  ## Überprüfung
</div>

* Überwache die Gateway-Logs auf eine verringerte Anzahl von `cron.add`-INVALID&#95;REQUEST-Fehlern.
* Stelle sicher, dass der Cron-Status in der Control UI nach dem Aktualisieren die Anzahl der Jobs anzeigt.

<div id="optional-follow-ups">
  ## Optionale Folgeaufgaben
</div>

* Manueller Control-UI-Smoke-Test: Cronjob pro Anbieter hinzufügen + Anzahl der Status-Jobs überprüfen.

<div id="open-questions">
  ## Offene Fragen
</div>

* Sollte `cron.add` expliziten `state` von Clients akzeptieren (derzeit durch das Schema nicht erlaubt)?
* Sollten wir `webchat` als expliziten Zustellungsanbieter erlauben (derzeit bei der Zustellungsauflösung herausgefiltert)?