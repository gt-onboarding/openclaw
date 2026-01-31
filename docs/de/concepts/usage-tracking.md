---
title: Nutzungsverfolgung
summary: "Oberflächen zur Nutzungsverfolgung und Anforderungen an Zugangsdaten"
read_when:
  - Du bindest Oberflächen für Anbieter-Nutzung oder -Kontingente ein
  - Du musst das Verhalten der Nutzungsverfolgung oder die Anforderungen an Zugangsdaten erklären
---

<div id="usage-tracking">
  # Nutzungsprotokollierung
</div>

<div id="what-it-is">
  ## Was es ist
</div>

- Ruft Nutzungs- und Quoteninformationen der anbieter direkt über deren Usage-Endpunkte ab.
- Keine geschätzten Kosten; nur die vom anbieter gemeldeten Zeiträume.

<div id="where-it-shows-up">
  ## Wo es angezeigt wird
</div>

- `/status` in Chats: Emoji‑reiche Statuskarte mit Sitzungs‑Tokens + geschätzten Kosten (nur API‑Schlüssel). Die Anbieter‑Nutzung wird für den **aktuellen Modell‑Anbieter** angezeigt, wenn verfügbar.
- `/usage off|tokens|full` in Chats: Nutzungs‑Footer pro Antwort (OAuth zeigt nur Token).
- `/usage cost` in Chats: lokale Kostenzusammenfassung, aggregiert aus OpenClaw‑Sitzungslogs.
- CLI: `openclaw status --usage` gibt eine vollständige Aufschlüsselung pro Anbieter aus.
- CLI: `openclaw channels list` gibt denselben Nutzungs‑Snapshot zusammen mit der Anbieter‑Konfiguration aus (verwende `--no-usage`, um dies zu überspringen).
- macOS‑Menüleiste: „Usage“-Abschnitt unter „Context“ (nur falls verfügbar).

<div id="providers-credentials">
  ## Anbieter + Zugangsdaten
</div>

- **Anthropic (Claude)**: OAuth-Token in Auth-Profilen.
- **GitHub Copilot**: OAuth-Token in Auth-Profilen.
- **Gemini CLI**: OAuth-Token in Auth-Profilen.
- **Antigravity**: OAuth-Token in Auth-Profilen.
- **OpenAI Codex**: OAuth-Token in Auth-Profilen (accountId wird verwendet, falls vorhanden).
- **MiniMax**: API-Schlüssel (Coding-Plan-Schlüssel; `MINIMAX_CODE_PLAN_KEY` oder `MINIMAX_API_KEY`); nutzt den 5‑Stunden‑Coding‑Plan‑Zeitraum.
- **z.ai**: API-Schlüssel über Env-/Config-/Auth-Store.

Die Nutzungsanzeige wird ausgeblendet, wenn keine entsprechenden OAuth-/API-Zugangsdaten vorhanden sind.