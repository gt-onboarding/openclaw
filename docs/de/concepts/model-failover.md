---
title: Modell-Failover
summary: "Wie OpenClaw Auth-Profile wechselt und auf andere Modelle zurückfällt"
read_when:
  - Diagnostizieren von Rotationen und Cooldowns von Auth-Profilen oder von Fallback-Verhalten zwischen Modellen
  - Aktualisieren von Failover-Regeln für Auth-Profile oder Modelle
---

<div id="model-failover">
  # Modell-Failover
</div>

OpenClaw behandelt Ausfälle in zwei Stufen:

1. **Auth-Profile-Rotation** innerhalb des aktuellen Anbieters.
2. **Modell-Fallback** auf das nächste Modell in `agents.defaults.model.fallbacks`.

Diese Dokumentation erläutert die Laufzeitregeln und die ihnen zugrunde liegenden Daten.

<div id="auth-storage-keys-oauth">
  ## Auth-Speicherung (Keys + OAuth)
</div>

OpenClaw verwendet **Auth-Profile** sowohl für API-Keys als auch für OAuth-Tokens.

* Secrets liegen in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (legacy: `~/.openclaw/agent/auth-profiles.json`).
* Die Konfigurationseinträge `auth.profiles` / `auth.order` sind **nur für Metadaten + Routing** (keine Secrets).
* Legacy-OAuth-Datei nur für Import: `~/.openclaw/credentials/oauth.json` (wird bei der ersten Verwendung in `auth-profiles.json` importiert).

Mehr Details: [/concepts/oauth](/de/concepts/oauth)

Credential-Typen:

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` für einige Anbieter)

<div id="profile-ids">
  ## Profil-IDs
</div>

OAuth-Anmeldungen erzeugen separate Profile, damit mehrere Konten nebeneinander existieren können.

* Standard: `provider:default`, wenn keine E-Mail-Adresse verfügbar ist.
* OAuth mit E-Mail: `provider:<email>` (zum Beispiel `google-antigravity:user@gmail.com`).

Profile befinden sich in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` unter `profiles`.

<div id="rotation-order">
  ## Rotationsreihenfolge
</div>

Wenn ein Anbieter mehrere Profile hat, wählt OpenClaw die Reihenfolge wie folgt:

1. **Explizite Konfiguration**: `auth.order[provider]` (falls gesetzt).
2. **Konfigurierte Profile**: `auth.profiles`, gefiltert nach Anbieter.
3. **Gespeicherte Profile**: Einträge in `auth-profiles.json` für den Anbieter.

Wenn keine explizite Reihenfolge konfiguriert ist, verwendet OpenClaw eine Round-Robin-Reihenfolge:

* **Primärer Schlüssel:** Profiltyp (**OAuth vor API-Schlüsseln**).
* **Sekundärer Schlüssel:** `usageStats.lastUsed` (innerhalb jedes Typs zuerst die ältesten).
* **Profile im Cooldown bzw. deaktivierte Profile** werden ans Ende verschoben und nach dem frühesten Ablaufzeitpunkt sortiert.

<div id="session-stickiness-cache-friendly">
  ### Sitzungs-Stickiness (cachefreundlich)
</div>

OpenClaw **pinnt das gewählte Auth‑Profil pro Sitzung**, um Anbieter‑Caches warm zu halten.
Es **rotiert nicht** bei jeder Anfrage. Das gepinnte Profil wird wiederverwendet, bis:

* die Sitzung zurückgesetzt wird (`/new` / `/reset`)
* eine Kompaktierung abgeschlossen ist (der Kompaktierungszähler erhöht wird)
* das Profil im Cooldown/deaktiviert ist

Die manuelle Auswahl über `/model …@<profileId>` setzt einen **Benutzer‑Override** für diese Sitzung
und wird nicht automatisch rotiert, bis eine neue Sitzung startet.

Automatisch gepinnte Profile (vom Sitzungsrouter ausgewählt) werden als **Präferenz** behandelt:
Sie werden zuerst verwendet, aber OpenClaw kann bei Rate‑Limits/Timeouts zu einem anderen Profil rotieren.
Benutzer‑gepinnte Profile bleiben auf dieses Profil fixiert; wenn es fehlschlägt und Modell‑Fallbacks
konfiguriert sind, wechselt OpenClaw zum nächsten Modell, statt das Profil zu wechseln.

<div id="why-oauth-can-look-lost">
  ### Warum OAuth „verloren wirken“ kann
</div>

Wenn du sowohl ein OAuth‑Profil als auch ein API‑Key‑Profil für denselben Anbieter hast, kann Round‑robin zwischen ihnen über Nachrichten hinweg wechseln, sofern nichts fixiert ist. Um ein einzelnes Profil zu erzwingen:

* Fixiere es mit `auth.order[provider] = ["provider:profileId"]`, oder
* Verwende einen Override pro Sitzung über `/model …` mit einem Profil‑Override (sofern dies von deiner UI/Chatoberfläche unterstützt wird).

<div id="cooldowns">
  ## Cooldowns
</div>

Wenn ein Profil aufgrund von Auth‑/Rate‑Limit‑Fehlern (oder eines Timeouts, das wie Rate‑Limiting aussieht) fehlschlägt, markiert OpenClaw es als im Cooldown und wechselt zum nächsten Profil.
Format‑/Invalid‑Request‑Fehler (zum Beispiel Validierungsfehler bei Cloud‑Code‑Assist‑Tool‑Call‑IDs) werden als failover‑würdig behandelt und verwenden dieselben Cooldowns.

Cooldowns verwenden exponentielles Backoff:

* 1 Minute
* 5 Minuten
* 25 Minuten
* 1 Stunde (Obergrenze)

Der Zustand wird in `auth-profiles.json` unter `usageStats` gespeichert:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## Deaktivierung bei Abrechnungsfehlern
</div>

Abrechnungs‑/Guthabenfehler (zum Beispiel „insufficient credits“ / „credit balance too low“) werden als Failover‑auslösend behandelt, sind aber in der Regel nicht vorübergehend. Anstelle eines kurzen Cooldowns markiert OpenClaw das Profil als **disabled** (mit einem längeren Backoff) und wechselt zum nächsten Profil/Anbieter.

Der Zustand wird in `auth-profiles.json` gespeichert:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Standardwerte:

* Das Backoff für Abrechnungsversuche beginnt bei **5 Stunden**, verdoppelt sich bei jedem Abrechnungsfehler und ist bei **24 Stunden** begrenzt.
* Die Backoff-Zähler werden zurückgesetzt, wenn das Profil **24 Stunden lang** (konfigurierbar) keinen Fehler hatte.

<div id="model-fallback">
  ## Modell-Fallback
</div>

Wenn alle Profile für einen Anbieter fehlschlagen, wechselt OpenClaw zum nächsten Modell in
`agents.defaults.model.fallbacks`. Das gilt für Authentifizierungsfehler, Ratenbegrenzungen (Rate Limits) und
Timeouts, bei denen die Profilrotation ausgeschöpft wurde (andere Fehler führen nicht zum nächsten Fallback).

Wenn eine Ausführung mit einem Modell-Override (Hooks oder CLI) startet, enden Fallbacks trotzdem bei
`agents.defaults.model.primary`, nachdem alle konfigurierten Fallbacks ausprobiert wurden.

<div id="related-config">
  ## Zugehörige Konfiguration
</div>

Siehe [Gateway-Konfiguration](/de/gateway/configuration) für:

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* Routing von `agents.defaults.imageModel`

Siehe [Modelle](/de/concepts/models) für einen umfassenderen Überblick über Modellauswahl und Fallback-Verhalten.