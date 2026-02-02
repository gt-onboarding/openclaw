---
title: Docker
summary: "Configurazione e procedura di onboarding opzionali basate su Docker per OpenClaw"
read_when:
  - Vuoi un Gateway in container invece di installazioni locali
  - Stai verificando il flusso Docker
---

<div id="docker-optional">
  # Docker (opzionale)
</div>

Docker è **opzionale**. Usalo solo se vuoi un Gateway containerizzato o per verificare il flusso Docker.

<div id="is-docker-right-for-me">
  ## Docker fa al caso mio?
</div>

* **Sì**: vuoi un ambiente Gateway isolato e usa e getta oppure eseguire OpenClaw su un host senza installazioni locali.
* **No**: stai eseguendo OpenClaw sulla tua macchina e vuoi solo il ciclo di sviluppo più rapido possibile. Usa invece il normale flusso di installazione.
* **Nota sul sandboxing**: il sandboxing degli agent usa anch’esso Docker, ma **non** richiede che l’intero Gateway venga eseguito in Docker. Vedi [Sandboxing](/it/gateway/sandboxing).

Questa guida copre:

* Gateway containerizzato (OpenClaw completo in Docker)
* Sandbox per agente per sessione (Gateway sull’host + strumenti dell’agente isolati in Docker)

Dettagli sul sandboxing: [Sandboxing](/it/gateway/sandboxing)

<div id="requirements">
  ## Requisiti
</div>

* Docker Desktop (o Docker Engine) + Docker Compose v2
* Spazio su disco sufficiente per immagini e log

<div id="containerized-gateway-docker-compose">
  ## Gateway containerizzato con Docker Compose
</div>

<div id="quick-start-recommended">
  ### Avvio rapido (consigliato)
</div>

Dalla directory root del repository:

```bash
./docker-setup.sh
```

Questo script:

* crea l’immagine del Gateway
* esegue la procedura guidata di onboarding
* stampa suggerimenti facoltativi per la configurazione dei provider
* avvia il Gateway tramite Docker Compose
* genera un token del Gateway e lo scrive in `.env`

Variabili d&#39;ambiente opzionali:

* `OPENCLAW_DOCKER_APT_PACKAGES` — installa pacchetti apt aggiuntivi durante la build
* `OPENCLAW_EXTRA_MOUNTS` — aggiunge bind mount dell&#39;host aggiuntivi
* `OPENCLAW_HOME_VOLUME` — mantiene `/home/node` in un volume denominato

Al termine:

* Apri `http://127.0.0.1:18789/` nel browser.
* Incolla il token nella Control UI (Settings → token).

Scrive configurazione e spazio di lavoro sull&#39;host:

* `~/.openclaw/`
* `~/.openclaw/workspace`

Lo esegui su una VPS? Vedi [Hetzner (Docker VPS)](/it/platforms/hetzner).

<div id="manual-flow-compose">
  ### Procedura manuale (Compose)
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### Mount aggiuntivi (opzionale)
</div>

Se vuoi montare directory aggiuntive dell&#39;host nei container, imposta
`OPENCLAW_EXTRA_MOUNTS` prima di eseguire `docker-setup.sh`. Questa variabile accetta
un elenco di bind mount Docker separati da virgola e li applica sia a
`openclaw-gateway` che a `openclaw-cli` generando `docker-compose.extra.yml`.

Esempio:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Note:

* I path devono essere condivisi con Docker Desktop su macOS/Windows.
* Se modifichi `OPENCLAW_EXTRA_MOUNTS`, esegui di nuovo `docker-setup.sh` per rigenerare
  il file di compose aggiuntivo.
* `docker-compose.extra.yml` viene generato automaticamente. Non modificarlo a mano.

<div id="persist-the-entire-container-home-optional">
  ### Rendere persistente l&#39;intera home del container (opzionale)
</div>

Se vuoi che `/home/node` persista tra una ricreazione e l&#39;altra del container, imposta un
volume denominato tramite `OPENCLAW_HOME_VOLUME`. Questo crea un volume Docker e lo monta in
`/home/node`, mantenendo allo stesso tempo i normali bind mount di configurazione/spazio di lavoro. Usa qui un
volume denominato (non un percorso bind); per i bind mount, usa
`OPENCLAW_EXTRA_MOUNTS`.

Esempio:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Puoi combinarlo con mount aggiuntivi:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Note:

* Se modifichi `OPENCLAW_HOME_VOLUME`, esegui nuovamente `docker-setup.sh` per rigenerare
  il file Compose aggiuntivo.
* Il volume denominato persiste finché non viene rimosso con `docker volume rm <name>`.

<div id="install-extra-apt-packages-optional">
  ### Installa pacchetti apt aggiuntivi (opzionale)
</div>

Se hai bisogno di pacchetti di sistema all&#39;interno dell&#39;immagine (ad esempio strumenti
di compilazione o librerie multimediali), imposta `OPENCLAW_DOCKER_APT_PACKAGES` prima di
eseguire `docker-setup.sh`. Questo installerà i pacchetti durante la creazione
dell&#39;immagine, in modo che rimangano disponibili anche se il container viene eliminato.

Esempio:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Note:

* Accetta un elenco di nomi di pacchetti apt separati da spazi.
* Se modifichi `OPENCLAW_DOCKER_APT_PACKAGES`, riesegui `docker-setup.sh` per ricreare
  l&#39;immagine.

<div id="faster-rebuilds-recommended">
  ### Ricostruzioni più rapide (consigliato)
</div>

Per velocizzare le ricostruzioni, organizza il Dockerfile in modo che i layer delle dipendenze vengano messi in cache.
Questo evita di dover rieseguire `pnpm install` a meno che i file di lock non cambino:

```dockerfile
FROM node:22-bookworm

# Installa Bun (necessario per gli script di build)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Mette in cache le dipendenze a meno che i metadati del package non cambino
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### Configurazione dei canali (opzionale)
</div>

Usa il container della CLI per configurare i canali, quindi riavvia il Gateway se necessario.

WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (token bot):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (token bot):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Documentazione: [WhatsApp](/it/channels/whatsapp), [Telegram](/it/channels/telegram), [Discord](/it/channels/discord)

<div id="health-check">
  ### Controllo dello stato
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### Smoke test E2E (Docker)
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### Smoke test importazione QR (Docker)
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### Note
</div>

* Per l&#39;uso in container, il binding predefinito del Gateway è `lan`.
* Il container del Gateway è la sorgente autorevole per le sessioni (`~/.openclaw/agents/<agentId>/sessions/`).

<div id="agent-sandbox-host-gateway-docker-tools">
  ## Sandbox dell&#39;agente (gateway host + strumenti Docker)
</div>

Approfondimento: [Sandboxing](/it/gateway/sandboxing)

<div id="what-it-does">
  ### Cosa fa
</div>

Quando `agents.defaults.sandbox` è abilitato, le **sessioni non principali** eseguono i tool all&#39;interno di un container
Docker. Il Gateway rimane sull&#39;host, ma l&#39;esecuzione dei tool è isolata:

* scope: `"agent"` per impostazione predefinita (un container + spazio di lavoro per agente)
* scope: `"session"` per l&#39;isolamento per sessione
* cartella dello spazio di lavoro per scope montata in `/workspace`
* accesso opzionale allo spazio di lavoro dell&#39;agente (`agents.defaults.sandbox.workspaceAccess`)
* policy allow/deny per i tool (deny ha la precedenza)
* i contenuti multimediali in ingresso vengono copiati nello spazio di lavoro della sandbox attiva (`media/inbound/*`) in modo che i tool possano leggerli (con `workspaceAccess: "rw"`, finiscono nello spazio di lavoro dell&#39;agente)

Avvertenza: `scope: "shared"` disabilita l&#39;isolamento tra sessioni. Tutte le sessioni condividono
un container e uno spazio di lavoro.

<div id="per-agent-sandbox-profiles-multi-agent">
  ### Profili di sandbox per agente (multi-agente)
</div>

Se usi l’instradamento multi-agente, ogni agente può sovrascrivere le impostazioni di sandbox + strumenti:
`agents.list[].sandbox` e `agents.list[].tools` (oltre a `agents.list[].tools.sandbox.tools`). Questo ti permette di eseguire
livelli di accesso misti in un unico Gateway:

* Accesso completo (agente personale)
* Strumenti in sola lettura + spazio di lavoro in sola lettura (agente famiglia/lavoro)
* Nessun strumento per filesystem o shell (agente pubblico)

Vedi [Multi-Agent Sandbox &amp; Tools](/it/multi-agent-sandbox-tools) per esempi,
ordine di precedenza e risoluzione dei problemi.

<div id="default-behavior">
  ### Comportamento predefinito
</div>

* Immagine: `openclaw-sandbox:bookworm-slim`
* Un container per agente
* Accesso allo spazio di lavoro dell&#39;agente: `workspaceAccess: "none"` (predefinito) usa `~/.openclaw/sandboxes`
  * `"ro"` mantiene lo spazio di lavoro della sandbox in `/workspace` e monta lo spazio di lavoro dell&#39;agente in sola lettura in `/agent` (disabilita `write`/`edit`/`apply_patch`)
  * `"rw"` monta lo spazio di lavoro dell&#39;agente in lettura/scrittura in `/workspace`
* Auto-prune (pulizia automatica): inattivo &gt; 24h OPPURE età &gt; 7g
* Rete: `none` per impostazione predefinita (devi abilitarla esplicitamente se ti serve traffico in uscita)
* Consentiti per impostazione predefinita: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* Negati per impostazione predefinita: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

<div id="enable-sandboxing">
  ### Abilita il sandboxing
</div>

Se prevedi di installare pacchetti in `setupCommand`, nota quanto segue:

* Il valore predefinito di `docker.network` è `"none"` (nessun traffico in uscita).
* `readOnlyRoot: true` blocca l&#39;installazione dei pacchetti.
* `user` deve essere root per `apt-get` (ometti `user` oppure imposta `user: "0:0"`).
  OpenClaw ricrea automaticamente i container quando `setupCommand` (o la configurazione Docker) cambia,
  a meno che il container non sia stato **utilizzato di recente** (entro ~5 minuti). I container “caldi”
  emettono un avviso nei log con il comando esatto `openclaw sandbox recreate ...`.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent è predefinito)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

I parametri di hardening si trovano sotto `agents.defaults.sandbox.docker`:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

Multi-agente: puoi sovrascrivere `agents.defaults.sandbox.{docker,browser,prune}.*` per ogni agente tramite `agents.list[].sandbox.{docker,browser,prune}.*`
(ignorato quando `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` è `"shared"`).

<div id="build-the-default-sandbox-image">
  ### Crea l&#39;immagine sandbox predefinita
</div>

```bash
scripts/sandbox-setup.sh
```

Questo comando crea l’immagine Docker `openclaw-sandbox:bookworm-slim` a partire da `Dockerfile.sandbox`.

<div id="sandbox-common-image-optional">
  ### Immagine sandbox comune (opzionale)
</div>

Se ti serve un&#39;immagine sandbox con i più comuni strumenti di build (Node, Go, Rust, ecc.), costruisci l&#39;immagine comune:

```bash
scripts/sandbox-common-setup.sh
```

Questo crea `openclaw-sandbox-common:bookworm-slim`. Per utilizzarla:

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### Immagine sandbox per il browser
</div>

Per eseguire lo strumento browser all&#39;interno della sandbox, effettua il build dell&#39;immagine del browser:

```bash
scripts/sandbox-browser-setup.sh
```

Questo crea `openclaw-sandbox-browser:bookworm-slim` usando
`Dockerfile.sandbox-browser`. Il container esegue Chromium con CDP abilitato e
un viewer noVNC opzionale (headful tramite Xvfb).

Note:

* La modalità headful (Xvfb) riduce il blocco dei bot rispetto alla modalità headless.
* La modalità headless può comunque essere usata impostando `agents.defaults.sandbox.browser.headless=true`.
* Non è necessario un ambiente desktop completo (GNOME); Xvfb fornisce il display.

Usa questa configurazione:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

Immagine Docker personalizzata per il browser:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

Quando è abilitato, l&#39;agente riceve:

* un URL di controllo del browser della sandbox (per lo strumento `browser`)
* un URL noVNC (se è abilitato e headless=false)

Ricorda: se usi una lista di autorizzati per gli strumenti, aggiungi `browser` (e rimuovilo da
deny) oppure lo strumento rimane bloccato.
Le regole di pruning (`agents.defaults.sandbox.prune`) si applicano anche ai container del browser.

<div id="custom-sandbox-image">
  ### Immagine sandbox personalizzata
</div>

Crea la tua immagine e impostane l&#39;uso nella configurazione:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### Policy degli strumenti (allow/deny)
</div>

* `deny` ha precedenza su `allow`.
* Se `allow` è vuoto: tutti gli strumenti (tranne quelli in `deny`) sono disponibili.
* Se `allow` non è vuoto: sono disponibili solo gli strumenti in `allow` (esclusi quelli in `deny`).

<div id="pruning-strategy">
  ### Strategia di pruning
</div>

Due parametri di configurazione:

* `prune.idleHours`: elimina i container non usati da X ore (0 = disattivato)
* `prune.maxAgeDays`: elimina i container più vecchi di X giorni (0 = disattivato)

Esempio:

* Mantieni attive le sessioni ma limita la durata:
  `idleHours: 24`, `maxAgeDays: 7`
* Non eseguire mai il pruning:
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### Note di sicurezza
</div>

* L&#39;hard wall si applica solo agli **strumenti** (exec/read/write/edit/apply&#95;patch).
* Gli strumenti che girano solo sull&#39;host, come browser/camera/canvas, sono bloccati per impostazione predefinita.
* Consentire `browser` nella sandbox **compromette l&#39;isolamento** (il browser gira sull&#39;host).

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

* Immagine mancante: esegui il build con [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) oppure imposta `agents.defaults.sandbox.docker.image`.
* Container non in esecuzione: verrà creato automaticamente, per ogni sessione, su richiesta.
* Errori di permessi nella sandbox: imposta `docker.user` su un UID:GID che corrisponda al proprietario
  dello spazio di lavoro montato (oppure esegui `chown` sulla cartella dello spazio di lavoro).
* Strumenti personalizzati non trovati: OpenClaw esegue i comandi con `sh -lc` (login shell), che
  carica `/etc/profile` e può reimpostare PATH. Imposta `docker.env.PATH` per anteporre i percorsi
  dei tuoi strumenti personalizzati (ad esempio, `/custom/bin:/usr/local/share/npm-global/bin`), oppure aggiungi
  uno script in `/etc/profile.d/` nel tuo Dockerfile.