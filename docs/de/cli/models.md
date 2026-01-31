---
title: Modelle
summary: "CLI-Referenz für `openclaw models` (status/list/set/scan, Aliasse, Fallbacks, Auth)"
read_when:
  - Du möchtest Standard-Modelle ändern oder den Authentifizierungsstatus von Anbietern anzeigen
  - Du möchtest verfügbare Modelle und Anbieter scannen und Authentifizierungsprofile debuggen
---

<div id="openclaw-models">
  # `openclaw models`
</div>

Modellerkennung, -Scans und -Konfiguration (Standardmodell, Fallbacks, Auth-Profile).

Verwandte Themen:

* Anbieter + Modelle: [Modelle](/de/providers/models)
* Anbieter-Authentifizierung einrichten: [Erste Schritte](/de/start/getting-started)

<div id="common-commands">
  ## Häufig verwendete Befehle
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` zeigt die aufgelösten Standardwerte/Fallbacks sowie eine Auth-Übersicht.
Wenn Nutzungs-Snapshots pro anbieter verfügbar sind, enthält der OAuth/Token-Statusabschnitt
entsprechende Nutzungs-Header.
Verwende `--probe`, um Live-Auth-Prüfungen gegen jedes konfigurierte anbieterprofil auszuführen.
Diese Prüfungen sind echte Anfragen (können Tokens verbrauchen und Rate-Limits auslösen).
Verwende `--agent <id>`, um den Modell-/Auth-Status eines konfigurierten agents anzuzeigen. Wenn weggelassen,
nutzt der Befehl `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`, falls gesetzt, andernfalls den
konfigurierten Standardagent.

Hinweise:

* `models set <model-or-alias>` akzeptiert `provider/model` oder einen Alias.
* Modellreferenzen werden geparst, indem an der **ersten** `/` getrennt wird. Wenn die Modell-ID `/` enthält (OpenRouter-Stil), füge den Anbieterpräfix hinzu (Beispiel: `openrouter/moonshotai/kimi-k2`).
* Wenn du den anbieter weglässt, behandelt OpenClaw die Eingabe als Alias oder als Modell für den **Standardanbieter** (funktioniert nur, wenn kein `/` in der Modell-ID enthalten ist).

<div id="models-status">
  ### `models status`
</div>

Optionen:

* `--json`
* `--plain`
* `--check` (Exit-Code 1=abgelaufen/fehlt, 2=läuft bald ab)
* `--probe` (Live-Überprüfung der konfigurierten Auth-Profile)
* `--probe-provider <name>` (einen Anbieter testen)
* `--probe-profile <id>` (Profil-IDs wiederholen oder kommasepariert angeben)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>` (konfigurierte Agent-ID; überschreibt `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

<div id="aliases-fallbacks">
  ## Aliasse + Fallbacks
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## Authentifizierungsprofile
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` führt den Auth-Flow (OAuth/API-Schlüssel) eines Anbieter-Plugins aus. Verwende
`openclaw plugins list`, um zu sehen, welche Anbieter installiert sind.

Hinweise:

* `setup-token` fragt nach einem setup-token-Wert (generiere ihn mit `claude setup-token` auf einer beliebigen Maschine).
* `paste-token` akzeptiert einen Token-String, der anderswo oder durch Automatisierung generiert wurde.
