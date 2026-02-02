---
title: Sicherheit
summary: "CLI-Referenz für `openclaw security` (Überprüfung und Behebung typischer Sicherheitsfallen)"
read_when:
  - Du möchtest schnell eine Sicherheitsprüfung für Konfiguration/Zustand durchführen
  - Du möchtest sichere Korrekturvorschläge anwenden (chmod, Defaults verschärfen)
---

<div id="openclaw-security">
  # `openclaw security`
</div>

Sicherheitswerkzeuge (Audit und optionale Korrekturen).

Verwandt:

* Sicherheitsleitfaden: [Sicherheit](/de/gateway/security)

<div id="audit">
  ## Audit
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Das Audit warnt, wenn mehrere DM-Absender dieselbe Hauptsitzung nutzen, und empfiehlt `session.dmScope="per-channel-peer"` (oder `per-account-channel-peer` für Multi-Account-Kanäle) für gemeinsam genutzte Posteingänge.
Es warnt außerdem, wenn kleine Modelle (`<=300B`) ohne sandbox und mit aktivierten Web-/Browser-Tools verwendet werden.
