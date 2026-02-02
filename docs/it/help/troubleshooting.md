---
title: Risoluzione dei problemi
summary: "Hub per la risoluzione dei problemi: sintomi → verifiche → soluzioni"
read_when:
  - Vedi un errore e vuoi il percorso per risolverlo
  - L'installer indica “success” ma la CLI non funziona
---

<div id="troubleshooting">
  # Risoluzione dei problemi
</div>

<div id="first-60-seconds">
  ## Primi 60 secondi
</div>

Esegui questi passaggi in ordine:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Se il Gateway è raggiungibile, esegui una diagnostica approfondita:

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## Casi comuni in cui “si rompe tutto”
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

Si tratta quasi sempre di un problema di PATH di Node/npm. Inizia da qui:

* [Installazione (verifica PATH di Node/npm)](/it/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### L&#39;installer non va a buon fine (o ti servono i log completi)
</div>

Esegui nuovamente l&#39;installer in modalità dettagliata per visualizzare l&#39;intera traccia e l&#39;output di npm:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

Per le installazioni beta:

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

Puoi anche impostare `OPENCLAW_VERBOSE=1` invece di usare il flag.

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway “non autorizzato”, impossibile connettersi o si riconnette continuamente
</div>

* [Risoluzione dei problemi del Gateway](/it/gateway/troubleshooting)
* [Autenticazione del Gateway](/it/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Control UI non funziona tramite HTTP (richiede l&#39;identità del dispositivo)
</div>

* [Risoluzione dei problemi del Gateway](/it/gateway/troubleshooting)
* [Control UI](/it/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` mostra un errore SSL (Comcast/Xfinity)
</div>

Alcune connessioni Comcast/Xfinity bloccano l’accesso a `docs.openclaw.ai` tramite Xfinity Advanced Security.
Disabilita Advanced Security oppure aggiungi `docs.openclaw.ai` alla lista di autorizzati, quindi riprova.

* Guida a Xfinity Advanced Security: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* Controlli rapidi: prova con un hotspot mobile o una VPN per confermare che si tratti di filtraggio a livello di ISP

<div id="service-says-running-but-rpc-probe-fails">
  ### Il servizio risulta in esecuzione, ma il controllo RPC non va a buon fine
</div>

* [Risoluzione dei problemi del Gateway](/it/gateway/troubleshooting)
* [Processo in background / servizio](/it/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### Errori di modello/autenticazione (limite di richieste, fatturazione, «tutti i modelli sono falliti»)
</div>

* [Modelli](/it/cli/models)
* [Concetti OAuth/autenticazione](/it/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` dice `model not allowed`
</div>

Di solito significa che `agents.defaults.models` è configurato come lista di autorizzati. Quando non è vuota,
solo quelle chiavi provider/model possono essere selezionate.

* Controlla la lista di autorizzati: `openclaw config get agents.defaults.models`
* Aggiungi il modello che ti serve (o svuota la lista di autorizzati) e riprova `/model`
* Usa `/models` per esplorare i provider/modelli consentiti

<div id="when-filing-an-issue">
  ### Quando apri una segnalazione
</div>

Incolla un report privo di dati sensibili:

```bash
openclaw status --all
```

Se puoi, allega la coda dei log rilevante ottenuta con `openclaw logs --follow`.
