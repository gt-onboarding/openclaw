---
title: Formale Verifikation (Sicherheitsmodelle)
summary: Maschinell geprüfte Sicherheitsmodelle für die risikoreichsten Angriffspfade in OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Formale Verifikation (Sicherheitsmodelle)
</div>

Diese Seite führt OpenClaws **formale Sicherheitsmodelle** auf (heute TLA+/TLC; weitere nach Bedarf).

> Hinweis: Einige ältere Links können sich auf den vorherigen Projektnamen beziehen.

**Ziel (Nordstern):** Eine maschinell geprüfte Argumentation liefern, dass OpenClaw seine
beabsichtigte Sicherheitsrichtlinie (Autorisierung, Sitzungsisolierung, Tool-Gating und
Sicherheit gegenüber Fehlkonfigurationen) unter expliziten Annahmen durchsetzt.

**Was das (heute) ist:** Eine ausführbare, angreifergetriebene **Sicherheits-Regression-Suite**:

- Jede Aussage verfügt über einen ausführbaren Model-Checking-Lauf über einen endlichen Zustandsraum.
- Viele Aussagen haben ein zugehöriges **Negativmodell**, das einen Gegenbeispiel-Trace für eine realistische Fehlerklasse erzeugt.

**Was das noch nicht ist:** Ein Beweis, dass „OpenClaw in jeder Hinsicht sicher ist“ oder dass die vollständige TypeScript-Implementierung korrekt ist.

<div id="where-the-models-live">
  ## Wo die Modelle abgelegt sind
</div>

Die Modelle werden in einem separaten Repository verwaltet: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Wichtige Einschränkungen
</div>

- Dies sind **Modelle**, nicht die vollständige TypeScript-Implementierung. Abweichungen zwischen Modell und Code sind möglich.
- Ergebnisse sind durch den von TLC untersuchten Zustandsraum begrenzt; „grün“ impliziert keine Sicherheit über die modellierten Annahmen und Grenzen hinaus.
- Einige Aussagen beruhen auf expliziten Annahmen über die Umgebung (z. B. korrekte Bereitstellung, korrekte Konfigurationsparameter).

<div id="reproducing-results">
  ## Ergebnisse reproduzieren
</div>

Heute werden Ergebnisse reproduziert, indem du das Model-Repository lokal klonst und TLC ausführst (siehe unten). Eine zukünftige Iteration könnte Folgendes bieten:

* in der CI ausgeführte Modelle mit öffentlichen Artefakten (Gegenbeispiel-Traces, Ausführungs-Logs)
* einen gehosteten „Dieses Modell ausführen“-Workflow für kleine, abgegrenzte Prüfungen

Erste Schritte:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ erforderlich (TLC läuft auf der JVM).
# Das Repository enthält eine gepinnte `tla2tools.jar` (TLA+-Tools) und stellt `bin/tlc` + Make-Targets bereit.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Gateway-Exposition und Fehlkonfiguration eines offenen Gateways
</div>

**These:** Ein Binden an mehr als nur das Loopback-Interface ohne Authentifizierung kann eine Remote-Kompromittierung ermöglichen bzw. die Angriffsfläche vergrößern; Token/Passwort sperren nicht authentisierte Angreifer aus (gemäß den Modellannahmen).

- Grüne Durchläufe:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rot (erwartet):
  - `make gateway-exposure-v2-negative`

Siehe auch: `docs/gateway-exposure-matrix.md` im Models-Repository.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run-Pipeline (Fähigkeit mit dem höchsten Risiko)
</div>

**Aussage:** `nodes.run` erfordert (a) eine Allowlist für Knotenbefehle sowie deklarierte Befehle und (b) eine Live-Genehmigung, falls entsprechend konfiguriert; Genehmigungen werden tokenisiert, um Replay zu verhindern (im Modell).

- Grüne Durchläufe:
  - `make nodes-pipeline`
  - `make approvals-token`
- Rote Durchläufe (erwartet):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Kopplungs-Store (DM-Gating)
</div>

**These:** Kopplungsanfragen respektieren TTL und Limits für ausstehende Anfragen.

- Grüne Durchläufe:
  - `make pairing`
  - `make pairing-cap`
- Rot (erwartet):
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress-Gating (Erwähnungen + Bypass von Steuerkommandos)
</div>

**Behauptung:** In Gruppenkontexten, in denen eine Erwähnung erforderlich ist, kann ein nicht autorisiertes „Steuerkommando“ das Erwähnungs-Gating nicht umgehen.

- Grün:
  - `make ingress-gating`
- Rot (erwartet):
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Routing-/Sitzungsschlüssel-Isolierung
</div>

**Behauptung:** DMs von unterschiedlichen Peers werden nicht zu derselben Sitzung zusammengeführt, es sei denn, sie sind explizit verknüpft/konfiguriert.

- Grün:
  - `make routing-isolation`
- Rot (erwartet):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: zusätzliche beschränkte Modelle (Nebenläufigkeit, Wiederholversuche, Trace-Korrektheit)
</div>

Dies sind aufbauende Modelle, die die Genauigkeit im Hinblick auf reale Fehlermodi (nicht-atomare Updates, Wiederholversuche und Message-Fan-out) erhöhen.

<div id="pairing-store-concurrency-idempotency">
  ### Kopplungsspeicher: Nebenläufigkeit / Idempotenz
</div>

**Behauptung:** Ein Kopplungsspeicher sollte `MaxPending` und Idempotenz auch bei Interleavings erzwingen (d. h. „check-then-write“ muss atomar/gesperrt sein; Refreshes/Aktualisierungen dürfen keine Duplikate erzeugen).

Was das bedeutet:

- Bei gleichzeitigen Anfragen darf `MaxPending` für einen Kanal nicht überschritten werden.
- Wiederholte Anfragen/Refreshes für dasselbe `(channel, sender)` dürfen keine doppelten noch offenen Pending-Einträge erzeugen.

- Grüne Durchläufe:
  - `make pairing-race` (atomare/gesperrte Cap-Prüfung)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Rot (erwartet):
  - `make pairing-race-negative` (nicht-atomares Begin/Commit-Cap-Race)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Ingress-Trace-Korrelation / Idempotenz
</div>

**Behauptung:** Ingestion sollte die Trace-Korrelation über Fan-out hinweg beibehalten und bei Anbieter-Retries idempotent sein.

Was das bedeutet:

- Wenn ein externes Event zu mehreren internen Nachrichten wird, behält jeder Teil dieselbe Trace-/Event-Identität.
- Retries führen nicht zu doppelter Verarbeitung.
- Wenn Anbieter-Event-IDs fehlen, greift die Deduplizierung auf einen sicheren Schlüssel zurück (z. B. Trace-ID), um zu vermeiden, dass unterschiedliche Events verworfen werden.

- Grün:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rot (erwartet):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Routing-dmScope-Priorität + identityLinks
</div>

**Behauptung:** Routing muss DM-Sitzungen standardmäßig isoliert halten und Sitzungen nur dann zusammenführen, wenn dies explizit konfiguriert ist (Kanal-Priorität + identityLinks).

Das bedeutet:

- Kanalspezifische dmScope-Overrides müssen gegenüber globalen Standardwerten Vorrang haben.
- identityLinks sollten nur innerhalb explizit verknüpfter Gruppen zusammengeführt werden, nicht über nicht verbundene Peers hinweg.

- Grün:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rot (erwartet):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`