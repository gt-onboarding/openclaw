---
title: Plugin
summary: "Plugin/estensioni di OpenClaw: scoperta, configurazione e sicurezza"
read_when:
  - Quando aggiungi o modifichi plugin/estensioni
  - Quando documenti le regole di installazione o caricamento dei plugin
---

<div id="plugins-extensions">
  # Plugin (estensioni)
</div>

<div id="quick-start-new-to-plugins">
  ## Avvio rapido (nuovo ai plugin?)
</div>

Un plugin è semplicemente un **piccolo modulo di codice** che estende OpenClaw con
funzionalità aggiuntive (comandi, strumenti e RPC del Gateway).

Nella maggior parte dei casi userai i plugin quando ti serve una funzionalità che
non è ancora integrata nel core di OpenClaw (oppure vuoi tenere le funzionalità
opzionali fuori dall&#39;installazione principale).

Procedura rapida:

1. Verifica cosa è già caricato:

```bash
openclaw plugins list
```

2. Installa un plugin ufficiale (ad esempio: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

3. Riavvia il Gateway, quindi configura in `plugins.entries.<id>.config`.

Consulta [Voice Call](/it/plugins/voice-call) per un esempio concreto di plugin.

<div id="available-plugins-official">
  ## Plugin disponibili (ufficiali)
</div>

* Microsoft Teams è disponibile solo come plugin a partire dalla versione 2026.1.15; installa `@openclaw/msteams` se usi Teams.
* Memory (Core) — plugin di ricerca nella memoria fornito in bundle (abilitato per impostazione predefinita tramite `plugins.slots.memory`)
* Memory (LanceDB) — plugin di memoria a lungo termine fornito in bundle (richiamo/cattura automatici; imposta `plugins.slots.memory = "memory-lancedb"`)
* [Voice Call](/it/plugins/voice-call) — `@openclaw/voice-call`
* [Zalo Personal](/it/plugins/zalouser) — `@openclaw/zalouser`
* [Matrix](/it/channels/matrix) — `@openclaw/matrix`
* [Nostr](/it/channels/nostr) — `@openclaw/nostr`
* [Zalo](/it/channels/zalo) — `@openclaw/zalo`
* [Microsoft Teams](/it/channels/msteams) — `@openclaw/msteams`
* Google Antigravity OAuth (autenticazione provider) — fornito in bundle come `google-antigravity-auth` (disabilitato per impostazione predefinita)
* Gemini CLI OAuth (autenticazione provider) — fornito in bundle come `google-gemini-cli-auth` (disabilitato per impostazione predefinita)
* Qwen OAuth (autenticazione provider) — fornito in bundle come `qwen-portal-auth` (disabilitato per impostazione predefinita)
* Copilot Proxy (autenticazione provider) — bridge locale per VS Code Copilot Proxy; distinto dall’accesso tramite dispositivo integrato `github-copilot` (fornito in bundle, disabilitato per impostazione predefinita)

I plugin OpenClaw sono **moduli TypeScript** caricati a runtime tramite jiti. **La
validazione della configurazione non esegue il codice del plugin**; utilizza invece il manifest del plugin e lo schema JSON. Consulta [Plugin manifest](/it/plugins/manifest).

I plugin possono registrare:

* Metodi RPC del Gateway
* Handler HTTP del Gateway
* Strumenti dell’agente
* Comandi CLI
* Servizi in background
* Validazione opzionale della configurazione
* **Abilità** (elencando le directory `skills` nel manifest del plugin)
* **Comandi di risposta automatica** (eseguiti senza invocare l’agente AI)

I plugin vengono eseguiti **in‑process** con il Gateway, quindi trattali come codice attendibile.
Guida alla creazione di strumenti: [Plugin agent tools](/it/plugins/agent-tools).

<div id="runtime-helpers">
  ## Helper di runtime
</div>

I plugin possono accedere ad alcuni helper del core selezionati tramite `api.runtime`. Per il TTS per la telefonia:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Note:

* Usa la configurazione core `messages.tts` (OpenAI o ElevenLabs).
* Restituisce un buffer audio PCM + frequenza di campionamento. I plugin devono effettuare il resampling e la codifica per i provider.
* Edge TTS non è supportato per la telefonia.

<div id="discovery-precedence">
  ## Scoperta e precedenza
</div>

OpenClaw esegue la scansione, in quest&#39;ordine:

1. Percorsi di configurazione

* `plugins.load.paths` (file o directory)

2. Estensioni dello spazio di lavoro

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. Estensioni globali

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. Estensioni incluse nel pacchetto (fornite con OpenClaw, **disabilitate per impostazione predefinita**)

* `<openclaw>/extensions/*`

I plugin inclusi nel pacchetto devono essere abilitati esplicitamente tramite `plugins.entries.<id>.enabled`
oppure `openclaw plugins enable <id>`. I plugin installati sono abilitati per impostazione predefinita,
ma possono essere disabilitati nello stesso modo.

Ogni plugin deve includere un file `openclaw.plugin.json` nella propria directory radice. Se un percorso
punta a un file, la radice del plugin è la directory del file e deve contenere il manifest.

Se più plugin risultano avere lo stesso ID, prevale la prima corrispondenza nell&#39;ordine sopra
e le copie con precedenza inferiore vengono ignorate.

<div id="package-packs">
  ### Pack di pacchetti
</div>

La directory di un plugin può includere un `package.json` con `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Ogni voce diventa un plugin. Se il pacchetto elenca più estensioni, il plugin ID
diventa `name/<fileBase>`.

Se il tuo plugin importa dipendenze npm, installale in quella directory così che
`node_modules` sia disponibile (`npm install` / `pnpm install`).

<div id="channel-catalog-metadata">
  ### Metadati del catalogo dei canali
</div>

I plugin di canale possono esporre metadati relativi all&#39;onboarding tramite `openclaw.channel` e
suggerimenti di installazione tramite `openclaw.install`. Questo mantiene il catalogo principale libero da questi dati.

Esempio:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Chat self-hosted tramite bot webhook di Nextcloud Talk.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw può anche unire **cataloghi di canali esterni** (ad esempio
un’esportazione da un registry MPM). Posiziona un file JSON in una delle
seguenti posizioni:

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

Oppure imposta `OPENCLAW_PLUGIN_CATALOG_PATHS` (o `OPENCLAW_MPM_CATALOG_PATHS`)
su uno o più file JSON (separati da virgola, punto e virgola o secondo le
convenzioni di `PATH`). Ogni file deve contenere
`{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

<div id="plugin-ids">
  ## ID dei plugin
</div>

ID predefiniti dei plugin:

* Pacchetti: `name` in `package.json`
* File autonomo: nome di base del file (`~/.../voice-call.ts` → `voice-call`)

Se un plugin esporta `id`, OpenClaw lo utilizza ma mostra un avviso quando non corrisponde all&#39;ID configurato.

<div id="config">
  ## Configurazione
</div>

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } }
    }
  }
}
```

Campi:

* `enabled`: interruttore principale (predefinito: true)
* `allow`: lista di autorizzati (opzionale)
* `deny`: denylist (lista di esclusi; opzionale, `deny` ha la precedenza)
* `load.paths`: file/cartelle plugin aggiuntivi
* `entries.<id>`: interruttori + configurazione per plugin

Le modifiche alla configurazione **richiedono un riavvio del Gateway**.

Regole di validazione (rigorose):

* Gli ID di plugin sconosciuti in `entries`, `allow`, `deny` o `slots` sono **errori**.
* Le chiavi `channels.<id>` sconosciute sono **errori**, a meno che un manifest del plugin
  dichiari l’ID del canale.
* La configurazione del plugin è validata usando lo schema JSON incorporato in
  `openclaw.plugin.json` (`configSchema`).
* Se un plugin è disabilitato, la sua configurazione viene preservata e viene emesso un **avviso**.

<div id="plugin-slots-exclusive-categories">
  ## Slot dei plugin (categorie esclusive)
</div>

Alcune categorie di plugin sono **esclusive** (solo uno attivo alla volta). Usa
`plugins.slots` per selezionare quale plugin occupa lo slot:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // oppure "none" per disabilitare i plugin di memoria
    }
  }
}
```

Se più plugin dichiarano `kind: "memory"`, solo quello selezionato viene caricato. Gli altri
vengono disabilitati con messaggi di diagnostica.

<div id="control-ui-schema-labels">
  ## Control UI (schema + etichette)
</div>

La Control UI utilizza `config.schema` (JSON Schema + `uiHints`) per generare moduli migliori.

OpenClaw arricchisce `uiHints` in fase di esecuzione in base ai plugin rilevati:

* Aggiunge etichette specifiche per plugin per `plugins.entries.<id>` / `.enabled` / `.config`
* Unifica gli hint opzionali sui campi di configurazione forniti dal plugin sotto:
  `plugins.entries.<id>.config.<field>`

Se vuoi che i campi di configurazione del tuo plugin mostrino etichette/placeholder chiari (e che i segreti siano contrassegnati come sensibili),
fornisci `uiHints` insieme al tuo JSON Schema nel manifest del plugin.

Esempio:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Regione", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copia un file/directory locale in ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # relative path ok
openclaw plugins install ./plugin.tgz           # install from a local tarball
openclaw plugins install ./plugin.zip           # install from a local zip
openclaw plugins install -l ./extensions/voice-call # link (no copy) for dev
openclaw plugins install @openclaw/voice-call # install from npm
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` funziona solo per le installazioni tramite npm tracciate in `plugins.installs`.

I plugin possono anche registrare comandi di primo livello propri (per esempio: `openclaw voicecall`).

<div id="plugin-api-overview">
  ## API dei plugin (panoramica)
</div>

I plugin esportano uno dei seguenti elementi:

* Una funzione: `(api) => { ... }`
* Un oggetto: `{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## Hook dei plugin
</div>

I plugin possono fornire hook e registrarli in fase di esecuzione. Questo permette a un plugin di integrare
automazioni basate su eventi senza richiedere l’installazione separata di un pacchetto di hook.

<div id="example">
  ### Esempio
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

Note:

* Le directory degli hook seguono la normale struttura degli hook (`HOOK.md` + `handler.ts`).
* Le regole di idoneità degli hook restano valide (requisiti di sistema operativo/binari/variabili d&#39;ambiente/configurazione).
* Gli hook gestiti dal plugin compaiono in `openclaw hooks list` con `plugin:<id>`.
* Non è possibile abilitare o disabilitare gli hook gestiti dal plugin tramite `openclaw hooks`; abilita o disabilita invece il plugin.

<div id="provider-plugins-model-auth">
  ## Plugin provider (autenticazione del modello)
</div>

I plugin possono registrare flussi di **autenticazione per i provider di modelli** in modo che gli utenti possano eseguire la configurazione OAuth o
API key all&#39;interno di OpenClaw (senza la necessità di script esterni).

Registra un provider tramite `api.registerProvider(...)`. Ogni provider espone uno
o più metodi di autenticazione (OAuth, API key, device code, ecc.). Questi metodi vengono utilizzati da:

* `openclaw models auth login --provider <id> [--method <id>]`

Esempio:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Esegui il flusso OAuth e restituisci i profili di autenticazione.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Note:

* `run` riceve un `ProviderAuthContext` con gli helper `prompter`, `runtime`,
  `openUrl` e `oauth.createVpsAwareHandlers`.
* Restituisci `configPatch` quando devi aggiungere modelli predefiniti o la configurazione del provider.
* Restituisci `defaultModel` in modo che `--set-default` possa aggiornare i valori predefiniti dell’agente.

<div id="register-a-messaging-channel">
  ### Registrare un canale di messaggistica
</div>

I plugin possono registrare **plugin di canale** che si comportano come i canali integrati
(WhatsApp, Telegram, ecc.). La configurazione del canale è definita in `channels.&lt;id&gt;` ed è
validata dal codice del tuo plugin di canale.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Note:

* Inserisci la configurazione sotto `channels.<id>` (non `plugins.entries`).
* `meta.label` viene usato per le etichette negli elenchi CLI/UI.
* `meta.aliases` aggiunge ID alternativi per la normalizzazione e gli input della CLI.
* `meta.preferOver` elenca gli ID di canale per cui evitare l’abilitazione automatica quando entrambi risultano configurati.
* `meta.detailLabel` e `meta.systemImage` permettono alle UI di mostrare etichette/icone di canale più ricche.

<div id="write-a-new-messaging-channel-stepbystep">
  ### Creare un nuovo canale di messaggistica (passo passo)
</div>

Usa questa procedura quando vuoi una **nuova interfaccia di chat** (un “canale di messaggistica”), non un provider di modelli.
La documentazione sui provider di modelli si trova sotto `/providers/*`.

1. Scegli un id + la struttura della config

* Tutta la config del canale è definita sotto `channels.<id>`.
* Preferisci `channels.<id>.accounts.<accountId>` per configurazioni multi‑account.

2. Definisci i metadati del canale

* `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` controllano gli elenchi nella CLI/UI.
* `meta.docsPath` dovrebbe puntare a una pagina di documentazione come `/channels/<id>`.
* `meta.preferOver` consente a un plugin di sostituire un altro canale (l’abilitazione automatica lo preferirà).
* `meta.detailLabel` e `meta.systemImage` sono usati dalle UI per testi/icone di dettaglio.

3. Implementa gli adapter richiesti

* `config.listAccountIds` + `config.resolveAccount`
* `capabilities` (tipi di chat, media, thread, ecc.)
* `outbound.deliveryMode` + `outbound.sendText` (per l’invio di base)

4. Aggiungi adapter opzionali secondo necessità

* `setup` (procedura guidata), `security` (criteri per i DM/messaggi diretti), `status` (stato/diagnostica)
* `gateway` (start/stop/login), `mentions`, `threading`, `streaming`
* `actions` (azioni sui messaggi), `commands` (comportamento dei comandi nativi)

5. Registra il canale nel tuo plugin

* `api.registerChannel({ plugin })`

Esempio di config minimale:

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true }
      }
    }
  }
}
```

Plugin di canale minimale (solo in uscita):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "Canale di messaggistica AcmeChat.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // consegna `text` al tuo canale qui
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Carica il plugin (cartella delle estensioni o `plugins.load.paths`), riavvia il Gateway,
quindi configura `channels.<id>` nel file di configurazione.

<div id="agent-tools">
  ### Strumenti dell&#39;agente
</div>

Consulta la guida dedicata: [Strumenti agente dei plugin](/it/plugins/agent-tools).

<div id="register-a-gateway-rpc-method">
  ### Registrare un metodo RPC per il Gateway
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### Registrare comandi CLI
</div>

```ts
export default function (api) {
  api.registerCli(({ program }) => {
    program.command("mycmd").action(() => {
      console.log("Hello");
    });
  }, { commands: ["mycmd"] });
}
```

<div id="register-auto-reply-commands">
  ### Registrare comandi di risposta automatica
</div>

I plugin possono registrare comandi slash personalizzati che vengono eseguiti **senza invocare l&#39;agente AI**. È utile per comandi di attivazione/disattivazione, controlli di stato o azioni rapide che non richiedono elaborazione da parte di un LLM.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin in esecuzione! Canale: ${ctx.channel}`,
    }),
  });
}
```

Contesto dell&#39;handler di comando:

* `senderId`: L&#39;ID del mittente (se disponibile)
* `channel`: Il canale in cui è stato inviato il comando
* `isAuthorizedSender`: Indica se il mittente è un utente autorizzato
* `args`: Argomenti passati dopo il comando (se `acceptsArgs: true`)
* `commandBody`: Il testo completo del comando
* `config`: La config OpenClaw corrente

Opzioni del comando:

* `name`: Nome del comando (senza lo slash iniziale `/`)
* `description`: Testo di aiuto mostrato negli elenchi dei comandi
* `acceptsArgs`: Indica se il comando accetta argomenti (valore predefinito: false). Se è false e vengono forniti argomenti, il comando non verrà abbinato e il messaggio passerà ad altri handler
* `requireAuth`: Indica se richiedere un mittente autorizzato (valore predefinito: true)
* `handler`: Funzione che restituisce `{ text: string }` (può essere async/asincrona)

Esempio con autorizzazione e argomenti:

```ts
api.registerCommand({
  name: "setmode",
  description: "Imposta modalità plugin",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Note:

* I comandi dei plugin vengono elaborati **prima** dei comandi integrati e dell&#39;Agente AI
* I comandi sono registrati globalmente e funzionano in tutti i canali
* I nomi dei comandi non sono sensibili alle maiuscole (`/MyStatus` corrisponde a `/mystatus`)
* I nomi dei comandi devono iniziare con una lettera e contenere solo lettere, numeri, trattini e caratteri di sottolineatura
* I nomi di comando riservati (come `help`, `status`, `reset`, ecc.) non possono essere sovrascritti dai plugin
* La registrazione duplicata di comandi tra plugin non andrà a buon fine e genererà un errore diagnostico

<div id="register-background-services">
  ### Registrare i servizi in background
</div>

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

<div id="naming-conventions">
  ## Convenzioni di denominazione
</div>

* Metodi del Gateway: `pluginId.action` (esempio: `voicecall.status`)
* Tool: `snake_case` (esempio: `voice_call`)
* Comandi CLI: kebab-case o camelCase, ma evita conflitti con i comandi core

<div id="skills">
  ## Abilità
</div>

I plugin possono fornire un&#39;abilità nel repository (`skills/<name>/SKILL.md`).
Abilitala con `plugins.entries.<id>.enabled` (o altri gate di configurazione) e assicurati
che sia presente nelle posizioni previste dello spazio di lavoro o delle abilità gestite.

<div id="distribution-npm">
  ## Distribuzione (npm)
</div>

Packaging consigliato:

* Pacchetto principale: `openclaw` (questo repository)
* Plugin: pacchetti npm separati sotto `@openclaw/*` (esempio: `@openclaw/voice-call`)

Contratto di pubblicazione:

* Il `package.json` del plugin deve includere `openclaw.extensions` con uno o più file di entrypoint.
* I file di entrypoint possono essere `.js` oppure `.ts` (jiti carica i file TS a runtime).
* `openclaw plugins install <npm-spec>` usa `npm pack`, estrae in `~/.openclaw/extensions/<id>/` e lo abilita nella configurazione.
* Stabilità delle chiavi di configurazione: i pacchetti con scope vengono normalizzati all&#39;ID **senza scope** per `plugins.entries.*`.

<div id="example-plugin-voice-call">
  ## Plugin di esempio: Voice Call
</div>

Questo repository include un plugin per le chiamate vocali (Twilio o fallback basato su log):

* Sorgente: `extensions/voice-call`
* Abilità: `skills/voice-call`
* CLI: `openclaw voicecall start|status`
* Tool: `voice_call`
* RPC: `voicecall.start`, `voicecall.status`
* Configurazione (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from` (opzionali `statusCallbackUrl`, `twimlUrl`)
* Configurazione (dev): `provider: "log"` (nessuna connessione di rete)

Consulta [Voice Call](/it/plugins/voice-call) e `extensions/voice-call/README.md` per configurazione e utilizzo.

<div id="safety-notes">
  ## Avvertenze sulla sicurezza
</div>

I plugin vengono eseguiti **in-process** con il Gateway. Considerali come codice attendibile:

* Installa solo plugin di cui ti fidi.
* Preferisci la lista di autorizzati `plugins.allow`.
* Riavvia il Gateway dopo ogni modifica.

<div id="testing-plugins">
  ## Test dei plugin
</div>

I plugin possono (e dovrebbero) fornire test:

* I plugin all&#39;interno del repository possono mantenere i test Vitest sotto `src/**` (esempio: `src/plugins/voice-call.plugin.test.ts`).
* I plugin pubblicati separatamente dovrebbero eseguire una propria CI (lint/build/test) e verificare che `openclaw.extensions` punti all&#39;entrypoint compilato (`dist/index.js`).