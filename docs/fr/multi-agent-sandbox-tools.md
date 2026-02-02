---
summary: "Restrictions de sandbox et d’outils par agent, priorités et exemples"
title: "Sandbox multi-agent & outils"
read_when: "Vous souhaitez une sandbox par agent ou des politiques d’autorisation/interdiction d’outils par agent dans un Gateway multi-agent."
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # Configuration du sandbox multi-agents et des outils
</div>

<div id="overview">
  ## Vue d’ensemble
</div>

Chaque agent dans une configuration multi-agent peut désormais avoir :

* **Configuration de sandbox** (`agents.list[].sandbox` remplace `agents.defaults.sandbox`)
* **Restrictions d’outils** (`tools.allow` / `tools.deny`, plus `agents.list[].tools`)

Cela vous permet d’exécuter plusieurs agents avec différents profils de sécurité :

* Assistant personnel avec accès complet
* Agents familiaux/professionnels avec outils restreints
* Agents accessibles au public dans des sandboxes

`setupCommand` relève de `sandbox.docker` (global ou par agent) et s’exécute une fois
lorsque le conteneur est créé.

L’authentification est gérée par agent : chaque agent lit dans son propre magasin d’authentification `agentDir`, situé à :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants ne sont **pas** partagés entre les agents. Ne réutilisez jamais `agentDir` entre plusieurs agents.
Si vous voulez partager des identifiants, copiez `auth-profiles.json` dans le `agentDir` de l’autre agent.

Pour comprendre le comportement du sandbox à l’exécution, voir [Sandboxing](/fr/gateway/sandboxing).
Pour diagnostiquer « pourquoi est-ce bloqué ? », voir [Sandbox vs Tool Policy vs Elevated](/fr/gateway/sandbox-vs-tool-policy-vs-elevated) et `openclaw sandbox explain`.

***

<div id="configuration-examples">
  ## Exemples de configuration
</div>

<div id="example-1-personal-restricted-family-agent">
  ### Exemple 1 : Agent personnel et familial à accès restreint
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Résultat :**

* agent `main` : s’exécute sur l’hôte, accès complet aux outils
* agent `family` : s’exécute dans Docker (un conteneur par agent), ne dispose que de l’outil `read`

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### Exemple 2 : Agent de travail avec sandbox partagée
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### Exemple 2b : profil de codage global + agent dédié à la messagerie
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Résultat :**

* les agents par défaut disposent d’outils de développement
* l’agent `support` est limité à la messagerie (+ outil Slack)

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### Exemple 3 : modes de sandbox distincts par agent
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // Remplacement : public toujours en sandbox
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## Ordre de priorité de la configuration
</div>

Lorsque des configurations globales (`agents.defaults.*`) et spécifiques à un agent (`agents.list[].*`) sont définies :

<div id="sandbox-config">
  ### Configuration de la sandbox
</div>

Les paramètres spécifiques à un agent remplacent les paramètres globaux :

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Remarques :**

* `agents.list[].sandbox.{docker,browser,prune}.*` a priorité sur `agents.defaults.sandbox.{docker,browser,prune}.*` pour cet agent (ignoré lorsque la portée du sandbox est évaluée à `"shared"`).

<div id="tool-restrictions">
  ### Restrictions sur les outils
</div>

L’ordre de filtrage est :

1. **Profil d’outil** (`tools.profile` ou `agents.list[].tools.profile`)
2. **Profil d’outil du fournisseur** (`tools.byProvider[provider].profile` ou `agents.list[].tools.byProvider[provider].profile`)
3. **Politique globale des outils** (`tools.allow` / `tools.deny`)
4. **Politique d’outils du fournisseur** (`tools.byProvider[provider].allow/deny`)
5. **Politique d’outils spécifique à l’agent** (`agents.list[].tools.allow/deny`)
6. **Politique du fournisseur pour l’agent** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Politique d’outils de la sandbox** (`tools.sandbox.tools` ou `agents.list[].tools.sandbox.tools`)
8. **Politique d’outils des sous-agents** (`tools.subagents.tools`, le cas échéant)

Chaque niveau peut restreindre davantage les outils, mais ne peut pas réautoriser des outils refusés par les niveaux précédents.
Si `agents.list[].tools.sandbox.tools` est défini, il remplace `tools.sandbox.tools` pour cet agent.
Si `agents.list[].tools.profile` est défini, il remplace `tools.profile` pour cet agent.
Les clés d’outils liées au fournisseur acceptent soit `provider` (par ex. `google-antigravity`), soit `provider/model` (par ex. `openai/gpt-5.2`).

<div id="tool-groups-shorthands">
  ### Groupes d&#39;outils (abréviations)
</div>

Les politiques d’outils (globales, par agent, par sandbox) prennent en charge les entrées `group:*` qui correspondent à plusieurs outils concrets :

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: tous les outils OpenClaw intégrés (hors plugins de fournisseurs)

<div id="elevated-mode">
  ### Mode élevé
</div>

`tools.elevated` est la base globale (liste d’autorisation basée sur l’expéditeur). `agents.list[].tools.elevated` peut restreindre davantage l’accès en mode élevé pour des agents spécifiques (les deux doivent l’autoriser).

Stratégies d’atténuation :

* Refuser `exec` pour les agents non fiables (`agents.list[].tools.deny: ["exec"]`)
* Éviter de mettre sur liste d’autorisation des expéditeurs qui routent vers des agents restreints
* Désactiver l’élévation globalement (`tools.elevated.enabled: false`) si vous ne voulez que de l’exécution en sandbox
* Désactiver l’élévation par agent (`agents.list[].tools.elevated.enabled: false`) pour les profils sensibles

***

<div id="migration-from-single-agent">
  ## Migration à partir d’un agent unique
</div>

**Avant (agent unique) :**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Après (plusieurs agents avec des profils différents) :**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Les anciennes configurations `agent.*` sont migrées par `openclaw doctor` ; privilégiez désormais `agents.defaults` + `agents.list`.

***

<div id="tool-restriction-examples">
  ## Exemples de restrictions sur les outils
</div>

<div id="read-only-agent">
  ### Agent en lecture seule
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### Agent d&#39;exécution sûr (sans modification de fichiers)
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### Agent dédié uniquement à la communication
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## Erreur fréquente : &quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` est basé sur `session.mainKey` (par défaut `"main"`),
et non sur l’identifiant de l’agent. Les sessions de groupe/canal obtiennent toujours leurs
propres clés ; elles sont donc considérées comme &quot;non-main&quot; et seront exécutées dans un sandbox. Si vous
voulez qu’un agent ne soit jamais exécuté dans un sandbox, définissez `agents.list[].sandbox.mode: "off"`.

***

<div id="testing">
  ## Tests
</div>

Après avoir configuré la sandbox multi-agent et les outils :

1. **Vérifier la résolution des agents :**
   ```exec
   openclaw agents list --bindings
   ```

2. **Vérifier les conteneurs de la sandbox :**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Tester les restrictions d’outils :**
   * Envoyer un message nécessitant des outils soumis à des restrictions
   * Vérifier que l’agent ne peut pas utiliser les outils interdits

4. **Surveiller les journaux :**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### Agent non exécuté dans un sandbox malgré `mode: "all"`
</div>

* Vérifiez si un `agents.defaults.sandbox.mode` global est défini et le surdéfinit
* La configuration spécifique à l&#39;agent prend le pas, définissez donc `agents.list[].sandbox.mode: "all"`

<div id="tools-still-available-despite-deny-list">
  ### Outils toujours disponibles malgré la liste de blocage
</div>

* Vérifiez l’ordre de filtrage des outils : global → agent → sandbox → sous-agent
* Chaque niveau ne peut qu’ajouter des restrictions, jamais en retirer
* Vérifiez via les journaux : `[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### Conteneur non isolé par agent
</div>

* Définissez `scope: "agent"` dans la configuration de sandbox spécifique à l’agent
* La valeur par défaut est `"session"`, ce qui crée un conteneur par session

***

<div id="see-also">
  ## Voir aussi
</div>

* [Routage multi‑agents](/fr/concepts/multi-agent)
* [Configuration de la sandbox](/fr/gateway/configuration#agentsdefaults-sandbox)
* [Gestion des sessions](/fr/concepts/session)