---
title: Verifica formale (modelli di sicurezza)
summary: Modelli di sicurezza verificati automaticamente tramite strumenti formali per i percorsi più critici di OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Verifica formale (modelli di sicurezza)
</div>

Questa pagina tiene traccia dei **modelli di sicurezza formali** di OpenClaw (oggi TLA+/TLC; altri all’occorrenza).

> Nota: alcuni link meno recenti possono fare riferimento al precedente nome del progetto.

**Obiettivo (stella polare):** fornire un argomento verificato automaticamente che dimostri che OpenClaw applica la
propria politica di sicurezza prevista (autorizzazione, isolamento delle sessioni, controllo degli strumenti e
sicurezza rispetto alla misconfigurazione), a fronte di assunzioni esplicite.

**Che cos’è (oggi):** una **suite di test di regressione di sicurezza** eseguibile e guidata dall’attaccante:

- Ogni asserzione ha una verifica del modello eseguibile su uno spazio di stati finito.
- Molte asserzioni hanno un **modello negativo** associato che produce una traccia di controesempio per una classe di bug realistica.

**Che cosa non è (ancora):** una dimostrazione che “OpenClaw è sicuro sotto ogni aspetto” o che l’intera implementazione TypeScript sia corretta.

<div id="where-the-models-live">
  ## Dove si trovano i modelli
</div>

I modelli sono gestiti in un repository separato: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Avvertenze importanti
</div>

- Questi sono **modelli**, non l'implementazione completa in TypeScript. È possibile che si verifichi un disallineamento tra modello e codice.
- I risultati sono limitati dallo spazio degli stati esplorato da TLC; lo stato “verde” non implica sicurezza oltre le assunzioni e i limiti modellati.
- Alcune conclusioni si basano su assunzioni ambientali esplicite (ad esempio, deployment corretto, input di configurazione corretti).

<div id="reproducing-results">
  ## Riprodurre i risultati
</div>

Oggi i risultati vengono riprodotti clonando localmente il repository dei modelli ed eseguendo TLC (vedi sotto). Una futura iterazione potrebbe offrire:

* modelli eseguiti in CI con artifacts pubblici (tracce di controesempi, log di esecuzione)
* un workflow ospitato &quot;esegui questo modello&quot; per verifiche piccole e con limiti ben definiti

Per iniziare:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Richiesto Java 11+ (TLC viene eseguito su JVM).
# Il repository include una versione bloccata di `tla2tools.jar` (strumenti TLA+) e fornisce `bin/tlc` + target Make.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Esposizione del Gateway ed errata configurazione di un gateway aperto
</div>

**Affermazione:** esporre il binding oltre l’interfaccia di loopback senza autenticazione può rendere possibile un compromesso remoto / aumenta l’esposizione; un token o una password blocca gli attaccanti non autenticati (in base alle assunzioni del modello).

- Esecuzioni verdi:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rosso (previsto):
  - `make gateway-exposure-v2-negative`

Vedi anche: `docs/gateway-exposure-matrix.md` nel repository dei modelli.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Pipeline di Nodes.run (funzionalità a rischio più elevato)
</div>

**Dichiarazione:** `nodes.run` richiede (a) una lista di autorizzati per i comandi del nodo più i comandi dichiarati e (b) un’approvazione in tempo reale quando configurato; le approvazioni sono tokenizzate per impedire attacchi di replay (nel modello).

- Esecuzioni “green” (attese come riuscite):
  - `make nodes-pipeline`
  - `make approvals-token`
- Esecuzioni “red” (fallimento previsto):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Archivio di abbinamento (DM gating)
</div>

**Verifica:** le richieste di abbinamento rispettano il TTL e il limite massimo di richieste in sospeso.

- Esecuzioni verdi:
  - `make pairing`
  - `make pairing-cap`
- Rosso (atteso):
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress gating (menzioni + bypass dei comandi di controllo)
</div>

**Affermazione:** in contesti di gruppo in cui è richiesta la menzione, un “comando di controllo” non autorizzato non può aggirare l’ingress gating basato sulla menzione.

- Verde:
  - `make ingress-gating`
- Rosso (atteso):
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Isolamento di routing/chiave di sessione
</div>

**Enunciato:** I DM da peer diversi non vengono aggregati nella stessa sessione a meno che non siano esplicitamente collegati o configurati.

- Verde:
  - `make routing-isolation`
- Rosso (atteso):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: modelli aggiuntivi con vincoli (concorrenza, retry, correttezza delle tracce di esecuzione)
</div>

Sono modelli successivi che affinano la fedeltà alle modalità di guasto nel mondo reale (aggiornamenti non atomici, retry e fan-out dei messaggi).

<div id="pairing-store-concurrency-idempotency">
  ### Concorrenza / idempotenza dello store di abbinamento
</div>

**Affermazione:** uno store di abbinamento deve applicare `MaxPending` e l'idempotenza anche in presenza di interleaving (cioè l'operazione "check-then-write" deve essere atomica / bloccata; un refresh non dovrebbe creare duplicati).

In pratica:

- In presenza di richieste concorrenti, non si deve superare `MaxPending` per un canale.
- Richieste/refresh ripetuti per la stessa coppia `(channel, sender)` non devono creare righe pendenti attive duplicate.

- Esecuzioni verdi:
  - `make pairing-race` (controllo del limite atomico/bloccato)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Rosso (previsto):
  - `make pairing-race-negative` (race condition sul limite con begin/commit non atomico)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Correlazione delle tracce di ingresso / idempotenza
</div>

**Affermazione:** l’ingestione deve preservare la correlazione delle tracce durante il fan-out ed essere idempotente in presenza di retry del provider.

Cosa significa:

- Quando un evento esterno diventa più messaggi interni, ogni parte mantiene la stessa identità di traccia/evento.
- I retry non devono portare a una doppia elaborazione.
- Se gli ID di evento del provider mancano, la deduplicazione usa come fallback una chiave sicura (ad es. l’ID di traccia) per evitare di scartare eventi distinti.

- Verde:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rosso (atteso):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Precedenza di routing dmScope + identityLinks
</div>

**Affermazione:** il routing deve mantenere le sessioni DM isolate per impostazione predefinita e deve unire le sessioni solo quando configurato in modo esplicito (precedenza del canale + identityLinks).

Cosa significa:

- Le impostazioni dmScope specifiche del canale devono prevalere sui valori globali predefiniti.
- identityLinks devono accorpare le sessioni solo all'interno di gruppi collegati esplicitamente, non tra peer non correlati.

- Verde:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rosso (previsto):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`