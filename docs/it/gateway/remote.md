---
title: Accesso remoto
summary: "Accesso remoto tramite tunnel SSH (WS del Gateway) e tailnet"
read_when:
  - Esecuzione o risoluzione dei problemi relativi a configurazioni remote del Gateway
---

<div id="remote-access-ssh-tunnels-and-tailnets">
  # Accesso remoto (SSH, tunnel e tailnet)
</div>

Questo repository supporta l’“accesso remoto via SSH” mantenendo un singolo Gateway (il principale) in esecuzione su un host dedicato (desktop/server) e collegando i client ad esso.

* Per **operatori (tu / l’app macOS)**: il tunneling SSH è il fallback universale.
* Per **nodi (iOS/Android e dispositivi futuri)**: connetti il nodo al Gateway tramite **WebSocket** (LAN/tailnet o tunnel SSH secondo necessità).

<div id="the-core-idea">
  ## L&#39;idea principale
</div>

* Il WebSocket del Gateway effettua il bind all&#39;interfaccia **loopback** sulla porta configurata (per impostazione predefinita 18789).
* Per l&#39;uso remoto, inoltra quella porta di loopback tramite SSH (oppure usa una tailnet/VPN e riduci il tunneling).

<div id="common-vpntailnet-setups-where-the-agent-lives">
  ## Configurazioni VPN/tailnet comuni (dove vive l&#39;agente)
</div>

Pensa all&#39;**host del Gateway** come al &quot;luogo in cui vive l&#39;agente&quot;. Gestisce sessioni, profili di autenticazione, canali e stato.
Il tuo laptop o desktop (e i nodi) si connettono a quell&#39;host.

<div id="1-always-on-gateway-in-your-tailnet-vps-or-home-server">
  ### 1) Gateway sempre attivo nel tuo tailnet (VPS o server domestico)
</div>

Esegui il Gateway su un host persistente e raggiungilo tramite **Tailscale** o SSH.

* **Migliore UX:** mantieni `gateway.bind: "loopback"` e usa **Tailscale Serve** per la Control UI.
* **Fallback:** mantieni loopback + tunnel SSH da qualsiasi macchina che deve accedervi.
* **Esempi:** [exe.dev](/it/platforms/exe-dev) (VM semplice) o [Hetzner](/it/platforms/hetzner) (VPS di produzione).

Questa configurazione è ideale quando il tuo laptop va spesso in sospensione ma vuoi che l&#39;agente rimanga sempre attivo.

<div id="2-home-desktop-runs-the-gateway-laptop-is-remote-control">
  ### 2) Il desktop di casa esegue il Gateway, il laptop è il controllo remoto
</div>

Il laptop **non** esegue l’agente. Si connette da remoto:

* Usa la modalità **Remote over SSH** dell’app macOS (Settings → General → “OpenClaw runs”).
* L’app apre e gestisce il tunnel, quindi WebChat + i controlli di stato “funzionano e basta”.

Runbook: [accesso remoto macOS](/it/platforms/mac/remote).

<div id="3-laptop-runs-the-gateway-remote-access-from-other-machines">
  ### 3) Il laptop esegue il Gateway, accesso remoto da altre macchine
</div>

Mantieni il Gateway in locale ma esponilo in modo sicuro:

* tunnel SSH verso il laptop dalle altre macchine, oppure
* Tailscale Serve la Control UI e mantieni il Gateway in ascolto solo su loopback.

Guida: [Tailscale](/it/gateway/tailscale) e [Panoramica Web](/it/web).

<div id="command-flow-what-runs-where">
  ## Flusso dei comandi (cosa viene eseguito dove)
</div>

Un servizio Gateway gestisce lo stato e i canali. I nodi sono periferiche.

Esempio di flusso (Telegram → nodo):

* Un messaggio Telegram arriva al **Gateway**.
* Il Gateway esegue l’**agente** e decide se chiamare uno strumento del nodo.
* Il Gateway chiama il **nodo** tramite il WebSocket del Gateway (RPC `node.*`).
* Il nodo restituisce il risultato; il Gateway risponde a sua volta a Telegram.

Note:

* **I nodi non eseguono il servizio Gateway.** Devi eseguire un solo Gateway per host, a meno che tu non voglia intenzionalmente usare profili isolati (vedi [Multiple gateways](/it/gateway/multiple-gateways)).
* La modalità nodo dell’app macOS è semplicemente un client di nodo sul WebSocket del Gateway.

<div id="ssh-tunnel-cli-tools">
  ## Tunnel SSH (CLI + strumenti)
</div>

Crea un tunnel locale verso il WS del Gateway remoto:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Con il tunnel attivo:

* `openclaw health` e `openclaw status --deep` ora raggiungono il Gateway remoto tramite `ws://127.0.0.1:18789`.
* `openclaw gateway {status,health,send,agent,call}` può anche puntare all&#39;URL inoltrato tramite `--url` quando necessario.

Nota: sostituisci `18789` con il tuo `gateway.port` configurato (oppure `--port`/`OPENCLAW_GATEWAY_PORT`).

<div id="cli-remote-defaults">
  ## Impostazioni predefinite remote della CLI
</div>

Puoi salvare una destinazione remota in modo che i comandi della CLI la usino come predefinita:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token"
    }
  }
}
```

Quando il Gateway è limitato all&#39;interfaccia di loopback, lascia l&#39;URL impostato su `ws://127.0.0.1:18789` e apri prima il tunnel SSH.

<div id="chat-ui-over-ssh">
  ## UI di chat via SSH
</div>

WebChat non utilizza più una porta HTTP separata. La UI di chat SwiftUI si connette direttamente al WebSocket del Gateway.

* Inoltra la porta `18789` via SSH (vedi sopra), quindi collega i client a `ws://127.0.0.1:18789`.
* Su macOS, utilizza preferibilmente la modalità “Remote over SSH” dell’app, che gestisce automaticamente il tunnel.

<div id="macos-app-remote-over-ssh">
  ## App macOS “Remote over SSH”
</div>

L&#39;app della barra dei menu di macOS può gestire lo stesso setup end-to-end (verifiche di stato remote, WebChat e inoltro di Voice Wake).

Runbook: [accesso remoto macOS](/it/platforms/mac/remote).

<div id="security-rules-remotevpn">
  ## Regole di sicurezza (remoto/VPN)
</div>

Versione breve: **mantieni il Gateway solo su loopback** a meno che tu non sia sicuro di aver bisogno di un bind.

* **Loopback + SSH/Tailscale Serve** è l’impostazione predefinita più sicura (nessuna esposizione pubblica).
* I **bind non-loopback** (`lan`/`tailnet`/`custom`, oppure `auto` quando il loopback non è disponibile) devono usare token/password di autenticazione.
* `gateway.remote.token` serve **solo** per le chiamate CLI remote — **non** abilita l’autenticazione locale.
* `gateway.remote.tlsFingerprint` esegue il pinning del certificato TLS remoto quando usi `wss://`.
* **Tailscale Serve** può autenticare tramite identity headers quando `gateway.auth.allowTailscale: true`.
  Impostalo su `false` se preferisci usare token/password.
* Considera il controllo via browser come accesso da operatore: solo tramite tailnet + abbinamento del nodo eseguito deliberatamente.

Approfondimento: [Sicurezza](/it/gateway/security).