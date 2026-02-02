---
title: Exe Dev
summary: "Esegui OpenClaw Gateway su exe.dev (VM + proxy HTTPS) per l'accesso remoto"
read_when:
  - Vuoi un host Linux economico e sempre attivo per il Gateway
  - Vuoi l'accesso remoto alla Control UI senza gestire un VPS personale
---

<div id="exedev">
  # exe.dev
</div>

Obiettivo: OpenClaw Gateway in esecuzione su una VM exe.dev, accessibile dal tuo laptop tramite: `https://<vm-name>.exe.xyz`

Questa pagina presuppone l&#39;immagine predefinita **exeuntu** di exe.dev. Se hai scelto una distribuzione diversa, adatta i pacchetti di conseguenza.

<div id="beginner-quick-path">
  ## Percorso rapido per principianti
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Inserisci la tua chiave/token di autenticazione, se necessario
3. Fai clic su &quot;Agent&quot; accanto alla tua VM e attendi...
4. ???
5. Profit

<div id="what-you-need">
  ## Requisiti
</div>

* un account exe.dev
* accesso SSH (`ssh exe.dev`) alle macchine virtuali su [exe.dev](https://exe.dev) (opzionale)

<div id="automated-install-with-shelley">
  ## Installazione automatizzata con Shelley
</div>

Shelley, l&#39;agente di [exe.dev](https://exe.dev), può installare OpenClaw in modo immediato usando il nostro prompt. Il prompt utilizzato è il seguente:

```
Configura OpenClaw (https://docs.openclaw.ai/install) su questa VM. Utilizza i flag non-interactive e accept-risk per l'onboarding di openclaw. Aggiungi l'autenticazione o il token fornito come necessario. Configura nginx per inoltrare dalla porta predefinita 18789 alla posizione root nella configurazione del sito abilitato predefinito, assicurandoti di abilitare il supporto WebSocket. L'abbinamento si effettua tramite "openclaw devices list" e "openclaw device approve <request id>". Verifica che la dashboard mostri che lo stato di salute di OpenClaw sia OK. exe.dev gestisce l'inoltro dalla porta 8000 alla porta 80/443 e HTTPS per noi, quindi il "reachable" finale dovrebbe essere <vm-name>.exe.xyz, senza specificazione della porta.
```

<div id="manual-installation">
  ## Installazione manuale
</div>

<div id="1-create-the-vm">
  ## 1) Crea la VM
</div>

Dal tuo dispositivo:

```bash
ssh exe.dev new 
```

Quindi connettiti:

```bash
ssh <vm-name>.exe.xyz
```

Suggerimento: mantieni questa VM con stato persistente (**stateful**). OpenClaw memorizza lo stato in `~/.openclaw/` e `~/.openclaw/workspace/`.

<div id="2-install-prerequisites-on-the-vm">
  ## 2) Installa i prerequisiti (sulla VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) Installa OpenClaw
</div>

Esegui lo script di installazione di OpenClaw:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) Configura nginx per fare da proxy verso OpenClaw sulla porta 8000
</div>

Modifica `/etc/nginx/sites-enabled/default` come segue

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Impostazioni di timeout per connessioni di lunga durata
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) Accedi a OpenClaw e concedi i privilegi
</div>

Accedi a `https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL`. Approva
i dispositivi con `openclaw devices list` e `openclaw device approve`. In caso di dubbi,
usa Shelley direttamente dal browser!

<div id="remote-access">
  ## Accesso remoto
</div>

L&#39;accesso remoto è gestito tramite l&#39;autenticazione di [exe.dev](https://exe.dev). Per
impostazione predefinita, il traffico HTTP dalla porta 8000 viene inoltrato a `https://<vm-name>.exe.xyz`
con autenticazione via email.

<div id="updating">
  ## Aggiornamento
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Guida: [Aggiornamento](/it/install/updating)
