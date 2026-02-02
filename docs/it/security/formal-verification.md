---
title: Verifica formale (modelli di sicurezza)
summary: Modelli di sicurezza verificati automaticamente per i percorsi più a rischio di OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Verifica formale (modelli di sicurezza)
</div>

Questa pagina documenta i **modelli di sicurezza formali** di OpenClaw (TLA+/TLC al momento; altri saranno aggiunti se necessario).

> Nota: alcuni link meno recenti potrebbero fare riferimento al precedente nome del progetto.

**Obiettivo (stella polare):** fornire un’argomentazione verificata automaticamente che dimostri che OpenClaw applica la
propria politica di sicurezza prevista (autorizzazione, isolamento delle sessioni, controllo degli strumenti
e sicurezza rispetto a errori di configurazione), sotto assunzioni esplicite.

**Che cos’è (oggi):** una **suite di regressione di sicurezza**, eseguibile e guidata dall’attaccante:

- Ogni asserzione dispone di una verifica del modello (model checking) eseguibile su uno spazio di stati finito.
- Molte asserzioni hanno un **modello negativo** abbinato che produce una traccia di controesempio per una classe realistica di bug.

**Che cosa non è (ancora):** una dimostrazione che “OpenClaw è sicuro sotto ogni profilo” o che l’intera implementazione TypeScript è corretta.

<div id="where-the-models-live">
  ## Dove si trovano i modelli
</div>

I modelli sono mantenuti in un repository separato: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Avvertenze importanti
</div>

- Si tratta di **modelli**, non dell'implementazione completa in TypeScript. È possibile che vi sia discrepanza tra modello e codice.
- I risultati sono limitati dallo spazio degli stati esplorato da TLC; un “verde” non implica sicurezza oltre le ipotesi e i limiti del modello.
- Alcune affermazioni si basano su ipotesi esplicite sull'ambiente (ad esempio, deployment corretto, parametri di configurazione corretti).

<div id="reproducing-results">
  ## Riproduzione dei risultati
</div>

Oggi i risultati vengono riprodotti clonando in locale il repository dei modelli ed eseguendo TLC (vedi sotto). Una futura iterazione potrebbe offrire:

* modelli eseguiti in CI con artefatti pubblici (tracce di controesempio, log di esecuzione)
* un workflow ospitato “esegui questo modello” per verifiche di piccole dimensioni e con limiti ben definiti

Per iniziare:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Richiesto Java 11+ (TLC viene eseguito su JVM).
# Il repository include una versione bloccata di `tla2tools.jar` (strumenti TLA+) e fornisce `bin/tlc` + target Make.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Esposizione del Gateway e errata configurazione di un gateway aperto
</div>

**Affermazione:** mettere il servizio in ascolto su interfacce diverse dal loopback senza autenticazione può rendere possibile una compromissione remota e aumenta l'esposizione; token/password bloccano gli attaccanti non autenticati (in base alle assunzioni del modello).

- Esecuzioni verdi:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rosso (atteso):
  - `make gateway-exposure-v2-negative`

Vedi anche: `docs/gateway-exposure-matrix.md` nel repository dei modelli.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Pipeline di nodes.run (capacità a rischio massimo)
</div>

**Assunto:** `nodes.run` richiede (a) una lista di autorizzati per i comandi del nodo, oltre ai comandi dichiarati, e (b) un'approvazione live quando così configurato; le approvazioni sono tokenizzate per prevenire attacchi di replay (nel modello).

- Esecuzioni verdi:
  - `make nodes-pipeline`
  - `make approvals-token`
- Rosse (attese):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Store di abbinamento (DM gating)
</div>

**Assunto:** le richieste di abbinamento rispettano il TTL e i limiti per le richieste in sospeso.

- Esecuzioni riuscite (verde):
  - `make pairing`
  - `make pairing-cap`
- Esecuzioni non riuscite (rosso, previsto):
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress gating (menzioni + bypass dei comandi di controllo)
</div>

**Affermazione:** in contesti di gruppo che richiedono una menzione, un “comando di controllo” non autorizzato non può eludere il filtro sulle menzioni.

- Verde:
  - `make ingress-gating`
- Rosso (atteso):
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Isolamento delle chiavi di routing/sessione
</div>

**Affermazione:** I DM provenienti da peer distinti non confluiscono nella stessa sessione a meno che non siano esplicitamente collegati/configurati.

- Verde:
  - `make routing-isolation`
- Rosso (atteso):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: modelli limitati aggiuntivi (concorrenza, retry, correttezza delle tracce di esecuzione)
</div>

Questi sono modelli successivi che migliorano la fedeltà rispetto alle modalità di guasto del mondo reale (aggiornamenti non atomici, retry e fan-out dei messaggi).

<div id="pairing-store-concurrency-idempotency">
  ### Concorrenza / idempotenza dello store di abbinamento
</div>

**Affermazione:** uno store di abbinamento deve applicare `MaxPending` e garantire l’idempotenza anche in presenza di interleaving (cioè l’operazione “check-then-write” deve essere atomica / bloccata; il refresh non deve creare duplicati).

Cosa significa:

- In presenza di richieste concorrenti, non puoi superare `MaxPending` per un canale.
- Richieste/refresh ripetuti per la stessa coppia `(channel, sender)` non devono creare righe pendenti attive duplicate.

- Esecuzioni verdi:
  - `make pairing-race` (controllo del limite atomico/bloccato)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Rosso (atteso):
  - `make pairing-race-negative` (race sul limite con begin/commit non atomici)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Correlazione delle tracce in ingresso / idempotenza
</div>

**Affermazione:** l'ingestione deve preservare la correlazione delle tracce durante il fan-out ed essere idempotente in caso di retry da parte del provider.

Cosa significa:

- Quando un evento esterno genera più messaggi interni, ogni parte mantiene la stessa identità di traccia/evento.
- I retry non devono comportare un'elaborazione doppia.
- Se gli ID degli eventi del provider non sono disponibili, la deduplicazione utilizza come fallback una chiave sicura (ad esempio il trace ID) per evitare di scartare eventi distinti.

- Verde:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rosso (previsto):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Precedenza di routing dmScope + identityLinks
</div>

**Affermazione:** il routing deve mantenere le sessioni DM isolate per impostazione predefinita e deve accorpare le sessioni solo quando è configurato in modo esplicito (precedenza del canale + identity links).

Cosa significa:

- Gli override dmScope specifici del canale devono avere la precedenza sui valori predefiniti globali.
- identityLinks deve accorpare le sessioni solo all'interno di gruppi esplicitamente collegati, non tra peer non correlati.

- Verde:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rosso (previsto):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`