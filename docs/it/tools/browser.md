---
title: Browser
summary: "Servizio integrato di controllo del browser + comandi di azione"
read_when:
  - Aggiungere l'automazione del browser controllata da un agente
  - Eseguire il debug del motivo per cui openclaw interferisce con il tuo Chrome
  - Implementare le impostazioni e il ciclo di vita del browser nell'app macOS
---

<div id="browser-openclaw-managed">
  # Browser (gestito da openclaw)
</div>

OpenClaw può eseguire un **profilo dedicato di Chrome/Brave/Edge/Chromium** controllato dall&#39;agente.
È isolato dal tuo browser personale ed è gestito tramite un piccolo servizio locale
di controllo all&#39;interno del Gateway (solo su loopback).

Vista per principianti:

* Consideralo come un **browser separato, usato solo dall&#39;agente**.
* Il profilo `openclaw` **non** tocca il profilo del tuo browser personale.
* L&#39;agente può **aprire schede, leggere pagine, fare clic e digitare** in un contesto sicuro.
* Il profilo predefinito `chrome` usa il **browser Chromium predefinito di sistema** tramite il
  relay dell&#39;estensione; passa a `openclaw` per il browser isolato e gestito.

<div id="what-you-get">
  ## Cosa ottieni
</div>

* Un profilo del browser separato chiamato **openclaw** (con accento arancione predefinito).
* Controllo deterministico delle schede (elenca/apri/porta in primo piano/chiudi).
* Azioni dell&#39;agente (clic/digita/trascina/seleziona), snapshot, screenshot, PDF.
* Supporto opzionale multi‑profilo (`openclaw`, `work`, `remote`, ...).

Questo browser **non** è quello che usi ogni giorno. È un ambiente sicuro e isolato per
l&#39;automazione e la verifica da parte dell&#39;agente.

<div id="quick-start">
  ## Guida rapida
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Se visualizzi il messaggio «Browser disabled», abilitalo nella configurazione (vedi sotto) e riavvia il
Gateway.

<div id="profiles-openclaw-vs-chrome">
  ## Profili: `openclaw` vs `chrome`
</div>

* `openclaw`: browser gestito e isolato (nessuna estensione richiesta).
* `chrome`: relay tramite estensione verso il **browser di sistema** (richiede che l&#39;estensione OpenClaw
  sia agganciata a una scheda).

Imposta `browser.defaultProfile: "openclaw"` se vuoi la modalità gestita come profilo predefinito.

<div id="configuration">
  ## Configurazione
</div>

Le impostazioni del browser sono in `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true,                    // default: true
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    remoteCdpTimeoutMs: 1500,         // remote CDP HTTP timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // timeout handshake WebSocket CDP remoto (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

Note:

* Il servizio di controllo del browser si associa all&#39;interfaccia di loopback su una porta derivata da `gateway.port`
  (predefinita: `18791`, cioè gateway + 2). Il relay usa la porta successiva (`18792`).
* Se sovrascrivi la porta del Gateway (`gateway.port` o `OPENCLAW_GATEWAY_PORT`),
  le porte del browser derivate vengono spostate per rimanere nella stessa “famiglia”.
* `cdpUrl`, se non impostato, per impostazione predefinita usa la porta del relay.
* `remoteCdpTimeoutMs` si applica ai controlli di raggiungibilità CDP remoti (non loopback).
* `remoteCdpHandshakeTimeoutMs` si applica ai controlli di raggiungibilità CDP WebSocket remoti.
* `attachOnly: true` significa “non avviare mai un browser locale; collegati solo se è già in esecuzione.”
* `color` + `color` per profilo applicano una tinta alla UI del browser in modo che tu possa vedere quale profilo è attivo.
* Il profilo predefinito è `chrome` (relay tramite estensione). Usa `defaultProfile: "openclaw"` per il browser gestito.
* Ordine di auto-rilevamento: browser predefinito di sistema se basato su Chromium; altrimenti Chrome → Brave → Edge → Chromium → Chrome Canary.
* I profili `openclaw` locali assegnano automaticamente `cdpPort`/`cdpUrl` — imposta questi solo per CDP remoti.

<div id="use-brave-or-another-chromium-based-browser">
  ## Usa Brave (o un altro browser basato su Chromium)
</div>

Se il **browser predefinito del sistema** è basato su Chromium (Chrome/Brave/Edge/ecc.),
OpenClaw lo utilizza automaticamente. Imposta `browser.executablePath` per
sovrascrivere il rilevamento automatico:

Esempio CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## Controllo locale vs remoto
</div>

* **Controllo locale (predefinito):** il Gateway avvia il servizio di controllo loopback e può aprire un browser locale.
* **Controllo remoto (nodo host):** esegui un nodo host sulla macchina che ha il browser; il Gateway inoltra le azioni del browser a tale nodo.
* **CDP remoto:** imposta `browser.profiles.<name>.cdpUrl` (o `browser.cdpUrl`) per
  collegarti a un browser remoto basato su Chromium. In questo caso, OpenClaw non avvierà un browser locale.

Gli URL CDP remoti possono includere l&#39;autenticazione:

* Token nella stringa di query (ad esempio, `https://provider.example?token=<token>`)
* Autenticazione HTTP Basic (ad esempio, `https://user:pass@provider.example`)

OpenClaw preserva l&#39;autenticazione quando chiama gli endpoint `/json/*` e quando si connette
al WebSocket CDP. Usa preferibilmente variabili d&#39;ambiente o servizi di gestione dei secret per i token, invece di inserirli nei file di configurazione.

<div id="node-browser-proxy-zero-config-default">
  ## Proxy del browser del nodo (impostazione predefinita zero-config)
</div>

Se esegui un **nodo host** sulla macchina su cui gira il tuo browser, OpenClaw può
instradare automaticamente le chiamate degli strumenti del browser a quel nodo senza alcuna configurazione aggiuntiva del browser.
Questo è il percorso predefinito per i gateway remoti.

Note:

* Il nodo host espone il proprio server locale di controllo del browser tramite un **comando proxy**.
* I profili provengono dalla configurazione `browser.profiles` del nodo stesso (come in locale).
* Disabilitalo se non lo vuoi usare:
  * Sul nodo: `nodeHost.browserProxy.enabled=false`
  * Sul gateway: `gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless (CDP remoto ospitato)
</div>

[Browserless](https://browserless.io) è un servizio Chromium ospitato che espone
endpoint CDP tramite HTTPS. Puoi configurare un profilo browser di OpenClaw per usare un
endpoint di regione Browserless e autenticarti con la tua chiave API.

Esempio:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

Note:

* Sostituisci `<BROWSERLESS_API_KEY>` con il tuo token Browserless effettivo.
* Scegli l&#39;endpoint di regione che corrisponde al tuo account Browserless (consulta la loro documentazione).

<div id="security">
  ## Sicurezza
</div>

Concetti chiave:

* Il controllo del browser è limitato al loopback; l’accesso passa tramite l’autenticazione del Gateway o l’abbinamento del nodo.
* Mantieni il Gateway e tutti gli host dei nodi su una rete privata (Tailscale); evita l’esposizione pubblica.
* Tratta gli URL/token CDP remoti come segreti; preferisci variabili d’ambiente o un gestore di segreti.

Suggerimenti per CDP remoti:

* Preferisci endpoint HTTPS e token di breve durata quando possibile.
* Evita di incorporare direttamente nei file di configurazione token di lunga durata.

<div id="profiles-multi-browser">
  ## Profili (multi-browser)
</div>

OpenClaw supporta più profili denominati (configurazioni di instradamento). I profili possono essere:

* **openclaw-managed**: un&#39;istanza dedicata di browser basato su Chromium con la propria directory dei dati utente + porta CDP
* **remote**: un URL CDP esplicito (browser basato su Chromium in esecuzione altrove)
* **extension relay**: le tue schede Chrome esistenti tramite il relay locale + estensione Chrome

Impostazioni predefinite:

* Il profilo `openclaw` viene creato automaticamente se non esiste.
* Il profilo `chrome` è integrato per l&#39;extension relay di Chrome (punta a `http://127.0.0.1:18792` per impostazione predefinita).
* Le porte CDP locali vengono allocate nell&#39;intervallo **18800–18899** per impostazione predefinita.
* L&#39;eliminazione di un profilo sposta la sua directory dati locale nel Cestino.

Tutti gli endpoint di controllo accettano `?profile=<name>`; la CLI usa `--browser-profile`.

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Relay dell&#39;estensione Chrome (usa il tuo Chrome esistente)
</div>

OpenClaw può anche controllare **le tue schede Chrome esistenti** (nessuna istanza di Chrome “openclaw” separata) tramite un relay locale CDP + un&#39;estensione Chrome.

Guida completa: [Chrome extension](/it/tools/chrome-extension)

Flusso:

* Il Gateway viene eseguito in locale (stessa macchina) oppure un host del nodo viene eseguito sulla macchina del browser.
* Un **server di relay** locale resta in ascolto su un `cdpUrl` di loopback (predefinito: `http://127.0.0.1:18792`).
* Fai clic sull&#39;icona dell&#39;estensione **OpenClaw Browser Relay** su una scheda per collegarla (non si collega automaticamente).
* L&#39;agente controlla quella scheda tramite il normale strumento `browser`, selezionando il profilo corretto.

Se il Gateway viene eseguito altrove, esegui un host del nodo sulla macchina del browser affinché il Gateway possa fare da proxy per le azioni del browser.

<div id="sandboxed-sessions">
  ### Sessioni in sandbox
</div>

Se la sessione dell&#39;agente è in sandbox, lo strumento `browser` potrebbe usare per impostazione predefinita `target="sandbox"` (browser in sandbox).
La presa di controllo del relay dell&#39;estensione Chrome richiede il controllo del browser host, quindi:

* esegui la sessione senza sandbox, oppure
* imposta `agents.defaults.sandbox.browser.allowHostControl: true` e usa `target="host"` quando chiami lo strumento.

<div id="setup">
  ### Configurazione
</div>

1. Carica l&#39;estensione (dev/unpacked):

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → abilita “Modalità sviluppatore”
* “Carica estensione non pacchettizzata” → seleziona la directory stampata da `openclaw browser extension path`
* Appunta l&#39;estensione, quindi cliccala sulla scheda che vuoi controllare (il badge mostra `ON`).

2. Usalo:

* CLI: `openclaw browser --browser-profile chrome tabs`
* Strumento agente: `browser` con `profile="chrome"`

Opzionale: se vuoi un nome o una porta di inoltro diversi, crea un tuo profilo:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Note:

* Questa modalità si basa su Playwright-on-CDP per la maggior parte delle operazioni (screenshot/istantanee/azioni).
* Scollega facendo nuovamente clic sull&#39;icona dell&#39;estensione.

<div id="isolation-guarantees">
  ## Garanzie di isolamento
</div>

* **Cartella dati utente dedicata**: non modifica mai il tuo profilo personale del browser.
* **Porte dedicate**: evita `9222` per prevenire conflitti con i flussi di lavoro di sviluppo.
* **Controllo deterministico delle schede**: individua le schede tramite `targetId`, non l’“ultima scheda”.

<div id="browser-selection">
  ## Selezione del browser
</div>

Quando viene avviato in locale, OpenClaw sceglie il primo browser disponibile:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Puoi sovrascrivere questa scelta con `browser.executablePath`.

Piattaforme:

* macOS: controlla `/Applications` e `~/Applications`.
* Linux: cerca `google-chrome`, `brave`, `microsoft-edge`, `chromium`, ecc.
* Windows: controlla i percorsi di installazione più comuni.

<div id="control-api-optional">
  ## API di controllo (opzionale)
</div>

Solo per integrazioni locali, il Gateway espone una piccola API HTTP di loopback:

* Stato/avvio/arresto: `GET /`, `POST /start`, `POST /stop`
* Schede: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* Istantanea/screenshot: `GET /snapshot`, `POST /screenshot`
* Azioni: `POST /navigate`, `POST /act`
* Hook: `POST /hooks/file-chooser`, `POST /hooks/dialog`
* Download: `POST /download`, `POST /wait/download`
* Debugging: `GET /console`, `POST /pdf`
* Debugging: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* Rete: `POST /response/body`
* Stato: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* Stato: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* Impostazioni: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Tutti gli endpoint accettano il parametro `?profile=<name>`.

<div id="playwright-requirement">
  ### Requisiti Playwright
</div>

Alcune funzionalità (navigate/act/AI snapshot/role snapshot, screenshot degli elementi, PDF) richiedono
Playwright. Se Playwright non è installato, quegli endpoint restituiscono un errore 501 esplicito.
Gli snapshot ARIA e gli screenshot di base continuano a funzionare per Chrome gestito da openclaw.
Per il driver relay dell’estensione Chrome, gli snapshot ARIA e gli screenshot richiedono Playwright.

Se visualizzi `Playwright is not available in this gateway build`, installa il pacchetto
Playwright completo (non `playwright-core`) e riavvia il Gateway, oppure reinstalla
OpenClaw con il supporto per il browser.

<div id="how-it-works-internal">
  ## Come funziona (interno)
</div>

Flusso generale:

* Un piccolo **server di controllo** accetta richieste HTTP.
* Si connette a browser basati su Chromium (Chrome/Brave/Edge/Chromium) tramite **CDP**.
* Per azioni avanzate (click/digitazione/istantanea/PDF), usa **Playwright** sopra
  CDP.
* Quando Playwright non è disponibile, sono disponibili solo operazioni che non usano Playwright.

Questo design mantiene l&#39;agente ancorato a un&#39;interfaccia stabile e deterministica, permettendoti
al tempo stesso di sostituire browser e profili locali/remoti.

<div id="cli-quick-reference">
  ## Riferimento rapido CLI
</div>

Tutti i comandi accettano `--browser-profile <name>` per usare uno specifico profilo.
Tutti i comandi accettano anche `--json` per un output adatto all’elaborazione automatica (payload stabili).

Nozioni di base:

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

Ispezione:

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

Azioni:

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

Stato:

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

Note:

* `upload` e `dialog` sono chiamate di **arming**; eseguili prima del clic/pressione
  che attiva il selettore/finestra di dialogo.
* `upload` può anche impostare direttamente gli input file tramite `--input-ref` o `--element`.
* `snapshot`:
  * `--format ai` (predefinito quando Playwright è installato): restituisce uno snapshot AI con riferimenti numerici (`aria-ref="<n>"`).
  * `--format aria`: restituisce l&#39;albero di accessibilità (senza riferimenti; solo per ispezione).
  * `--efficient` (o `--mode efficient`): preset compatto per snapshot basati sui ruoli (interattivo + compatto + profondità + maxChars inferiore).
  * Impostazione di configurazione predefinita (solo tool/CLI): imposta `browser.snapshotDefaults.mode: "efficient"` per usare snapshot efficienti quando il chiamante non specifica una modalità (vedi [Gateway configuration](/it/gateway/configuration#browser-openclaw-managed-browser)).
  * Le opzioni di snapshot per ruoli (`--interactive`, `--compact`, `--depth`, `--selector`) forzano uno snapshot basato sui ruoli con riferimenti come `ref=e12`.
  * `--frame "<iframe selector>"` restringe gli snapshot per ruoli a un iframe (associato a riferimenti di ruolo come `e12`).
  * `--interactive` produce un elenco piatto e facilmente selezionabile di elementi interattivi (ideale per pilotare le azioni).
  * `--labels` aggiunge uno screenshot limitato alla viewport con etichette di riferimento sovrapposte (stampa `MEDIA:<path>`).
* `click`/`type`/ecc. richiedono un `ref` da `snapshot` (sia numerico `12` sia riferimento di ruolo `e12`).
  I selettori CSS non sono supportati di proposito per le azioni.

<div id="snapshots-and-refs">
  ## Snapshot e ref
</div>

OpenClaw supporta due stili di “snapshot”:

* **Snapshot AI (ref numeriche)**: `openclaw browser snapshot` (predefinito; `--format ai`)
  * Output: uno snapshot testuale che include ref numeriche.
  * Azioni: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  * Internamente, la ref viene risolta tramite `aria-ref` di Playwright.

* **Snapshot per ruolo (ref di ruolo come `e12`)**: `openclaw browser snapshot --interactive` (oppure `--compact`, `--depth`, `--selector`, `--frame`)
  * Output: un elenco/albero basato sui ruoli con `[ref=e12]` (ed eventualmente `[nth=1]`).
  * Azioni: `openclaw browser click e12`, `openclaw browser highlight e12`.
  * Internamente, la ref viene risolta tramite `getByRole(...)` (più `nth()` per i duplicati).
  * Aggiungi `--labels` per includere uno screenshot del viewport con etichette `e12` sovrapposte.

Comportamento delle ref:

* Le ref **non sono stabili tra una navigazione e l’altra**; se qualcosa fallisce, riesegui `snapshot` e usa una nuova ref.
* Se lo snapshot per ruolo è stato acquisito con `--frame`, le ref di ruolo restano limitate a quell’iframe fino al successivo snapshot per ruolo.

<div id="wait-power-ups">
  ## Funzionalità avanzate di attesa
</div>

Puoi attendere in base non solo al tempo o al testo:

* Attendi un URL (pattern glob supportati da Playwright):
  * `openclaw browser wait --url "**/dash"`
* Attendi uno stato di caricamento:
  * `openclaw browser wait --load networkidle`
* Attendi un predicato JS:
  * `openclaw browser wait --fn "window.ready===true"`
* Attendi che un selettore diventi visibile:
  * `openclaw browser wait "#main"`

Queste opzioni possono essere combinate:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## Workflow di debug
</div>

Quando un&#39;azione non riesce (ad es. &quot;non visibile&quot;, &quot;violazione della modalità strict&quot;, &quot;coperto&quot;):

1. `openclaw browser snapshot --interactive`
2. Usa `click <ref>` / `type <ref>` (preferisci i ref di ruolo in modalità interattiva)
3. Se non funziona ancora: `openclaw browser highlight <ref>` per vedere quale elemento sta puntando Playwright
4. Se la pagina si comporta in modo anomalo:
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. Per un debug più approfondito: registra una traccia (trace):
   * `openclaw browser trace start`
   * riproduci il problema
   * `openclaw browser trace stop` (mostra `TRACE:<path>`)

<div id="json-output">
  ## Output JSON
</div>

`--json` è pensato per l&#39;uso in script e strumenti strutturati.

Esempi:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Gli snapshot dei ruoli in JSON includono `refs` e un piccolo blocco `stats` (righe/caratteri/riferimenti/interactive), così che gli strumenti possano ragionare sulla dimensione e sulla densità del payload.

<div id="state-and-environment-knobs">
  ## Controlli di stato e ambiente
</div>

Questi sono utili per i workflow del tipo “fai comportare il sito come X”:

* Cookie: `cookies`, `cookies set`, `cookies clear`
* Storage: `storage local|session get|set|clear`
* Offline: `set offline on|off`
* Header: `set headers --json '{"X-Debug":"1"}'` (oppure `--clear`)
* Autenticazione HTTP basic: `set credentials user pass` (oppure `--clear`)
* Geolocalizzazione: `set geo <lat> <lon> --origin "https://example.com"` (oppure `--clear`)
* Media: `set media dark|light|no-preference|none`
* Fuso orario / locale: `set timezone ...`, `set locale ...`
* Dispositivo / viewport:
  * `set device "iPhone 14"` (preset di dispositivi di Playwright)
  * `set viewport 1280 720`

<div id="security-privacy">
  ## Sicurezza e privacy
</div>

* Il profilo del browser openclaw può contenere sessioni attive; trattalo come sensibile.
* `browser act kind=evaluate` / `openclaw browser evaluate` e `wait --fn`
  eseguono JavaScript arbitrario nel contesto della pagina. La prompt injection può indirizzare questo comportamento. Disabilita questa funzione con `browser.evaluateEnabled=false` se non ti serve.
* Per informazioni su login e meccanismi anti‑bot (X/Twitter, ecc.), vedi [Browser login + X/Twitter posting](/it/tools/browser-login).
* Mantieni privato l&#39;host del Gateway/nodo (solo loopback o solo tailnet).
* Gli endpoint CDP remoti sono molto potenti; instradali tramite un tunnel e proteggili adeguatamente.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

Per problemi specifici su Linux (in particolare con Chromium installato tramite snap), consulta
[Risoluzione dei problemi del browser](/it/tools/browser-linux-troubleshooting).

<div id="agent-tools-how-control-works">
  ## Strumenti dell&#39;agente + come funziona il controllo
</div>

L&#39;agente dispone di **un solo strumento** per l&#39;automazione del browser:

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

Come funziona la mappatura:

* `browser snapshot` restituisce un albero UI stabile (AI o ARIA).
* `browser act` usa gli ID `ref` dello snapshot per fare clic/digitare/trascinare/selezionare.
* `browser screenshot` cattura i pixel (pagina intera o singolo elemento).
* `browser` accetta:
  * `profile` per scegliere un profilo browser nominato (openclaw, chrome o CDP remoto).
  * `target` (`sandbox` | `host` | `node`) per selezionare dove è in esecuzione il browser.
  * Nelle sessioni in sandbox, `target: "host"` richiede `agents.defaults.sandbox.browser.allowHostControl=true`.
  * Se `target` è omesso: le sessioni in sandbox usano per impostazione predefinita `sandbox`, le sessioni non in sandbox usano `host`.
  * Se è connesso un nodo con funzionalità browser, lo strumento può instradare automaticamente verso di esso a meno che tu non imposti esplicitamente `target="host"` o `target="node"`.

Questo mantiene l&#39;agente deterministico ed evita selettori fragili.