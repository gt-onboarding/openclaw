---
title: Scoperta
summary: "Scoperta dei nodi e trasporti (Bonjour, Tailscale, SSH) per individuare il Gateway"
read_when:
  - Implementare o modificare la scoperta/pubblicazione Bonjour
  - Configurare le modalità di connessione remota (diretta vs SSH)
  - Progettare la scoperta e l’abbinamento per nodi remoti
---

<div id="discovery-transports">
  # Discovery &amp; transports
</div>

OpenClaw deve affrontare due problemi distinti che in superficie sembrano simili:

1. **Controllo remoto da parte dell’operatore**: l’app della barra dei menu di macOS che controlla un Gateway in esecuzione altrove.
2. **Abbinamento del nodo**: iOS/Android (e i nodi futuri) che devono trovare un Gateway ed effettuare un abbinamento sicuro.

L’obiettivo progettuale è mantenere tutta la network discovery/advertising nel **Node Gateway** (`openclaw gateway`) e lasciare i client (app Mac, iOS) nel ruolo di consumer.

<div id="terms">
  ## Termini
</div>

* **Gateway**: un singolo processo Gateway di lunga durata che gestisce lo stato (sessioni, abbinamento, registro dei nodi) ed esegue i canali. La maggior parte delle installazioni ne usa uno per host; sono possibili configurazioni con più Gateway isolati.
* **Gateway WS (control plane)**: l&#39;endpoint WebSocket su `127.0.0.1:18789` per impostazione predefinita; può essere associato alla LAN/tailnet tramite `gateway.bind`.
* **Trasporto WS diretto**: un endpoint Gateway WS esposto su LAN/tailnet (senza SSH).
* **Trasporto SSH (fallback)**: controllo remoto tramite inoltro di `127.0.0.1:18789` su SSH.
* **Bridge TCP legacy (deprecato/rimosso)**: vecchio trasporto dei nodi (vedi [Bridge protocol](/it/gateway/bridge-protocol)); non è più pubblicizzato per la scoperta automatica.

Dettagli del protocollo:

* [Gateway protocol](/it/gateway/protocol)
* [Bridge protocol (legacy)](/it/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## Perché manteniamo sia la modalità “diretta” che SSH
</div>

* **WS diretto** offre la migliore UX sulla stessa rete e all&#39;interno di una tailnet:
  * individuazione automatica sulla LAN tramite Bonjour
  * token di abbinamento + ACL gestiti dal Gateway
  * non è richiesto l&#39;accesso alla shell; la superficie del protocollo può rimanere minima e verificabile
* **SSH** rimane il fallback universale:
  * funziona ovunque tu abbia accesso SSH (anche tra reti non correlate)
  * resiste a problemi di multicast/mDNS
  * non richiede nuove porte in ingresso oltre a SSH

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## Input di individuazione (come i client individuano dove si trova il Gateway)
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS (solo LAN)
</div>

Bonjour funziona a “miglior sforzo” e non attraversa reti diverse. Viene utilizzato solo come comodità per dispositivi sulla stessa LAN.

Direzione prevista:

* Il **Gateway** annuncia il proprio endpoint WS tramite Bonjour.
* I client eseguono la ricerca e mostrano un elenco in cui puoi “scegliere un gateway”, quindi memorizzano l’endpoint selezionato.

Risoluzione dei problemi e dettagli sul beacon: [Bonjour](/it/gateway/bonjour).

<div id="service-beacon-details">
  #### Dettagli del beacon di servizio
</div>

* Tipi di servizio:
  * `_openclaw-gw._tcp` (beacon di trasporto del Gateway)
* Chiavi TXT (non segrete):
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22` (o qualunque porta venga annunciata)
  * `gatewayPort=18789` (Gateway WS + HTTP)
  * `gatewayTls=1` (solo quando TLS è abilitato)
  * `gatewayTlsSha256=<sha256>` (solo quando TLS è abilitato e l&#39;impronta è disponibile)
  * `canvasPort=18793` (porta predefinita dell&#39;host canvas; serve `/__openclaw__/canvas/`)
  * `cliPath=<path>` (opzionale; percorso assoluto verso un entrypoint o binario `openclaw` eseguibile)
  * `tailnetDns=<magicdns>` (suggerimento opzionale; rilevato automaticamente quando Tailscale è disponibile)

Disabilitare/sovrascrivere:

* `OPENCLAW_DISABLE_BONJOUR=1` disabilita l&#39;annuncio del servizio.
* `gateway.bind` in `~/.openclaw/openclaw.json` controlla la modalità di bind del Gateway.
* `OPENCLAW_SSH_PORT` sovrascrive la porta SSH annunciata nel TXT (predefinita 22).
* `OPENCLAW_TAILNET_DNS` pubblica un suggerimento `tailnetDns` (MagicDNS).
* `OPENCLAW_CLI_PATH` sovrascrive il percorso CLI annunciato.

<div id="2-tailnet-cross-network">
  ### 2) Tailnet (tra reti)
</div>

Per configurazioni in stile Londra/Vienna, Bonjour non sarà d’aiuto. Il “target” diretto consigliato è:

* nome Tailscale MagicDNS (preferito) o un IP Tailnet stabile.

Se il Gateway rileva che è in esecuzione su Tailscale, pubblica `tailnetDns` come suggerimento opzionale per i client (inclusi i beacon wide-area).

<div id="3-manual-ssh-target">
  ### 3) Manuale / destinazione SSH
</div>

Quando non esiste una route diretta (o l&#39;instradamento diretto è disabilitato), i client possono sempre connettersi tramite SSH inoltrando la porta del Gateway sulla loopback locale.

Consulta [Accesso remoto](/it/gateway/remote).

<div id="transport-selection-client-policy">
  ## Selezione del trasporto (criteri client)
</div>

Comportamento del client consigliato:

1. Se è configurato e raggiungibile un endpoint diretto associato, usalo.
2. Altrimenti, se Bonjour trova un Gateway sulla LAN, offri un&#39;opzione con un solo tocco &quot;Usa questo Gateway&quot; e salvalo come endpoint diretto.
3. Altrimenti, se è configurato un DNS/IP di tailnet, prova la connessione diretta.
4. Altrimenti, passa a SSH.

<div id="pairing-auth-direct-transport">
  ## Abbinamento + autenticazione (trasporto diretto)
</div>

Il Gateway è la fonte di verità per l’ammissione dei nodi/client.

* Le richieste di abbinamento vengono create, approvate o rifiutate nel Gateway (vedi [Gateway pairing](/it/gateway/pairing)).
* Il Gateway applica:
  * autenticazione (token / coppia di chiavi)
  * scope/ACL (il Gateway non funge da proxy diretto per tutti i metodi)
  * limiti di frequenza

<div id="responsibilities-by-component">
  ## Responsabilità per componente
</div>

* **Gateway**: annuncia i beacon di discovery, gestisce le decisioni di abbinamento e ospita l&#39;endpoint WS.
* **app macOS**: ti aiuta a scegliere un Gateway, mostra i prompt di abbinamento e usa SSH solo come fallback.
* **nodi iOS/Android**: usano Bonjour per comodità e si connettono al WS del Gateway abbinato.