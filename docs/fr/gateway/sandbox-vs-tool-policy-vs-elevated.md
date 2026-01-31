---
title: Sandbox vs politique d’outils vs Elevated
summary: "Pourquoi un outil est bloqué : environnement d’exécution de la sandbox, politique d’autorisation/interdiction des outils et garde‑fous d’exécution pour Elevated"
read_when: "Vous vous heurtez à la « sandbox jail » ou voyez un refus d’outil ou d’Elevated et vous voulez connaître la clé de configuration précise à modifier."
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # Sandbox vs Tool Policy vs Elevated
</div>

OpenClaw propose trois mécanismes liés (mais distincts) :

1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) détermine **où les outils s’exécutent** (Docker vs hôte).
2. **Tool policy** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) détermine **quels outils sont disponibles/autorisés**.
3. **Elevated** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) est une **échappatoire réservée à l’exécution** pour s’exécuter sur l’hôte lorsque vous êtes en sandbox.

<div id="quick-debug">
  ## Débogage rapide
</div>

Utilisez l’inspecteur pour voir ce que fait *réellement* OpenClaw :

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Il affiche :

* mode de sandbox effectif / portée / accès à l’espace de travail
* si la session est actuellement en sandbox (main vs non-main)
* règles effectives d’autorisation/interdiction des outils dans le sandbox (et si elles viennent de l’agent/global/default)
* gates d’élévation et chemins de clés « fix-it »

<div id="sandbox-where-tools-run">
  ## Sandbox : où s’exécutent les outils
</div>

L’utilisation du sandbox est contrôlée par `agents.defaults.sandbox.mode` :

* `"off"` : tout s’exécute sur l’hôte.
* `"non-main"` : seules les sessions non principales sont exécutées dans le sandbox (source fréquente de « surprises » pour les groupes/canaux).
* `"all"` : tout est exécuté dans le sandbox.

Voir [Sandboxing](/fr/gateway/sandboxing) pour la matrice complète (portée, montages de l’espace de travail, images).

<div id="bind-mounts-security-quick-check">
  ### Points de montage bind (vérification rapide de la sécurité)
</div>

* `docker.binds` *perce* le système de fichiers du sandbox : tout ce que vous montez est visible dans le conteneur avec le mode que vous définissez (`:ro` ou `:rw`).
* Par défaut, le mode est lecture-écriture si vous omettez le mode ; privilégiez `:ro` pour le code source et les secrets.
* `scope: "shared"` ignore les binds par agent (seuls les binds globaux sont pris en compte).
* Monter `/var/run/docker.sock` revient effectivement à donner le contrôle de l’hôte au sandbox ; ne le faites que de manière intentionnelle.
* L’accès à l’espace de travail (`workspaceAccess: "ro"`/`"rw"`) est indépendant des modes de bind.

<div id="tool-policy-which-tools-existare-callable">
  ## Stratégie d’outils : quels outils existent/sont appelables
</div>

Deux couches sont importantes :

* **Profil d’outils** : `tools.profile` et `agents.list[].tools.profile` (liste d’autorisation de base)
* **Profil d’outils du fournisseur** : `tools.byProvider[provider].profile` et `agents.list[].tools.byProvider[provider].profile`
* **Stratégie d’outils globale/par agent** : `tools.allow`/`tools.deny` et `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **Stratégie d’outils du fournisseur** : `tools.byProvider[provider].allow/deny` et `agents.list[].tools.byProvider[provider].allow/deny`
* **Stratégie d’outils de sandbox** (ne s’applique qu’en mode sandbox) : `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` et `agents.list[].tools.sandbox.tools.*`

Règles générales :

* `deny` gagne toujours.
* Si `allow` n’est pas vide, tout le reste est considéré comme bloqué.
* La stratégie d’outils est la barrière stricte : `/exec` ne peut pas passer outre un outil `exec` refusé.
* `/exec` ne change que les valeurs par défaut de la session pour les expéditeurs autorisés ; cela n’accorde pas l’accès aux outils.

Les clés d’outils de fournisseur acceptent soit `provider` (par ex. `google-antigravity`), soit `provider/model` (par ex. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Groupes d’outils (raccourcis)
</div>

Les politiques d’outils (globales, agent, sandbox) prennent en charge les entrées `group:*` qui correspondent à plusieurs outils :

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

Groupes disponibles :

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tous les outils OpenClaw intégrés (à l’exclusion des plugins de fournisseurs)

<div id="elevated-exec-only-run-on-host">
  ## Élevé : exécution uniquement (`exec`) « sur l’hôte »
</div>

Le mode élevé ne donne **pas** d’outils supplémentaires ; il n’affecte que `exec`.

* Si vous êtes en sandbox, `/elevated on` (ou `exec` avec `elevated: true`) s’exécute sur l’hôte (les approbations peuvent toujours s’appliquer).
* Utilisez `/elevated full` pour ignorer les approbations d’exec pour la session.
* Si vous êtes déjà en exécution directe, le mode élevé est essentiellement un no-op (toujours soumis aux contrôles).
* Le mode élevé n’est **pas** limité à une compétence et n’outrepasse **pas** les règles d’autorisation/interdiction d’outils.
* `/exec` est distinct du mode élevé. Il ne modifie que les valeurs par défaut d’exec par session pour les expéditeurs autorisés.

Garde-fous :

* Activation : `tools.elevated.enabled` (et éventuellement `agents.list[].tools.elevated.enabled`)
* Listes d’autorisation des expéditeurs : `tools.elevated.allowFrom.<provider>` (et éventuellement `agents.list[].tools.elevated.allowFrom.<provider>`)

Voir [Mode Elevated](/fr/tools/elevated).

<div id="common-sandbox-jail-fixes">
  ## Correctifs courants pour la « prison sandbox »
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### « Outil X bloqué par la politique de sandbox des outils »
</div>

Clés de résolution (choisissez-en une) :

* Désactiver le sandbox : `agents.defaults.sandbox.mode=off` (ou par agent `agents.list[].sandbox.mode=off`)
* Autoriser l’outil dans le sandbox :
  * le retirer de `tools.sandbox.tools.deny` (ou par agent `agents.list[].tools.sandbox.tools.deny`)
  * ou l’ajouter à `tools.sandbox.tools.allow` (ou à la liste d’autorisation par agent)

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### « Je pensais que c’était main, pourquoi est-ce en sandbox ? »
</div>

En mode `"non-main"`, les clés de groupe/canal ne sont *pas* main. Utilisez la clé de session main (affichée par `sandbox explain`) ou passez le mode sur `"off"`.