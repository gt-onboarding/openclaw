---
title: Remoto
summary: "Flusso dell'app macOS per controllare in remoto un Gateway OpenClaw tramite SSH"
read_when:
  - Configurazione o debug del controllo remoto da macOS
---

<div id="remote-openclaw-macos-remote-host">
  # OpenClaw remoto (macOS ⇄ host remoto)
</div>

Questo flusso permette all’app macOS di agire come controllo remoto completo per un Gateway OpenClaw in esecuzione su un altro host (desktop/server). È la funzionalità di **Remote over SSH** (esecuzione remota) dell’app. Tutte le funzionalità — health check, inoltro di Voice Wake e Web Chat — riutilizzano la stessa configurazione SSH remota da *Settings → General*.

<div id="modes">
  ## Modalità
</div>

* **Locale (questo Mac)**: Tutto viene eseguito sul portatile. Nessun uso di SSH.
* **Remoto tramite SSH (predefinito)**: I comandi OpenClaw vengono eseguiti sull&#39;host remoto. L&#39;app Mac apre una connessione SSH con `-o BatchMode`, più l&#39;identità/chiave scelta e un inoltro di porta locale.
* **Remoto diretto (ws/wss)**: Nessun tunnel SSH. L&#39;app Mac si connette direttamente all&#39;URL del Gateway (ad esempio tramite Tailscale Serve o un reverse proxy HTTPS pubblico).

<div id="remote-transports">
  ## Trasporti remoti
</div>

La modalità remota supporta due trasporti:

* **Tunnel SSH** (predefinito): usa `ssh -N -L ...` per inoltrare la porta del Gateway su localhost. Il Gateway vedrà l’IP del nodo come `127.0.0.1` perché il tunnel è in loopback.
* **Diretto (ws/wss)**: si connette direttamente all’URL del Gateway. Il Gateway vede il vero IP del client.

<div id="prereqs-on-the-remote-host">
  ## Prerequisiti sull&#39;host remoto
</div>

1. Installa Node + pnpm e compila/installa la CLI di OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2. Assicurati che `openclaw` sia nel PATH per le shell non interattive (crea un symlink in `/usr/local/bin` o `/opt/homebrew/bin` se necessario).
3. Configura SSH con autenticazione tramite chiave. Ti consigliamo di usare gli IP di **Tailscale** per una connettività stabile al di fuori della LAN.

<div id="macos-app-setup">
  ## Configurazione dell&#39;app macOS
</div>

1. Apri *Impostazioni → Generali*.
2. In **OpenClaw runs**, seleziona **Remote over SSH** e imposta:
   * **Transport**: **SSH tunnel** oppure **Direct (ws/wss)**.
   * **SSH target**: `user@host` (facoltativo `:port`).
     * Se il Gateway è sulla stessa LAN e si pubblicizza tramite Bonjour, selezionalo dall&#39;elenco dei dispositivi rilevati per compilare automaticamente questo campo.
   * **Gateway URL** (solo Direct): `wss://gateway.example.ts.net` (oppure `ws://...` per uso locale/LAN).
   * **Identity file** (avanzato): percorso della tua chiave.
   * **Project root** (avanzato): percorso remoto del checkout usato per i comandi.
   * **CLI path** (avanzato): percorso facoltativo verso un `openclaw` eseguibile (valorizzato automaticamente quando annunciato).
3. Premi **Test remote**. Se ha esito positivo significa che il comando remoto `openclaw status --json` viene eseguito correttamente. Gli errori di solito indicano problemi con PATH/CLI; il codice di uscita 127 significa che la CLI non viene trovata in remoto.
4. I controlli di stato (health checks) e la Web Chat ora verranno eseguiti automaticamente attraverso questo tunnel SSH.

<div id="web-chat">
  ## Web Chat
</div>

* **Tunnel SSH**: Web Chat si connette al Gateway tramite la porta di controllo WS inoltrata (WebSocket, predefinita 18789).
* **Diretto (ws/wss)**: Web Chat si connette direttamente all&#39;URL del Gateway configurato.
* Non esiste più un server HTTP separato per Web Chat.

<div id="permissions">
  ## Permessi
</div>

* L&#39;host remoto richiede le stesse autorizzazioni TCC della macchina locale (Automazione, Accessibilità, Registrazione schermo, Microfono, Riconoscimento vocale, Notifiche). Esegui la procedura di onboarding su quella macchina per concederle una sola volta.
* I nodi espongono il proprio stato delle autorizzazioni tramite `node.list` / `node.describe`, così che gli agenti sappiano quali funzionalità sono disponibili.

<div id="security-notes">
  ## Note sulla sicurezza
</div>

* Preferisci il bind in loopback sull&#39;host remoto e connettiti tramite SSH o Tailscale.
* Se esponi il Gateway su un&#39;interfaccia non loopback, richiedi l&#39;autenticazione tramite token/password.
* Consulta [Security](/it/gateway/security) e [Tailscale](/it/gateway/tailscale).

<div id="whatsapp-login-flow-remote">
  ## Flusso di login WhatsApp (remoto)
</div>

* Esegui `openclaw channels login --verbose` **sull&#39;host remoto**. Scansiona il codice QR con WhatsApp sul tuo telefono.
* Esegui nuovamente il login su quell&#39;host se l&#39;autenticazione scade. L&#39;health check segnalerà eventuali problemi di collegamento.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* **exit 127 / not found**: `openclaw` non è nel PATH per le shell non di login. Aggiungilo a `/etc/paths`, al tuo file rc della shell, oppure crea un symlink in `/usr/local/bin`/`/opt/homebrew/bin`.
* **Health probe failed**: verifica la raggiungibilità SSH, il PATH e che Baileys sia connesso (`openclaw status --json`).
* **Web Chat bloccata**: conferma che il Gateway sia in esecuzione sull’host remoto e che la porta inoltrata corrisponda alla porta WS del Gateway; la UI richiede una connessione WS funzionante.
* **IP del nodo mostra 127.0.0.1**: è normale con il tunnel SSH. Passa **Transport** a **Direct (ws/wss)** se vuoi che il Gateway veda il vero IP del client.
* **Voice Wake**: le frasi di attivazione vengono inoltrate automaticamente in modalità remota; non è necessario un componente di inoltro separato.

<div id="notification-sounds">
  ## Suoni di notifica
</div>

Scegli i suoni per ogni notifica dagli script usando `openclaw` e `node.invoke`, ad esempio:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Gateway remoto pronto" --sound Glass
```

Non esiste più un&#39;impostazione globale di “suono predefinito” nell&#39;app; chi effettua la chiamata sceglie un suono (o nessuno) per ogni richiesta.
