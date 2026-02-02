---
title: Estensione Chrome
summary: "Estensione Chrome: lascia che OpenClaw controlli una scheda di Chrome esistente"
read_when:
  - Vuoi che l'agente controlli una scheda di Chrome esistente (pulsante sulla barra degli strumenti)
  - Ti serve un Gateway remoto + automazione del browser locale tramite Tailscale
  - Vuoi capire le implicazioni di sicurezza della presa di controllo del browser
---

<div id="chrome-extension-browser-relay">
  # Estensione Chrome (browser relay)
</div>

L&#39;estensione Chrome di OpenClaw consente all&#39;agente di controllare le **schede di Chrome già aperte** (la normale finestra di Chrome) invece di avviare un profilo di Chrome separato gestito da openclaw.

Il collegamento/scollegamento avviene tramite **un singolo pulsante nella barra degli strumenti di Chrome**.

<div id="what-it-is-concept">
  ## Che cos&#39;è (concetto)
</div>

È composto da tre parti:

* **Servizio di controllo del browser** (Gateway o nodo): l&#39;API che l&#39;agente/strumento invoca (tramite il Gateway)
* **Server di relay locale** (loopback CDP): fa da ponte tra il server di controllo e l&#39;estensione (`http://127.0.0.1:18792` per impostazione predefinita)
* **Estensione Chrome MV3**: si collega alla scheda attiva usando `chrome.debugger` e instrada i messaggi CDP verso il relay

OpenClaw controlla quindi la scheda associata tramite la consueta interfaccia dello strumento `browser` (selezionando il profilo corretto).

<div id="install-load-unpacked">
  ## Installazione / caricamento (non pacchettizzata)
</div>

1. Installa l&#39;estensione in un percorso locale fisso:

```bash
openclaw browser extension install
```

2. Stampa il percorso della directory in cui è installata l&#39;estensione:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* Abilita “Modalità sviluppatore”
* “Carica estensione non pacchettizzata” → seleziona la directory indicata sopra

4. Blocca l&#39;estensione.

<div id="updates-no-build-step">
  ## Aggiornamenti (nessuna fase di build)
</div>

L&#39;estensione è inclusa all&#39;interno della release di OpenClaw (pacchetto npm) come file statici. Non esiste una fase di “build” separata.

Dopo aver aggiornato OpenClaw:

* Esegui nuovamente `openclaw browser extension install` per aggiornare i file installati nella directory dello stato di OpenClaw.
* In Chrome → `chrome://extensions` → fai clic su “Ricarica” sull&#39;estensione.

<div id="use-it-no-extra-config">
  ## Usalo (nessuna configurazione aggiuntiva)
</div>

OpenClaw viene fornito con un profilo browser integrato chiamato `chrome` che punta al relay dell&#39;estensione sulla porta predefinita.

Usalo:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Strumento Agente: `browser` con `profile="chrome"`

Se vuoi un nome diverso o una porta del relay diversa, crea il tuo profilo personalizzato:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## Collega / scollega (pulsante barra degli strumenti)
</div>

* Apri la scheda che vuoi che OpenClaw controlli.
* Fai clic sull&#39;icona dell&#39;estensione.
  * Il badge mostra `ON` quando è collegata.
* Fai clic di nuovo per scollegare.

<div id="which-tab-does-it-control">
  ## Quale scheda controlla?
</div>

* **Non** controlla automaticamente “la scheda che hai davanti”.
* Controlla **solo le schede che hai collegato esplicitamente** facendo clic sul pulsante della barra degli strumenti.
* Per passare a un&#39;altra scheda: apri l’altra scheda e fai clic sull’icona dell’estensione in quella scheda.

<div id="badge-common-errors">
  ## Badge + errori comuni
</div>

* `ON`: collegata; OpenClaw può controllare quella scheda.
* `…`: connessione al relay locale.
* `!`: relay non raggiungibile (caso più comune: il server del relay del browser non è in esecuzione su questa macchina).

Se vedi `!`:

* Assicurati che il Gateway sia in esecuzione in locale (configurazione predefinita), oppure esegui un nodo host su questa macchina se il Gateway è in esecuzione altrove.
* Apri la pagina delle opzioni dell&#39;estensione; mostra se il relay è raggiungibile.

<div id="remote-gateway-use-a-node-host">
  ## Gateway remoto (usa un nodo host)
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### Gateway locale (stessa macchina di Chrome) — di solito **nessuna configurazione aggiuntiva**
</div>

Se il Gateway è in esecuzione sulla stessa macchina su cui gira Chrome, avvia il servizio di controllo del browser sulla loopback
e avvia automaticamente il relay server. L&#39;estensione comunica con il relay locale; le chiamate CLI/tool vanno al Gateway.

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### Remote Gateway (Gateway runs elsewhere) — **esegui un nodo host**
</div>

Se il tuo Gateway è in esecuzione su un&#39;altra macchina, avvia un nodo host sulla macchina su cui gira Chrome.
Il Gateway farà da proxy per le azioni del browser verso quel nodo; l&#39;estensione e il relay rimangono locali alla macchina del browser.

Se sono connessi più nodi, fissane uno con `gateway.nodes.browser.node` oppure imposta `gateway.nodes.browser.mode`.

<div id="sandboxing-tool-containers">
  ## Sandboxing (container degli strumenti)
</div>

Se la tua sessione dell&#39;agente è in sandbox (`agents.defaults.sandbox.mode != "off"`), lo strumento `browser` può essere soggetto a limitazioni:

* Per impostazione predefinita, le sessioni in sandbox spesso puntano al **browser sandbox** (`target="sandbox"`), non al Chrome dell&#39;host.
* Il takeover tramite relay dell&#39;estensione di Chrome richiede il controllo del server di controllo del browser **host**.

Opzioni:

* Opzione più semplice: usa l&#39;estensione da una sessione/agente **non in sandbox**.
* Oppure consenti il controllo del browser dell&#39;host per le sessioni in sandbox:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Quindi assicurati che lo strumento non sia bloccato dalla tool policy e, se necessario, chiama `browser` con `target="host"`.

Debugging: `openclaw sandbox explain`

<div id="remote-access-tips">
  ## Suggerimenti per l’accesso remoto
</div>

* Mantieni il Gateway e l’host del nodo sulla stessa tailnet; evita di esporre le porte di relay sulla LAN o su Internet pubblico.
* Associa i nodi in modo esplicito; disabilita il routing del proxy del browser se non vuoi il controllo remoto (`gateway.nodes.browser.mode="off"`).

<div id="how-extension-path-works">
  ## Come funziona “extension path”
</div>

`openclaw browser extension path` mostra la directory sul disco in cui è **installata** l&#39;estensione e che contiene i relativi file.

La CLI per progettazione **non** mostra un percorso `node_modules`. Esegui sempre prima `openclaw browser extension install` per copiare l&#39;estensione in una posizione stabile all&#39;interno della directory di stato di OpenClaw.

Se sposti o elimini quella directory di installazione, Chrome contrassegnerà l&#39;estensione come danneggiata finché non la ricarichi da un percorso valido.

<div id="security-implications-read-this">
  ## Implicazioni di sicurezza (leggi questo)
</div>

Questo è potente e rischioso. Trattalo come se stessi dando al modello “le mani sul tuo browser”.

* L&#39;estensione utilizza l&#39;API di debug di Chrome (`chrome.debugger`). Quando è collegata, il modello può:
  * fare clic/scrivere/navigare in quella scheda
  * leggere il contenuto della pagina
  * accedere a tutto ciò a cui può accedere la sessione con accesso effettuato in quella scheda
* **Questo non è isolato** come il profilo dedicato gestito da openclaw.
  * Se ti colleghi al tuo profilo/scheda di uso quotidiano, stai concedendo accesso allo stato di quell&#39;account.

Raccomandazioni:

* Preferisci un profilo Chrome dedicato (separato dalla tua navigazione personale) per l&#39;uso del relay dell&#39;estensione.
* Mantieni il Gateway e qualsiasi host dei nodi accessibili solo tramite Tailnet; fai affidamento sull&#39;autenticazione del Gateway + abbinamento del nodo.
* Evita di esporre le porte del relay sulla LAN (`0.0.0.0`) ed evita Funnel (pubblico).

Correlato:

* Panoramica dello strumento Browser: [Browser](/it/tools/browser)
* Audit di sicurezza: [Security](/it/gateway/security)
* Configurazione Tailscale: [Tailscale](/it/gateway/tailscale)