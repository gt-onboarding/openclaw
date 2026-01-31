---
title: Clawnet
summary: "Clawnet-Refaktorierung: Netzwerkprotokoll, Rollen, Auth, Freigaben und Identität vereinheitlichen"
read_when:
  - Wenn du ein einheitliches Netzwerkprotokoll für Knoten + Operator-Clients planst
  - Wenn du Freigaben, Kopplung, TLS und Präsenz geräteübergreifend überarbeitest
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Clawnet-Refactoring (Vereinheitlichung von Protokoll und Authentifizierung)
</div>

<div id="hi">
  ## Hi
</div>

Hi Peter – gute Richtung; das ermöglicht eine einfachere UX und stärkt die Sicherheit.

<div id="purpose">
  ## Zweck
</div>

Ein einziges, präzises Dokument für:

- Aktueller Stand: Protokolle, Abläufe, Vertrauensgrenzen.
- Pain Points: Freigaben, Multi‑Hop‑Routing, UI‑Duplizierung.
- Vorgeschlagener Zielzustand: ein Protokoll, klar abgegrenzte Rollen, vereinheitlichte Auth/kopplung, TLS‑Pinning.
- Identitätsmodell: stabile IDs + niedliche Slugs.
- Migrationsplan, Risiken, offene Fragen.

<div id="goals-from-discussion">
  ## Ziele (aus der Diskussion)
</div>

- Ein Protokoll für alle Clients (macOS-App, CLI, iOS, Android, Headless-Knoten).
- Jeder Netzwerkteilnehmer ist authentifiziert und gepairt.
- Klare Rollentrennung: Knoten vs. Operatoren.
- Zentrale Freigaben werden dorthin geleitet, wo sich der Benutzer befindet.
- TLS-Verschlüsselung + optionales Pinning für sämtlichen entfernten Datenverkehr.
- Minimale Code-Duplizierung.
- Eine einzelne Maschine soll nur einmal erscheinen (kein doppelter UI-/Knoten-Eintrag).

<div id="nongoals-explicit">
  ## Nichtziele (explizit)
</div>

- Trennung von Berechtigungen aufheben (Least-Privilege-Prinzip wird weiterhin benötigt).
- Die vollständige Gateway-Control-Plane ohne Scope-Prüfungen exponieren.
- Authentifizierung von menschlich vergebenen Labels abhängig machen (Slugs bleiben sicherheitsunerheblich).

---

<div id="current-state-asis">
  # Aktueller Stand (Ist-Zustand)
</div>

<div id="two-protocols">
  ## Zwei Protokolle
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket (Control-Plane)
</div>

- Gesamte API-Oberfläche: Konfiguration, Kanäle, Modelle, Sitzungen, Agent-Ausführungen, Logs, Knoten usw.
- Standard-Bindung: Loopback. Entfernter Zugriff über SSH/Tailscale.
- Authentifizierung: Token/Passwort über `connect`.
- Kein TLS-Pinning (stützt sich auf Loopback/Tunnel).
- Code:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge (Node-Transport)
</div>

- Kleine Angriffsfläche durch Allowlist, Knotenidentität + Kopplung.
- JSONL über TCP; optional TLS + Zertifikat-Fingerprint-Pinning.
- TLS kündigt den Fingerprint im Discovery-TXT-Eintrag an.
- Code:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## Control-Plane-Clients heute
</div>

- CLI → Gateway WS über `callGateway` (`src/gateway/call.ts`).
- macOS-App-UI → Gateway WS (`GatewayConnection`).
- Web Control UI → Gateway WS.
- ACP → Gateway WS.
- Die Browser-Steuerung verwendet ihren eigenen HTTP-Control-Server.

<div id="nodes-today">
  ## Knoten heute
</div>

- macOS‑App im Knotenmodus stellt eine Verbindung zur Gateway‑Bridge her (`MacNodeBridgeSession`).
- iOS/Android‑Apps stellen eine Verbindung zur Gateway‑Bridge her.
- Kopplung + pro‑Knoten‑Token wird auf dem Gateway gespeichert.

<div id="current-approval-flow-exec">
  ## Aktueller Genehmigungsablauf (exec)
</div>

- Agent verwendet `system.run` über das Gateway.
- Gateway ruft den Knoten über die Bridge auf.
- Knoten‑Runtime entscheidet über die Genehmigung.
- UI‑Dialog wird von der Mac‑App angezeigt (wenn Knoten == Mac‑App).
- Knoten gibt `invoke-res` an das Gateway zurück.
- Mehrere Hops; UI ist an den Knoten‑Host gebunden.

<div id="presence-identity-today">
  ## Präsenz + Identität heute
</div>

- Gateway-Präsenz-Einträge von WS-Clients.
- Knoten-Präsenz-Einträge von der Bridge.
- macOS-App kann zwei Einträge für dieselbe Maschine anzeigen (UI + Knoten).
- Knoten-Identität im Kopplungs-Store gespeichert; UI-Identität separat.

<div id="problems-pain-points">
  # Probleme / Schmerzpunkte
</div>

- Zwei Protokoll-Stacks zu pflegen (WS + Bridge).
- Freigaben auf entfernten Knoten: Bestätigungsdialog erscheint auf dem Knoten-Host, nicht dort, wo der Benutzer ist.
- TLS-Pinning existiert nur für Bridge; WS hängt von SSH/Tailscale ab.
- Duplizierte Identitäten: derselbe Rechner erscheint als mehrere Instanzen.
- Mehrdeutige Rollen: UI-, Knoten- und CLI-Funktionen sind nicht klar getrennt.

---

<div id="proposed-new-state-clawnet">
  # Vorgeschlagener neuer Status (Clawnet)
</div>

<div id="one-protocol-two-roles">
  ## Ein Protokoll, zwei Rollen
</div>

Ein einziges WS-Protokoll mit Rolle + Scope.

- **Rolle: knoten** (Fähigkeiten-Host)
- **Rolle: Operator** (Control-Plane)
- Optionaler **Scope** für Operator:
  - `operator.read` (Status + Anzeigen)
  - `operator.write` (Agent-Ausführung, Senden)
  - `operator.admin` (Konfiguration, Kanäle, Modelle)

<div id="role-behaviors">
  ### Rollenverhalten
</div>

**Knoten**

- Kann Fähigkeiten registrieren (`caps`, `commands`, Berechtigungen).
- Kann `invoke`-Befehle empfangen (`system.run`, `camera.*`, `canvas.*`, `screen.record` usw.).
- Kann Events senden: `voice.transcript`, `agent.request`, `chat.subscribe`.
- Kann keine Config/Models/Channels/Sessions/Agent-Control-Plane-APIs aufrufen.

**Operator**

- Voller Control-Plane-API-Zugriff, über Scope gesteuert.
- Empfängt alle Genehmigungen.
- Führt keine OS-Aktionen direkt aus, sondern leitet an Knoten weiter.

<div id="key-rule">
  ### Wichtige Regel
</div>

Die Rolle ist verbindungsbezogen, nicht gerätebezogen. Ein Gerät kann beide Rollen getrennt aufbauen.
---

# Einheitliche Authentifizierung + Kopplung

<div id="pairing-flow-unified">
  ## Client-Identität
</div>

Jeder Client stellt Folgendes bereit:

- `deviceId` (stabil, abgeleitet vom Geräteschlüssel).
- `displayName` (angezeigter, menschenlesbarer Name).
- `role` + `scope` + `caps` + `commands`.

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## Kopplungsablauf (vereinheitlicht)
</div>

- Client stellt eine Verbindung ohne Authentifizierung her.
- Gateway erstellt eine **Kopplungsanfrage** für diese `deviceId`.
- Operator erhält eine Aufforderung und genehmigt oder verweigert sie.
- Gateway stellt Zugangsdaten aus, die an Folgendes gebunden sind:
  - öffentlicher Geräteschlüssel
  - Rolle(n)
  - Scope(s)
  - Fähigkeiten/Befehle
- Client speichert das Token und stellt die Verbindung erneut authentifiziert her.

<div id="silent-approval-ssh-heuristic">
  ## Gerätegebundene Authentifizierung (Replay von Bearer-Tokens vermeiden)
</div>

Bevorzugt: Geräte-Schlüsselpaare.

- Gerät erzeugt einmalig ein Schlüsselpaar.
- `deviceId = fingerprint(publicKey)`.
- Gateway sendet Nonce; Gerät signiert; Gateway verifiziert.
- Token werden für einen öffentlichen Schlüssel ausgestellt (Besitznachweis/Proof-of-Possession), nicht für eine Zeichenkette.

Alternativen:

- mTLS (Client-Zertifikate): am stärksten, höhere Betriebskomplexität.
- Kurzlebige Bearer-Tokens nur als temporäre Phase einsetzen (frühzeitig rotieren + widerrufen).

## Stille Zustimmung (SSH‑Heuristik)

Definiere sie präzise, um ein schwaches Glied zu vermeiden. Bevorzuge eine der folgenden Optionen:

- **Nur lokal**: automatisches Pairing, wenn sich der Client über Loopback oder Unix‑Socket verbindet.
- **Challenge über SSH**: Gateway gibt eine Nonce aus; der Client weist den SSH‑Zugriff nach, indem er sie abruft.
- **Zeitfenster mit physischer Anwesenheit**: Nach einer lokalen Freigabe in der UI des Gateway‑Hosts automatisches Pairing für ein kurzes Zeitfenster zulassen (z. B. 10 Minuten).

Protokolliere und erfasse alle automatischen Zustimmungen.

---

# TLS durchgängig (Dev und Prod)

<div id="apply-to-ws">
  ## Bestehendes Bridge-TLS wiederverwenden
</div>

Verwende die bestehende TLS-Runtime plus Fingerprint-Pinning:

- `src/infra/bridge/server/tls.ts`
- Fingerprint-Verifizierungslogik in `src/node-host/bridge-client.ts`

<div id="why">
  ## Gilt für WS
</div>

- WS-Server unterstützen TLS mit demselben Zertifikat/Schlüssel und Fingerabdruck.
- WS-Clients können den Fingerabdruck pinnen (optional).
- Discovery gibt TLS + Fingerabdruck für alle Endpunkte bekannt.
  - Discovery ist nur ein Locator-Hinweis; niemals ein Vertrauensanker.

## Warum

- Die Abhängigkeit von SSH/Tailscale zur Wahrung der Vertraulichkeit reduzieren.
- Mobile Remote-Verbindungen standardmäßig sicher machen.

---

# Neugestaltung des Freigabesystems (zentralisiert)

<div id="proposed">
  ## Aktuell
</div>

Die Freigabe erfolgt auf dem Knoten-Host (Knoten-Laufzeit der Mac-App). Die Aufforderung erscheint dort, wo der Knoten ausgeführt wird.

## Vorgeschlagen

Freigaben sind **Gateway‑gehostet**, die UI wird an Operator-Clients ausgeliefert.

<div id="approval-semantics-hardening">
  ### Neuer Ablauf
</div>

1) Gateway erhält den `system.run`-Intent (agent).
2) Gateway erstellt einen Genehmigungsdatensatz: `approval.requested`.
3) Operator-UIs zeigen einen Prompt an.
4) Die Genehmigungsentscheidung wird an das Gateway gesendet: `approval.resolve`.
5) Gateway ruft den Knotenbefehl auf, falls genehmigt.
6) Knoten führt aus und gibt `invoke-res` zurück.

### Genehmigungssemantik (Härtung)

- Wird an alle Operatoren gesendet; nur die aktive UI zeigt einen modalen Dialog (andere erhalten eine Toast-Benachrichtigung).
- Die erste Entscheidung zählt; das Gateway lehnt nachfolgende Entscheidungen als bereits abgeschlossen ab.
- Standard-Timeout: nach N Sekunden (z. B. 60 s) ablehnen, Grund protokollieren.
- Für die Entscheidung ist der `operator.approvals` Scope erforderlich.

## Vorteile

- Prompt erscheint dort, wo sich der Nutzer befindet (Mac/Telefon).
- Einheitliche Genehmigungen für Remote-Knoten.
- Knoten-Runtime bleibt headless; keine UI-Abhängigkeit.

---

# Beispiele zur Rollenklärung

<div id="macos-app">
  ## iPhone-App
</div>

- **Rolle als Knoten** für: Mikrofon, Kamera, Sprachchat, Standort, Push‑to‑Talk.
- Optionales **operator.read** für Status- und Chatansicht.
- Optionales **operator.write/admin** nur, wenn ausdrücklich aktiviert.

<div id="cli">
  ## macOS-App
</div>

- Standardmäßig Rolle „Operator“ (Control UI).
- Rolle „Knoten“, wenn „Mac node“ aktiviert ist (system.run, screen, camera).
- Gleiche deviceId für beide Verbindungen → zusammengeführter UI-Eintrag.

## CLI

- Rolle „Operator“ immer erforderlich.
- Scope wird durch Unterbefehl abgeleitet:
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - Genehmigungen + Kopplung → `operator.approvals` / `operator.pairing`

---

# Identität + Slugs

<div id="cute-slug-lobsterthemed">
  ## Stabile ID
</div>

Wird für die Authentifizierung benötigt; ändert sich nie.
Bevorzugt:

- Keypair-Fingerabdruck (Public-Key-Hash).

<div id="ui-grouping">
  ## Niedlicher Slug (Lobster-Thema)
</div>

Nur menschliche Bezeichnung.

- Beispiel: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- In der Gateway-Registrierung gespeichert; editierbar.
- Kollisionsbehandlung: `-2`, `-3`.

## UI-Gruppierung

Gleiche `deviceId` für alle Rollen → eine einzelne „Instanz“-Zeile:

- Badge: `operator`, `node`.
- Zeigt Funktionen + Zeitstempel „zuletzt gesehen“.

# Migrationsstrategie

<div id="phase-1-add-rolesscopes-to-ws">
  ## Phase 0: Dokumentieren + Abgleichen
</div>

- Dieses Dokument veröffentlichen.
- Alle Protokollaufrufe und Genehmigungsabläufe erfassen.

<div id="phase-2-bridge-compatibility">
  ## Phase 1: Rollen/Scopes zu WS hinzufügen
</div>

- `connect`-Parameter um `role`, `scope`, `deviceId` erweitern.
- Allowlist-basierte Zugriffskontrolle für die Knotenrolle hinzufügen.

<div id="phase-3-central-approvals">
  ## Phase 2: Bridge-Kompatibilität
</div>

- Bridge weiterlaufen lassen.
- Parallel WS-Knoten-Unterstützung hinzufügen.
- Features hinter einem Konfigurations-Flag steuern.

<div id="phase-4-tls-unification">
  ## Phase 3: Zentrale Freigaben
</div>

- Freigabeanforderungs- und Resolve-Events in WS hinzufügen.
- macOS-App-UI aktualisieren, damit sie Aufforderungen stellt und darauf reagiert.
- Die Knoten-Runtime hört auf, UI-Eingaben anzufordern.

<div id="phase-5-deprecate-bridge">
  ## Phase 4: TLS-Vereinheitlichung
</div>

- TLS-Konfiguration für WS mithilfe der TLS-Runtime der Bridge hinzufügen.
- Certificate Pinning in den Clients hinzufügen.

<div id="phase-6-devicebound-auth">
  ## Phase 5: Bridge außer Betrieb nehmen
</div>

- iOS-/Android-/Mac-Knoten auf WS migrieren.
- Bridge als Fallback beibehalten; entfernen, sobald stabil läuft.

## Phase 6: Gerätegebundene Authentifizierung

- Schlüsselbasierte Identität für alle nicht‑lokalen Verbindungen erzwingen.
- UI für Widerruf und Rotation hinzufügen.

---

<div id="streaming-large-payloads-node-media">
  # Sicherheitshinweise
</div>

- Rollen/Allowlist werden an der Gateway-Grenze durchgesetzt.
- Kein Client erhält vollen API-Zugriff ohne Operator-Scope.
- Kopplung ist für *alle* Verbindungen erforderlich.
- TLS + Pinning reduziert das MITM-Risiko für Mobilgeräte.
- Stille SSH-Genehmigung ist eine Komfortfunktion; wird trotzdem protokolliert und ist widerrufbar.
- Discovery ist niemals ein Vertrauensanker.
- Capability-Claims werden anhand der Server-Allowlists nach Plattform/Typ verifiziert.

<div id="capability-command-policy">
  # Streaming + große Payloads (Node‑Medien)
</div>

Die WS‑Steuerebene ist für kleine Nachrichten in Ordnung, aber Nodes übernehmen außerdem:

- Kamera‑Clips
- Bildschirmaufnahmen
- Audio‑Streams

Optionen:

1) WS‑binäre Frames + Chunking + Backpressure‑Regeln.
2) Separater Streaming‑Endpoint (weiterhin TLS + Auth).
3) Bridge für medienintensive Befehle länger beibehalten, zuletzt migrieren.

Lege eine Variante vor der Implementierung fest, um Drift zu vermeiden.

<div id="audit-rate-limiting">
  # Richtlinie für Fähigkeiten und Befehle
</div>

- Vom Knoten gemeldete Capabilities und Befehle werden als **Claims** behandelt.
- Das Gateway erzwingt plattformspezifische Allowlists.
- Jeder neue Befehl erfordert die Zustimmung des Betreibers oder eine explizite Allowlist-Änderung.
- Protokolliere Änderungen mit Zeitstempeln.

<div id="protocol-hygiene">
  # Audits + Rate Limiting
</div>

- Protokolliere: Kopplungsanfragen, Genehmigungen/Ablehnungen, Token-Ausstellung/-Rotation/-Widerruf.
- Begrenze Kopplungs-Spam und Genehmigungsaufforderungen durch Rate Limiting.

<div id="open-questions">
  # Protokollhygiene
</div>

- Explizite Protokollversion + Fehlercodes.
- Regeln für Wiederverbindungen + Herzschlag‑Richtlinie.
- Presence‑TTL und Last‑Seen‑Semantik.

---

<div id="summary-tldr">
  # Offene Fragen
</div>

1) Einzelnes Gerät mit beiden Rollen: Token-Modell
   - Empfohlen: separate Tokens pro Rolle (knoten vs Operator).
   - Gleiche deviceId; unterschiedliche Scopes; klarere Widerrufbarkeit.

2) Granularität des Operator-Scopes
   - read/write/admin + approvals + pairing (minimal funktionsfähig).
   - Später Scopes pro Feature in Betracht ziehen.

3) Token-Rotation + Widerrufs-UX
   - Automatisch rotieren bei Rollenwechsel.
   - UI zum Widerruf nach deviceId + Rolle.

4) Discovery
   - Aktuellen Bonjour-TXT-Eintrag erweitern, um WS‑TLS‑Fingerprint + Rollenhinweise einzuschließen.
   - Nur als Locator-Hinweise behandeln.

5) Netzwerkübergreifende Genehmigung
   - An alle Operator-Clients broadcasten; aktives UI zeigt ein Modal.
   - Erste Antwort gewinnt; Gateway erzwingt Atomarität.

---

# Zusammenfassung (TL;DR)

- Heute: WS-Control-Plane + Bridge-Knoten-Transport.
- Problem: Freigaben + Duplizierung + zwei Stacks.
- Vorschlag: ein WS-Protokoll mit expliziten Rollen + Scopes, vereinheitlichte Kopplung + TLS-Pinning, gateway‑gehostete Freigaben, stabile Geräte-IDs + niedliche Slugs.
- Ergebnis: einfachere UX, höhere Sicherheit, weniger Duplizierung, besseres Mobile-Routing.