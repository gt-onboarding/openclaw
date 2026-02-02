---
title: Doctor
summary: "Riferimento CLI per `openclaw doctor` (controlli di integrità + correzioni guidate)"
read_when:
  - Hai problemi di connettività/autenticazione e vuoi una procedura guidata per risolverli
  - Hai eseguito un aggiornamento e vuoi un controllo rapido
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

Controlli di stato e correzioni rapide per il Gateway e i canali.

Correlato:

* Risoluzione dei problemi: [Troubleshooting](/it/gateway/troubleshooting)
* Verifica di sicurezza: [Security](/it/gateway/security)

<div id="examples">
  ## Esempi
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Note:

* I prompt interattivi (come le correzioni per portachiavi/OAuth) vengono eseguiti solo quando stdin è un TTY e `--non-interactive` **non** è impostato. Le esecuzioni headless (cron, Telegram, senza terminale) ignoreranno i prompt.
* `--fix` (alias di `--repair`) crea un backup in `~/.openclaw/openclaw.json.bak` ed elimina le chiavi di configurazione sconosciute, elencando ogni rimozione.

<div id="macos-launchctl-env-overrides">
  ## macOS: override delle variabili d&#39;ambiente con `launchctl`
</div>

Se in precedenza hai eseguito `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (o `...PASSWORD`), quel valore ha la precedenza rispetto al file di configurazione e può causare errori persistenti di tipo &quot;unauthorized&quot;.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
