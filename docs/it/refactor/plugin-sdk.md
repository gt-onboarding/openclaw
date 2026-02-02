---
title: Plugin SDK
summary: "Piano: un unico Plugin SDK + runtime per tutti i connettori di messaggistica"
read_when:
  - Definire o rifattorizzare l'architettura dei plugin
  - Migrare i connettori dei canali al Plugin SDK/runtime
---

<div id="plugin-sdk-runtime-refactor-plan">
  # Piano di refactoring di Plugin SDK + Runtime
</div>

Obiettivo: ogni connettore di messaggistica è un plugin (incluso nel bundle o esterno) che utilizza un&#39;unica API stabile.
Nessun plugin importa direttamente da `src/**`. Tutte le dipendenze passano tramite l&#39;SDK o il runtime.

<div id="why-now">
  ## Perché ora
</div>

* I connettori attuali mescolano diversi pattern: importazioni dirette del core, bridge basati solo su dist e helper personalizzati.
* Questo rende gli aggiornamenti fragili e impedisce di esporre una superficie pulita per i plugin esterni.

<div id="target-architecture-two-layers">
  ## Architettura target (due livelli)
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) Plugin SDK (compile-time, stabile, pubblicabile)
</div>

Ambito: tipi, helper e utility di configurazione. Nessuno stato a runtime, nessun effetto collaterale.

Contenuti (esempi):

* Tipi: `ChannelPlugin`, adapter, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
* Helper per la configurazione: `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
* Helper per l&#39;abbinamento: `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
* Helper per l&#39;onboarding: `promptChannelAccessConfig`, `addWildcardAllowFrom`, tipi di onboarding.
* Helper per i parametri dei tool: `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
* Helper per i link alla documentazione: `formatDocsLink`.

Distribuzione:

* Pubblica come `openclaw/plugin-sdk` (o esporta dal core sotto `openclaw/plugin-sdk`).
* Semver con garanzie esplicite di stabilità.

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) Runtime del plugin (superficie di esecuzione, iniettata)
</div>

Ambito: tutto ciò che riguarda il comportamento del runtime principale.
Accessibile tramite `OpenClawPluginApi.runtime` in modo che i plugin non importino mai `src/**`.

Superficie proposta (minima ma completa):

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: { text?: string; mediaUrls?: string[]; mediaUrl?: string }) =>
            void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // adattatore per flussi stile Teams
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: "dm" | "group" | "channel"; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(cfg: OpenClawConfig, channel: string, accountId: string, groupId: string): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```

Note:

* Il runtime è l&#39;unico modo per accedere alle funzionalità core.
* L’SDK è deliberatamente piccolo e stabile.
* Ogni metodo del runtime corrisponde a un&#39;implementazione core esistente (senza duplicazioni).

<div id="migration-plan-phased-safe">
  ## Piano di migrazione (per fasi, sicuro)
</div>

<div id="phase-0-scaffolding">
  ### Fase 0: struttura di base
</div>

* Introdurre `openclaw/plugin-sdk`.
* Aggiungere `api.runtime` a `OpenClawPluginApi` con l’interfaccia descritta sopra.
* Mantenere gli import esistenti durante il periodo di transizione (con avvisi di deprecazione).

<div id="phase-1-bridge-cleanup-low-risk">
  ### Fase 1: pulizia del bridge (basso rischio)
</div>

* Sostituisci `core-bridge.ts` per ciascuna estensione con `api.runtime`.
* Migra prima BlueBubbles, Zalo, Zalo Personal (sono già quasi pronti).
* Rimuovi il codice del bridge duplicato.

<div id="phase-2-light-direct-import-plugins">
  ### Fase 2: plugin con importazione diretta leggera
</div>

* Migra Matrix su SDK + runtime.
* Verifica l&#39;onboarding, la directory, la logica delle menzioni di gruppo.

<div id="phase-3-heavy-direct-import-plugins">
  ### Fase 3: plugin con import diretto massiccio
</div>

* Esegui la migrazione di MS Teams (il set più ampio di helper di runtime).
* Verifica che la semantica di risposta/digitazione corrisponda al comportamento attuale.

<div id="phase-4-imessage-pluginization">
  ### Fase 4: trasformazione di iMessage in plugin
</div>

* Sposta iMessage in `extensions/imessage`.
* Sostituisci le chiamate dirette al core con `api.runtime`.
* Mantieni le chiavi di configurazione, il comportamento della CLI e la documentazione invariati.

<div id="phase-5-enforcement">
  ### Fase 5: enforcement
</div>

* Aggiungi una regola di lint / un controllo CI: nessun import da `extensions/**` in `src/**`.
* Aggiungi controlli di compatibilità di versione per il plugin SDK (semver per runtime + SDK).

<div id="compatibility-and-versioning">
  ## Compatibilità e versionamento
</div>

* SDK: semver, modifiche pubblicate e documentate.
* Runtime: versionato per ogni release del core. Aggiungi il campo `api.runtime.version`.
* I plugin dichiarano un intervallo di runtime richiesto (ad esempio, `openclawRuntime: ">=2026.2.0"`).

<div id="testing-strategy">
  ## Strategia di test
</div>

* Test unitari a livello di adapter (funzioni di runtime esercitate con l&#39;implementazione core reale).
* Golden tests per plugin: garantisci che non ci siano variazioni di comportamento (routing, abbinamento, lista di autorizzati, mention gating).
* Un singolo esempio di plugin end-to-end utilizzato in CI (installazione + esecuzione + smoke test).

<div id="open-questions">
  ## Questioni aperte
</div>

* Dove ospitare i tipi dell&#39;SDK: pacchetto separato o export del core?
* Distribuzione dei tipi di runtime: nell&#39;SDK (solo tipi) o nel core?
* Come esporre i link alla documentazione per i plugin inclusi nel bundle vs esterni?
* Vogliamo consentire import diretti, seppur limitati, del core per i plugin in-repo durante la transizione?

<div id="success-criteria">
  ## Criteri di successo
</div>

* Tutti i connettori di canale sono plugin che utilizzano SDK + runtime.
* Nessun import `extensions/**` da `src/**`.
* I nuovi modelli di connettore dipendono solo da SDK + runtime.
* I plugin esterni possono essere sviluppati e aggiornati senza accesso al codice sorgente del core.

Documentazione correlata: [Plugin](/it/plugin), [Canali](/it/channels/index), [Configurazione](/it/gateway/configuration).