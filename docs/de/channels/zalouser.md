---
title: Zalouser
summary: "Unterstützung für persönliche Zalo-Konten über zca-cli (QR-Login), Funktionsumfang und Konfiguration"
read_when:
  - Einrichten von Zalo Personal für OpenClaw
  - Fehlerbehebung bei Zalo-Personal-Anmeldung oder Nachrichtenfluss
---

<div id="zalo-personal-unofficial">
  # Zalo Personal (inoffiziell)
</div>

Status: experimentell. Diese Integration automatisiert ein **persönliches Zalo-Konto** über `zca-cli`.

> **Warnung:** Dies ist eine inoffizielle Integration und kann zur Sperrung oder Deaktivierung des Kontos führen. Nutzung auf eigenes Risiko.

<div id="plugin-required">
  ## Erforderliches Plugin
</div>

Zalo Personal wird als Plugin ausgeliefert und ist nicht in der Core-Installation enthalten.

* Installation über die CLI: `openclaw plugins install @openclaw/zalouser`
* Oder aus einem Quellcode-Checkout: `openclaw plugins install ./extensions/zalouser`
* Details: [Plugins](/de/plugin)

<div id="prerequisite-zca-cli">
  ## Voraussetzung: zca-cli
</div>

Auf der Gateway-Maschine muss das `zca`-Binary im `PATH` verfügbar sein.

* Prüfen: `zca --version`
* Falls nicht vorhanden, installiere zca-cli (siehe `extensions/zalouser/README.md` oder die Upstream-Dokumentation zu zca-cli).

<div id="quick-setup-beginner">
  ## Schnellsetup (Einsteiger)
</div>

1. Installiere das Plugin (siehe oben).
2. Melde dich an (per QR auf dem Gateway-Rechner):
   * `openclaw channels login --channel zalouser`
   * Scanne den QR-Code im Terminal mit der Zalo Mobile App.
3. Aktiviere den Channel:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```

4. Starte den Gateway neu (oder schließe das Onboarding ab).
5. DM-Zugriff verwendet standardmäßig kopplung; bestätige den kopplungscode beim ersten Kontakt.

<div id="what-it-is">
  ## Was es ist
</div>

* Verwendet `zca listen`, um eingehende Nachrichten zu empfangen.
* Verwendet `zca msg ...`, um Antworten (Text/Medien/Links) zu senden.
* Ausgelegt für Anwendungsfälle mit persönlichen Konten, in denen die Zalo Bot API nicht verfügbar ist.

<div id="naming">
  ## Benennung
</div>

Die Channel-ID lautet `zalouser`, um deutlich zu machen, dass damit ein **persönliches Zalo-Benutzerkonto** (inoffiziell) automatisiert wird. `zalo` bleibt für eine mögliche zukünftige offizielle Zalo-API-Integration reserviert.

<div id="finding-ids-directory">
  ## IDs ermitteln (directory)
</div>

Verwende die directory-CLI, um Peers/Gruppen und ihre IDs zu ermitteln:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## Beschränkungen
</div>

* Ausgehende Texte werden in Abschnitte von ca. 2000 Zeichen aufgeteilt (Beschränkungen des Zalo-Clients).
* Streaming ist standardmäßig blockiert.

<div id="access-control-dms">
  ## Zugriffskontrolle (DMs)
</div>

`channels.zalouser.dmPolicy` unterstützt: `pairing | allowlist | open | disabled` (Standard: `pairing`; bei `open` werden Direktnachrichten von allen Nutzer:innen ohne Einschränkung akzeptiert).
`channels.zalouser.allowFrom` akzeptiert Nutzer-IDs oder Namen. Der Assistent löst Namen, sofern möglich, über `zca friend find` in IDs auf.

Freigabe via:

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## Gruppenzugriff (optional)
</div>

* Standard: `channels.zalouser.groupPolicy = "open"` (Gruppen erlaubt; `open` = keine Einschränkung, Nachrichten von allen Gruppen werden akzeptiert). Verwende `channels.defaults.groupPolicy`, um den Standardwert zu überschreiben, wenn er nicht gesetzt ist.
* Auf eine Allowlist beschränken mit:
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups` (Schlüssel sind Gruppen‑IDs oder ‑Namen)
* Alle Gruppen blockieren: `channels.zalouser.groupPolicy = "disabled"`.
* Der Konfigurationsassistent kann nach Gruppen‑Allowlists fragen.
* Beim Start löst OpenClaw Gruppen‑/Benutzernamen in Allowlists zu IDs auf und protokolliert die Zuordnung; nicht auflösbare Einträge bleiben wie eingegeben erhalten.

Beispiel:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## Multi-Account
</div>

Accounts entsprechen `zca`-Profilen. Beispiel:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Fehlerbehebung
</div>

**`zca` nicht gefunden:**

* Installiere zca-cli und stelle sicher, dass es im `PATH` des Gateway-Prozesses verfügbar ist.

**Anmeldung wird nicht beibehalten:**

* `openclaw channels status --probe`
* Erneut anmelden: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`