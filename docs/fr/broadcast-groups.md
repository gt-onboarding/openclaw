---
title: Groupes de diffusion
summary: "Diffuser un message WhatsApp à plusieurs agents"
read_when:
  - Configuration des groupes de diffusion
  - Débogage des réponses multi-agents dans WhatsApp
status: experimental
---

<div id="broadcast-groups">
  # Groupes de diffusion
</div>

**Statut :** Expérimental\
**Version :** Introduit dans la version 2026.1.9

<div id="overview">
  ## Vue d’ensemble
</div>

Les groupes de diffusion permettent à plusieurs agents de traiter et de répondre simultanément au même message. Ils vous permettent de créer des équipes d’agents spécialisés qui travaillent ensemble dans un seul groupe WhatsApp ou en message privé (DM) — tout en utilisant un seul numéro de téléphone.

Portée actuelle : **WhatsApp uniquement** (canal web).

Les groupes de diffusion sont évalués après les listes d’autorisation de canal et les règles d’activation de groupe. Dans les groupes WhatsApp, cela signifie que les diffusions ont lieu lorsque OpenClaw répondrait normalement (par exemple lors d’une mention, en fonction des paramètres de votre groupe).

<div id="use-cases">
  ## Cas d’usage
</div>

<div id="1-specialized-agent-teams">
  ### 1. Équipes d&#39;agents spécialisés
</div>

Déployez plusieurs agents avec des responsabilités précises et ciblées :

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Chaque agent traite le même message et apporte son point de vue spécialisé.

<div id="2-multi-language-support">
  ### 2. Prise en charge multilingue
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. Flux de travail pour l’assurance qualité
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (fournit la réponse)
  - QAAgent (examine la qualité, ne répond que si des problèmes sont détectés)
```

<div id="4-task-automation">
  ### 4. Automatisation des tâches
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (met à jour la base de données de tâches)
  - TimeLogger (enregistre le temps passé)
  - ReportGenerator (crée des résumés)
```

<div id="configuration">
  ## Configuration
</div>

<div id="basic-setup">
  ### Configuration de base
</div>

Ajoutez une section de premier niveau `broadcast` (à côté de `bindings`). Les clés correspondent aux identifiants WhatsApp des pairs :

* discussions de groupe : JID de groupe (par ex. `120363403215116621@g.us`)
* messages directs (DM) : numéro de téléphone au format E.164 (par ex. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Résultat :** Chaque fois qu’OpenClaw répondra dans cette discussion, il exécutera les trois agents.

<div id="processing-strategy">
  ### Stratégie de traitement
</div>

Définissez la façon dont les agents traitent les messages :

<div id="parallel-default">
  #### Parallèle (par défaut)
</div>

Tous les agents traitent en parallèle :

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### Séquentiel
</div>

Les agents s’exécutent dans l’ordre (chacun attend que le précédent ait terminé) :

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### Exemple complet
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## Fonctionnement
</div>

<div id="message-flow">
  ### Flux des messages
</div>

1. Un **message entrant** arrive dans un groupe WhatsApp
2. **Contrôle de broadcast** : le système vérifie si l’ID du pair est dans `broadcast`
3. **S’il est dans la liste broadcast** :
   * Tous les agents listés traitent le message
   * Chaque agent a sa propre clé de session et un contexte isolé
   * Les agents traitent le message en parallèle (par défaut) ou séquentiellement
4. **S’il n’est pas dans la liste broadcast** :
   * Le routage normal s’applique (premier binding correspondant)

Remarque : les groupes de broadcast ne contournent pas la liste d’autorisation du canal ni les règles d’activation du groupe (mentions/commandes/etc.). Ils changent uniquement *quels agents s’exécutent* lorsqu’un message est éligible au traitement.

<div id="session-isolation">
  ### Isolation des sessions
</div>

Chaque agent dans un groupe de diffusion maintient strictement séparés :

* **Clés de session** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
* **Historique de conversation** (l&#39;agent ne voit pas les messages des autres agents)
* **Espace de travail** (sandboxes séparés si configurés)
* **Accès aux outils** (listes d&#39;autorisation/interdiction différentes)
* **Mémoire/contexte** (IDENTITY.md, SOUL.md, etc. séparés)
* **Tampon de contexte de groupe** (messages récents du groupe utilisés pour le contexte), qui est partagé par pair, de sorte que tous les agents de diffusion voient le même contexte lorsqu&#39;ils sont déclenchés

Cela permet à chaque agent d&#39;avoir :

* Des personnalités différentes
* Des droits d&#39;accès aux outils différents (par exemple, lecture seule vs lecture-écriture)
* Des modèles différents (par exemple, opus vs sonnet)
* Des ensembles de compétences différents installés

<div id="example-isolated-sessions">
  ### Exemple : sessions isolées
</div>

Dans le groupe `120363403215116621@g.us` avec les agents `["alfred", "baerbel"]` :

**Contexte d&#39;Alfred :**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contexte de Bärbel :**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [message utilisateur, réponses précédentes de baerbel]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

<div id="best-practices">
  ## Bonnes pratiques
</div>

<div id="1-keep-agents-focused">
  ### 1. Gardez les Agents concentrés
</div>

Concevez chaque agent avec une responsabilité unique et clairement définie :

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Correct :** Chaque agent a un seul rôle
❌ **À éviter :** Un seul agent générique &quot;dev-helper&quot;

<div id="2-use-descriptive-names">
  ### 2. Utilisez des noms explicites
</div>

Indiquez clairement ce que fait chaque agent :

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. Configurer différents accès aux outils
</div>

Donnez aux agents uniquement les outils dont ils ont besoin :

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // Lecture-écriture
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. Surveillez les performances
</div>

Avec de nombreux agents, envisagez :

* D’utiliser `"strategy": "parallel"` (par défaut) pour de meilleures performances
* De limiter les groupes de diffusion à 5 à 10 agents
* D’utiliser des modèles plus rapides pour les agents les plus simples

<div id="5-handle-failures-gracefully">
  ### 5. Gérer correctement les défaillances
</div>

Les agents peuvent échouer indépendamment. L&#39;erreur d&#39;un agent n&#39;empêche pas les autres de continuer à fonctionner :

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

<div id="compatibility">
  ## Compatibilité
</div>

<div id="providers">
  ### Fournisseurs
</div>

Les groupes de diffusion fonctionnent actuellement avec :

* ✅ WhatsApp (pris en charge)
* 🚧 Telegram (prévu)
* 🚧 Discord (prévu)
* 🚧 Slack (prévu)

<div id="routing">
  ### Routage
</div>

Les groupes de diffusion fonctionnent en complément du routage existant :

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: seul alfred répond (routage normal)
* `GROUP_B`: agent1 ET agent2 répondent (diffusion)

**Priorité :** `broadcast` a priorité sur `bindings`.

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="agents-not-responding">
  ### Agents qui ne répondent pas
</div>

**Vérifications :**

1. Les ID d’agent existent dans `agents.list`
2. Le format de l’ID de pair est correct (par ex. `120363403215116621@g.us`)
3. Les agents ne figurent pas dans des listes de blocage

**Débogage :**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### Un seul Agent répond
</div>

**Cause :** l’ID du pair peut se trouver dans `bindings` mais pas dans `broadcast`.

**Correctif :** ajoutez cet ID à la configuration `broadcast` ou supprimez-le de `bindings`.

<div id="performance-issues">
  ### Problèmes de performance
</div>

**En cas de lenteur avec de nombreux agents :**

* Réduisez le nombre d’agents par groupe
* Utilisez des modèles plus légers (sonnet au lieu d’opus)
* Vérifiez le temps de démarrage du sandbox

<div id="examples">
  ## Exemples
</div>

<div id="example-1-code-review-team">
  ### Exemple 1 : Équipe de revue de code
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**L&#39;utilisateur envoie :** Extrait de code
**Réponses :**

* code-formatter: « Indentation corrigée et annotations de type ajoutées »
* security-scanner: « ⚠️ Vulnérabilité d&#39;injection SQL à la ligne 12 »
* test-coverage: « Couverture à 45 %, tests manquants pour les cas d&#39;erreur »
* docs-checker: « Docstring manquante pour la fonction `process_data` »

<div id="example-2-multi-language-support">
  ### Exemple 2 : prise en charge multilingue
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## Référence de l’API
</div>

<div id="config-schema">
  ### Schéma de configuration
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### Champs
</div>

* `strategy` (facultatif) : Comment traiter les agents
  * `"parallel"` (par défaut) : Tous les agents sont traités simultanément
  * `"sequential"` : Les agents sont traités dans l’ordre du tableau

* `[peerId]` : JID de groupe WhatsApp, numéro E.164 ou autre identifiant de pair
  * Valeur : Tableau d’identifiants d’agent qui doivent traiter les messages

<div id="limitations">
  ## Limitations
</div>

1. **Nombre maximal d&#39;agents :** Pas de limite stricte, mais au-delà de 10 agents les performances peuvent diminuer
2. **Contexte partagé :** Les agents ne voient pas les réponses des autres (comportement prévu)
3. **Ordre des messages :** Les réponses parallèles peuvent arriver dans n&#39;importe quel ordre
4. **Limites de débit :** Tous les agents comptent dans les limites de débit WhatsApp

<div id="future-enhancements">
  ## Améliorations futures
</div>

Fonctionnalités prévues :

* [ ] Mode de contexte partagé (les agents voient les réponses les uns des autres)
* [ ] Coordination des agents (les agents peuvent s&#39;envoyer des signaux)
* [ ] Sélection dynamique des agents (choisir les agents en fonction du contenu du message)
* [ ] Priorités des agents (certains agents répondent avant les autres)

<div id="see-also">
  ## Voir aussi
</div>

* [Configuration multi-agents](/fr/multi-agent-sandbox-tools)
* [Configuration de routage](/fr/concepts/channel-routing)
* [Gestion des sessions](/fr/concepts/sessions)