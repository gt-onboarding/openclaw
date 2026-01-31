---
title: Plugin
summary: "Plugins/extensions OpenClaw : découverte, configuration et sécurité"
read_when:
  - Ajout ou modification de plugins/extensions
  - Documentation des règles d’installation ou de chargement de plugins
---

<div id="plugins-extensions">
  # Plugins (extensions)
</div>

<div id="quick-start-new-to-plugins">
  ## Démarrage rapide (vous débutez avec les plugins ?)
</div>

Un plugin est simplement un **petit module de code** qui étend OpenClaw avec des
fonctionnalités supplémentaires (commandes, outils et Gateway RPC).

La plupart du temps, vous utiliserez des plugins lorsque vous aurez besoin d’une fonctionnalité qui n’est pas encore intégrée
au cœur d’OpenClaw (ou si vous souhaitez garder certaines fonctionnalités optionnelles en dehors de votre
installation principale).

Voie rapide :

1. Vérifiez ce qui est déjà chargé :

```bash
openclaw plugins list
```

2. Installez un plugin officiel (par exemple : Voice Call) :

```bash
openclaw plugins install @openclaw/voice-call
```

3. Redémarrez le Gateway, puis configurez-le dans `plugins.entries.<id>.config`.

Voir [Voice Call](/fr/plugins/voice-call) pour un exemple concret de plugin.

<div id="available-plugins-official">
  ## Plugins disponibles (officiels)
</div>

* Microsoft Teams est uniquement disponible sous forme de plugin depuis 2026.1.15 ; installez `@openclaw/msteams` si vous utilisez Teams.
* Memory (Core) — plugin de recherche de mémoire intégré (activé par défaut via `plugins.slots.memory`)
* Memory (LanceDB) — plugin de mémoire à long terme intégré (rappel/capture automatiques ; définissez `plugins.slots.memory = "memory-lancedb"`)
* [Voice Call](/fr/plugins/voice-call) — `@openclaw/voice-call`
* [Zalo Personal](/fr/plugins/zalouser) — `@openclaw/zalouser`
* [Matrix](/fr/channels/matrix) — `@openclaw/matrix`
* [Nostr](/fr/channels/nostr) — `@openclaw/nostr`
* [Zalo](/fr/channels/zalo) — `@openclaw/zalo`
* [Microsoft Teams](/fr/channels/msteams) — `@openclaw/msteams`
* Google Antigravity OAuth (authentification fournisseur) — fourni sous la forme `google-antigravity-auth` (désactivé par défaut)
* Gemini CLI OAuth (authentification fournisseur) — fourni sous la forme `google-gemini-cli-auth` (désactivé par défaut)
* Qwen OAuth (authentification fournisseur) — fourni sous la forme `qwen-portal-auth` (désactivé par défaut)
* Copilot Proxy (authentification fournisseur) — passerelle locale Copilot Proxy pour VS Code ; distincte de la connexion appareil intégrée `github-copilot` (fournie, désactivée par défaut)

Les plugins OpenClaw sont des **modules TypeScript** chargés à l’exécution via jiti. **La validation de configuration n’exécute pas le code du plugin** ; elle utilise à la place le manifeste du plugin et le JSON Schema. Voir [Plugin manifest](/fr/plugins/manifest).

Les plugins peuvent enregistrer :

* des méthodes RPC du Gateway
* des gestionnaires HTTP du Gateway
* des outils d’agent
* des commandes CLI
* des services d’arrière‑plan
* une validation de configuration optionnelle
* des **compétences** (en référençant des répertoires `skills` dans le manifeste du plugin)
* des **commandes de réponse automatique** (exécutées sans invoquer l’agent IA)

Les plugins s’exécutent **dans le même processus** que le Gateway ; ils doivent donc être considérés comme du code de confiance.
Guide d’écriture d’outils : [Plugin agent tools](/fr/plugins/agent-tools).

<div id="runtime-helpers">
  ## Utilitaires d&#39;exécution
</div>

Les plugins peuvent accéder à certains utilitaires de base via `api.runtime`. Pour la synthèse vocale (TTS) en téléphonie :

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes :

* Utilise la configuration TTS centrale `messages.tts` (OpenAI ou ElevenLabs).
* Retourne un buffer audio PCM + un taux d’échantillonnage. Les plugins doivent rééchantillonner et encoder pour les fournisseurs.
* Edge TTS n’est pas pris en charge pour la téléphonie.

<div id="discovery-precedence">
  ## Découverte et ordre de priorité
</div>

OpenClaw scanne, dans cet ordre :

1. Chemins de configuration

* `plugins.load.paths` (fichier ou répertoire)

2. Extensions de l’espace de travail

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. Extensions globales

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. Extensions intégrées (fournies avec OpenClaw, **désactivées par défaut**)

* `<openclaw>/extensions/*`

Les plugins intégrés doivent être activés explicitement via `plugins.entries.<id>.enabled`
ou `openclaw plugins enable <id>`. Les plugins installés sont activés par défaut,
mais peuvent être désactivés de la même manière.

Chaque plugin doit inclure un fichier `openclaw.plugin.json` à sa racine. Si un chemin
pointe vers un fichier, la racine du plugin est le répertoire de ce fichier et doit contenir
le manifeste.

Si plusieurs plugins aboutissent au même ID, la première correspondance dans l’ordre ci-dessus
l’emporte et les copies de priorité inférieure sont ignorées.

<div id="package-packs">
  ### Packs de packages npm
</div>

Un répertoire de plugin peut contenir un fichier `package.json` avec la clé `openclaw.extensions` :

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Chaque entrée devient un plugin. Si le pack répertorie plusieurs extensions, l’identifiant du plugin
devient `name/<fileBase>`.

Si votre plugin importe des dépendances npm, installez-les dans ce répertoire afin que
`node_modules` soit disponible (`npm install` / `pnpm install`).

<div id="channel-catalog-metadata">
  ### Métadonnées du catalogue de canaux
</div>

Les plugins de canal peuvent déclarer des métadonnées d’onboarding via `openclaw.channel` et
des indications d’installation via `openclaw.install`. Cela permet de garder le catalogue principal exempt de données.

Exemple :

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
      "blurb": "Chat auto-hébergé via les bots webhook Nextcloud Talk.",
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

OpenClaw peut également fusionner des **catalogues de canaux externes** (par
exemple, un export de registre MPM). Déposez un fichier JSON dans l’un des
emplacements suivants :

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

Ou définissez `OPENCLAW_PLUGIN_CATALOG_PATHS` (ou `OPENCLAW_MPM_CATALOG_PATHS`)
vers un ou plusieurs fichiers JSON (séparés par des virgules, des
points-virgules ou selon la convention `PATH`). Chaque fichier doit contenir `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

<div id="plugin-ids">
  ## Identifiants de plugin
</div>

Identifiants de plugin par défaut :

* Plugins fournis sous forme de paquet : champ `name` de `package.json`
* Fichier autonome : nom de base du fichier (`~/.../voice-call.ts` → `voice-call`)

Si un plugin exporte `id`, OpenClaw l’utilise mais affiche un avertissement lorsqu’il ne correspond pas à l’ID configuré.

<div id="config">
  ## Configuration
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

Champs :

* `enabled`: interrupteur général (par défaut : true)
* `allow`: liste d’autorisation (facultatif)
* `deny`: liste de blocage (facultatif ; la règle `deny` est prioritaire)
* `load.paths`: fichiers/répertoires de plugin supplémentaires
* `entries.<id>`: commutateurs et configuration par plugin

Les modifications de configuration **nécessitent un redémarrage du Gateway**.

Règles de validation strictes :

* Les identifiants de plugin inconnus dans `entries`, `allow`, `deny` ou `slots` sont des **erreurs**.
* Les clés `channels.<id>` inconnues sont des **erreurs**, sauf si un manifeste de plugin déclare l’identifiant de canal.
* La configuration du plugin est validée à l’aide du schéma JSON intégré dans
  `openclaw.plugin.json` (`configSchema`).
* Si un plugin est désactivé, sa configuration est conservée et un **avertissement** est émis.

<div id="plugin-slots-exclusive-categories">
  ## Emplacements de plugin (catégories exclusives)
</div>

Certaines catégories de plugin sont **exclusives** (un seul actif à la fois). Utilisez
`plugins.slots` pour sélectionner quel plugin occupe l’emplacement :

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // ou "none" pour désactiver les plugins de mémoire
    }
  }
}
```

Si plusieurs plugins déclarent `kind: "memory"`, seul le plugin sélectionné est chargé. Les autres
sont désactivés et signalés dans les diagnostics.

<div id="control-ui-schema-labels">
  ## Control UI (schéma + libellés)
</div>

Le Control UI utilise `config.schema` (JSON Schema + `uiHints`) pour afficher des formulaires plus ergonomiques.

OpenClaw complète `uiHints` à l’exécution en fonction des plugins détectés :

* Ajoute des libellés par plugin pour `plugins.entries.<id>` / `.enabled` / `.config`
* Fusionne les indications optionnelles fournies par le plugin pour les champs de configuration sous :
  `plugins.entries.<id>.config.<field>`

Si vous voulez que les champs de configuration de votre plugin affichent de bons libellés/espaces réservés (et marquent les secrets comme sensibles),
fournissez des `uiHints` en plus de votre JSON Schema dans le manifeste du plugin.

Exemple :

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
    "region": { "label": "Région", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copier un fichier/répertoire local dans ~/.openclaw/extensions/<id>
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

`plugins update` ne fonctionne que pour les installations npm suivies dans `plugins.installs`.

Les plugins peuvent également enregistrer leurs propres commandes de premier niveau (par exemple : `openclaw voicecall`).

<div id="plugin-api-overview">
  ## API des plugins (aperçu)
</div>

Les plugins exportent soit :

* Une fonction : `(api) => { ... }`
* Un objet : `{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## Hooks de plugin
</div>

Les plugins peuvent fournir des hooks et les enregistrer à l&#39;exécution. Cela permet à un plugin d&#39;embarquer une logique d&#39;automatisation pilotée par les événements sans installer un pack de hooks distinct.

<div id="example">
  ### Exemple
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

Remarques :

* Les répertoires de hooks suivent la structure de hook standard (`HOOK.md` + `handler.ts`).
* Les règles d’éligibilité des hooks s’appliquent toujours (prérequis OS/binaires/env/config).
* Les hooks gérés par des plugins apparaissent dans `openclaw hooks list` avec `plugin:<id>`.
* Vous ne pouvez pas activer ou désactiver les hooks gérés par des plugins via `openclaw hooks` ; activez ou désactivez plutôt le plugin lui-même.

<div id="provider-plugins-model-auth">
  ## Plugins de fournisseurs (authentification des modèles)
</div>

Les plugins peuvent enregistrer des flux d’**authentification de fournisseurs de modèles** afin que les utilisateurs puissent configurer OAuth ou des clés API directement dans OpenClaw (aucun script externe n’est nécessaire).

Enregistrez un fournisseur avec `api.registerProvider(...)`. Chaque fournisseur expose une ou plusieurs méthodes d’authentification (OAuth, clé API, code appareil, etc.). Ces méthodes sont utilisées par :

* `openclaw models auth login --provider &lt;id&gt; [--method &lt;id&gt;]`

Exemple :

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
        // Exécute le flux OAuth et renvoie les profils d'authentification.
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

Remarques :

* `run` reçoit un `ProviderAuthContext` avec les fonctions utilitaires `prompter`, `runtime`,
  `openUrl` et `oauth.createVpsAwareHandlers`.
* Retournez `configPatch` lorsque vous devez ajouter des modèles par défaut ou la configuration du fournisseur.
* Retournez `defaultModel` afin que `--set-default` puisse mettre à jour les valeurs par défaut de l’agent.

<div id="register-a-messaging-channel">
  ### Enregistrer un canal de messagerie
</div>

Les plugins peuvent enregistrer des **plugins de canal** qui se comportent comme des canaux intégrés
(WhatsApp, Telegram, etc.). La configuration du canal est définie sous `channels.<id>` et est
validée par le code de votre plugin de canal.

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

Remarques :

* Placez la configuration sous `channels.<id>` (et non `plugins.entries`).
* `meta.label` est utilisé pour les libellés dans les listes CLI/UI.
* `meta.aliases` ajoute des IDs alternatifs pour la normalisation et les saisies en CLI.
* `meta.preferOver` répertorie les IDs de canal à ignorer pour l’activation automatique lorsque les deux sont configurés.
* `meta.detailLabel` et `meta.systemImage` permettent aux UI d’afficher des libellés/icônes de canal plus détaillés.

<div id="write-a-new-messaging-channel-stepbystep">
  ### Créer un nouveau canal de messagerie (étape par étape)
</div>

Utilisez ceci lorsque vous voulez une **nouvelle interface de chat** (un « canal de messagerie »), et non un fournisseur de modèles.
La documentation des fournisseurs de modèles se trouve sous `/providers/*`.

1. Choisir un id + une structure de configuration

* Toute la configuration des canaux se trouve sous `channels.<id>`.
* Préférez `channels.<id>.accounts.<accountId>` pour les configurations multi‑comptes.

2. Définir les métadonnées du canal

* `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` contrôlent les listes dans la CLI/UI.
* `meta.docsPath` doit pointer vers une page de documentation comme `/channels/<id>`.
* `meta.preferOver` permet à un plugin de remplacer un autre canal (l’activation automatique le privilégie).
* `meta.detailLabel` et `meta.systemImage` sont utilisés par les UI pour le texte détaillé et les icônes.

3. Implémenter les adaptateurs requis

* `config.listAccountIds` + `config.resolveAccount`
* `capabilities` (types de chat, médias, fils de discussion, etc.)
* `outbound.deliveryMode` + `outbound.sendText` (pour l’envoi de base)

4. Ajouter des adaptateurs optionnels selon les besoins

* `setup` (assistant de configuration), `security` (politique de DM), `status` (état/diagnostics)
* `gateway` (démarrage/arrêt/connexion), `mentions`, `threading`, `streaming`
* `actions` (actions de message), `commands` (comportement de commandes natives)

5. Enregistrer le canal dans votre plugin

* `api.registerChannel({ plugin })`

Exemple de configuration minimale :

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

Plugin de canal minimal (uniquement sortant) :

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
      // envoyer `text` à votre canal ici
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Chargez le plugin (répertoire des extensions ou `plugins.load.paths`), redémarrez le Gateway,
puis configurez `channels.<id>` dans votre configuration.

<div id="agent-tools">
  ### Outils d’agent
</div>

Consultez le guide dédié : [Outils d’agent pour les plugins](/fr/plugins/agent-tools).

<div id="register-a-gateway-rpc-method">
  ### Enregistrer une méthode RPC de Gateway
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### Enregistrer les commandes de la CLI
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
  ### Enregistrer des commandes de réponse automatique
</div>

Les plugins peuvent enregistrer des commandes slash personnalisées qui s’exécutent **sans faire appel à l’agent IA**. C’est utile pour des commandes d’activation/désactivation, des contrôles d’état ou des actions rapides qui n’ont pas besoin de traitement par un LLM.

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

Contexte du gestionnaire de commandes :

* `senderId`: L&#39;identifiant de l&#39;expéditeur (si disponible)
* `channel`: Le canal où la commande a été envoyée
* `isAuthorizedSender`: Indique si l&#39;expéditeur est un utilisateur autorisé
* `args`: Arguments passés après la commande (si `acceptsArgs: true`)
* `commandBody`: Le texte complet de la commande
* `config`: La configuration OpenClaw actuelle

Options de commande :

* `name`: Nom de la commande (sans le `/` initial)
* `description`: Texte d&#39;aide affiché dans les listes de commandes
* `acceptsArgs`: Indique si la commande accepte des arguments (par défaut : false). Si false et que des arguments sont fournis, la commande ne sera pas prise en compte et le message sera transmis à d&#39;autres gestionnaires
* `requireAuth`: Indique si un expéditeur autorisé est requis (par défaut : true)
* `handler`: Fonction qui renvoie `{ text: string }` (peut être asynchrone)

Exemple avec autorisation et arguments :

```ts
api.registerCommand({
  name: "setmode",
  description: "Définir le mode du plugin",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Remarques :

* Les commandes de plugins sont traitées **avant** les commandes intégrées et l&#39;agent IA
* Les commandes sont enregistrées globalement et fonctionnent sur tous les canaux
* Les noms de commande ne sont pas sensibles à la casse (`/MyStatus` correspond à `/mystatus`)
* Les noms de commande doivent commencer par une lettre et ne contenir que des lettres, des chiffres, des tirets et des caractères de soulignement (`_`)
* Les noms de commande réservés (comme `help`, `status`, `reset`, etc.) ne peuvent pas être redéfinis par des plugins
* L&#39;enregistrement d&#39;une même commande dans plusieurs plugins échouera avec une erreur de diagnostic

<div id="register-background-services">
  ### Enregistrer les services d’arrière-plan
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
  ## Conventions de nommage
</div>

* Méthodes du Gateway : `pluginId.action` (exemple : `voicecall.status`)
* Outils : `snake_case` (exemple : `voice_call`)
* Commandes CLI : kebab-case ou camelCase, mais évitez toute collision avec les commandes principales (core)

<div id="skills">
  ## Compétences
</div>

Les plugins peuvent inclure une compétence dans le référentiel (`skills/<name>/SKILL.md`).
Activez-la avec `plugins.entries.<id>.enabled` (ou d’autres verrous de configuration) et assurez-vous
qu’elle est présente dans les emplacements de compétences de votre espace de travail ou gérées.

<div id="distribution-npm">
  ## Distribution (npm)
</div>

Mode de distribution recommandé :

* Paquet principal : `openclaw` (ce dépôt)
* Plugins : paquets npm distincts sous `@openclaw/*` (exemple : `@openclaw/voice-call`)

Contrat de publication :

* Le `package.json` du plugin doit inclure `openclaw.extensions` avec un ou plusieurs fichiers d’entrée.
* Les fichiers d’entrée peuvent être en `.js` ou `.ts` (jiti charge le TS à l’exécution).
* `openclaw plugins install <npm-spec>` utilise `npm pack`, extrait le contenu dans `~/.openclaw/extensions/<id>/` et l’active dans la configuration.
* Stabilité des clés de configuration : les paquets *scoped* sont normalisés vers l’identifiant **non scoped** pour `plugins.entries.*`.

<div id="example-plugin-voice-call">
  ## Exemple de plugin : Appel vocal
</div>

Ce dépôt contient un plugin d&#39;appel vocal (Twilio ou journal en mode de repli) :

* Source : `extensions/voice-call`
* Compétence : `skills/voice-call`
* CLI : `openclaw voicecall start|status`
* Outil : `voice_call`
* RPC : `voicecall.start`, `voicecall.status`
* Configuration (Twilio) : `provider: "twilio"` + `twilio.accountSid/authToken/from` (paramètres `statusCallbackUrl`, `twimlUrl` optionnels)
* Configuration (dev) : `provider: "log"` (sans réseau)

Voir [Voice Call](/fr/plugins/voice-call) et `extensions/voice-call/README.md` pour la configuration et l&#39;utilisation.

<div id="safety-notes">
  ## Notes de sécurité
</div>

Les plugins s’exécutent dans le même processus que le Gateway. Considérez-les comme du code digne de confiance :

* Installez uniquement les plugins auxquels vous faites confiance.
* Privilégiez les listes d’autorisation `plugins.allow`.
* Redémarrez le Gateway après les modifications.

<div id="testing-plugins">
  ## Tester les plugins
</div>

Les plugins peuvent (et devraient) être fournis avec des tests :

* Les plugins situés dans le dépôt peuvent placer leurs tests Vitest sous `src/**` (exemple : `src/plugins/voice-call.plugin.test.ts`).
* Les plugins publiés séparément doivent exécuter leur propre CI (lint/build/test) et vérifier que `openclaw.extensions` pointe vers le point d&#39;entrée généré (`dist/index.js`).