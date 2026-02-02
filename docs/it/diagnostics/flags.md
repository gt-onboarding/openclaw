---
title: Flag di diagnostica
summary: "Flag di diagnostica per generare log di debug mirati"
read_when:
  - Ti servono log di debug mirati senza aumentare il livello di logging globale
  - Devi acquisire log specifici di sottosistemi per il supporto
---

<div id="diagnostics-flags">
  # Flag di diagnostica
</div>

I flag di diagnostica consentono di abilitare log di debug mirati senza dover attivare il logging dettagliato ovunque. I flag sono opt-in e non hanno alcun effetto finché non vengono verificati da un sottosistema.

<div id="how-it-works">
  ## Come funziona
</div>

* I flag sono stringhe (non fanno distinzione tra maiuscole e minuscole).
* Puoi abilitare i flag nella configurazione o tramite un override con variabile d’ambiente.
* I caratteri jolly sono supportati:
  * `telegram.*` corrisponde a `telegram.http`
  * `*` abilita tutti i flag

<div id="enable-via-config">
  ## Abilitazione tramite configurazione
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Più flag:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Riavvia il Gateway dopo aver cambiato i flag.

<div id="env-override-one-off">
  ## Override temporaneo dell&#39;ambiente
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Disattiva tutti i flag:

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## Dove vengono scritti i log
</div>

I flag inviano i log al file di log di diagnostica standard. Per impostazione predefinita:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Se imposti `logging.file`, usa invece quel percorso. I log sono in formato JSONL (un oggetto JSON per riga). L&#39;offuscamento resta comunque attivo in base a `logging.redactSensitive`.

<div id="extract-logs">
  ## Estrai i log
</div>

Scegli il file di log più recente:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtro per la diagnostica HTTP di Telegram:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Oppure esegui tail mentre riproduci il problema:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Per i Gateway remoti, puoi anche utilizzare `openclaw logs --follow` (vedi [/cli/logs](/it/cli/logs)).

<div id="notes">
  ## Note
</div>

* Se `logging.level` è impostato a un valore superiore a `warn`, questi log potrebbero essere soppressi. Il valore predefinito `info` va bene.
* I flag possono essere lasciati abilitati senza problemi; influenzano solo il volume dei log per il sottosistema specifico.
* Usa [/logging](/it/logging) per modificare le destinazioni dei log, i livelli e l&#39;oscuramento dei dati.