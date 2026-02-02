---
title: Formale Verifikation (Sicherheitsmodelle)
summary: Maschinell geprüfte Sicherheitsmodelle für die höchstrisikoreichen Pfade von OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Formale Verifikation (Sicherheitsmodelle)
</div>

Diese Seite dokumentiert OpenClaws **formale Sicherheitsmodelle** (heute TLA+/TLC; weitere bei Bedarf).

> Hinweis: Einige ältere Links können sich noch auf den früheren Projektnamen beziehen.

**Ziel (Nordstern):** Eine maschinell geprüfte Argumentation liefern, dass OpenClaw seine
beabsichtigte Sicherheitsrichtlinie (Autorisierung, Sitzungsisolation, Tool-Gating und
Robustheit gegenüber Fehlkonfigurationen) unter expliziten Annahmen durchsetzt.

**Was das (heute) ist:** eine ausführbare, angriffsgetriebene **Sicherheits-Regressionstest-Suite**:

- Jede Aussage verfügt über eine ausführbare Modellprüfung über einen endlichen Zustandsraum.
- Viele Aussagen haben ein zugehöriges **negatives Modell**, das einen Gegenbeispiel-Trace für eine realistische Fehlerklasse erzeugt.

**Was das (noch) nicht ist:** ein Beweis, dass „OpenClaw in jeder Hinsicht sicher ist“ oder dass die vollständige TypeScript-Implementierung korrekt ist.

<div id="where-the-models-live">
  ## Wo die Modelle verwaltet werden
</div>

Die Modelle werden in einem separaten Repository verwaltet: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Wichtige Einschränkungen
</div>

- Dies sind **Modelle**, nicht die vollständige TypeScript-Implementierung. Abweichungen zwischen Modell und Code sind möglich.
- Die Ergebnisse sind durch den von TLC explorierten Zustandsraum begrenzt; „Grün“ bedeutet keine Sicherheit über die modellierten Annahmen und Grenzen hinaus.
- Einige Aussagen stützen sich auf explizite Umgebungsannahmen (z. B. korrekte Bereitstellung, korrekte Konfigurationseingaben).

<div id="reproducing-results">
  ## Ergebnisse reproduzieren
</div>

Aktuell werden Ergebnisse reproduziert, indem du das Model-Repository lokal klonst und TLC ausführst (siehe unten). Eine zukünftige Iteration könnte Folgendes bieten:

* In der CI ausgeführte Modelle mit öffentlich zugänglichen Artefakten (Gegenbeispiel-Traces, Ausführungsprotokolle)
* Einen gehosteten „Dieses Modell ausführen“-Workflow für kleine, begrenzte Prüfungen

Erste Schritte:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ erforderlich (TLC läuft auf der JVM).
# Das Repository enthält eine gepinnte `tla2tools.jar` (TLA+-Tools) und stellt `bin/tlc` + Make-Targets bereit.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Exponierung des Gateways und Fehlkonfiguration eines offenen Gateways
</div>

**Behauptung:** Das Binden an andere Interfaces als Loopback ohne Authentifizierung kann eine Remote-Kompromittierung ermöglichen bzw. die Exponierung erhöhen; ein Token/Passwort blockiert nicht authentifizierte Angreifer (gemäß den Modellannahmen).

- Grüne Durchläufe:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rot (erwartet):
  - `make gateway-exposure-v2-negative`

Siehe auch: `docs/gateway-exposure-matrix.md` im Models-Repo.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run-Pipeline (Fähigkeit mit höchstem Risiko)
</div>

**Behauptung:** `nodes.run` erfordert (a) eine Allowlist für Knotenbefehle plus deklarierte Befehle und (b) eine Live-Freigabe, wenn so konfiguriert; Genehmigungen werden tokenisiert, um Replay-Angriffe zu verhindern (im Modell).

- Grün (sollte erfolgreich sein):
  - `make nodes-pipeline`
  - `make approvals-token`
- Rot (soll fehlschlagen):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Kopplungsspeicher (DM-Gating)
</div>

**Aussage:** Kopplungsanforderungen beachten TTL und Obergrenzen für ausstehende Anfragen.

- Grüne Läufe:
  - `make pairing`
  - `make pairing-cap`
- Rote Läufe (erwartet):
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress-Gating (Erwähnungen + Bypass von Control-Commands)
</div>

**Behauptung:** In Gruppenkontexten, in denen eine Erwähnung erforderlich ist, kann ein nicht autorisierter „Control-Command“ das Mention-Gating nicht umgehen.

- Grün:
  - `make ingress-gating`
- Rot (erwartet):
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Isolation von Routing-/Sitzungsschlüsseln
</div>

**Behauptung:** DMs (Direktnachrichten) von unterschiedlichen Peers werden nicht in derselben Sitzung zusammengeführt, sofern sie nicht explizit miteinander verknüpft bzw. entsprechend konfiguriert werden.

- Grün:
  - `make routing-isolation`
- Rot (erwartet):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: zusätzliche begrenzte Modelle (Nebenläufigkeit, Retries, Trace-Korrektheit)
</div>

Dies sind weiterführende Modelle, die die Genauigkeit in Bezug auf Fehlermodi in der Praxis (nicht-atomare Updates, Retries und Message-Fan-out) weiter erhöhen.

<div id="pairing-store-concurrency-idempotency">
  ### Kopplungs-Store: Nebenläufigkeit / Idempotenz
</div>

**Behauptung:** Ein Kopplungs-Store sollte `MaxPending` und Idempotenz auch unter Interleavings konkurrierender Vorgänge erzwingen (d. h. „check-then-write“ muss atomar / gesperrt sein; Refreshes sollten keine Duplikate erzeugen).

Was das bedeutet:

- Unter gleichzeitigen Anfragen darfst du `MaxPending` für einen Channel nicht überschreiten.
- Wiederholte Anfragen/Refreshes für dasselbe `(channel, sender)` dürfen keine doppelten aktiven Pending-Einträge erzeugen.

- Grün (soll bestehen):
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
- Wenn Anbieter-Event-IDs fehlen, greift das Deduplizieren auf einen sicheren Schlüssel (z. B. Trace-ID) zurück, um zu vermeiden, dass unterschiedliche Events verworfen werden.

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

**Behauptung:** Routing muss DM-Sitzungen standardmäßig isoliert halten und Sitzungen nur dann zusammenführen, wenn dies explizit konfiguriert ist (Kanal-Priorität + Identity Links).

Was das bedeutet:

- Kanal-spezifische dmScope-Overrides müssen gegenüber globalen Standardeinstellungen Vorrang haben.
- identityLinks sollten Sitzungen nur innerhalb explizit verknüpfter Gruppen zusammenführen, nicht über nicht verwandte Peers hinweg.

- Grün:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rot (erwartet):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`