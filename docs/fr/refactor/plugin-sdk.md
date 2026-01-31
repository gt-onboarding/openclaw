---
title: SDK de plugin
summary: "Plan : un SDK de plugin et un runtime unifiés pour tous les connecteurs de messagerie"
read_when:
  - Définir ou refactoriser l'architecture des plugins
  - Migrer les connecteurs de canaux vers le SDK/runtime de plugin
---

<div id="plugin-sdk-runtime-refactor-plan">
  # Plan de refonte du SDK de plugins et du runtime
</div>

Objectif : faire en sorte que chaque connecteur de messagerie soit un plugin (intégré ou externe) utilisant une API unique et stable.
Aucun plugin n’importe directement depuis `src/**`. Toutes les dépendances passent par le SDK ou le runtime.

<div id="why-now">
  ## Pourquoi maintenant
</div>

* Les connecteurs actuels mélangent plusieurs approches : imports directs du core, ponts limités au dist et helpers personnalisés.
* Cela rend les mises à jour fragiles et empêche de définir une surface de plugin externe propre.

<div id="target-architecture-two-layers">
  ## Architecture cible à deux couches
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) SDK de plugin (temps de compilation, stable, publiable)
</div>

Portée : types, helpers et utilitaires de configuration. Aucun état à l’exécution, aucun effet de bord.

Contenu (exemples) :

* Types : `ChannelPlugin`, adaptateurs, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
* Helpers de configuration : `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
* Helpers d’appairage : `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
* Helpers d’onboarding : `promptChannelAccessConfig`, `addWildcardAllowFrom`, types d’onboarding.
* Helpers de paramètres d’outil : `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
* Helper de lien vers la documentation : `formatDocsLink`.

Livraison :

* Publier en tant que `openclaw/plugin-sdk` (ou exporter depuis le core sous `openclaw/plugin-sdk`).
* SemVer avec garanties explicites de stabilité.

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) Runtime de plugin (surface d’exécution, injectée)
</div>

Portée : tout ce qui concerne le comportement du runtime cœur.
Accessible via `OpenClawPluginApi.runtime` pour que les plugins n’importent jamais `src/**`.

Surface proposée (minimale mais complète) :

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
      createReplyDispatcherWithTyping?: unknown; // adaptateur pour les flux de style Teams
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

Notes :

* Le runtime est le seul moyen d’accéder au comportement du cœur du système.
* Le SDK est volontairement réduit et stable.
* Chaque méthode du runtime correspond à une implémentation centrale existante (aucune duplication).

<div id="migration-plan-phased-safe">
  ## Plan de migration (par étapes, sécurisé)
</div>

<div id="phase-0-scaffolding">
  ### Phase 0 : mise en place
</div>

* Introduire `openclaw/plugin-sdk`.
* Ajouter `api.runtime` à `OpenClawPluginApi` avec la surface d’API décrite ci-dessus.
* Conserver les imports existants pendant une période de transition (avertissements de dépréciation).

<div id="phase-1-bridge-cleanup-low-risk">
  ### Phase 1 : nettoyage du bridge (faible risque)
</div>

* Remplacer `core-bridge.ts` spécifique à chaque extension par `api.runtime`.
* Migrer d&#39;abord BlueBubbles, Zalo, Zalo Personal (déjà quasiment prêts).
* Supprimer le code de bridge en double.

<div id="phase-2-light-direct-import-plugins">
  ### Phase 2 : plugins légers en import direct
</div>

* Migrer Matrix vers le SDK + runtime.
* Valider la logique d&#39;onboarding, de l&#39;annuaire et des mentions de groupe.

<div id="phase-3-heavy-direct-import-plugins">
  ### Phase 3 : plugins à import direct lourds
</div>

* Migrer MS Teams (plus gros ensemble de helpers d&#39;exécution).
* Veiller à ce que la sémantique des réponses et de la saisie corresponde au comportement actuel.

<div id="phase-4-imessage-pluginization">
  ### Phase 4 : conversion d’iMessage en plugin
</div>

* Déplacer iMessage dans `extensions/imessage`.
* Remplacer les appels directs au noyau par `api.runtime`.
* Conserver les clés de configuration, le comportement CLI et la documentation inchangés.

<div id="phase-5-enforcement">
  ### Phase 5 : mise en application
</div>

* Ajouter une règle de lint / un contrôle CI : aucun import de `extensions/**` dans `src/**`.
* Ajouter des vérifications de compatibilité de versions plugin/SDK (semver runtime + SDK).

<div id="compatibility-and-versioning">
  ## Compatibilité et gestion des versions
</div>

* SDK : semver, changements publiés et documentés.
* Runtime : versionné à chaque publication du core. Ajoutez `api.runtime.version`.
* Les plugins déclarent une plage de versions de runtime requise (par exemple : `openclawRuntime: ">=2026.2.0"`).

<div id="testing-strategy">
  ## Stratégie de test
</div>

* Tests unitaires au niveau de l’adaptateur (fonctions runtime exercées avec l’implémentation core réelle).
* Tests de référence (« golden ») par plugin : s’assurer de l’absence de dérive de comportement (routage, appairage, liste d’autorisation, filtrage par mention).
* Un seul exemple de plugin de bout en bout utilisé en CI (installation + exécution + smoke test).

<div id="open-questions">
  ## Questions ouvertes
</div>

* Où placer les types du SDK : paquet séparé ou export du core ?
* Distribution des types au runtime : dans le SDK (types uniquement) ou dans le core ?
* Comment exposer les liens de documentation pour les plugins intégrés par rapport aux plugins externes ?
* Autoriser des imports directs limités depuis le core pour les plugins au sein du dépôt pendant la transition ?

<div id="success-criteria">
  ## Critères de réussite
</div>

* Tous les connecteurs de canaux sont des plugins utilisant le SDK + le runtime.
* Aucun import `extensions/**` depuis `src/**`.
* Les nouveaux templates de connecteur ne dépendent que du SDK + du runtime.
* Des plugins externes peuvent être développés et mis à jour sans accès au code source du core.

Documentation associée : [Plugins](/fr/plugin), [Channels](/fr/channels/index), [Configuration](/fr/gateway/configuration).