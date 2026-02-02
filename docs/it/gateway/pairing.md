---
title: Abbinamento
summary: "Abbinamento dei nodi gestito dal Gateway (Opzione B) per iOS e altri nodi remoti"
read_when:
  - Implementare le approvazioni di abbinamento dei nodi senza UI macOS
  - Aggiungere flussi CLI per approvare nodi remoti
  - Estendere il protocollo del Gateway con la gestione dei nodi
---

<div id="gateway-owned-pairing-option-b">
  # Abbinamento gestito dal Gateway (Opzione B)
</div>

Nell'abbinamento gestito dal **Gateway**, il **Gateway** è la fonte di verità su quali nodi
sono autorizzati a collegarsi. Le UI (app macOS, client futuri) sono solo frontend che
approvano o rifiutano le richieste in sospeso.

**Importante:** i nodi WS usano l’**abbinamento del dispositivo** (ruolo `node`) durante `connect`.
`node.pair.*` è un archivio di abbinamenti separato e **non** regola l’handshake WS.
Solo i client che chiamano esplicitamente `node.pair.*` usano questo flusso.

<div id="concepts">
  ## Concetti
</div>

- **Richiesta in sospeso**: un nodo ha chiesto di unirsi; richiede approvazione.
- **Nodo associato**: nodo approvato a cui è stato emesso un token di autenticazione.
- **Trasporto**: l'endpoint WS del Gateway inoltra le richieste ma non decide
  l'appartenenza. (Il supporto legacy per il bridge TCP è deprecato ed è stato rimosso.)

<div id="how-pairing-works">
  ## Come funziona l'abbinamento
</div>

1. Un nodo si connette al WS del Gateway e richiede l'abbinamento.
2. Il Gateway registra una **richiesta in sospeso** ed emette `node.pair.requested`.
3. Tu approvi o rifiuti la richiesta (da CLI o UI).
4. In caso di approvazione, il Gateway genera un **nuovo token** (i token vengono ruotati a ogni nuovo abbinamento).
5. Il nodo si riconnette usando il token ed è ora abbinato.

Le richieste in sospeso scadono automaticamente dopo **5 minuti**.

<div id="cli-workflow-headless-friendly">
  ## Flusso di lavoro CLI (adatto all&#39;esecuzione headless)
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` mostra i nodi associati o connessi e le rispettive capacità.


<div id="api-surface-gateway-protocol">
  ## Superficie API (protocollo del Gateway)
</div>

Eventi:

- `node.pair.requested` — generato quando viene creata una nuova richiesta pendente.
- `node.pair.resolved` — generato quando una richiesta viene approvata/rifiutata/va in scadenza.

Metodi:

- `node.pair.request` — crea o riutilizza una richiesta pendente.
- `node.pair.list` — elenca i nodi pendenti + accoppiati.
- `node.pair.approve` — approva una richiesta pendente (emette un token).
- `node.pair.reject` — rifiuta una richiesta pendente.
- `node.pair.verify` — verifica `{ nodeId, token }`.

Note:

- `node.pair.request` è idempotente per nodo: le chiamate ripetute restituiscono la stessa
  richiesta pendente.
- L'approvazione genera **sempre** un nuovo token; nessun token viene mai restituito da
  `node.pair.request`.
- Le richieste possono includere `silent: true` come hint per i flussi di approvazione automatica.

<div id="auto-approval-macos-app">
  ## Approvazione automatica (app macOS)
</div>

L'app macOS può facoltativamente tentare un'approvazione silenziosa quando:

- la richiesta è contrassegnata come `silent`, e
- l'app può verificare una connessione SSH all'host del Gateway usando lo stesso utente.

Se l'approvazione silenziosa non riesce, l'app torna alla normale richiesta “Approva/Rifiuta”.

<div id="storage-local-private">
  ## Archiviazione (locale, privata)
</div>

Lo stato di abbinamento è memorizzato nella directory di stato del Gateway (predefinita `~/.openclaw`):

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Se ridefinisci `OPENCLAW_STATE_DIR`, la cartella `nodes/` si sposta di conseguenza.

Note di sicurezza:

- I token sono segreti; tratta `paired.json` come contenuto sensibile.
- La rotazione di un token richiede una nuova approvazione (o l'eliminazione della voce del nodo).

<div id="transport-behavior">
  ## Comportamento del trasporto
</div>

- Il trasporto è **stateless**; non tiene traccia dei membri.
- Se il Gateway è offline o l'abbinamento è disabilitato, i nodi non possono effettuare l'abbinamento.
- Se il Gateway è in modalità remota, l'abbinamento avviene comunque sullo store del Gateway remoto.