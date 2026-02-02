---
title: Plugin
summary: "OpenClaw-Plugins/-Erweiterungen: Erkennung, Konfiguration und Sicherheit"
read_when:
  - Plugins/Erweiterungen hinzufügen oder ändern
  - Installations- oder Laderegeln für Plugins/Erweiterungen dokumentieren
---

<div id="plugins-extensions">
  # Plugins (Erweiterungen)
</div>

<div id="quick-start-new-to-plugins">
  ## Schnellstart (neu bei Plugins?)
</div>

Ein Plugin ist einfach ein **kleines Code-Modul**, das OpenClaw um zusätzliche
Features erweitert (Befehle, Tools und Gateway-RPC).

Meistens verwendest du Plugins, wenn du ein Feature brauchst, das im
OpenClaw-Core noch nicht eingebaut ist (oder wenn du optionale Features aus
deiner Hauptinstallation heraushalten willst).

Schneller Weg:

1. Sieh dir an, was bereits geladen ist:

```bash
openclaw plugins list
```

2. Installiere ein offizielles Plugin (z. B. Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

3. Starte das Gateway neu und konfiguriere es anschließend unter `plugins.entries.<id>.config`.

Siehe [Voice Call](/de/plugins/voice-call) für ein konkretes Beispiel-Plugin.

<div id="available-plugins-official">
  ## Verfügbare Plugins (offiziell)
</div>

* Microsoft Teams ist seit 2026.1.15 ausschließlich als Plugin verfügbar; installiere `@openclaw/msteams`, wenn du Teams verwendest.
* Memory (Core) — mitgeliefertes Memory-Such-Plugin (standardmäßig über `plugins.slots.memory` aktiviert)
* Memory (LanceDB) — mitgeliefertes Langzeit-Memory-Plugin (Auto-Recall/-Capture; setze `plugins.slots.memory = "memory-lancedb"`)
* [Voice Call](/de/plugins/voice-call) — `@openclaw/voice-call`
* [Zalo Personal](/de/plugins/zalouser) — `@openclaw/zalouser`
* [Matrix](/de/channels/matrix) — `@openclaw/matrix`
* [Nostr](/de/channels/nostr) — `@openclaw/nostr`
* [Zalo](/de/channels/zalo) — `@openclaw/zalo`
* [Microsoft Teams](/de/channels/msteams) — `@openclaw/msteams`
* Google Antigravity OAuth (Anbieter-Auth) — mitgeliefert als `google-antigravity-auth` (standardmäßig deaktiviert)
* Gemini CLI OAuth (Anbieter-Auth) — mitgeliefert als `google-gemini-cli-auth` (standardmäßig deaktiviert)
* Qwen OAuth (Anbieter-Auth) — mitgeliefert als `qwen-portal-auth` (standardmäßig deaktiviert)
* Copilot Proxy (Anbieter-Auth) — lokale Proxy-Bridge für VS Code Copilot; unterscheidet sich vom eingebauten `github-copilot`-Device-Login (mitgeliefert, standardmäßig deaktiviert)

OpenClaw-Plugins sind **TypeScript-Module**, die zur Laufzeit über jiti geladen werden. **Config-
Validierung führt keinen Plugin-Code aus**; sie verwendet stattdessen das Plugin-Manifest und JSON
Schema. Siehe [Plugin-Manifest](/de/plugins/manifest).

Plugins können registrieren:

* Gateway-RPC-Methoden
* Gateway-HTTP-Handler
* Agent-Tools
* CLI-Befehle
* Hintergrunddienste
* Optionale Config-Validierung
* **Fähigkeiten** (durch Auflisten von `skills`-Verzeichnissen im Plugin-Manifest)
* **Auto-Antwort-Befehle** (Ausführung ohne Aufruf des AI-Agenten)

Plugins laufen **im selben Prozess** wie das Gateway; behandle sie daher als vertrauenswürdigen Code.
Leitfaden zur Tool-Erstellung: [Plugin agent tools](/de/plugins/agent-tools).

<div id="runtime-helpers">
  ## Runtime-Helfer
</div>

Plugins können über `api.runtime` auf ausgewählte Core-Helfer zugreifen. Für Telephony-TTS:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Hinweise:

* Verwendet die zentrale `messages.tts`-Konfiguration (OpenAI oder ElevenLabs).
* Gibt einen PCM-Audiopuffer + Abtastrate zurück. Plugins müssen das Audio für Anbieter resamplen/encodieren.
* Edge TTS wird für Telefonie nicht unterstützt.

<div id="discovery-precedence">
  ## Erkennung &amp; Priorität
</div>

OpenClaw durchsucht in folgender Reihenfolge:

1. Konfigurationspfade

* `plugins.load.paths` (Datei oder Verzeichnis)

2. Erweiterungen im Arbeitsbereich

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. Globale Erweiterungen

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. Mitgelieferte Erweiterungen (mit OpenClaw ausgeliefert, **standardmäßig deaktiviert**)

* `<openclaw>/extensions/*`

Mitgelieferte Plugins müssen explizit über `plugins.entries.<id>.enabled`
oder `openclaw plugins enable <id>` aktiviert werden. Installierte Plugins sind
standardmäßig aktiviert, können aber auf dieselbe Weise deaktiviert werden.

Jedes Plugin muss eine `openclaw.plugin.json`-Datei in seinem Stammverzeichnis enthalten. Wenn ein Pfad auf eine Datei zeigt, ist das Stammverzeichnis des Plugins das Verzeichnis dieser Datei und muss das Manifest enthalten.

Wenn mehrere Plugins auf dieselbe ID aufgelöst werden, gewinnt der erste Treffer
in der obigen Reihenfolge und Kopien mit niedrigerer Priorität werden ignoriert.

<div id="package-packs">
  ### Package-Packs
</div>

Ein Plugin-Verzeichnis kann eine Datei `package.json` enthalten, die `openclaw.extensions` definiert:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Jeder Eintrag wird zu einem Plugin. Wenn der Pack mehrere Erweiterungen auflistet, lautet die Plugin-ID `name/<fileBase>`.

Wenn dein Plugin npm-Abhängigkeiten importiert, installiere sie in diesem Verzeichnis, damit `node_modules` zur Verfügung steht (`npm install` / `pnpm install`).

<div id="channel-catalog-metadata">
  ### Metadaten für den Channel-Katalog
</div>

Channel-Plugins können Onboarding-Metadaten über `openclaw.channel` und
Installationshinweise über `openclaw.install` bereitstellen. Dadurch bleibt der zentrale Katalog selbst datenfrei.

Beispiel:

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
      "blurb": "Selbst-gehosteter Chat über Nextcloud Talk Webhook-Bots.",
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

OpenClaw kann außerdem **externe Channel-Kataloge** zusammenführen (z. B. einen
MPM-Registry-Export). Lege eine JSON-Datei an einem der folgenden Orte ab:

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

Oder setze `OPENCLAW_PLUGIN_CATALOG_PATHS` (oder `OPENCLAW_MPM_CATALOG_PATHS`) auf
eine oder mehrere JSON-Dateien (durch Komma, Semikolon oder wie `PATH` getrennt). Jede Datei
sollte `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }` enthalten.

<div id="plugin-ids">
  ## Plugin-IDs
</div>

Standardmäßige Plugin-IDs:

* Paketpakete: `package.json` `name`
* Eigenständige Datei: Basis-Dateiname (`~/.../voice-call.ts` → `voice-call`)

Wenn ein Plugin eine `id` exportiert, verwendet OpenClaw diese, warnt jedoch, wenn sie nicht mit der
konfigurierten ID übereinstimmt.

<div id="config">
  ## Konfiguration
</div>

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"], // Erlaubnisliste
    deny: ["untrusted-plugin"], // Sperrliste
    load: { paths: ["~/Projects/oss/voice-call-extension"] }, // Zusätzliche Plugin-Pfade
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } }
    }
  }
}
```

Felder:

* `enabled`: Hauptschalter (Standard: true)
* `allow`: Allowlist (optional)
* `deny`: Denylist (optional; deny hat Vorrang)
* `load.paths`: zusätzliche Plugin‑Dateien und ‑Verzeichnisse
* `entries.<id>`: Plugin‑spezifische Schalter und Konfiguration

Konfigurationsänderungen **erfordern einen Neustart des Gateways**.

Validierungsregeln (strikt):

* Unbekannte Plugin‑IDs in `entries`, `allow`, `deny` oder `slots` sind **Fehler**.
* Unbekannte `channels.<id>`‑Schlüssel sind **Fehler**, sofern nicht ein Plugin‑Manifest
  die Channel‑ID deklariert.
* Die Plugin‑Konfiguration wird anhand des in
  `openclaw.plugin.json` (`configSchema`) eingebetteten JSON‑Schemas validiert.
* Wenn ein Plugin deaktiviert ist, bleibt seine Konfiguration erhalten und eine **Warnung** wird ausgegeben.

<div id="plugin-slots-exclusive-categories">
  ## Plugin-Slots (exklusive Kategorien)
</div>

Einige Plugin-Kategorien sind **exklusiv** (es kann immer nur ein Plugin gleichzeitig aktiv sein). Verwende
`plugins.slots`, um festzulegen, welchem Plugin der Slot zugewiesen wird:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // oder "none", um Memory-Plugins zu deaktivieren
    }
  }
}
```

Wenn mehrere Plugins `kind: "memory"` deklarieren, wird nur das ausgewählte geladen. Die anderen
werden mit entsprechenden Diagnosemeldungen deaktiviert.

<div id="control-ui-schema-labels">
  ## Control UI (Schema + Beschriftungen)
</div>

Die Control UI verwendet `config.schema` (JSON Schema + `uiHints`), um bessere Formulare zu rendern.

OpenClaw ergänzt `uiHints` zur Laufzeit basierend auf erkannten Plugins:

* Fügt Plugin-spezifische Beschriftungen für `plugins.entries.<id>` / `.enabled` / `.config` hinzu
* Führt optionale, vom Plugin bereitgestellte Hinweise zu Konfigurationsfeldern zusammen unter:
  `plugins.entries.<id>.config.<field>`

Wenn du möchtest, dass deine Plugin-Konfigurationsfelder sinnvolle Beschriftungen/Platzhalter anzeigen (und Secrets als sensibel markieren),
liefere `uiHints` zusammen mit deinem JSON Schema im Plugin-Manifest.

Beispiel:

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
    "apiKey": { "label": "API-Schlüssel", "sensitive": true },
    "region": { "label": "Region", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # kopiert eine lokale Datei/ein Verzeichnis nach ~/.openclaw/extensions/<id>
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

`plugins update` funktioniert nur für npm‑Installationen, die unter `plugins.installs` verwaltet werden.

Plugins können auch eigene Top‑Level‑Befehle registrieren (Beispiel: `openclaw voicecall`).

<div id="plugin-api-overview">
  ## Plugin-API (Übersicht)
</div>

Plugins exportieren entweder:

* eine Funktion: `(api) => { ... }`
* ein Objekt: `{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## Plugin-Hooks
</div>

Plugins können Hooks mitliefern und sie zur Laufzeit registrieren. Dadurch kann ein Plugin ereignisgesteuerte Automatisierung mitbündeln, ohne dass ein separates Hook-Paket installiert werden muss.

<div id="example">
  ### Beispiel
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

Hinweise:

* Hook-Verzeichnisse folgen der normalen Hook-Struktur (`HOOK.md` + `handler.ts`).
* Bedingungen für die Verwendung von Hooks gelten weiterhin (OS-/Binär-/Env-/Config-Anforderungen).
* Von Plugins verwaltete Hooks werden in `openclaw hooks list` mit `plugin:&lt;id&gt;` aufgeführt.
* Du kannst von Plugins verwaltete Hooks nicht über `openclaw hooks` aktivieren/deaktivieren; aktiviere/deaktiviere stattdessen das Plugin.

<div id="provider-plugins-model-auth">
  ## Provider-Plugins (Modellauthentifizierung)
</div>

Plugins können **Authentifizierungs-Flows für Modell-Anbieter** registrieren, sodass Nutzer OAuth- oder API-Schlüssel-Einrichtung direkt in OpenClaw durchführen können (keine externen Skripte nötig).

Registriere einen Anbieter über `api.registerProvider(...)`. Jeder Anbieter stellt eine oder mehrere Authentifizierungsmethoden bereit (OAuth, API-Schlüssel, Device Code usw.). Diese Methoden werden von folgendem Befehl verwendet:

* `openclaw models auth login --provider <id> [--method <id>]`

Beispiel:

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
        // OAuth-Ablauf ausführen und Authentifizierungsprofile zurückgeben.
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

Hinweise:

* `run` erhält einen `ProviderAuthContext` mit den Hilfsfunktionen `prompter`, `runtime`,
  `openUrl` und `oauth.createVpsAwareHandlers`.
* Gib `configPatch` zurück, wenn du Standardmodelle oder Anbieter-Konfiguration hinzufügen musst.
* Gib `defaultModel` zurück, damit `--set-default` die Agent-Standardwerte aktualisieren kann.

<div id="register-a-messaging-channel">
  ### Einen Messaging‑Kanal registrieren
</div>

Plugins können **Channel‑Plugins** registrieren, die sich wie eingebaute Kanäle
(WhatsApp, Telegram usw.) verhalten. Die Channel‑Konfiguration befindet sich unter `channels.<id>` und wird
von deinem Channel‑Plugin‑Code validiert.

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

Hinweise:

* Lege die Konfiguration unter `channels.<id>` ab (nicht unter `plugins.entries`).
* `meta.label` wird für Labels in CLI-/UI-Listen verwendet.
* `meta.aliases` fügt alternative IDs für Normalisierung und CLI-Eingaben hinzu.
* `meta.preferOver` listet Channel-IDs auf, für die die automatische Aktivierung übersprungen wird, wenn beide konfiguriert sind.
* `meta.detailLabel` und `meta.systemImage` ermöglichen es UIs, detailliertere Channel-Labels/-Icons anzuzeigen.

<div id="write-a-new-messaging-channel-stepbystep">
  ### Neuen Messaging‑Kanal schreiben (Schritt für Schritt)
</div>

Verwende dies, wenn du eine **neue Chat‑Oberfläche** (einen „Messaging‑Kanal“) und keinen Modellanbieter benötigst.
Dokumentation zu Modellanbietern befindet sich unter `/providers/*`.

1. Wähle eine id + Konfigurationsschema

* Alle Kanal‑Konfiguration liegt unter `channels.<id>`.
* Bevorzuge `channels.<id>.accounts.<accountId>` für Multi‑Account‑Setups.

2. Definiere die Kanal‑Metadaten

* `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` steuern CLI/UI‑Listen.
* `meta.docsPath` sollte auf eine Doku‑Seite wie `/channels/<id>` zeigen.
* `meta.preferOver` ermöglicht es einem Plugin, einen anderen Kanal zu ersetzen (Auto‑Enable bevorzugt ihn).
* `meta.detailLabel` und `meta.systemImage` werden von UIs für Detailtexte/Icons verwendet.

3. Implementiere die benötigten Adapter

* `config.listAccountIds` + `config.resolveAccount`
* `capabilities` (Chat‑Typen, Medien, Threads, etc.)
* `outbound.deliveryMode` + `outbound.sendText` (für einfaches Senden)

4. Füge nach Bedarf optionale Adapter hinzu

* `setup` (Assistent), `security` (DM‑Richtlinie), `status` (Health/Diagnose)
* `gateway` (Start/Stop/Login), `mentions`, `threading`, `streaming`
* `actions` (Nachrichtenaktionen), `commands` (native Befehlssemantik)

5. Registriere den Kanal in deinem Plugin

* `api.registerChannel({ plugin })`

Minimales Konfigurationsbeispiel:

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

Minimales Channel-Plugin (nur ausgehend):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
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
      // `text` hier an Ihren Kanal ausliefern
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Lade das Plugin (Verzeichnis `extensions` oder `plugins.load.paths`), starte das Gateway neu
und konfiguriere anschließend `channels.<id>` in deiner Konfigurationsdatei.

<div id="agent-tools">
  ### Agent-Tools
</div>

Siehe die entsprechende Anleitung: [Plugin-Agent-Tools](/de/plugins/agent-tools).

<div id="register-a-gateway-rpc-method">
  ### Gateway-RPC-Methode registrieren
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### CLI-Befehle registrieren
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
  ### Auto-Antwort-Befehle registrieren
</div>

Plugins können benutzerdefinierte Slash-Befehle registrieren, die **ohne den
AI-Agent** ausgeführt werden. Das ist nützlich für Toggle-Befehle, Statusprüfungen oder schnelle Aktionen,
die keine LLM-Verarbeitung benötigen.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

Command-Handler-Kontext:

* `senderId`: Die ID des Absenders (falls verfügbar)
* `channel`: Der Kanal, über den der Befehl gesendet wurde
* `isAuthorizedSender`: Ob der Absender ein autorisierter Benutzer ist
* `args`: Argumente, die nach dem Befehl übergeben werden (wenn `acceptsArgs: true`)
* `commandBody`: Der vollständige Befehlstext
* `config`: Die aktuelle OpenClaw-Konfiguration

Befehlsoptionen:

* `name`: Befehlsname (ohne führenden `/`)
* `description`: Hilfetext, der in Befehlslisten angezeigt wird
* `acceptsArgs`: Ob der Befehl Argumente akzeptiert (Standard: false). Wenn false ist und Argumente angegeben werden, passt der Befehl nicht und die Nachricht wird an andere Handler weitergereicht
* `requireAuth`: Ob ein autorisierter Absender erforderlich ist (Standard: true)
* `handler`: Funktion, die `{ text: string }` zurückgibt (kann asynchron sein)

Beispiel mit Autorisierung und Argumenten:

```ts
api.registerCommand({
  name: "setmode",
  description: "Plugin-Modus setzen",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Hinweise:

* Plugin-Befehle werden **vor** integrierten Befehlen und dem AI-Agent verarbeitet
* Befehle werden global registriert und funktionieren in allen Kanälen
* Bei Befehlsnamen wird die Groß- und Kleinschreibung nicht beachtet (`/MyStatus` entspricht `/mystatus`)
* Befehlsnamen müssen mit einem Buchstaben beginnen und dürfen nur Buchstaben, Zahlen, Bindestriche und Unterstriche enthalten
* Reservierte Befehlsnamen (wie `help`, `status`, `reset` usw.) können von Plugins nicht überschrieben werden
* Doppelte Befehlsregistrierungen über Plugins hinweg führen zu einem Diagnosefehler

<div id="register-background-services">
  ### Hintergrunddienste registrieren
</div>

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("bereit"),
    stop: () => api.logger.info("bye"),
  });
}
```

<div id="naming-conventions">
  ## Namenskonventionen
</div>

* Gateway-Methoden: `pluginId.action` (Beispiel: `voicecall.status`)
* Tools: `snake_case` (Beispiel: `voice_call`)
* CLI-Befehle: Kebab- oder CamelCase-Schreibweise, aber Kollisionen mit Kernbefehlen vermeiden

<div id="skills">
  ## Fähigkeiten
</div>

Plugins können eine Fähigkeit im Repo mitliefern (`skills/<name>/SKILL.md`).
Aktiviere sie mit `plugins.entries.<id>.enabled` (oder anderen Konfigurationsoptionen) und stelle sicher,
dass sie in deinem Arbeitsbereich bzw. an den verwalteten Speicherorten für Fähigkeiten vorhanden ist.

<div id="distribution-npm">
  ## Distribution (npm)
</div>

Empfohlene Paketierung:

* Hauptpaket: `openclaw` (dieses Repo)
* Plugins: separate npm-Pakete unter `@openclaw/*` (Beispiel: `@openclaw/voice-call`)

Veröffentlichungsanforderungen:

* Die `package.json` des Plugins muss `openclaw.extensions` mit einer oder mehreren Entry-Dateien enthalten.
* Entry-Dateien können `.js` oder `.ts` sein (jiti lädt TS zur Laufzeit).
* `openclaw plugins install <npm-spec>` verwendet `npm pack`, entpackt nach `~/.openclaw/extensions/<id>/` und aktiviert das Plugin in der Konfiguration.
* Stabilität des Konfigurationsschlüssels: Scoped-Pakete werden für `plugins.entries.*` auf die **unscoped** ID normalisiert.

<div id="example-plugin-voice-call">
  ## Beispiel-Plugin: Voice Call
</div>

Dieses Repo enthält ein Voice-Call-Plugin (Twilio oder Log‑Fallback):

* Quelle: `extensions/voice-call`
* Fähigkeit: `skills/voice-call`
* CLI: `openclaw voicecall start|status`
* Tool: `voice_call`
* RPC: `voicecall.start`, `voicecall.status`
* Config (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from` (optional `statusCallbackUrl`, `twimlUrl`)
* Config (dev): `provider: "log"` (kein Netzwerk)

Siehe [Voice Call](/de/plugins/voice-call) und `extensions/voice-call/README.md` für Einrichtung und Verwendung.

<div id="safety-notes">
  ## Sicherheitshinweise
</div>

Plugins laufen im selben Prozess wie das Gateway. Behandle sie als vertrauenswürdigen Code:

* Installiere nur Plugins, denen du vertraust.
* Bevorzuge die Allowlist `plugins.allow`.
* Starte das Gateway nach Änderungen neu.

<div id="testing-plugins">
  ## Plugins testen
</div>

Plugins können (und sollten) Tests mitliefern:

* Repo-interne Plugins können Vitest-Tests unter `src/**` ablegen (Beispiel: `src/plugins/voice-call.plugin.test.ts`).
* Separat veröffentlichte Plugins sollten ihre eigene CI (lint/build/test) ausführen und prüfen, dass `openclaw.extensions` auf den gebauten Entry Point (`dist/index.js`) verweist.