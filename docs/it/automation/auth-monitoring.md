---
title: Monitoraggio dell'autenticazione
summary: "Monitora la scadenza OAuth per i provider di modelli"
read_when:
  - Configurazione del monitoraggio o degli avvisi sulla scadenza dell'autenticazione
  - Automazione dei controlli di refresh OAuth per Claude Code / Codex
---

<div id="auth-monitoring">
  # Monitoraggio dell'autenticazione
</div>

OpenClaw espone lo stato di salute delle scadenze OAuth tramite `openclaw models status`. Usalo per
l'automazione e l'alerting; gli script sono solo un'aggiunta opzionale per i workflow su telefono.

<div id="preferred-cli-check-portable">
  ## Consigliato: controllo via CLI (portabile)
</div>

```bash
openclaw models status --check
```

Codici di ritorno:

* `0`: OK
* `1`: credenziali scadute o mancanti
* `2`: in scadenza a breve (entro 24 ore)

Questo funziona con cron/systemd e non richiede script aggiuntivi.


<div id="optional-scripts-ops-phone-workflows">
  ## Script opzionali (workflow operativi / telefono)
</div>

Questi si trovano sotto `scripts/` e sono **opzionali**. Presuppongono l’accesso SSH
all’host del gateway e sono ottimizzati per systemd + Termux.

- `scripts/claude-auth-status.sh` ora usa `openclaw models status --json` come
  fonte di verità (con fallback alla lettura diretta dei file se la CLI non è disponibile),
  quindi mantieni `openclaw` nel `PATH` per i timer.
- `scripts/auth-monitor.sh`: destinazione per timer cron/systemd; invia avvisi (ntfy o telefono).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: timer utente systemd.
- `scripts/claude-auth-status.sh`: controllo autenticazione Claude Code + OpenClaw (full/json/simple).
- `scripts/mobile-reauth.sh`: flusso guidato di ri‑autenticazione via SSH.
- `scripts/termux-quick-auth.sh`: stato widget con un tocco + apertura URL di autenticazione.
- `scripts/termux-auth-widget.sh`: flusso guidato completo via widget.
- `scripts/termux-sync-widget.sh`: sincronizza le credenziali Claude Code → OpenClaw.

Se non ti servono automazioni da telefono o timer systemd, puoi ignorare questi script.