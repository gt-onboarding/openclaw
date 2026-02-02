---
title: Bonjour
summary: "Scoperta e debug di Bonjour/mDNS (segnali/beacon del Gateway, client e cause di errore più comuni)"
read_when:
  - Risoluzione di problemi di scoperta Bonjour su macOS/iOS
  - Modifica dei tipi di servizio mDNS, dei record TXT o dell'UX di scoperta
---

<div id="bonjour-mdns-discovery">
  # Bonjour / rilevamento mDNS
</div>

OpenClaw utilizza Bonjour (mDNS / DNS‑SD) come **comodità limitata alla sola LAN** per individuare
un Gateway attivo (endpoint WebSocket). È una funzionalità best‑effort e **non** sostituisce la connettività basata su SSH o su Tailnet.

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## Bonjour su vasta area (DNS‑SD unicast) tramite Tailscale
</div>

Se il nodo e il Gateway si trovano su reti diverse, il mDNS multicast non attraverserà
il confine tra le reti. Puoi mantenere la stessa UX di discovery passando a **DNS‑SD unicast**
(&quot;Wide‑Area Bonjour&quot;) tramite Tailscale.

Passaggi generali:

1. Esegui un server DNS sull&#39;host del Gateway (raggiungibile tramite Tailnet).
2. Pubblica i record DNS‑SD per `_openclaw-gw._tcp` sotto una zona dedicata
   (esempio: `openclaw.internal.`).
3. Configura lo **split DNS** di Tailscale in modo che il dominio scelto venga risolto tramite quel
   server DNS per i client (inclusi iOS).

OpenClaw supporta qualsiasi dominio di discovery; `openclaw.internal.` è solo un esempio.
I nodi iOS/Android eseguono il browsing sia di `local.` sia del tuo dominio wide‑area configurato.

<div id="gateway-config-recommended">
  ### Configurazione del Gateway (raccomandata)
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // abilita la pubblicazione DNS-SD wide-area
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### Configurazione iniziale del server DNS (host del Gateway)
</div>

```bash
openclaw dns setup --apply
```

Questo installa CoreDNS e lo configura per:

* ascoltare sulla porta 53 solo sulle interfacce Tailscale del Gateway
* servire il dominio scelto (per esempio: `openclaw.internal.`) da `~/.openclaw/dns/<domain>.db`

Verifica da una macchina connessa alla tailnet:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Impostazioni DNS di Tailscale
</div>

Nella console di amministrazione di Tailscale:

* Aggiungi un nameserver che punti all&#39;IP tailnet del Gateway (UDP/TCP 53).
* Configura lo split DNS in modo che il dominio di discovery utilizzi quel nameserver.

Quando i client utilizzano il DNS del tailnet, i nodi iOS possono individuare
`_openclaw-gw._tcp` nel dominio di discovery senza multicast.

<div id="gateway-listener-security-recommended">
  ### Sicurezza del listener del Gateway (consigliato)
</div>

La porta WS del Gateway (predefinita `18789`) si associa all’interfaccia di loopback per impostazione predefinita. Per l’accesso da LAN/tailnet, esegui un bind esplicito e mantieni l’autenticazione abilitata.

Per configurazioni limitate alla sola tailnet:

* Imposta `gateway.bind: "tailnet"` in `~/.openclaw/openclaw.json`.
* Riavvia il Gateway (o riavvia l’app nella barra dei menu di macOS).

<div id="what-advertises">
  ## Chi effettua l&#39;annuncio
</div>

Solo il Gateway annuncia `_openclaw-gw._tcp`.

<div id="service-types">
  ## Tipi di servizio
</div>

* `_openclaw-gw._tcp` — beacon di trasporto del Gateway (utilizzato dai nodi macOS/iOS/Android).

<div id="txt-keys-nonsecret-hints">
  ## Chiavi TXT (indicazioni non segrete)
</div>

Il Gateway espone alcune indicazioni non segrete per rendere più comodi i flussi nella UI:

* `role=gateway`
* `displayName=<friendly name>` (nome descrittivo)
* `lanHost=<hostname>.local`
* `gatewayPort=<port>` (Gateway WS + HTTP)
* `gatewayTls=1` (solo quando TLS è attivo)
* `gatewayTlsSha256=<sha256>` (solo quando TLS è attivo e l&#39;impronta digitale è disponibile)
* `canvasPort=<port>` (solo quando l&#39;host canvas è abilitato; valore predefinito `18793`)
* `sshPort=<port>` (impostato a 22 se non sovrascritto)
* `transport=gateway`
* `cliPath=<path>` (facoltativo; percorso assoluto a un entrypoint `openclaw` eseguibile)
* `tailnetDns=<magicdns>` (indicazione facoltativa quando Tailnet è disponibile)

<div id="debugging-on-macos">
  ## Debugging su macOS
</div>

Strumenti integrati utili:

* Esplora le istanze:
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* Risolvi una singola istanza (sostituisci `<instance>`):
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Se l&#39;esplorazione funziona ma la risoluzione no, in genere il problema è dovuto a una policy della LAN o al resolver mDNS.

<div id="debugging-in-gateway-logs">
  ## Debugging nei log del Gateway
</div>

Il Gateway scrive un file di log ciclico (mostrato all&#39;avvio come
`gateway log file: ...`). Cerca le righe con `bonjour:`, in particolare:

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## Debug sul nodo iOS
</div>

Il nodo iOS usa `NWBrowser` per rilevare `_openclaw-gw._tcp`.

Per acquisire i log:

* Impostazioni → Gateway → Avanzate → **Discovery Debug Logs**
* Impostazioni → Gateway → Avanzate → **Discovery Logs** → riproduci il problema → **Copy**

Il log include le transizioni di stato del browser e le modifiche al set di risultati.

<div id="common-failure-modes">
  ## Modalità di guasto comuni
</div>

* **Bonjour non attraversa le reti**: usa Tailnet o SSH.
* **Multicast bloccato**: alcune reti Wi‑Fi disabilitano mDNS.
* **Sospensione / cambi frequenti di interfaccia**: macOS può temporaneamente scartare i risultati mDNS; riprova.
* **L’esplorazione funziona ma la risoluzione non riesce**: mantieni i nomi delle macchine semplici (evita emoji o
  punteggiatura), quindi riavvia il Gateway. Il nome dell&#39;istanza del servizio deriva dal nome host, quindi nomi troppo complessi possono confondere alcuni resolver.

<div id="escaped-instance-names-032">
  ## Nomi di istanza con caratteri di escape (`\032`)
</div>

Bonjour/DNS‑SD spesso rappresenta i byte nei nomi di istanza del servizio come sequenze decimali con escape `\DDD`
(ad esempio gli spazi diventano `\032`).

* Questo è previsto a livello di protocollo.
* Le UI dovrebbero decodificarli per la visualizzazione (iOS usa `BonjourEscapes.decode`).

<div id="disabling-configuration">
  ## Disabilitazione / configurazione
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` disabilita la pubblicazione (legacy: `OPENCLAW_DISABLE_BONJOUR`).
* `gateway.bind` in `~/.openclaw/openclaw.json` controlla la modalità di bind del Gateway.
* `OPENCLAW_SSH_PORT` esegue l’override della porta SSH pubblicata nei record TXT (legacy: `OPENCLAW_SSH_PORT`).
* `OPENCLAW_TAILNET_DNS` pubblica un hint MagicDNS nei record TXT (legacy: `OPENCLAW_TAILNET_DNS`).
* `OPENCLAW_CLI_PATH` esegue l’override del percorso della CLI pubblicato (legacy: `OPENCLAW_CLI_PATH`).

<div id="related-docs">
  ## Documentazione correlata
</div>

* Criteri di individuazione e selezione del trasporto: [Discovery](/it/gateway/discovery)
* Abbinamento dei nodi e approvazioni: [Gateway pairing](/it/gateway/pairing)