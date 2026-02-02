---
title: Abilità
summary: "UI delle impostazioni delle abilità su macOS e stato gestito dal Gateway"
read_when:
  - Aggiornare la UI delle impostazioni delle abilità su macOS
  - Modificare i controlli di accesso o il comportamento di installazione delle abilità
---

<div id="skills-macos">
  # Abilità (macOS)
</div>

L'app macOS rende disponibili le abilità di OpenClaw tramite il Gateway; non elabora le abilità localmente.

<div id="data-source">
  ## Origine dei dati
</div>

- `skills.status` (Gateway) restituisce l'elenco completo delle abilità, insieme all'idoneità e ai requisiti mancanti
  (incluse le voci della lista di autorizzati che bloccano le abilità fornite in bundle).
- I requisiti sono derivati da `metadata.openclaw.requires` in ogni `SKILL.md`.

<div id="install-actions">
  ## Azioni di installazione
</div>

- `metadata.openclaw.install` definisce le opzioni di installazione (brew/node/go/uv).
- L'app chiama `skills.install` per eseguire i programmi di installazione sull'host del Gateway.
- Il Gateway mostra un solo metodo di installazione preferito quando ne vengono forniti più di uno
  (brew quando disponibile, altrimenti il node manager da `skills.install`, con npm come predefinito).

<div id="envapi-keys">
  ## Chiavi Env/API
</div>

- L'applicazione memorizza le chiavi in `~/.openclaw/openclaw.json` all'interno di `skills.entries.<skillKey>`.
- `skills.update` aggiorna `enabled`, `apiKey` e `env`.

<div id="remote-mode">
  ## Modalità remota
</div>

- L'installazione e gli aggiornamenti della configurazione vengono eseguiti sull'host del Gateway (non sul Mac locale).