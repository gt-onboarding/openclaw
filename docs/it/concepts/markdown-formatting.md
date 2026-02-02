---
title: Formattazione Markdown
summary: "Pipeline di formattazione Markdown per i canali in uscita"
read_when:
  - Stai modificando la formattazione Markdown o il chunking per i canali in uscita
  - Stai aggiungendo un nuovo formatter di canale o una mappatura degli stili
  - Stai eseguendo il debug di regressioni di formattazione tra i vari canali
---

<div id="markdown-formatting">
  # Formattazione Markdown
</div>

OpenClaw formatta il Markdown in uscita convertendolo in una rappresentazione
intermedia (IR) condivisa prima di generare l&#39;output specifico per ogni canale.
L&#39;IR mantiene il testo originale intatto associando gli intervalli di stile/link,
così che la suddivisione in chunk e il rendering rimangano coerenti su tutti i canali.

<div id="goals">
  ## Obiettivi
</div>

* **Coerenza:** una sola fase di parsing, più renderer.
* **Suddivisione sicura:** suddividi il testo prima del rendering in modo che la formattazione inline non venga mai spezzata tra i blocchi.
* **Adattamento ai canali:** mappa la stessa IR su Slack mrkdwn, HTML di Telegram e intervalli di stile di Signal senza dover ripetere il parsing del Markdown.

<div id="pipeline">
  ## Pipeline
</div>

1. **Esegui il parsing di Markdown -&gt; IR**
   * L&#39;IR è testo semplice più segmenti di stile (grassetto/corsivo/barrato/codice/spoiler) e segmenti di link.
   * Gli offset sono unità di codice UTF-16, in modo che gli intervalli di stile di Signal siano allineati con la sua API.
   * Le tabelle vengono analizzate solo quando un canale abilita esplicitamente la conversione delle tabelle.
2. **Suddividi l&#39;IR in chunk (format-first)**
   * La suddivisione in chunk avviene sul testo dell&#39;IR prima del rendering.
   * La formattazione inline non viene spezzata tra chunk; i segmenti vengono sezionati per chunk.
3. **Rendering per canale**
   * **Slack:** token mrkdwn (grassetto/corsivo/barrato/codice), link come `<url|label>`.
   * **Telegram:** tag HTML (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   * **Signal:** testo semplice + intervalli `text-style`; i link diventano `label (url)` quando l&#39;etichetta è diversa dall&#39;URL.

<div id="ir-example">
  ## Esempio di IR
</div>

Markdown di input:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (schema):

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }
  ]
}
```

<div id="where-it-is-used">
  ## Dove viene utilizzato
</div>

* Gli adattatori in uscita per Slack, Telegram e Signal eseguono il rendering dall&#39;IR.
* Altri canali (WhatsApp, iMessage, MS Teams, Discord) utilizzano ancora testo semplice oppure
  le proprie regole di formattazione, con la conversione delle tabelle Markdown applicata prima
  del chunking quando è abilitata.

<div id="table-handling">
  ## Gestione delle tabelle
</div>

Le tabelle Markdown non sono supportate in modo coerente tra i vari client di chat. Usa
`markdown.tables` per controllare la conversione per canale (e per account).

* `code`: visualizza le tabelle come blocchi di codice (valore predefinito per la maggior parte dei canali).
* `bullets`: converte ogni riga in punti elenco (valore predefinito per Signal + WhatsApp).
* `off`: disabilita l&#39;analisi e la conversione delle tabelle; il testo grezzo della tabella passa attraverso invariato.

Chiavi di configurazione:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

<div id="chunking-rules">
  ## Regole di suddivisione in chunk
</div>

* I limiti dei chunk provengono dagli adapter dei canali e dalla relativa configurazione e vengono applicati al testo IR.
* I blocchi di codice racchiusi tra fence vengono preservati come un singolo blocco con un a capo finale, così che i canali
  li renderizzino correttamente.
* I prefissi di elenco e i prefissi di citazione fanno parte del testo IR, quindi la suddivisione in chunk
  non avviene a metà prefisso.
* Gli stili in linea (grassetto/corsivo/barrato/codice in linea/spoiler) non vengono mai spezzati tra
  chunk; il renderer riapre gli stili all&#39;interno di ogni chunk.

Per maggiori dettagli sul comportamento di suddivisione in chunk tra canali, vedi
[Streaming + chunking](/it/concepts/streaming).

<div id="link-policy">
  ## Policy sui link
</div>

* **Slack:** `[label](url)` -&gt; `<url|label>`; gli URL in chiaro rimangono tali. L&#39;autolink
  è disabilitato in fase di analisi per evitare doppi link.
* **Telegram:** `[label](url)` -&gt; `<a href="url">label</a>` (modalità di analisi HTML).
* **Signal:** `[label](url)` -&gt; `label (url)` a meno che l&#39;etichetta coincida con l&#39;URL.

<div id="spoilers">
  ## Spoiler
</div>

I marcatori spoiler (`||spoiler||`) vengono interpretati solo per Signal, dove
vengono mappati a intervalli di stile SPOILER. Gli altri canali li trattano come testo normale.

<div id="how-to-add-or-update-a-channel-formatter">
  ## Come aggiungere o aggiornare un formatter di canale
</div>

1. **Effettua il parsing una sola volta:** usa l&#39;helper condiviso `markdownToIR(...)` con opzioni
   appropriate al canale (autolink, stile dei titoli, prefisso dei blocchi di citazione).
2. **Esegui il rendering:** implementa un renderer con `renderMarkdownWithMarkers(...)` e una
   mappa dei marker di stile (o intervalli di stile Signal).
3. **Suddividi in chunk:** chiama `chunkMarkdownIR(...)` prima del rendering; esegui il rendering di ogni chunk.
4. **Collega l&#39;adapter:** aggiorna l&#39;adapter in uscita del canale per usare il nuovo chunker
   e renderer.
5. **Esegui i test:** aggiungi o aggiorna i test di formattazione e un test di consegna in uscita se il
   canale usa il chunking.

<div id="common-gotchas">
  ## Problemi comuni
</div>

* I token di Slack tra parentesi angolari (`<@U123>`, `<#C123>`, `<https://...>`) devono
  essere preservati; esegui in modo sicuro l&#39;escape del codice HTML grezzo.
* L&#39;HTML di Telegram richiede l&#39;escape del testo al di fuori dei tag per evitare markup non valido.
* Gli intervalli in stile Signal si basano su offset UTF-16; non usare offset in code point.
* Conserva le newline finali per i blocchi di codice delimitati, in modo che i marcatori di chiusura
  si trovino sulla propria riga.