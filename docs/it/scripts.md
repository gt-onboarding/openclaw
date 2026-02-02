---
title: Script
summary: "Script del repository: scopo, ambito e note sulla sicurezza"
read_when:
  - Quando esegui script dal repository
  - Quando aggiungi o modifichi script in ./scripts
---

<div id="scripts">
  # Script
</div>

La directory `scripts/` contiene script di supporto per i workflow locali e le attività operative.
Usa questi script quando un&#39;attività è chiaramente associata a uno script; in caso contrario, preferisci la CLI.

<div id="conventions">
  ## Convenzioni
</div>

* Gli script sono **opzionali** salvo quando esplicitamente richiamati nella documentazione o nelle checklist di rilascio.
* Quando possibile, prediligi l&#39;uso della CLI (ad esempio: il monitoraggio dell&#39;autenticazione usa `openclaw models status --check`).
* Considera gli script specifici per l&#39;host; leggili prima di eseguirli su una nuova macchina.

<div id="git-hooks">
  ## Hook di Git
</div>

* `scripts/setup-git-hooks.js`: configurazione best-effort di `core.hooksPath` quando ci si trova all&#39;interno di un repository Git.
* `scripts/format-staged.js`: formatter pre-commit per i file `src/` e `test/` già indicizzati (staged).

<div id="auth-monitoring-scripts">
  ## Script di monitoraggio dell&#39;autenticazione
</div>

Gli script di monitoraggio dell&#39;autenticazione sono descritti qui:
[/automation/auth-monitoring](/it/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## Quando aggiungi script
</div>

* Mantieni gli script mirati e ben documentati.
* Aggiungi una breve voce nella documentazione correlata (o creane una se manca).