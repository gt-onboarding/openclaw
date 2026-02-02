---
title: Configura
summary: "Riferimento alla CLI per `openclaw configure` (prompt di configurazione interattivi)"
read_when:
  - Vuoi modificare in modo interattivo credenziali, dispositivi o valori predefiniti degli agenti
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

Procedura guidata interattiva per configurare credenziali, dispositivi e valori predefiniti degli agenti.

Nota: la sezione **Model** ora include una selezione multipla per la lista di autorizzati `agents.defaults.models` (ossia ciò che viene mostrato in `/model` e nel selettore dei modelli).

Suggerimento: `openclaw config` senza un sottocomando apre la stessa procedura guidata. Usa
`openclaw config get|set|unset` per modifiche non interattive.

Correlato:

* Riferimento alla configurazione del Gateway: [Configuration](/it/gateway/configuration)
* CLI di configurazione: [Config](/it/cli/config)

Note:

* La scelta di dove eseguire il Gateway aggiorna sempre `gateway.mode`. Puoi selezionare &quot;Continue&quot; senza altre sezioni se è tutto ciò di cui hai bisogno.
* I servizi orientati ai canali (Slack/Discord/Matrix/Microsoft Teams) richiedono liste di autorizzati per canali/stanze durante la configurazione. Puoi inserire nomi o ID; la procedura guidata risolve i nomi in ID quando possibile.

<div id="examples">
  ## Esempi
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
