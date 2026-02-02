---
title: Test
summary: "Come eseguire i test in locale (vitest) e quando usare le modalità force/coverage"
read_when:
  - Eseguire o correggere i test
---

<div id="tests">
  # Test
</div>

* Kit di test completo (suite, live, Docker): [Testing](/it/testing)

* `pnpm test:force`: termina qualsiasi processo gateway residuo che occupa la porta di controllo predefinita, quindi esegue l’intera suite Vitest con una porta gateway isolata, in modo che i test del server non entrino in conflitto con un’istanza già in esecuzione. Usa questo comando quando una precedente esecuzione del gateway ha lasciato la porta 18789 occupata.

* `pnpm test:coverage`: esegue Vitest con copertura V8. Le soglie globali sono 70% per linee/branch/funzioni/statement. La copertura esclude gli entry point con molte integrazioni (wiring della CLI, bridge gateway/telegram, server statico webchat) per mantenere il target focalizzato sulla logica testabile tramite unit test.

* `pnpm test:e2e`: esegue smoke test end-to-end del gateway (abbinamento multi-istanza WS/HTTP/nodo).

* `pnpm test:live`: esegue i test live dei provider (minimax/zai). Richiede chiavi API e `LIVE=1` (oppure `*_LIVE_TEST=1` specifico per provider) per evitare che i test vengano saltati.

<div id="model-latency-bench-local-keys">
  ## Benchmark della latenza dei modelli (chiavi locali)
</div>

Script: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Utilizzo:

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* Variabili d&#39;ambiente opzionali: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
* Prompt predefinito: &quot;Reply with a single word: ok. No punctuation or extra text.&quot;

Ultima esecuzione (2025-12-31, 20 esecuzioni):

* minimax latenza mediana 1279 ms (min 1114, max 2431)
* opus latenza mediana 2454 ms (min 1224, max 3170)

<div id="onboarding-e2e-docker">
  ## Onboarding E2E (Docker)
</div>

Docker è opzionale; serve solo per gli smoke test di onboarding containerizzati.

Flusso completo di cold start in un container Linux pulito:

```bash
scripts/e2e/onboard-docker.sh
```

Questo script gestisce la procedura guidata interattiva tramite uno pseudo-tty, verifica i file di configurazione/spazio di lavoro/sessione, quindi avvia il Gateway ed esegue `openclaw health`.

<div id="qr-import-smoke-docker">
  ## Smoke test per l&#39;import QR (Docker)
</div>

Verifica che `qrcode-terminal` venga caricato con Node 22+ in Docker:

```bash
pnpm test:docker:qr
```
