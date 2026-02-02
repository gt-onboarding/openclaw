---
title: Protocollo Bridge
summary: "Protocollo Bridge (nodi legacy): TCP JSONL, abbinamento, RPC con scope"
read_when:
  - Sviluppo o debug di client nodo (modalità nodo iOS/Android/macOS)
  - Analisi di problemi di abbinamento o di autenticazione del Bridge
  - Verifica della superficie del nodo esposta dal Gateway
---

<div id="bridge-protocol-legacy-node-transport">
  # Protocollo Bridge (trasporto legacy per i nodi)
</div>

Il protocollo Bridge è un trasporto **legacy** per i nodi (TCP JSONL). I nuovi client nodo
dovrebbero invece usare il protocollo WebSocket unificato del Gateway.

Se stai sviluppando un operatore o un client nodo, usa il
[protocollo del Gateway](/it/gateway/protocol).

**Nota:** Le versioni attuali di OpenClaw non includono più il listener TCP del bridge; questo documento è mantenuto solo come riferimento storico.
Le chiavi di configurazione legacy `bridge.*` non fanno più parte dello schema di configurazione.

<div id="why-we-have-both">
  ## Perché usiamo entrambi
</div>

* **Perimetro di sicurezza**: il bridge espone una piccola lista di autorizzati invece
  dell&#39;intera superficie API del Gateway.
* **Abbinamento + identità del nodo**: l&#39;ammissione dei nodi è gestita dal Gateway ed è collegata
  a un token specifico per ogni nodo.
* **Discovery UX**: i nodi possono individuare i Gateway tramite Bonjour sulla LAN oppure connettersi
  direttamente attraverso una tailnet.
* **Loopback WS**: il piano di controllo WS completo resta locale, a meno che non venga instradato tramite SSH.

<div id="transport">
  ## Trasporto
</div>

* TCP, un oggetto JSON per riga (JSONL).
* TLS opzionale (quando `bridge.tls.enabled` è `true`).
* La porta di ascolto predefinita legacy era `18790` (le versioni attuali non avviano un bridge TCP).

Quando TLS è abilitato, i record TXT di discovery includono `bridgeTls=1` più
`bridgeTlsSha256` in modo che i nodi possano effettuare il pinning del certificato.

<div id="handshake-pairing">
  ## Handshake + abbinamento
</div>

1. Il client invia `hello` con i metadati del nodo + token (se già abbinato).
2. Se non è ancora abbinato, il Gateway risponde con `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Il client invia `pair-request`.
4. Il Gateway attende l&#39;approvazione, quindi invia `pair-ok` e `hello-ok`.

`hello-ok` restituisce `serverName` e può includere `canvasHostUrl`.

<div id="frames">
  ## Frame
</div>

Client → Gateway:

* `req` / `res`: RPC del Gateway con scope (chat, sessioni, config, health, voicewake, skills.bins)
* `event`: segnali del nodo (trascrizione vocale, richiesta agente, sottoscrizione chat, ciclo di vita di exec)

Gateway → Client:

* `invoke` / `invoke-res`: comandi del nodo (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
* `event`: aggiornamenti della chat per le sessioni iscritte
* `ping` / `pong`: keepalive

L&#39;implementazione legacy della lista di autorizzati risiedeva in `src/gateway/server-bridge.ts` (rimossa).

<div id="exec-lifecycle-events">
  ## Eventi del ciclo di vita di Exec
</div>

I nodi possono emettere eventi `exec.finished` o `exec.denied` per segnalare l&#39;attività di system.run.
Questi vengono mappati su eventi di sistema nel Gateway. (I nodi legacy potrebbero ancora emettere `exec.started`.)

Campi del payload (tutti opzionali se non indicato diversamente):

* `sessionKey` (obbligatorio): sessione dell&#39;agente che deve ricevere l&#39;evento di sistema.
* `runId`: ID univoco dell&#39;esecuzione per il raggruppamento.
* `command`: stringa di comando grezza o formattata.
* `exitCode`, `timedOut`, `success`, `output`: dettagli di completamento (solo per `exec.finished`).
* `reason`: motivo del rifiuto (solo per `exec.denied`).

<div id="tailnet-usage">
  ## Utilizzo del tailnet
</div>

* Associa il bridge a un IP del tailnet: `bridge.bind: "tailnet"` in
  `~/.openclaw/openclaw.json`.
* I client si connettono tramite nome MagicDNS o IP del tailnet.
* Bonjour **non** attraversa reti diverse; usa host/porta specificati manualmente o wide‑area DNS‑SD
  quando necessario.

<div id="versioning">
  ## Versioning
</div>

Bridge è attualmente alla **v1 implicita** (nessuna negoziazione di versione min/max). La retrocompatibilità è prevista; aggiungi un campo di versione del protocollo Bridge prima di introdurre qualsiasi modifica incompatibile.