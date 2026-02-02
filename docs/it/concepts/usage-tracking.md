---
title: Tracciamento dell'utilizzo
summary: "Esposizione dell'utilizzo e requisiti delle credenziali"
read_when:
  - Stai integrando le metriche di utilizzo/quota del provider
  - Devi spiegare il comportamento del tracciamento dell'utilizzo o i requisiti di autenticazione
---

<div id="usage-tracking">
  # Tracciamento dell'utilizzo
</div>

<div id="what-it-is">
  ## Che cos'è
</div>

- Recupera i dati di utilizzo e di quota del provider direttamente dai relativi endpoint di utilizzo.
- Nessun costo stimato; solo gli intervalli temporali riportati dal provider.

<div id="where-it-shows-up">
  ## Dove viene visualizzato
</div>

- `/status` nelle chat: scheda di stato ricca di emoji con token di sessione + costo stimato (solo chiave API). L'utilizzo del provider viene mostrato per il **provider di modello corrente** quando disponibile.
- `/usage off|tokens|full` nelle chat: footer di utilizzo per ogni risposta (OAuth mostra solo i token).
- `/usage cost` nelle chat: riepilogo locale dei costi aggregato dai log di sessione di OpenClaw.
- CLI: `openclaw status --usage` stampa un dettaglio completo per provider.
- CLI: `openclaw channels list` stampa la stessa istantanea di utilizzo insieme alla configurazione del provider (usa `--no-usage` per saltarlo).
- Barra dei menu di macOS: sezione "Usage" sotto "Context" (solo se disponibile).

<div id="providers-credentials">
  ## Provider e credenziali
</div>

- **Anthropic (Claude)**: token OAuth nei profili di autenticazione.
- **GitHub Copilot**: token OAuth nei profili di autenticazione.
- **Gemini CLI**: token OAuth nei profili di autenticazione.
- **Antigravity**: token OAuth nei profili di autenticazione.
- **OpenAI Codex**: token OAuth nei profili di autenticazione (accountId utilizzato se presente).
- **MiniMax**: API key (coding plan key; `MINIMAX_CODE_PLAN_KEY` o `MINIMAX_API_KEY`); utilizza una finestra di 5 ore del coding plan.
- **z.ai**: API key tramite env/config/auth store.

I dati di utilizzo non vengono mostrati se non esistono credenziali OAuth/API corrispondenti.