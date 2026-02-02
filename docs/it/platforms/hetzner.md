---
title: Hetzner
summary: "Esegui OpenClaw Gateway 24/7 su un VPS Hetzner economico (Docker) con stato persistente e binari integrati"
read_when:
  - Vuoi che OpenClaw sia in esecuzione 24/7 su un VPS nel cloud (non sul tuo laptop)
  - Vuoi un Gateway sempre attivo, pronto per l'ambiente di produzione, sul tuo VPS
  - Vuoi il pieno controllo su persistenza, binari e comportamento di riavvio
  - Stai eseguendo OpenClaw in Docker su Hetzner o su un provider simile
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # OpenClaw su Hetzner (Docker, guida alla VPS di produzione)
</div>

<div id="goal">
  ## Obiettivo
</div>

Eseguire un Gateway OpenClaw persistente su un VPS Hetzner usando Docker, con stato persistente, binari incorporati nell&#39;immagine e comportamento di riavvio sicuro.

Se vuoi “OpenClaw 24/7 per circa 5 $”, questa è la configurazione più semplice e affidabile.
I prezzi di Hetzner variano; scegli il VPS Debian/Ubuntu più piccolo e scala verso l’alto se riscontri errori OOM (out-of-memory).

<div id="what-are-we-doing-simple-terms">
  ## Cosa stiamo facendo (in termini semplici)?
</div>

* Noleggiare un piccolo server Linux (VPS Hetzner)
* Installare Docker (runtime isolato per le applicazioni)
* Avviare il Gateway OpenClaw in Docker
* Rendere persistenti `~/.openclaw` + `~/.openclaw/workspace` sull&#39;host (in modo che sopravvivano a riavvii/ricostruzioni)
* Accedere al Control UI dal tuo laptop tramite un tunnel SSH

Il Gateway può essere raggiunto tramite:

* Inoltro di porte SSH dal tuo laptop
* Esposizione diretta della porta se gestisci personalmente firewall e token

Questa guida presuppone Ubuntu o Debian su Hetzner.\
Se utilizzi un altro VPS Linux, adatta i pacchetti di conseguenza.
Per il flusso Docker generico, vedi [Docker](/it/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Percorso rapido (operatori esperti)
</div>

1. Esegui il provisioning di una VPS Hetzner
2. Installa Docker
3. Clona il repository OpenClaw
4. Crea le directory persistenti sull&#39;host
5. Configura `.env` e `docker-compose.yml`
6. Integra i binari necessari nell&#39;immagine
7. `docker compose up -d`
8. Verifica la persistenza e l&#39;accesso al Gateway

***

<div id="what-you-need">
  ## Cosa ti serve
</div>

* VPS Hetzner con accesso root
* Accesso SSH dal tuo laptop
* Familiarità di base con SSH + copia/incolla
* ~20 minuti
* Docker e Docker Compose
* Credenziali per l&#39;autenticazione dei modelli
* Credenziali opzionali dei provider
  * Codice QR di WhatsApp
  * Token del bot Telegram
  * OAuth per Gmail

***

<div id="1-provision-the-vps">
  ## 1) Provisiona la VPS
</div>

Crea una VPS Ubuntu o Debian su Hetzner.

Accedi come root:

```bash
ssh root@YOUR_VPS_IP
```

Questa guida presuppone che il VPS sia stateful.
Non considerarlo infrastruttura usa e getta.

***

<div id="2-install-docker-on-the-vps">
  ## 2) Installa Docker (sul VPS)
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Verifica:

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3) Clona il repository di OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Questa guida presuppone che tu crei un&#39;immagine personalizzata per garantire la persistenza dei binari.

***

<div id="4-create-persistent-host-directories">
  ## 4) Crea le directory persistenti sull&#39;host
</div>

I container Docker sono effimeri.
Tutto lo stato persistente deve risiedere sull&#39;host.

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# Imposta la proprietà sull'utente del container (uid 1000):
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5) Configura le variabili d&#39;ambiente
</div>

Crea il file `.env` nella directory principale del repository.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Genera segreti robusti:

```bash
openssl rand -hex 32
```

**Non fare il commit di questo file.**

***

<div id="6-docker-compose-configuration">
  ## 6) Configurazione di Docker Compose
</div>

Crea o aggiorna `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Facoltativo: solo se si eseguono nodi iOS/Android su questo VPS e si necessita dell'host Canvas.
      # Se esposto pubblicamente, leggere /gateway/security e configurare il firewall di conseguenza.
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) Includi i binari richiesti nell&#39;immagine (critico)
</div>

Installare binari all&#39;interno di un container in esecuzione è una trappola.
Qualsiasi cosa installata a runtime andrà persa al riavvio.

Tutti i binari esterni richiesti dalle abilità devono essere installati in fase di build dell&#39;immagine.

Gli esempi seguenti mostrano solo tre binari comuni:

* `gog` per l&#39;accesso a Gmail
* `goplaces` per Google Places
* `wacli` per WhatsApp

Questi sono esempi, non un elenco completo.
Puoi installare tutti i binari necessari seguendo lo stesso schema.

Se in seguito aggiungi nuove abilità che dipendono da altri binari, devi:

1. Aggiornare il Dockerfile
2. Ricostruire l&#39;immagine
3. Riavviare i container

**Esempio di Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Aggiungi altri binari di seguito usando lo stesso schema

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="8-build-and-launch">
  ## 8) Build e avvio
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verifica i file binari:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Risultato atteso:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="9-verify-gateway">
  ## 9) Verifica del Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Esito positivo:

```
[gateway] listening on ws://0.0.0.0:18789
```

Dal tuo portatile:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Apri:

`http://127.0.0.1:18789/`

Incolla il token del tuo Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Cosa persiste e dove (source of truth)
</div>

OpenClaw viene eseguito in Docker, ma Docker non è la source of truth.
Tutto lo stato persistente deve sopravvivere a riavvii, rebuild e reboot.

| Componente | Percorso | Meccanismo di persistenza | Note |
|---|---|---|---|
| Configurazione del Gateway | `/home/node/.openclaw/` | Volume dell&#39;host montato | Include `openclaw.json`, token |
| Profili di autenticazione dei modelli | `/home/node/.openclaw/` | Volume dell&#39;host montato | Token OAuth, chiavi API |
| Configurazioni delle abilità | `/home/node/.openclaw/skills/` | Volume dell&#39;host montato | Stato a livello di abilità |
| Spazio di lavoro dell&#39;agente | `/home/node/.openclaw/workspace/` | Volume dell&#39;host montato | Codice e artefatti dell&#39;agente |
| Sessione WhatsApp | `/home/node/.openclaw/` | Volume dell&#39;host montato | Preserva il login tramite QR |
| Keyring di Gmail | `/home/node/.openclaw/` | Volume dell&#39;host + password | Richiede `GOG_KEYRING_PASSWORD` |
| Binari esterni | `/usr/local/bin/` | Immagine Docker | Deve essere incluso nell&#39;immagine in fase di build |
| Runtime Node | Filesystem del container | Immagine Docker | Ricostruito a ogni build dell&#39;immagine |
| Pacchetti OS | Filesystem del container | Immagine Docker | Non installare a runtime |
| Container Docker | Effimero | Riavviabile | Può essere eliminato senza rischi |