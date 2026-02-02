---
title: Più Gateway
summary: "Eseguire più Gateway OpenClaw su un unico host (isolamento, porte e profili)"
read_when:
  - Stai eseguendo più di un Gateway sulla stessa macchina
  - Hai bisogno di configurazioni/stato/porte isolati per ogni Gateway
---

<div id="multiple-gateways-same-host">
  # Più Gateway (stesso host)
</div>

Nella maggior parte dei casi è preferibile usare un solo Gateway, perché un singolo Gateway può gestire più connessioni di messaggistica e più agenti. Se hai bisogno di un isolamento o di una ridondanza più rigorosi (ad esempio, un bot di emergenza), esegui Gateway separati con profili e porte isolati.

<div id="isolation-checklist-required">
  ## Checklist di isolamento (obbligatoria)
</div>

- `OPENCLAW_CONFIG_PATH` — file di configurazione per istanza
- `OPENCLAW_STATE_DIR` — sessioni, credenziali, cache per istanza
- `agents.defaults.workspace` — radice dello spazio di lavoro per istanza
- `gateway.port` (o `--port`) — univoca per ogni istanza
- Le porte derivate (browser/canvas) non devono sovrapporsi

Se questi parametri sono condivisi, riscontrerai race condition sulla configurazione e conflitti di porta.

<div id="recommended-profiles-profile">
  ## Consigliato: profili (`--profile`)
</div>

I profili configurano automaticamente `OPENCLAW_STATE_DIR` e `OPENCLAW_CONFIG_PATH` per il profilo e aggiungono un suffisso ai nomi dei servizi.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Servizi per singolo profilo:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## Guida al bot di soccorso
</div>

Esegui un secondo Gateway sullo stesso host con i propri:

- profilo/config
- directory di stato
- spazio di lavoro
- porta base (più le porte derivate)

In questo modo il bot di soccorso rimane isolato dal bot principale, così può eseguire il debug o applicare modifiche di configurazione se il bot primario è inattivo.

Spaziatura delle porte: lascia almeno 20 porte tra le porte base, in modo che le porte derivate per browser/canvas/CDP non vadano mai in conflitto.

<div id="how-to-install-rescue-bot">
  ### Come installare (Rescue Bot)
</div>

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports 
openclaw onboard
openclaw gateway install

# Bot di emergenza (profilo e porte isolati)
openclaw --profile rescue onboard
# Note:
# - il nome dello spazio di lavoro avrà il suffisso -rescue per impostazione predefinita
# - La porta dovrebbe essere almeno 18789 + 20 porte,
#   meglio scegliere una porta base completamente diversa, come 19789,
# - il resto della procedura di onboarding è identico a quella normale

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## Mappatura delle porte (derivata)
</div>

Porta base = `gateway.port` (oppure `OPENCLAW_GATEWAY_PORT` / `--port`).

- porta del servizio di controllo del browser = base + 2 (solo loopback)
- `canvasHost.port = base + 4`
- Le porte CDP del profilo browser vengono allocate automaticamente a partire da `browser.controlPort + 9 .. + 108`

Se modifichi uno qualsiasi di questi valori nella configurazione o nelle variabili d'ambiente, devi mantenerli univoci per ogni istanza.

<div id="browsercdp-notes-common-footgun">
  ## Note su browser/CDP (errore comune)
</div>

- **Non** impostare `browser.cdpUrl` agli stessi valori su più istanze.
- Ogni istanza necessita della propria porta di controllo del browser e del proprio intervallo CDP (derivati dalla sua porta del Gateway).
- Se ti servono porte CDP esplicite, imposta `browser.profiles.<name>.cdpPort` per ogni istanza.
- Chrome remoto: usa `browser.profiles.<name>.cdpUrl` (per profilo, per istanza).

<div id="manual-env-example">
  ## Esempio di configurazione manuale delle variabili d&#39;ambiente
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## Controlli rapidi
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
