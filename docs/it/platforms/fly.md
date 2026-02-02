---
title: Fly.io
description: Distribuzione di OpenClaw su Fly.io
---

<div id="flyio-deployment">
  # Distribuzione su Fly.io
</div>

**Obiettivo:** eseguire OpenClaw Gateway su una macchina [Fly.io](https://fly.io) con archiviazione persistente, HTTPS automatico e accesso a Discord e ad altri canali.

<div id="what-you-need">
  ## Cosa ti serve
</div>

- [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) installata
- Account Fly.io (va bene il piano gratuito)
- Autenticazione dei modelli: chiave API Anthropic (o altre chiavi dei provider)
- Credenziali dei canali: token bot Discord, token Telegram, ecc.

<div id="beginner-quick-path">
  ## Percorso rapido per principianti
</div>

1. Clona il repository → personalizza `fly.toml`
2. Crea l'app e il volume → imposta i secret
3. Effettua il deploy con `fly deploy`
4. Accedi via SSH per creare la configurazione oppure usa Control UI

<div id="1-create-the-fly-app">
  ## 1) Crea l&#39;app Fly.io
</div>

```bash
# Clona il repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Crea una nuova app Fly (scegli un nome personalizzato)
fly apps create my-openclaw

# Crea un volume persistente (1GB è generalmente sufficiente)
fly volumes create openclaw_data --size 1 --region iad
```

**Suggerimento:** Scegli una regione il più possibile vicina a te. Opzioni comuni: `lhr` (Londra), `iad` (Virginia), `sjc` (San Jose).


<div id="2-configure-flytoml">
  ## 2) Configura fly.toml
</div>

Modifica `fly.toml` in modo che corrisponda al nome e ai requisiti della tua app.

**Nota sulla sicurezza:** La configurazione predefinita espone un URL pubblico. Per un deployment più sicuro senza IP pubblico, vedi [Private Deployment](#private-deployment-hardened) oppure usa `fly.private.toml`.

```toml
app = "my-openclaw"  # Nome della tua app
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Impostazioni chiave:**

| Impostazione                   | Perché                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| `--bind lan`                   | Effettua il bind su `0.0.0.0` così il proxy Fly può raggiungere il Gateway                 |
| `--allow-unconfigured`         | Avvia senza un file di configurazione (lo creerai dopo)                                    |
| `internal_port = 3000`         | Deve corrispondere a `--port 3000` (o `OPENCLAW_GATEWAY_PORT`) per gli health check di Fly |
| `memory = "2048mb"`            | 512MB è troppo poco; 2GB consigliati                                                       |
| `OPENCLAW_STATE_DIR = "/data"` | Rende persistente lo stato sul volume                                                      |


<div id="3-set-secrets">
  ## 3) Configura i segreti
</div>

```bash
# Obbligatorio: token del Gateway (per binding non-loopback)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Note:**

* I binding non-loopback (`--bind lan`) richiedono `OPENCLAW_GATEWAY_TOKEN` per motivi di sicurezza.
* Considera questi token come password.
* **Preferisci le variabili d&#39;ambiente al file di configurazione** per tutte le chiavi API e i token. Questo mantiene i segreti fuori da `openclaw.json`, dove potrebbero essere esposti o finire accidentalmente nei log.


<div id="4-deploy">
  ## 4) Esegui il deploy
</div>

```bash
fly deploy
```

Il primo deploy crea l&#39;immagine Docker (circa 2-3 minuti). I deploy successivi sono più veloci.

Dopo il deploy, verifica:

```bash
fly status
fly logs
```

Dovresti vedere:

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) Crea il file di configurazione
</div>

Connettiti alla macchina via SSH per creare un file di configurazione corretto:

```bash
fly ssh console
```

Crea la directory e il file di configurazione:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Nota:** Con `OPENCLAW_STATE_DIR=/data`, il percorso del file di configurazione è `/data/openclaw.json`.

**Nota:** Il token di Discord può provenire da:

* Variabile d&#39;ambiente: `DISCORD_BOT_TOKEN` (consigliata per dati sensibili)
* File di configurazione: `channels.discord.token`

Se usi la variabile d&#39;ambiente, non è necessario aggiungere il token alla configurazione. Il Gateway legge automaticamente `DISCORD_BOT_TOKEN`.

Riavvia per applicare le modifiche:

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6) Accedi al Gateway
</div>

<div id="control-ui">
  ### Control UI
</div>

Apri nel browser:

```bash
fly open
```

Oppure vai su `https://my-openclaw.fly.dev/`

Incolla il tuo token del Gateway (quello da `OPENCLAW_GATEWAY_TOKEN`) per autenticarti.


<div id="logs">
  ### Log
</div>

```bash
fly logs              # Log in tempo reale
fly logs --no-tail    # Log recenti
```


<div id="ssh-console">
  ### Console SSH
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="app-is-not-listening-on-expected-address">
  ### "L'app non è in ascolto sull'indirizzo previsto"
</div>

Il Gateway si sta associando a `127.0.0.1` invece di `0.0.0.0`.

**Soluzione:** aggiungi `--bind lan` al comando del processo nel `fly.toml`.

<div id="health-checks-failing-connection-refused">
  ### Controlli di integrità non riusciti / connessione rifiutata
</div>

Fly non riesce a raggiungere il Gateway sulla porta configurata.

**Soluzione:** Assicurati che `internal_port` corrisponda alla porta del Gateway (imposta `--port 3000` oppure `OPENCLAW_GATEWAY_PORT=3000`).

<div id="oom-memory-issues">
  ### Problemi di memoria / OOM
</div>

Il container continua a riavviarsi o a essere terminato. Indicatori: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration` o riavvii silenziosi.

**Soluzione:** Aumenta la memoria in `fly.toml`:

```toml
[[vm]]
  memory = "2048mb"
```

Oppure aggiorna una macchina già esistente:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Nota:** 512MB sono troppo pochi. 1GB può funzionare, ma può andare in OOM sotto carico o con log verbosi. **Si raccomandano 2GB.**


<div id="gateway-lock-issues">
  ### Problemi di lock del Gateway
</div>

Il Gateway non si avvia e restituisce errori &quot;already running&quot;.

Questo accade quando il container viene riavviato ma il file di lock del PID persiste sul volume.

**Soluzione:** Elimina il file di lock:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

Il file di lock è in `/data/gateway.*.lock` (non in una sottodirectory).


<div id="config-not-being-read">
  ### La Configurazione Non Viene Letta
</div>

Se usi `--allow-unconfigured`, il Gateway crea una configurazione minima. La tua configurazione personalizzata in `/data/openclaw.json` dovrebbe essere caricata al riavvio.

Verifica che la configurazione esista:

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### Scrittura della configurazione tramite SSH
</div>

Il comando `fly ssh console -C` non supporta la redirezione della shell. Per scrivere un file di configurazione:

```bash
# Use echo + tee (pipe from local to remote)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Nota:** `fly sftp` potrebbe non andare a buon fine se il file esiste già. Eliminalo prima:

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### Stato non persistente
</div>

Se perdi credenziali o sessioni dopo un riavvio, la directory di stato viene scritta sul filesystem del container.

**Soluzione:** Assicurati di impostare `OPENCLAW_STATE_DIR=/data` in `fly.toml` e ridistribuisci.

<div id="updates">
  ## Aggiornamenti
</div>

```bash
# Pull latest changes
git pull

# Redeploy
fly deploy

# Check health
fly status
fly logs
```


<div id="updating-machine-command">
  ### Aggiornamento del comando della macchina
</div>

Se devi modificare il comando di avvio senza eseguire un redeploy completo:

```bash
# Get machine ID
fly machines list

# Update command
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Oppure con aumento della memoria
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Nota:** Dopo `fly deploy`, il comando della macchina potrebbe essere reimpostato in base a quanto definito in `fly.toml`. Se hai apportato modifiche manuali, riapplicale dopo il deploy.


<div id="private-deployment-hardened">
  ## Private Deployment (Hardened)
</div>

Per impostazione predefinita, Fly assegna IP pubblici, rendendo il tuo Gateway accessibile all’indirizzo `https://your-app.fly.dev`. Questo è comodo, ma significa che il tuo deployment è individuabile dagli scanner su Internet (Shodan, Censys, ecc.).

Per un deployment rafforzato, senza **alcuna esposizione pubblica**, usa il template privato.

<div id="when-to-use-private-deployment">
  ### Quando usare un deployment privato
</div>

- Effettui solo chiamate/messaggi **in uscita** (nessun webhook in ingresso)
- Usi tunnel **ngrok o Tailscale** per qualsiasi callback di webhook
- Accedi al Gateway tramite **SSH, proxy o WireGuard** anziché dal browser
- Vuoi che il deployment sia **non rilevabile dagli scanner di Internet**

<div id="setup">
  ### Configurazione
</div>

Utilizza `fly.private.toml` invece del file di configurazione standard:

```bash
# Deploy con config privata
fly deploy -c fly.private.toml
```

Oppure converti un deployment esistente:

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Passa alla configurazione privata in modo che i deploy futuri non rialloghino IP pubblici
# (rimuovi [http_service] o esegui il deploy con il template privato)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

A questo punto, `fly ips list` dovrebbe mostrare solo un IP di tipo `private`:

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### Accesso a un deployment privato
</div>

Poiché non esiste alcun URL pubblico, utilizza uno di questi metodi:

**Opzione 1: proxy locale (il più semplice)**

```bash
# Inoltra la porta locale 3000 all'app
fly proxy 3000:3000 -a my-openclaw

# Quindi apri http://localhost:3000 nel browser
```

**Opzione 2: WireGuard VPN**

```bash
# Crea la configurazione WireGuard (una tantum)
fly wireguard create

# Importa nel client WireGuard, quindi accedi tramite IPv6 interno
# Esempio: http://[fdaa:x:x:x:x::x]:3000
```

**Opzione 3: solo via SSH**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### Webhook con deployment privato
</div>

Se ti servono callback webhook (Twilio, Telnyx, ecc.) senza esposizione pubblica:

1. **tunnel ngrok** - Esegui ngrok nel container o come sidecar
2. **Tailscale Funnel** - Esponi percorsi specifici tramite Tailscale
3. **Solo in uscita** - Alcuni provider (Twilio) funzionano bene per chiamate in uscita senza webhook

Esempio di configurazione di chiamata vocale con ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

Il tunnel ngrok viene eseguito all&#39;interno del container e fornisce un URL webhook pubblico senza esporre direttamente l&#39;app Fly.


<div id="security-benefits">
  ### Vantaggi di sicurezza
</div>

| Aspetto | Pubblico | Privato |
|--------|----------|---------|
| Scanner Internet | Individuabile | Nascosto |
| Attacchi diretti | Possibili | Bloccati |
| Accesso alla Control UI | Browser | Proxy/VPN |
| Recapito webhook | Diretto | Tramite tunnel |

<div id="notes">
  ## Note
</div>

- Fly.io usa **architettura x86** (non ARM)
- Il Dockerfile è compatibile con entrambe le architetture
- Per l'onboarding su WhatsApp/Telegram, usa `fly ssh console`
- I dati persistenti sono memorizzati nel volume `/data`
- Signal richiede Java + signal-cli; usa un'immagine personalizzata e assegna almeno 2 GB di memoria.

<div id="cost">
  ## Costo
</div>

Con la configurazione consigliata (`shared-cpu-2x`, 2GB RAM):

- ~10-15 $/mese in base all'utilizzo
- Il livello gratuito include un certo quantitativo di risorse

Consulta la [pagina dei prezzi di Fly.io](https://fly.io/docs/about/pricing/) per i dettagli.