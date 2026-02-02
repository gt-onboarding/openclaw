---
title: Android
summary: "App Android (nodo): runbook di connessione + Canvas/Chat/Camera"
read_when:
  - Abbinare o riconnettere il nodo Android
  - Eseguire il debug dell’individuazione del Gateway o dell’autenticazione su Android
  - Verificare l’allineamento della cronologia chat tra i client
---

<div id="android-app-node">
  # App per Android (Node)
</div>

<div id="support-snapshot">
  ## Snapshot di supporto
</div>

* Ruolo: app nodo companion (Android non ospita il Gateway).
* Gateway necessario: sì (esegui il Gateway su macOS, Linux o Windows tramite WSL2).
* Installazione: [Guida introduttiva](/it/start/getting-started) + [Abbinamento](/it/gateway/pairing).
* Gateway: [Runbook](/it/gateway) + [Configurazione](/it/gateway/configuration).
  * Protocolli: [Protocollo Gateway](/it/gateway/protocol) (nodi + piano di controllo).

<div id="system-control">
  ## Controllo del sistema
</div>

Il controllo del sistema (launchd/systemd) è in esecuzione sull&#39;host del Gateway. Consulta [Gateway](/it/gateway).

<div id="connection-runbook">
  ## Runbook di connessione
</div>

App nodo Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android si connette direttamente al WebSocket del Gateway (indirizzo predefinito `ws://<host>:18789`) e utilizza una procedura di abbinamento gestita dal Gateway.

<div id="prerequisites">
  ### Prerequisiti
</div>

* Puoi eseguire il Gateway sulla macchina principale (master).
* Il dispositivo/emulatore Android deve poter raggiungere il WebSocket del Gateway:
  * Stessa LAN con mDNS/NSD, **oppure**
  * Stessa tailnet Tailscale utilizzando Wide-Area Bonjour / DNS-SD unicast (vedi sotto), **oppure**
  * Host/porta del Gateway impostati manualmente (come fallback)
* Puoi eseguire la CLI (`openclaw`) sulla macchina del Gateway (o tramite SSH).

<div id="1-start-the-gateway">
  ### 1) Avvia il Gateway
</div>

```bash
openclaw gateway --port 18789 --verbose
```

Conferma nei log che compaia qualcosa del tipo:

* `listening on ws://0.0.0.0:18789`

Per configurazioni solo su tailnet (consigliato per Vienna ⇄ London), associa il Gateway all&#39;IP della tailnet:

* Imposta `gateway.bind: "tailnet"` in `~/.openclaw/openclaw.json` sull&#39;host che esegue il Gateway.
* Riavvia il Gateway / l&#39;app della barra dei menu di macOS.

<div id="2-verify-discovery-optional">
  ### 2) Verificare il discovery (facoltativo)
</div>

Dalla macchina che esegue il Gateway:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Ulteriori note di debug: [Bonjour](/it/gateway/bonjour).

<div id="tailnet-vienna-london-discovery-via-unicast-dns-sd">
  #### Individuazione Tailnet (Vienna ⇄ London) tramite DNS-SD unicast
</div>

L’individuazione Android NSD/mDNS non funziona tra reti diverse. Se il tuo nodo Android e il Gateway sono su reti differenti ma connessi tramite Tailscale, usa invece Wide-Area Bonjour / DNS-SD unicast:

1. Configura una zona DNS-SD (ad esempio `openclaw.internal.`) sull’host del Gateway e pubblica i record `_openclaw-gw._tcp`.
2. Configura lo split DNS di Tailscale per il dominio scelto indirizzandolo a quel server DNS.

Dettagli ed esempio di configurazione CoreDNS: [Bonjour](/it/gateway/bonjour).

<div id="3-connect-from-android">
  ### 3) Connettersi da Android
</div>

Nell&#39;app Android:

* L&#39;app mantiene attiva la connessione al Gateway tramite un **servizio in primo piano** (notifica persistente).
* Apri **Settings**.
* In **Discovered Gateways**, seleziona il tuo Gateway e premi **Connect**.
* Se mDNS è bloccato, usa **Advanced → Manual Gateway** (host + porta) e **Connect (Manual)**.

Dopo il primo abbinamento riuscito, Android si riconnette automaticamente all&#39;avvio:

* Endpoint manuale (se abilitato), altrimenti
* L&#39;ultimo Gateway individuato (best effort).

<div id="4-approve-pairing-cli">
  ### 4) Approva l&#39;abbinamento (CLI)
</div>

Sulla macchina che esegue il Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Dettagli sull&#39;abbinamento: [Abbinamento del Gateway](/it/gateway/pairing).

<div id="5-verify-the-node-is-connected">
  ### 5) Verifica che il nodo sia connesso
</div>

* Tramite lo stato dei nodi:
  ```bash
  openclaw nodes status
  ```
* Tramite Gateway:
  ```bash
  openclaw gateway call node.list --params "{}"
  ```

<div id="6-chat-history">
  ### 6) Chat + cronologia
</div>

La scheda Chat del nodo Android usa la **chiave di sessione primaria** (`main`) del Gateway, quindi cronologia e risposte sono condivise con WebChat e altri client:

* Cronologia: `chat.history`
* Invia: `chat.send`
* Aggiornamenti push (best-effort): `chat.subscribe` → `event:"chat"`

<div id="7-canvas-camera">
  ### 7) Canvas + fotocamera
</div>

<div id="gateway-canvas-host-recommended-for-web-content">
  #### Gateway Canvas Host (consigliato per contenuti web)
</div>

Se vuoi che il nodo mostri HTML/CSS/JS reale che l&#39;agente può modificare su disco, configura il nodo in modo che punti al canvas host del Gateway.

Nota: i nodi usano il canvas host autonomo su `canvasHost.port` (predefinito `18793`).

1. Crea `~/.openclaw/workspace/canvas/index.html` sull&#39;host del Gateway.

2. Dal nodo, raggiungi quella pagina (LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__openclaw__/canvas/"}'
```

Tailnet (opzionale): se entrambi i dispositivi sono su Tailscale, utilizza un nome MagicDNS o l&#39;IP del tailnet invece di `.local`, ad es. `http://<gateway-magicdns>:18793/__openclaw__/canvas/`.

Questo server inserisce un client di live-reload nell&#39;HTML e ricarica la pagina quando i file cambiano.
L&#39;host A2UI è disponibile all&#39;indirizzo `http://<gateway-host>:18793/__openclaw__/a2ui/`.

Comandi Canvas (solo in primo piano):

* `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (usa `{"url":""}` oppure `{"url":"/"}` per tornare allo scaffold predefinito). `canvas.snapshot` restituisce `{ format, base64 }` (predefinito `format="jpeg"`).
* A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` alias legacy)

Comandi fotocamera (solo in primo piano; con controllo dei permessi):

* `camera.snap` (jpg)
* `camera.clip` (mp4)

Consulta il [nodo Camera](/it/nodes/camera) per i parametri e gli helper della CLI.
