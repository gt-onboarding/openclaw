---
title: SDK de complementos
summary: "Plan: un único SDK de complementos limpio y un entorno de ejecución para todos los conectores de mensajería"
read_when:
  - Definir o refactorizar la arquitectura de complementos
  - Migrar los conectores de canal al SDK y entorno de ejecución de complementos
---

<div id="plugin-sdk-runtime-refactor-plan">
  # Plan de refactorización del SDK y del runtime de complementos
</div>

Objetivo: que cada conector de mensajería sea un complemento (integrado o externo) que utilice una única API estable.
Ningún complemento importa directamente desde `src/**`. Todas las dependencias pasan por el SDK o por el runtime.

<div id="why-now">
  ## Por qué ahora
</div>

* Los conectores actuales mezclan varios patrones: importaciones directas del núcleo, bridges que sólo usan el dist y helpers personalizados.
* Esto hace que las actualizaciones sean frágiles e impide contar con una superficie de complemento externa limpia.

<div id="target-architecture-two-layers">
  ## Arquitectura objetivo (dos capas)
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) Plugin SDK (tiempo de compilación, estable, publicable)
</div>

Ámbito: tipos, helpers y utilidades de configuración. Sin estado en tiempo de ejecución ni efectos secundarios.

Contenido (ejemplos):

* Tipos: `ChannelPlugin`, adaptadores, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
* Helpers de configuración: `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
* Helpers de emparejamiento: `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
* Helpers de onboarding: `promptChannelAccessConfig`, `addWildcardAllowFrom`, tipos de onboarding.
* Helpers de parámetros de herramienta: `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
* Helper de enlace a la documentación: `formatDocsLink`.

Entrega:

* Publicar como `openclaw/plugin-sdk` (o exportar desde core bajo `openclaw/plugin-sdk`).
* SemVer con garantías explícitas de estabilidad.

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) Runtime de Complementos (superficie de ejecución, inyectada)
</div>

Ámbito: todo lo que afecta al comportamiento del runtime principal.
Se accede mediante `OpenClawPluginApi.runtime` para que los complementos nunca importen `src/**`.

Superficie propuesta (mínima pero completa):

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
      createReplyDispatcherWithTyping?: unknown; // adaptador para flujos de estilo Teams
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

Notas:

* Runtime es la única forma de acceder al comportamiento del núcleo.
* El SDK es intencionalmente pequeño y estable.
* Cada método de runtime se corresponde con una implementación de núcleo existente (sin duplicación).

<div id="migration-plan-phased-safe">
  ## Plan de migración (por fases y seguro)
</div>

<div id="phase-0-scaffolding">
  ### Fase 0: preparación
</div>

* Introducir `openclaw/plugin-sdk`.
* Añadir `api.runtime` a `OpenClawPluginApi` con la interfaz descrita arriba.
* Mantener las importaciones existentes durante una ventana de transición (advertencias de deprecación).

<div id="phase-1-bridge-cleanup-low-risk">
  ### Fase 1: limpieza del bridge (bajo riesgo)
</div>

* Reemplazar `core-bridge.ts` en cada extensión con `api.runtime`.
* Migrar primero BlueBubbles, Zalo, Zalo Personal (ya casi están).
* Eliminar el código duplicado del bridge.

<div id="phase-2-light-direct-import-plugins">
  ### Fase 2: complementos ligeros de importación directa
</div>

* Migrar Matrix al SDK + runtime.
* Validar el flujo de incorporación, el directorio y la lógica de menciones de grupo.

<div id="phase-3-heavy-direct-import-plugins">
  ### Fase 3: complementos con importación directa intensiva
</div>

* Migra MS Teams (el conjunto más grande de utilidades de tiempo de ejecución).
* Asegúrate de que la semántica de respuesta/escritura se corresponda con el comportamiento actual.

<div id="phase-4-imessage-pluginization">
  ### Fase 4: pluginización de iMessage
</div>

* Mover iMessage a `extensions/imessage`.
* Reemplazar las llamadas directas al núcleo con `api.runtime`.
* Mantener intactas las claves de configuración, el comportamiento de la CLI y la documentación.

<div id="phase-5-enforcement">
  ### Fase 5: cumplimiento
</div>

* Añadir una regla de lint o comprobación en CI: prohibir imports de `extensions/**` desde `src/**`.
* Añadir comprobaciones de compatibilidad de versiones de complementos y SDK (runtime + semver del SDK).

<div id="compatibility-and-versioning">
  ## Compatibilidad y versionado
</div>

* SDK: semver, publicado, con cambios documentados.
* Runtime: versionado por cada versión del núcleo. Añade `api.runtime.version`.
* Los complementos declaran un intervalo de versiones de runtime requerido (por ejemplo, `openclawRuntime: ">=2026.2.0"`).

<div id="testing-strategy">
  ## Estrategia de pruebas
</div>

* Pruebas unitarias a nivel de adaptador (funciones en tiempo de ejecución ejercitadas con la implementación real del núcleo).
* Pruebas de referencia para cada complemento: garantizar que no haya desviaciones de comportamiento (enrutamiento, emparejamiento, lista de permitidos, limitación por menciones).
* Un único ejemplo de complemento de extremo a extremo usado en CI (instalar + ejecutar + prueba de humo).

<div id="open-questions">
  ## Preguntas abiertas
</div>

* ¿Dónde alojar los tipos del SDK: en un paquete separado o como exportaciones del core?
* Distribución de tipos en tiempo de ejecución: ¿en el SDK (solo tipos) o en el core?
* ¿Cómo exponer enlaces de documentación para complementos incluidos vs externos?
* ¿Permitimos importaciones directas limitadas del core para complementos dentro del repositorio durante la transición?

<div id="success-criteria">
  ## Criterios de éxito
</div>

* Todos los conectores de canal son complementos que usan el SDK + runtime.
* No hay importaciones de `extensions/**` desde `src/**`.
* Las nuevas plantillas de conectores dependen únicamente del SDK + runtime.
* Los complementos externos se pueden desarrollar y actualizar sin acceso al código fuente principal.

Documentación relacionada: [Complementos](/es/plugin), [Canales](/es/channels/index), [Configuración](/es/gateway/configuration).