---
title: Canali
summary: "Riferimento CLI per `openclaw channels` (account, stato, login/logout, log)"
read_when:
  - Vuoi aggiungere o rimuovere account di canali (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage)
  - Vuoi controllare lo stato dei canali o visualizzare in tempo reale i log dei canali
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

Gestisce gli account dei canali di chat e il loro stato di esecuzione sul Gateway.

Documentazione correlata:

* Guide ai canali: [Canali](/it/channels/index)
* Configurazione del Gateway: [Configurazione](/it/gateway/configuration)

<div id="common-commands">
  ## Comandi comuni
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## Aggiungere / rimuovere account
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Suggerimento: `openclaw channels add --help` mostra i flag specifici del canale (token, token dell&#39;app, percorsi di signal-cli, ecc.).

<div id="login-logout-interactive">
  ## Accesso / disconnessione (interattivo)
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* Esegui `openclaw status --deep` per una diagnostica approfondita.
* Usa `openclaw doctor` per correzioni guidate.
* `openclaw channels list` stampa `Claude: HTTP 403 ... user:profile` → lo snapshot di utilizzo richiede lo scope `user:profile`. Usa `--no-usage`, oppure fornisci una chiave di sessione claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), oppure riesegui l&#39;autenticazione tramite Claude Code CLI.

<div id="capabilities-probe">
  ## Verifica delle funzionalità
</div>

Recupera indicazioni sulle funzionalità del provider (intents/scope quando disponibili), oltre al supporto per le funzionalità statiche:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Note:

* `--channel` è facoltativo; omettilo per elencare tutti i canali (incluse le estensioni).
* `--target` accetta `channel:<id>` oppure un ID numerico di canale e si applica solo a Discord.
* I probe sono specifici per provider: intent Discord + permessi di canale facoltativi; bot Slack + scope utente; flag del bot Telegram + webhook; versione del demone Signal; token dell&#39;app MS Teams + ruoli/scope Graph (annotati quando noti). I canali senza probe mostrano `Probe: unavailable`.

<div id="resolve-names-to-ids">
  ## Risolvi i nomi in ID
</div>

Risolvi i nomi di canali/utenti in ID usando la directory del provider:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Note:

* Usa `--kind user|group|auto` per forzare il tipo di destinazione.
* In fase di risoluzione vengono preferite le corrispondenze attive quando più voci condividono lo stesso nome.
