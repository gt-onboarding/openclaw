---
title: Processo figlio
summary: "Ciclo di vita del Gateway su macOS (launchd)"
read_when:
  - Integrazione dell'app macOS con il ciclo di vita del Gateway
---

<div id="gateway-lifecycle-on-macos">
  # Ciclo di vita del Gateway su macOS
</div>

L'app macOS **gestisce il Gateway tramite launchd** per impostazione predefinita e non
esegue il Gateway come processo figlio. Prima prova a connettersi a un Gateway
già in esecuzione sulla porta configurata; se nessuno è raggiungibile, abilita
il servizio launchd tramite la CLI `openclaw` esterna (nessun runtime integrato).
Questo garantisce un avvio automatico affidabile all'accesso e il riavvio in caso di crash.

La modalità a processo figlio (Gateway avviato direttamente dall'app) **non è utilizzata**
al momento. Se ti serve un'integrazione più stretta con la UI, esegui manualmente
il Gateway da terminale.

<div id="default-behavior-launchd">
  ## Comportamento predefinito (launchd)
</div>

* L&#39;app installa un LaunchAgent per singolo utente etichettato `bot.molt.gateway`
  (oppure `bot.molt.<profile>` quando usi `--profile`/`OPENCLAW_PROFILE`; il prefisso legacy `com.openclaw.*` è ancora supportato).
* Quando la modalità Local è abilitata, l&#39;app si assicura che il LaunchAgent sia caricato e
  avvia il Gateway se necessario.
* I log vengono scritti nel percorso di log del gateway di launchd (visibile in Debug Settings).

Comandi comuni:

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Sostituisci l&#39;etichetta con `bot.molt.&lt;profile&gt;` quando esegui un profilo denominato.


<div id="unsigned-dev-builds">
  ## Build di sviluppo non firmate
</div>

`scripts/restart-mac.sh --no-sign` è pensato per build locali veloci quando non disponi di
chiavi di firma. Per impedire a launchd di puntare a un binario del relay non firmato, lo script:

* Scrive `~/.openclaw/disable-launchagent`.

Le esecuzioni firmate di `scripts/restart-mac.sh` eliminano questo override se il marcatore è
presente. Per reimpostare manualmente:

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## Modalità attach-only
</div>

Per forzare l'app macOS a **non installare né gestire mai launchd**, avviala con
`--attach-only` (o `--no-launchd`). Questo crea/imposta `~/.openclaw/disable-launchagent`,
così l'app si limita ad agganciarsi a un Gateway già in esecuzione. Puoi attivare o disattivare lo stesso
comportamento nelle impostazioni di debug.

<div id="remote-mode">
  ## Modalità remota
</div>

La modalità remota non avvia alcun Gateway locale. L'applicazione utilizza un tunnel SSH verso
l'host remoto e si connette attraverso quel tunnel.

<div id="why-we-prefer-launchd">
  ## Perché preferiamo launchd
</div>

- Avvio automatico all'accesso.
- Semantiche integrate di riavvio/KeepAlive.
- Log e supervisione con comportamento prevedibile.

Se in futuro fosse nuovamente necessaria una vera modalità con processo figlio, andrebbe documentata come
una modalità separata, esplicita e riservata allo sviluppo.