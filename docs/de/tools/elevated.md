---
title: Erhöhter Modus
summary: "Erhöhter Ausführungsmodus und /elevated-Direktiven"
read_when:
  - Anpassen der Standardkonfiguration für den erhöhten Modus, Allowlists oder das Verhalten von Slash-Befehlen
---

<div id="elevated-mode-elevated-directives">
  # Erweiterter Berechtigungsmodus (/elevated directives)
</div>

<div id="what-it-does">
  ## Was es tut
</div>

- `/elevated on` läuft auf dem Gateway-Host und behält Exec-Bestätigungen bei (genauso wie `/elevated ask`).
- `/elevated full` läuft auf dem Gateway-Host **und** bestätigt Exec automatisch (überspringt Exec-Bestätigungen).
- `/elevated ask` läuft auf dem Gateway-Host, behält aber Exec-Bestätigungen bei (genauso wie `/elevated on`).
- `on`/`ask` erzwingen **nicht** `exec.security=full`; die konfigurierte Sicherheits-/Ask-Richtlinie gilt weiterhin.
- Ändert das Verhalten nur, wenn der agent in einer sandbox läuft (ansonsten läuft Exec bereits auf dem Host).
- Direktivenformen: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Es werden nur `on|off|ask|full` akzeptiert; alles andere gibt einen Hinweis zurück und ändert den Zustand nicht.

<div id="what-it-controls-and-what-it-doesnt">
  ## Was es steuert (und was nicht)
</div>

- **Verfügbarkeits-Gates**: `tools.elevated` ist die globale Basis. `agents.list[].tools.elevated` kann elevated zusätzlich pro Agent einschränken (beides muss erlauben).
- **Sitzungsbezogener Zustand**: `/elevated on|off|ask|full` setzt den elevated-Status für den aktuellen Sitzungsschlüssel.
- **Inline-Direktive**: `/elevated on|ask|full` innerhalb einer Nachricht gilt nur für diese Nachricht.
- **Gruppen**: In Gruppenchats werden elevated-Direktiven nur beachtet, wenn der Agent erwähnt wird. Reine Befehlsnachrichten, die die Erwähnungsanforderung umgehen, werden so behandelt, als wäre eine Erwähnung erfolgt.
- **Host-Ausführung**: elevated erzwingt `exec` auf dem Gateway-Host; `full` setzt zusätzlich `security=full`.
- **Freigaben**: `full` überspringt `exec`-Freigaben; `on`/`ask` berücksichtigen sie, wenn Allowlist-/`ask`-Regeln dies erfordern.
- **Nicht-sandboxed Agenten**: kein Effekt auf den Ausführungsort; beeinflusst nur Gating, Logging und Status.
- **Tool-Policy gilt weiterhin**: Wenn `exec` durch die Tool-Policy verweigert wird, kann elevated nicht verwendet werden.
- **Getrennt von `/exec`**: `/exec` passt sitzungsbezogene Standardwerte für autorisierte Sender an und erfordert kein elevated.

<div id="resolution-order">
  ## Reihenfolge der Auflösung
</div>

1. Inline-Direktive in der Nachricht (gilt nur für diese Nachricht).
2. Sitzungs-Override (gesetzt durch Senden einer Nachricht, die nur eine Direktive enthält).
3. Globaler Standardwert (`agents.defaults.elevatedDefault` in der Konfiguration).

<div id="setting-a-session-default">
  ## Standard für eine Sitzung festlegen
</div>

- Sende eine Nachricht, die **nur** aus der Direktive besteht (Leerzeichen erlaubt), z. B. `/elevated full`.
- Es wird eine Bestätigungsantwort gesendet (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Wenn erhöhter Zugriff deaktiviert ist oder der Absender nicht auf der genehmigten Allowlist steht, antwortet die Direktive mit einer umsetzbaren Fehlermeldung und ändert den Sitzungszustand nicht.
- Sende `/elevated` (oder `/elevated:`) ohne Argument, um den aktuellen Elevated-Level zu sehen.

<div id="availability-allowlists">
  ## Verfügbarkeit + Allowlists
</div>

- Feature-Gate: `tools.elevated.enabled` (Standardwert kann per Konfiguration deaktiviert sein, selbst wenn der Code es unterstützt).
- Absender-Allowlist: `tools.elevated.allowFrom` mit anbieterspezifischen Allowlists (z. B. `discord`, `whatsapp`).
- Per-Agent-Gate: `agents.list[].tools.elevated.enabled` (optional; kann nur weiter einschränken).
- Per-Agent-Allowlist: `agents.list[].tools.elevated.allowFrom` (optional; wenn gesetzt, muss der Absender **sowohl** die globale als auch die per-Agent-Allowlist erfüllen).
- Discord-Fallback: Wenn `tools.elevated.allowFrom.discord` weggelassen wird, wird die Liste `channels.discord.dm.allowFrom` als Fallback verwendet. Setze `tools.elevated.allowFrom.discord` (auch `[]`), um dies zu überschreiben. Per-Agent-Allowlists verwenden **nicht** den Fallback.
- Alle Gates müssen erfolgreich sein; andernfalls gilt elevated als nicht verfügbar.

<div id="logging-status">
  ## Logging + Status
</div>

- Elevated-Exec-Aufrufe werden auf Info-Level (`info`) protokolliert.
- Der Sitzungsstatus enthält den erhöhten Modus (z. B. `elevated=ask`, `elevated=full`).