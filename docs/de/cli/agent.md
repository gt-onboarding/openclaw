---
title: Agent
summary: "CLI-Referenz zu `openclaw agent` (einen agent-Schritt über das Gateway senden)"
read_when:
  - Wenn du einen einzelnen agent-Schritt aus Skripten ausführen und optional die Antwort zustellen möchtest
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

Führe einen agent-Turn über das Gateway aus (verwende `--local` für den eingebetteten Modus).
Verwende `--agent <id>`, um einen konfigurierten agent direkt anzusprechen.

Verwandtes:

* Tool „Agent send“: [Agent send](/de/tools/agent-send)

<div id="examples">
  ## Beispiele
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
