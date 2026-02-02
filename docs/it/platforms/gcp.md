---
title: Gcp
summary: "Esegui OpenClaw Gateway 24/7 su una VM GCP Compute Engine (Docker) con stato persistente"
read_when:
  - Vuoi eseguire OpenClaw 24/7 su GCP
  - Vuoi un Gateway sempre attivo, di livello production, sulla tua VM
  - Vuoi il controllo completo su persistenza, binari e comportamento di riavvio
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # OpenClaw su GCP Compute Engine (Docker, guida VPS per l&#39;ambiente di produzione)
</div>

<div id="goal">
  ## Obiettivo
</div>

Eseguire in modo permanente un Gateway OpenClaw su una VM GCP Compute Engine usando Docker, con stato persistente, binari inclusi nell&#39;immagine e comportamento di riavvio sicuro.

Se vuoi &quot;OpenClaw 24/7 per circa 5-12 $/mese&quot;, questa è una configurazione affidabile su Google Cloud.
I costi variano in base al tipo di macchina e alla regione; scegli la VM più piccola che soddisfa il tuo carico di lavoro e passa a una VM più grande se inizi a riscontrare errori OOM (out-of-memory).

<div id="what-are-we-doing-simple-terms">
  ## Cosa faremo (in termini semplici)?
</div>

* Creare un progetto GCP e abilitare la fatturazione
* Creare una VM Compute Engine
* Installare Docker (runtime applicativo isolato)
* Avviare il Gateway OpenClaw in Docker
* Rendere persistenti `~/.openclaw` + `~/.openclaw/workspace` sull&#39;host (sopravvivono a riavvii/ricostruzioni)
* Accedere alla Control UI dal tuo laptop tramite un tunnel SSH

Il Gateway è accessibile tramite:

* Port forwarding SSH dal tuo laptop
* Esposizione diretta della porta se gestisci direttamente firewall e token

Questa guida utilizza Debian su GCP Compute Engine.
Anche Ubuntu funziona; adegua i pacchetti di conseguenza.
Per il flusso Docker generico, vedi [Docker](/it/install/docker).

***

<div id="quick-path-experienced-operators">
  ## Procedura rapida (operatori esperti)
</div>

1. Crea un progetto GCP e abilita la Compute Engine API
2. Crea una VM Compute Engine (e2-small, Debian 12, 20GB)
3. Connettiti alla VM via SSH
4. Installa Docker
5. Clona il repository OpenClaw
6. Crea directory persistenti sull&#39;host
7. Configura `.env` e `docker-compose.yml`
8. Genera i binari necessari, esegui il build e avvia

***

<div id="what-you-need">
  ## Cosa ti serve
</div>

* Account GCP (il free tier consente l’uso di e2-micro)
* gcloud CLI installata (oppure utilizza Cloud Console)
* Accesso SSH dal tuo laptop
* Minima dimestichezza con SSH + copia/incolla
* ~20-30 minuti
* Docker e Docker Compose
* Credenziali di autenticazione per i modelli
* Credenziali opzionali dei provider
  * QR di WhatsApp
  * Token del bot Telegram
  * OAuth di Gmail

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) Installa gcloud CLI (o usa la Console)
</div>

**Opzione A: gcloud CLI** (consigliata per l&#39;automazione)

Installa da https://cloud.google.com/sdk/docs/install

Inizializza e autenticati:

```bash
gcloud init
gcloud auth login
```

**Opzione B: Cloud Console**

Tutti i passaggi possono essere completati tramite la UI web all&#39;indirizzo https://console.cloud.google.com

***

<div id="2-create-a-gcp-project">
  ## 2) Crea un progetto su GCP
</div>

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Abilita la fatturazione su https://console.cloud.google.com/billing (obbligatorio per Compute Engine).

Abilita l&#39;API Compute Engine:

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1. Vai su IAM &amp; Admin &gt; Create Project
2. Assegna un nome al progetto e crealo
3. Abilita la fatturazione per il progetto
4. Vai su APIs &amp; Services &gt; Enable APIs &gt; cerca &quot;Compute Engine API&quot; &gt; Enable

***

<div id="3-create-the-vm">
  ## 3) Crea la VM
</div>

**Tipi di macchina:**

| Tipo     | Specifiche                  | Costo               | Note                                        |
| -------- | --------------------------- | ------------------- | ------------------------------------------- |
| e2-small | 2 vCPU, 2GB RAM             | ~$12/mese           | Consigliata                                 |
| e2-micro | 2 vCPU (condivise), 1GB RAM | Idonea al free tier | Potrebbe andare in OOM sotto carico elevato |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1. Vai su Compute Engine &gt; Istanze VM &gt; Crea istanza
2. Nome: `openclaw-gateway`
3. Regione: `us-central1`, zona: `us-central1-a`
4. Tipo di macchina: `e2-small`
5. Disco di avvio: Debian 12, 20 GB
6. Fai clic su **Crea**

***

<div id="4-ssh-into-the-vm">
  ## 4) Collegati alla VM via SSH
</div>

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:**

Fai clic sul pulsante &quot;SSH&quot; accanto alla tua VM nella dashboard di Compute Engine.

Nota: la propagazione delle chiavi SSH può richiedere 1-2 minuti dopo la creazione della VM. Se la connessione viene rifiutata, attendi e riprova.

***

<div id="5-install-docker-on-the-vm">
  ## 5) Installa Docker (sulla VM)
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Disconnettiti e riconnettiti perché la modifica del gruppo abbia effetto:

```bash
exit
```

Quindi accedi di nuovo via SSH:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Verifica:

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6) Clona il repository di OpenClaw
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Questa guida presume che creerai un&#39;immagine personalizzata per garantire la persistenza dei binari.

***

<div id="7-create-persistent-host-directories">
  ## 7) Crea le directory persistenti sull&#39;host
</div>

I container Docker sono effimeri.
Tutto lo stato persistente deve risiedere sull&#39;host.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) Configura le variabili d&#39;ambiente
</div>

Crea `.env` nella directory root del repository.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Genera segreti robusti:

```bash
openssl rand -hex 32
```

**Non effettuare il commit di questo file.**

***

<div id="9-docker-compose-configuration">
  ## 9) Configurazione di Docker Compose
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
      # Consigliato: mantieni il Gateway solo in loopback sulla VM; accedi tramite tunnel SSH.
      # Per esporlo pubblicamente, rimuovi il prefisso `127.0.0.1:` e configura il firewall di conseguenza.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # Optional: only if you run iOS/Android nodes against this VM and need Canvas host.
      # If you expose this publicly, read /gateway/security and firewall accordingly.
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

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) Integrare i binari richiesti nell&#39;immagine (critico)
</div>

Installare binari all&#39;interno di un container in esecuzione è una trappola.
Qualsiasi cosa installata a runtime andrà persa al riavvio.

Tutti i binari esterni richiesti dalle abilità devono essere installati in fase di build dell&#39;immagine.

Gli esempi seguenti mostrano solo tre binari comuni:

* `gog` per l&#39;accesso a Gmail
* `goplaces` per Google Places
* `wacli` per WhatsApp

Questi sono esempi, non un elenco completo.
Puoi installare quanti binari sono necessari usando lo stesso modello.

Se in seguito aggiungi nuove abilità che dipendono da binari aggiuntivi, devi:

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

# Aggiungi ulteriori binari qui sotto seguendo lo stesso schema

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

<div id="11-build-and-launch">
  ## 11) Esegui la build e avvia
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

Output previsto:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="12-verify-gateway">
  ## 12) Verifica del Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

Operazione riuscita:

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) Accesso dal tuo laptop
</div>

Crea un tunnel SSH per inoltrare la porta del Gateway:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Apri nel browser:

`http://127.0.0.1:18789/`

Incolla il tuo token del Gateway.

***

<div id="what-persists-where-source-of-truth">
  ## Cosa persiste dove (fonte di verità)
</div>

OpenClaw viene eseguito in Docker, ma Docker non è la fonte di verità.
Tutto lo stato persistente deve sopravvivere a riavvii, ricostruzioni e reboot.

| Component | Location | Persistence mechanism | Notes |
|---|---|---|---|
| Gateway config | `/home/node/.openclaw/` | Host volume mount | Include `openclaw.json`, token |
| Model auth profiles | `/home/node/.openclaw/` | Host volume mount | Token OAuth, chiavi API |
| Skill configs | `/home/node/.openclaw/skills/` | Host volume mount | Stato specifico dell&#39;abilità |
| Agent workspace | `/home/node/.openclaw/workspace/` | Host volume mount | Codice e artefatti dell&#39;agente |
| WhatsApp session | `/home/node/.openclaw/` | Host volume mount | Preserva il login via QR |
| Gmail keyring | `/home/node/.openclaw/` | Host volume + password | Richiede `GOG_KEYRING_PASSWORD` |
| External binaries | `/usr/local/bin/` | Docker image | Devono essere inclusi in fase di build |
| Node runtime | Container filesystem | Docker image | Ricostruito a ogni build dell&#39;immagine |
| OS packages | Container filesystem | Docker image | Non installare a runtime |
| Docker container | Ephemeral | Restartable | Può essere distrutto in sicurezza |

***

<div id="updates">
  ## Aggiornamenti
</div>

Per aggiornare OpenClaw nella VM:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**Connessione SSH rifiutata**

La propagazione della chiave SSH può richiedere da 1 a 2 minuti dopo la creazione della VM. Attendi e riprova.

**Problemi con OS Login**

Controlla il tuo profilo OS Login:

```bash
gcloud compute os-login describe-profile
```

Assicurati che il tuo account disponga delle autorizzazioni IAM richieste (Compute OS Login o Compute OS Admin Login).

**Out of memory (OOM)**

Se utilizzi un&#39;istanza e2-micro e riscontri errori di out-of-memory (OOM), passa a e2-small o e2-medium:

```bash
# Arresta prima la VM
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Cambia il tipo di macchina
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Avvia la VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## Account di servizio (best practice di sicurezza)
</div>

Per uso personale, il tuo account utente predefinito va bene.

Per l&#39;automazione o le pipeline CI/CD, crea un account di servizio dedicato con permessi minimi:

1. Crea un account di servizio:
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Concedi il ruolo Compute Instance Admin (o un ruolo personalizzato più ristretto):
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Evita di usare il ruolo Owner per l&#39;automazione. Segui il principio del privilegio minimo.

Consulta https://cloud.google.com/iam/docs/understanding-roles per i dettagli sui ruoli IAM.

***

<div id="next-steps">
  ## Passaggi successivi
</div>

* Configura i canali di messaggistica: [Channels](/it/channels)
* Associa i dispositivi locali come nodi: [Nodes](/it/nodes)
* Configura il Gateway: [Gateway configuration](/it/gateway/configuration)