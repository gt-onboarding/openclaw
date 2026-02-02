---
title: iOS
summary: "App nodo iOS: connessione al Gateway, abbinamento, canvas e risoluzione dei problemi"
read_when:
  - Abbinamento o riconnessione del nodo iOS
  - Esecuzione dell'app iOS dal codice sorgente
  - Debug della discovery del Gateway o dei comandi canvas
---

<div id="ios-app-node">
  # iOS App (Nodo)
</div>

Disponibilità: in anteprima interna. L&#39;app iOS non è ancora distribuita pubblicamente.

<div id="what-it-does">
  ## Cosa fa
</div>

* Si connette a un Gateway tramite WebSocket (LAN o tailnet).
* Espone le capacità del nodo: Canvas, istantanea schermo, acquisizione fotocamera, posizione, modalità Talk, attivazione vocale (Voice wake).
* Riceve comandi `node.invoke` e invia eventi di stato del nodo.

<div id="requirements">
  ## Requisiti
</div>

* Gateway in esecuzione su un altro dispositivo (macOS, Linux o Windows tramite WSL2).
* Percorso di rete:
  * Stessa LAN tramite Bonjour, **oppure**
  * Tailnet tramite DNS-SD unicast (dominio di esempio: `openclaw.internal.`), **oppure**
  * Host/porta configurati manualmente (fallback).

<div id="quick-start-pair-connect">
  ## Avvio rapido (abbinamento + connessione)
</div>

1. Avvia il Gateway:

```bash
openclaw gateway --port 18789
```

2. Nell&#39;app iOS, apri Impostazioni e seleziona un Gateway individuato (oppure abilita Manual Host e inserisci host/porta).

3. Approva la richiesta di abbinamento sull&#39;host del Gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Verifica la connessione:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## Modalità di individuazione
</div>

<div id="bonjour-lan">
  ### Bonjour (LAN)
</div>

Il Gateway annuncia `_openclaw-gw._tcp` su `local.`. L&#39;app iOS li rileva e li elenca automaticamente.

<div id="tailnet-cross-network">
  ### Tailnet (cross-network)
</div>

Quando mDNS è bloccato, utilizza una zona DNS-SD unicast (scegli un dominio; ad esempio: `openclaw.internal.`) e lo split DNS di Tailscale.
Vedi [Bonjour](/it/gateway/bonjour) per un esempio con CoreDNS.

<div id="manual-hostport">
  ### Host/porta manuali
</div>

In Impostazioni, attiva **Manual Host** e inserisci l&#39;host e la porta del Gateway (predefinita `18789`).

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

Il nodo iOS esegue il rendering di un canvas WKWebView. Utilizza `node.invoke` per controllarlo:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

Note:

* L&#39;host canvas del Gateway espone `/__openclaw__/canvas/` e `/__openclaw__/a2ui/`.
* Il nodo iOS passa automaticamente ad A2UI alla connessione quando viene annunciato un URL dell&#39;host canvas.
* Torna allo scaffold integrato con `canvas.navigate` e `{"url":""}`.

<div id="canvas-eval-snapshot">
  ### Valutazione Canvas / snapshot
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## Attivazione vocale + modalità conversazione
</div>

* L&#39;attivazione vocale e la modalità conversazione sono disponibili in Impostazioni.
* iOS può sospendere l&#39;audio in background; considera le funzionalità vocali non garantite quando l&#39;app non è attiva.

<div id="common-errors">
  ## Errori comuni
</div>

* `NODE_BACKGROUND_UNAVAILABLE`: porta l&#39;app iOS in primo piano (i comandi canvas/camera/screen lo richiedono).
* `A2UI_HOST_NOT_CONFIGURED`: il Gateway non ha esposto un URL host canvas; controlla `canvasHost` nella [configurazione del Gateway](/it/gateway/configuration).
* Il prompt di abbinamento non viene mai mostrato: esegui `openclaw nodes pending` e approva manualmente.
* La riconnessione non riesce dopo la reinstallazione: il token di abbinamento nel Keychain è stato eliminato; esegui nuovamente l&#39;abbinamento del nodo.

<div id="related-docs">
  ## Documentazione correlata
</div>

* [Abbinamento](/it/gateway/pairing)
* [Discovery](/it/gateway/discovery)
* [Bonjour](/it/gateway/bonjour)