---
title: Plugin-SDK
summary: "Plan: ein einheitliches Plugin-SDK und eine einheitliche Runtime für alle Messaging-Connectoren"
read_when:
  - Definieren oder Refaktorieren der Plugin-Architektur
  - Migrieren von Channel-Connectoren auf das Plugin-SDK und die Runtime
---

<div id="plugin-sdk-runtime-refactor-plan">
  # Plugin-SDK- und Runtime-Refactoring-Plan
</div>

Ziel: Jeder Messaging-Connector ist ein Plugin (gebündelt oder extern), das eine stabile API verwendet.
Kein Plugin importiert direkt aus `src/**`; alle Abhängigkeiten laufen über das SDK oder die Runtime.

<div id="why-now">
  ## Warum jetzt
</div>

* Aktuelle Connectors mischen verschiedene Muster: direkte Core-Imports, reine Dist-Bridges und eigene Hilfsfunktionen.
* Das macht Upgrades anfällig und verhindert eine klar definierte externe Plugin-Schnittstelle.

<div id="target-architecture-two-layers">
  ## Zielarchitektur (zwei Ebenen)
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) Plugin SDK (Compile-Time, stabil, veröffentlichbar)
</div>

Umfang: Typen, Hilfsfunktionen und Konfig-Utilities. Kein Laufzeitstatus, keine Seiteneffekte.

Inhalte (Beispiele):

* Typen: `ChannelPlugin`, Adapter, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
* Konfig-Helfer: `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
* Kopplungs-Helfer: `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
* Onboarding-Helfer: `promptChannelAccessConfig`, `addWildcardAllowFrom`, Onboarding-Typen.
* Tool-Parameter-Helfer: `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
* Docs-Link-Helfer: `formatDocsLink`.

Bereitstellung:

* Veröffentlichen als `openclaw/plugin-sdk` (oder aus dem Core unter `openclaw/plugin-sdk` exportieren).
* Semver mit expliziten Stabilitätsgarantien.

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) Plugin-Laufzeit (Ausführungsoberfläche, injiziert)
</div>

Geltungsbereich: alles, was das zentrale Laufzeitverhalten betrifft.
Zugriff über `OpenClawPluginApi.runtime`, sodass Plugins niemals `src/**` importieren müssen.

Vorgeschlagene Oberfläche (minimal, aber vollständig):

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
      createReplyDispatcherWithTyping?: unknown; // Adapter für Teams-artige Abläufe
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

Hinweise:

* Die Runtime ist die einzige Möglichkeit, auf das Kernverhalten zuzugreifen.
* Das SDK ist bewusst klein und stabil gehalten.
* Jede Runtime-Methode ist direkt einer bestehenden Kernimplementierung zugeordnet (keine Duplikation).

<div id="migration-plan-phased-safe">
  ## Migrationsplan (stufenweise, sicher)
</div>

<div id="phase-0-scaffolding">
  ### Phase 0: Scaffolding
</div>

* Einführung von `openclaw/plugin-sdk`.
* Füge `api.runtime` zur `OpenClawPluginApi` mit der oben beschriebenen API-Oberfläche hinzu.
* Behalte bestehende Importe während eines Übergangszeitraums bei (Deprecation-Warnungen).

<div id="phase-1-bridge-cleanup-low-risk">
  ### Phase 1: Bridge-Bereinigung (geringes Risiko)
</div>

* Ersetze die `core-bridge.ts`-Datei je Extension durch `api.runtime`.
* Migriere zuerst BlueBubbles, Zalo, Zalo Personal (bereits weitgehend vorbereitet).
* Entferne doppelten Bridge-Code.

<div id="phase-2-light-direct-import-plugins">
  ### Phase 2: leichte Direct-Import-Plugins
</div>

* Migriere Matrix auf das SDK + die Runtime.
* Validiere das Onboarding sowie die Logik für Verzeichnis- und Gruppen-Mentions.

<div id="phase-3-heavy-direct-import-plugins">
  ### Phase 3: umfangreiche Direct-Import-Plugins
</div>

* MS Teams migrieren (größte Menge an Runtime-Hilfsfunktionen).
* Sicherstellen, dass Antwort-/Typing-Semantik dem aktuellen Verhalten entspricht.

<div id="phase-4-imessage-pluginization">
  ### Phase 4: iMessage-Pluginisierung
</div>

* Verschiebe iMessage in `extensions/imessage`.
* Ersetze direkte Core-Aufrufe durch `api.runtime`.
* Behalte Config-Keys, CLI-Verhalten und Dokumentation bei.

<div id="phase-5-enforcement">
  ### Phase 5: Durchsetzung
</div>

* Lint-Regel/CI-Check hinzufügen: keine Importe von `extensions/**` aus `src/**`.
* Kompatibilitätsprüfungen für Plugin-SDK/Versionen hinzufügen (Runtime + SDK-Semver).

<div id="compatibility-and-versioning">
  ## Kompatibilität und Versionierung
</div>

* SDK: semantische Versionierung (semver), veröffentlichte und dokumentierte Änderungen.
* Runtime: für jedes Core-Release versioniert. Ergänze `api.runtime.version`.
* Plugins geben einen erforderlichen Runtime-Bereich an (z. B. `openclawRuntime: ">=2026.2.0"`).

<div id="testing-strategy">
  ## Teststrategie
</div>

* Unit-Tests auf Adapter-Ebene (Runtime-Funktionen werden mit echter Core-Implementierung ausgeführt).
* Golden-Tests pro Plugin: sicherstellen, dass es keine Abweichungen im Verhalten gibt (Routing, Kopplung, Allowlist, Mention-Gating).
* Ein einziges End-to-End-Plugin-Beispiel, das in CI verwendet wird (install + run + Smoke-Test).

<div id="open-questions">
  ## Offene Fragen
</div>

* Wo sollen die SDK-Typen liegen: eigenes Paket oder Export aus dem Core?
* Verteilung der Runtime-Typen: im SDK (nur Typdefinitionen) oder im Core?
* Wie sollen Dokumentationslinks für gebündelte vs. externe Plugins bereitgestellt werden?
* Erlauben wir während der Übergangsphase eingeschränkte direkte Core-Imports für In-Repo-Plugins?

<div id="success-criteria">
  ## Erfolgskriterien
</div>

* Alle Channel-Connectoren sind Plugins, die SDK + Runtime verwenden.
* Keine `extensions/**`-Importe aus `src/**`.
* Neue Connector-Vorlagen hängen ausschließlich von SDK + Runtime ab.
* Externe Plugins können ohne Zugriff auf den Core-Quellcode entwickelt und aktualisiert werden.

Zugehörige Dokumentation: [Plugins](/de/plugin), [Kanäle](/de/channels/index), [Konfiguration](/de/gateway/configuration).