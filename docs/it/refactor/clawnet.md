---
title: Clawnet
summary: "Refactoring di Clawnet: unificazione di protocollo di rete, ruoli, autenticazione, approvazioni e identità"
read_when:
  - Pianificazione di un protocollo di rete unificato per nodi + client dell'operatore
  - Rielaborazione di approvazioni, abbinamento, TLS e presenza tra dispositivi
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Refactoring di Clawnet (unificazione del protocollo e dell'autenticazione)
</div>

<div id="hi">
  ## Ciao
</div>

Ciao Peter — ottima direzione: abilita una UX più semplice e una sicurezza più solida.

<div id="purpose">
  ## Scopo
</div>

Un unico documento rigoroso per:

- Stato attuale: protocolli, flussi, confini di fiducia.
- Criticità: approvazioni, instradamento multi‑hop, duplicazione della UI.
- Stato futuro proposto: un solo protocollo, ruoli con scope definiti, autenticazione/abbinamento unificati, pinning TLS.
- Modello di identità: ID stabili + slug carini.
- Piano di migrazione, rischi, questioni aperte.

<div id="goals-from-discussion">
  ## Obiettivi (dalla discussione)
</div>

- Un protocollo unico per tutti i client (app macOS, CLI, iOS, Android, nodo headless).
- Ogni partecipante alla rete deve essere autenticato e accoppiato (paired).
- Chiarezza dei ruoli: nodi vs operatori.
- Approvazioni centralizzate instradate verso il punto in cui si trova l'utente.
- Crittografia TLS + pinning opzionale per tutto il traffico remoto.
- Duplicazione minima del codice.
- Una singola macchina deve comparire una sola volta (nessuna voce duplicata in UI/nodo).

<div id="nongoals-explicit">
  ## Obiettivi non perseguiti (espliciti)
</div>

- Rimuovere la separazione delle funzionalità (il principio del minimo privilegio rimane necessario).
- Esporre l'intero piano di controllo del Gateway senza verifiche di scope.
- Rendere l'autenticazione dipendente da etichette umane (gli slug restano privi di valenza di sicurezza).

---

<div id="current-state-asis">
  # Stato attuale (as-is)
</div>

<div id="two-protocols">
  ## Due protocolli
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket (piano di controllo)
</div>

- Superficie completa delle API: config, canali, modelli, sessioni, esecuzioni di agenti, log, nodi, ecc.
- Binding predefinito: loopback. Accesso remoto tramite SSH/Tailscale.
- Autenticazione: token/password tramite `connect`.
- Nessun TLS pinning (si basa su loopback/tunnel).
- Codice:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge (trasporto del nodo)
</div>

- Superficie della lista di autorizzati ridotta, identità del nodo + abbinamento.
- JSONL su TCP; TLS opzionale + pinning dell’impronta del certificato.
- TLS annuncia l’impronta nel record TXT di discovery.
- Codice:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## Client del control plane allo stato attuale
</div>

- CLI → Gateway WS tramite `callGateway` (`src/gateway/call.ts`).
- UI dell'app macOS → Gateway WS (`GatewayConnection`).
- Web Control UI → Gateway WS.
- ACP → Gateway WS.
- Il controllo tramite browser utilizza un proprio server di controllo HTTP.

<div id="nodes-today">
  ## Nodi oggi
</div>

- L’applicazione macOS in modalità nodo si connette al bridge del Gateway (`MacNodeBridgeSession`).
- Le applicazioni iOS/Android si connettono al bridge del Gateway.
- Abbinamento e token specifico per nodo archiviati nel Gateway.

<div id="current-approval-flow-exec">
  ## Flusso di approvazione attuale (exec)
</div>

- L'agente utilizza `system.run` tramite il Gateway.
- Il Gateway invoca il nodo tramite il bridge.
- Il runtime del nodo decide se approvare.
- Prompt UI mostrato dall'app Mac (quando nodo == app Mac).
- Il nodo restituisce `invoke-res` al Gateway.
- Multi‑hop, UI collegata all'host del nodo.

<div id="presence-identity-today">
  ## Presenza + identità oggi
</div>

- Voci di presenza del Gateway provenienti dai client WS.
- Voci di presenza del nodo provenienti dal bridge.
- L'app macOS può mostrare due voci per la stessa macchina (UI + nodo).
- Identità del nodo memorizzata nello store di abbinamento; identità della UI separata.

---

<div id="problems-pain-points">
  # Problemi / criticità
</div>

- Due stack di protocolli da mantenere (WS + Bridge).
- Approvazioni sui nodi remoti: il prompt appare sull'host del nodo, non dove si trova l'utente.
- Il pinning TLS esiste solo per il bridge; WS dipende da SSH/Tailscale.
- Duplicazione dell'identità: la stessa macchina compare come più istanze.
- Ruoli ambigui: le funzionalità di UI + nodo + CLI non sono chiaramente separate.

---

<div id="proposed-new-state-clawnet">
  # Nuovo stato proposto (Clawnet)
</div>

<div id="one-protocol-two-roles">
  ## Un protocollo, due ruoli
</div>

Singolo protocollo WS con ruolo + scope.

- **Ruolo: nodo** (host di funzionalità)
- **Ruolo: operatore** (piano di controllo)
- **scope** opzionale per l'operatore:
  - `operator.read` (stato + visualizzazione)
  - `operator.write` (esecuzione dell'agente, invio)
  - `operator.admin` (configurazione, canali, modelli)

<div id="role-behaviors">
  ### Comportamenti dei ruoli
</div>

**Nodo**

- Può registrare capacità (`caps`, `commands`, autorizzazioni).
- Può ricevere comandi `invoke` (`system.run`, `camera.*`, `canvas.*`, `screen.record`, ecc.).
- Può inviare eventi: `voice.transcript`, `agent.request`, `chat.subscribe`.
- Non può chiamare le API del control plane per config/models/channels/sessions/agent.

**Operatore**

- Pieno accesso alle API del control plane, limitato dallo scope.
- Riceve tutte le approvazioni.
- Non esegue direttamente azioni sul sistema operativo; instrada le richieste verso i nodi.

<div id="key-rule">
  ### Regola principale
</div>

Il ruolo è per connessione, non per dispositivo. Un dispositivo può assumere entrambi i ruoli, separatamente.

---

<div id="unified-authentication-pairing">
  # Autenticazione e abbinamento unificati
</div>

<div id="client-identity">
  ## Identità del client
</div>

Ogni client fornisce:

- `deviceId` (stabile, derivato dalla chiave del dispositivo).
- `displayName` (nome visualizzato, leggibile dalle persone).
- `role` + `scope` + `caps` + `commands`.

<div id="pairing-flow-unified">
  ## Flusso di abbinamento (unificato)
</div>

- Il client si connette senza autenticazione.
- Il Gateway crea una **pairing request** (richiesta di abbinamento) per quel `deviceId`.
- L'operatore riceve una richiesta; approva o rifiuta.
- Il Gateway rilascia credenziali vincolate a:
  - chiave pubblica del dispositivo
  - ruoli
  - scope
  - capacità/comandi
- Il client salva il token e si riconnette autenticato.

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## Autenticazione vincolata al dispositivo (evitare il replay dei bearer token)
</div>

Preferibile: coppie di chiavi del dispositivo.

- Il dispositivo genera una coppia di chiavi una sola volta.
- `deviceId = fingerprint(publicKey)`.
- Il Gateway invia un nonce; il dispositivo firma; il Gateway verifica.
- I token sono emessi rispetto a una chiave pubblica (proof‑of‑possession), non rispetto a una stringa.

Alternative:

- mTLS (certificati client): più robusto, con maggiore complessità operativa.
- Bearer token di breve durata solo come fase temporanea (ruotare + revocare il prima possibile).

<div id="silent-approval-ssh-heuristic">
  ## Approvazione silenziosa (euristica SSH)
</div>

Definisci questa regola in modo preciso per evitare un punto debole. Preferisci una delle seguenti opzioni:

- **Solo locale**: associa automaticamente quando il client si connette tramite loopback/socket Unix.
- **Challenge via SSH**: il Gateway emette un nonce; il client dimostra l’accesso SSH recuperandolo.
- **Finestra di presenza fisica**: dopo un’approvazione locale sulla UI dell’host del Gateway, consenti l’associazione automatica per una breve finestra (es. 10 minuti).

Registra sempre nei log e conserva traccia delle approvazioni automatiche.

---

<div id="tls-everywhere-dev-prod">
  # TLS ovunque (dev + prod)
</div>

<div id="reuse-existing-bridge-tls">
  ## Riutilizza il TLS del bridge esistente
</div>

Usa il runtime TLS attuale + il pinning dell'impronta digitale:

- `src/infra/bridge/server/tls.ts`
- logica di verifica dell'impronta digitale in `src/node-host/bridge-client.ts`

<div id="apply-to-ws">
  ## Valido per WS
</div>

- Il server WS supporta TLS con lo stesso certificato/chiave + impronta digitale.
- I client WS possono effettuare il pinning dell'impronta digitale (opzionale).
- Il Discovery annuncia TLS + impronta digitale per tutti gli endpoint.
  - Il Discovery fornisce solo suggerimenti di individuazione; non è mai una radice di fiducia.

<div id="why">
  ## Perché
</div>

- Ridurre la dipendenza da SSH/Tailscale ai fini della riservatezza.
- Rendere sicure per impostazione predefinita le connessioni mobili remote.

---

<div id="approvals-redesign-centralized">
  # Redesign del sistema di approvazioni (centralizzato)
</div>

<div id="current">
  ## Stato attuale
</div>

L'approvazione avviene sul nodo host (runtime del nodo dell'app macOS). Il prompt viene visualizzato sul nodo in cui è in esecuzione.

<div id="proposed">
  ## Proposta
</div>

L'approvazione è **eseguita nel Gateway**, con la UI erogata ai client degli operatori.

<div id="new-flow">
  ### Nuovo flusso
</div>

1) Il Gateway riceve l'intent `system.run` (agente).
2) Il Gateway crea un record di approvazione: `approval.requested`.
3) Le UI dell'operatore visualizzano il prompt.
4) La decisione di approvazione viene inviata al Gateway: `approval.resolve`.
5) Il Gateway invoca il comando del nodo in caso di approvazione.
6) Il nodo esegue e restituisce `invoke-res`.

<div id="approval-semantics-hardening">
  ### Semantica delle approvazioni (hardening)
</div>

- Broadcast a tutti gli operatori; solo la UI attiva mostra una finestra modale (le altre ricevono un toast).
- La prima risoluzione prevale; il Gateway rifiuta le risoluzioni successive come già risolte.
- Timeout predefinito: l’operazione viene negata dopo N secondi (ad es. 60s), registrando il motivo nei log.
- La risoluzione richiede lo scope `operator.approvals`.

<div id="benefits">
  ## Vantaggi
</div>

- Il prompt appare dove si trova l'utente (Mac/telefono).
- Approvazioni uniformi per i nodi remoti.
- Il runtime del nodo resta headless, senza dipendenza dalla UI.

---

<div id="role-clarity-examples">
  # Esempi di definizione dei ruoli
</div>

<div id="iphone-app">
  ## App per iPhone
</div>

- **Ruolo di nodo** per: microfono, fotocamera, chat vocale, posizione, push‑to‑talk.
- **operator.read** opzionale per stato e visualizzazione della chat.
- **operator.write/admin** opzionale solo quando abilitato esplicitamente.

<div id="macos-app">
  ## app macOS
</div>

- Ruolo di operatore per impostazione predefinita (Control UI).
- Ruolo nodo quando "Mac node" è abilitato (system.run, screen, camera).
- Stesso deviceId per entrambe le connessioni → voce unificata nella UI.

<div id="cli">
  ## CLI
</div>

- Sempre con ruolo operatore.
- Lo scope è determinato dal sottocomando:
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - approvals + abbinamento → `operator.approvals` / `operator.pairing`

---

<div id="identity-slugs">
  # Identità e slug
</div>

<div id="stable-id">
  ## ID stabile
</div>

Obbligatorio per l'autenticazione; non cambia mai.
Preferibile:

- Impronta della coppia di chiavi (hash della chiave pubblica).

<div id="cute-slug-lobsterthemed">
  ## Cute slug (tema aragosta)
</div>

Solo etichetta leggibile dall'utente.

- Esempio: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Archiviato nel registro del Gateway, modificabile.
- Gestione delle collisioni: `-2`, `-3`.

<div id="ui-grouping">
  ## Raggruppamento UI
</div>

Stesso `deviceId` per i ruoli → singola riga "Instance":

- Badge: `operator`, `nodo`.
- Mostra funzionalità + ultima attività.

---

<div id="migration-strategy">
  # Strategia di migrazione
</div>

<div id="phase-0-document-align">
  ## Fase 0: Documentazione + allineamento
</div>

- Pubblica questo documento.
- Elenca tutte le chiamate di protocollo e i flussi di approvazione.

<div id="phase-1-add-rolesscopes-to-ws">
  ## Fase 1: Aggiungi ruoli/scope a WS
</div>

- Estendi i parametri di `connect` con `role`, `scope` e `deviceId`.
- Aggiungi una lista di autorizzati (allowlist) per controllare l’accesso al ruolo node.

<div id="phase-2-bridge-compatibility">
  ## Fase 2: Compatibilità del bridge
</div>

- Mantieni il bridge in esecuzione.
- Aggiungi in parallelo il supporto per il nodo WS.
- Metti le funzionalità dietro un flag di configurazione.

<div id="phase-3-central-approvals">
  ## Fase 3: Approvazioni centralizzate
</div>

- Aggiungi in WS gli eventi di richiesta di approvazione e di risoluzione.
- Aggiorna la UI dell'app Mac per mostrare le richieste e gestire le risposte.
- Il runtime del nodo smette di inviare richieste alla UI.

<div id="phase-4-tls-unification">
  ## Fase 4: unificazione TLS
</div>

- Aggiungi la configurazione TLS per WS utilizzando il runtime TLS del bridge.
- Aggiungi il pinning lato client.

<div id="phase-5-deprecate-bridge">
  ## Fase 5: Deprecare il bridge
</div>

- Migra i nodi iOS/Android/mac a WS.
- Mantieni il bridge come fallback; rimuovilo una volta che la migrazione è stabile.

<div id="phase-6-devicebound-auth">
  ## Fase 6: Autenticazione vincolata al dispositivo
</div>

- Richiedere un'identità basata su chiavi per tutte le connessioni non locali.
- Aggiungere una UI per la revoca e la rotazione delle chiavi.

---

<div id="security-notes">
  # Note sulla sicurezza
</div>

- Ruoli e liste di autorizzati sono applicati al perimetro del Gateway.
- Nessun client ottiene accesso “completo” alle API senza scope operatore.
- Abbinamento richiesto per *tutte* le connessioni.
- TLS + pinning riducono il rischio di attacchi MITM sui dispositivi mobili.
- L'approvazione silenziosa via SSH è una comodità; rimane comunque registrata e può essere revocata.
- La discovery non è mai una radice di fiducia.
- Le dichiarazioni di funzionalità vengono verificate rispetto alle liste di autorizzati del server in base a piattaforma/tipo.

<div id="streaming-large-payloads-node-media">
  # Streaming + payload di grandi dimensioni (media del nodo)
</div>

Il piano di controllo WS va bene per i messaggi piccoli, ma i nodi gestiscono anche:

- clip della fotocamera
- registrazioni dello schermo
- stream audio

Opzioni:

1) Frame binari WS + chunking + regole di backpressure.
2) Endpoint di streaming separato (sempre TLS + auth).
3) Mantenere il bridge attivo più a lungo per i comandi ad alto contenuto multimediale, migrarlo per ultimo.

Scegline una prima di implementare per evitare disallineamenti.

<div id="capability-command-policy">
  # Criteri per funzionalità e comandi
</div>

- Le funzionalità e i comandi segnalati dal nodo sono trattati come **asserzioni**.
- Il Gateway applica liste di autorizzati specifiche per piattaforma.
- Qualsiasi nuovo comando richiede l'approvazione dell'operatore o una modifica esplicita alla lista di autorizzati.
- Registra le modifiche con marca temporale.

<div id="audit-rate-limiting">
  # Audit + limitazione della frequenza
</div>

- Registra nel log: richieste di abbinamento, approvazioni/rifiuti, emissione/rotazione/revocazione dei token.
- Applica una limitazione di frequenza allo spam di richieste di abbinamento e alle richieste di approvazione.

<div id="protocol-hygiene">
  # Igiene del protocollo
</div>

- Versione esplicita del protocollo + codici di errore.
- Regole di riconnessione + policy di heartbeat.
- TTL di presenza e semantica dell’ultima presenza rilevata.

---

<div id="open-questions">
  # Questioni aperte
</div>

1) Singolo dispositivo con entrambi i ruoli: modello di token
   - Si raccomandano token separati per ruolo (nodo vs operatore).
   - Stesso deviceId; scope diversi; revoca più chiara.

2) Granularità dello scope dell’operatore
   - read/write/admin + approvals + abbinamento (minimo funzionale).
   - Valutare in seguito scope per singola funzionalità.

3) UX di rotazione + revoca dei token
   - Rotazione automatica al cambio di ruolo.
   - UI per revocare in base a deviceId + ruolo.

4) Discovery
   - Estendere l’attuale Bonjour TXT per includere fingerprint TLS WS + suggerimenti sul ruolo.
   - Usarli solo come indizi di localizzazione.

5) Approvazione tra reti
   - Inviare un broadcast a tutti i client dell’operatore; la UI attiva mostra una finestra modale.
   - Vince la prima risposta; il Gateway garantisce l’atomicità.

---

<div id="summary-tldr">
  # Riepilogo (TL;DR)
</div>

- Oggi: piano di controllo WS + trasporto nodo Bridge.
- Problema: approvazioni + duplicazione + due stack separati.
- Proposta: un unico protocollo WS con ruoli + scope espliciti, abbinamento unificato + pinning TLS, approvazioni gestite dal Gateway, ID dispositivo stabili + simpatici slug.
- Risultato: UX più semplice, sicurezza più robusta, meno duplicazione, instradamento migliore su dispositivi mobili.