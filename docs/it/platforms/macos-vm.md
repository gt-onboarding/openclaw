---
title: VM macOS
summary: "Esegui OpenClaw in una VM macOS sandboxed (locale o hosted) quando ti servono isolamento o iMessage"
read_when:
  - Vuoi OpenClaw isolato dal tuo ambiente macOS principale
  - Vuoi l'integrazione con iMessage (BlueBubbles) in una sandbox
  - Vuoi un ambiente macOS ripristinabile che puoi clonare
  - Vuoi confrontare le opzioni di VM macOS locali e hosted
---

<div id="openclaw-on-macos-vms-sandboxing">
  # OpenClaw su VM macOS (in sandbox)
</div>

<div id="recommended-default-most-users">
  ## Configurazione predefinita consigliata (per la maggior parte degli utenti)
</div>

* **Piccolo VPS Linux** per un Gateway sempre attivo e a basso costo. Consulta [VPS hosting](/it/vps).
* **Hardware dedicato** (Mac mini o server Linux) se vuoi il pieno controllo e un **IP residenziale** per l&#39;automazione del browser. Molti siti bloccano gli IP dei data center, quindi la navigazione locale spesso funziona meglio.
* **Ibrido:** mantieni il Gateway su un VPS economico e collega il tuo Mac come **nodo** quando ti serve l&#39;automazione del browser/UI. Consulta [Nodes](/it/nodes) e [Gateway remote](/it/gateway/remote).

Usa una VM macOS quando ti servono in modo specifico funzionalità disponibili solo su macOS (iMessage/BlueBubbles) o quando vuoi un isolamento rigoroso rispetto al tuo Mac di tutti i giorni.

<div id="macos-vm-options">
  ## Opzioni per VM macOS
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### VM locale sul tuo Mac con Apple Silicon (Lume)
</div>

Esegui OpenClaw in una VM macOS sandboxed sul tuo Mac con Apple Silicon utilizzando [Lume](https://cua.ai/docs/lume).

Questo ti consente di avere:

* Un ambiente macOS completo e isolato (l&#39;host rimane pulito)
* Supporto iMessage tramite BlueBubbles (impossibile su Linux/Windows)
* Ripristino istantaneo clonando le VM
* Nessun costo aggiuntivo per hardware o cloud

<div id="hosted-mac-providers-cloud">
  ### Provider Mac ospitati (cloud)
</div>

Se vuoi macOS nel cloud, vanno bene anche i provider Mac ospitati:

* [MacStadium](https://www.macstadium.com/) (Mac ospitati)
* Anche altri fornitori di Mac ospitati funzionano; segui la loro documentazione per VM + SSH

Una volta che disponi dell&#39;accesso SSH a una VM macOS, continua dal passaggio 6 qui sotto.

***

<div id="quick-path-lume-experienced-users">
  ## Procedura rapida (Lume, utenti esperti)
</div>

1. Installa Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Completa Setup Assistant e abilita Accesso remoto (SSH)
4. `lume run openclaw --no-display`
5. Accedi via SSH, installa OpenClaw e configura i canali
6. Fine

***

<div id="what-you-need-lume">
  ## Requisiti (Lume)
</div>

* Mac con Apple Silicon (M1/M2/M3/M4)
* macOS Sequoia o successivo sull&#39;host
* ~60 GB di spazio libero su disco per macchina virtuale
* ~20 minuti

***

<div id="1-install-lume">
  ## 1) Installa Lume
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Se `~/.local/bin` non è nel PATH:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Verifica:

```bash
lume --version
```

Documentazione: [Installazione di Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) Crea la VM di macOS
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

Questa operazione scarica macOS e crea la VM. Si aprirà automaticamente una finestra VNC.

Nota: il download può richiedere del tempo, a seconda della velocità della tua connessione.

***

<div id="3-complete-setup-assistant">
  ## 3) Completa l&#39;Assistente di configurazione
</div>

Nella finestra VNC:

1. Seleziona lingua e regione
2. Ignora l&#39;ID Apple (oppure accedi se vuoi usare iMessage in seguito)
3. Crea un account utente (ricorda nome utente e password)
4. Ignora tutte le funzionalità opzionali

Al termine della configurazione, abilita SSH:

1. Apri Impostazioni di sistema → Generali → Condivisione
2. Abilita &quot;Accesso remoto&quot;

***

<div id="4-get-the-vms-ip-address">
  ## 4) Recupera l&#39;indirizzo IP della VM
</div>

```bash
lume get openclaw
```

Cerca l&#39;indirizzo IP (in genere `192.168.64.x`).

***

<div id="5-ssh-into-the-vm">
  ## 5) Accedi alla VM tramite SSH
</div>

```bash
ssh youruser@192.168.64.X
```

Sostituisci `youruser` con l&#39;account che hai creato e l&#39;IP con l&#39;indirizzo IP della tua VM.

***

<div id="6-install-openclaw">
  ## 6) Installa OpenClaw
</div>

All&#39;interno della VM:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Segui i passaggi di onboarding per configurare il tuo provider di modelli (Anthropic, OpenAI, ecc.).

***

<div id="7-configure-channels">
  ## 7) Configura i canali
</div>

Modifica il file di configurazione:

```bash
nano ~/.openclaw/openclaw.json
```

Aggiungi i canali:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Poi accedi a WhatsApp (scansiona il codice QR):

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) Esegui la VM in modalità headless
</div>

Arresta la VM e riavviala senza interfaccia grafica:

```bash
lume stop openclaw
lume run openclaw --no-display
```

La VM è in esecuzione in background. Il demone di OpenClaw mantiene il Gateway in esecuzione.

Per verificare lo stato:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## Bonus: integrazione con iMessage
</div>

Questa è la funzionalità di punta dell&#39;esecuzione di OpenClaw su macOS. Usa [BlueBubbles](https://bluebubbles.app) per aggiungere iMessage a OpenClaw.

All&#39;interno della VM:

1. Scarica BlueBubbles da bluebubbles.app
2. Accedi con il tuo ID Apple
3. Abilita la Web API e imposta una password
4. Configura i webhook di BlueBubbles verso il tuo Gateway (esempio: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Aggiungi alla configurazione di OpenClaw:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Riavvia il Gateway. Ora il tuo agente può inviare e ricevere iMessage.

Dettagli completi sulla configurazione: [canale BlueBubbles](/it/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## Salva una golden image
</div>

Prima di procedere con ulteriori personalizzazioni, crea uno snapshot dello stato pulito:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Reimpostabile in qualsiasi momento:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## Esecuzione 24/7
</div>

Mantieni la VM sempre in esecuzione:

* Tenendo il Mac collegato all&#39;alimentazione
* Disattivando la sospensione in Impostazioni di sistema → Risparmio energia
* Usando `caffeinate` se necessario

Per un vero funzionamento always-on, valuta un Mac mini dedicato o un piccolo VPS. Consulta [hosting VPS](/it/vps).

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

| Problema | Soluzione |
|---------|-----------|
| Impossibile eseguire l&#39;accesso SSH alla VM | Verifica che &quot;Accesso remoto&quot; sia abilitato nelle Impostazioni di sistema della VM |
| IP della VM non visibile | Attendi che la VM completi l&#39;avvio, quindi esegui di nuovo `lume get openclaw` |
| Comando Lume non trovato | Aggiungi `~/.local/bin` al tuo PATH |
| Il QR di WhatsApp non viene scansionato | Assicurati di essere connesso alla VM (non all&#39;host) quando esegui `openclaw channels login` |

***

<div id="related-docs">
  ## Documentazione correlata
</div>

* [Hosting su VPS](/it/vps)
* [Nodi](/it/nodes)
* [Gateway remoto](/it/gateway/remote)
* [Canale BlueBubbles](/it/channels/bluebubbles)
* [Guida rapida Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Riferimento CLI di Lume](https://cua.ai/docs/lume/reference/cli-reference)
* [Configurazione non presidiata della VM](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avanzato)
* [Sandboxing con Docker](/it/install/docker) (metodo di isolamento alternativo)