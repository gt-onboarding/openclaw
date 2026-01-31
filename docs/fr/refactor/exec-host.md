---
title: Hôte Exec
summary: "Plan de refactorisation : routage de l'hôte Exec, approbations de nœud et exécuteur headless"
read_when:
  - Conception du routage de l'hôte Exec ou des approbations Exec
  - Mise en œuvre du runner de nœud + IPC de l'UI
  - Ajout des modes de sécurité de l'hôte Exec et des commandes slash
---

<div id="exec-host-refactor-plan">
  # Plan de refactorisation de l&#39;hôte d&#39;exécution
</div>

<div id="goals">
  ## Objectifs
</div>

* Ajouter `exec.host` + `exec.security` pour répartir l’exécution entre **sandbox**, **Gateway** et **nœud**.
* Conserver des paramètres par défaut **sûrs** : aucune exécution entre hôtes sauf si elle est explicitement activée.
* Scinder l’exécution en un **service d’exécution headless** avec une UI optionnelle (app macOS) via IPC local.
* Fournir une politique **par agent**, une liste d’autorisation, un mode de confirmation et un lien à un nœud.
* Prendre en charge des **modes de confirmation** qui fonctionnent *avec* ou *sans* liste d’autorisation.
* Multiplateforme : socket Unix + authentification par jeton (parité macOS/Linux/Windows).

<div id="non-goals">
  ## Objectifs hors périmètre
</div>

* Aucune migration d’ancienne liste d’autorisation ni prise en charge de schéma hérité.
* Aucun PTY/streaming pour node exec (sortie agrégée uniquement).
* Aucune nouvelle couche réseau au-delà de la couche Bridge + Gateway existante.

<div id="decisions-locked">
  ## Décisions (verrouillées)
</div>

* **Clés de configuration :** `exec.host` + `exec.security` (surcharge par agent autorisée).
* **Élévation :** conserver `/elevated` comme alias pour l’accès complet au Gateway.
* **Valeur par défaut de Ask :** `on-miss`.
* **Stockage des approbations :** `~/.openclaw/exec-approvals.json` (JSON, aucune migration d’anciens formats).
* **Runner :** service système sans interface (« headless ») ; l’application UI héberge un socket Unix pour les approbations.
* **Identité du nœud :** utiliser le `nodeId` existant.
* **Authentification du socket :** socket Unix + jeton (multiplateforme) ; à scinder plus tard si nécessaire.
* **État de l’hôte du nœud :** `~/.openclaw/node.json` (ID de nœud + jeton d’appairage).
* **Hôte d’exécution macOS :** exécuter `system.run` dans l’application macOS ; le service hôte du nœud transfère les requêtes via IPC local.
* **Pas de helper XPC :** rester sur socket Unix + jeton + contrôles du pair.

<div id="key-concepts">
  ## Concepts clés
</div>

<div id="host">
  ### Hôte
</div>

* `sandbox` : exec Docker (comportement actuel).
* `gateway` : exec sur l&#39;hôte Gateway.
* `node` : exec sur le runner du nœud via Bridge (`system.run`).

<div id="security-mode">
  ### Mode de sécurité
</div>

* `deny` : toujours refuser.
* `allowlist` : n’autoriser que les éléments figurant dans la liste d’autorisation.
* `full` : tout autoriser (équivaut à `elevated`).

<div id="ask-mode">
  ### Mode de demande
</div>

* `off` : ne jamais demander.
* `on-miss` : demander uniquement lorsqu’aucune entrée de la liste d’autorisation ne correspond.
* `always` : demander systématiquement.

Le mode de demande est **indépendant** de la liste d’autorisation ; la liste d’autorisation peut être utilisée avec `always` ou `on-miss`.

<div id="policy-resolution-per-exec">
  ### Résolution de la politique (par exécution)
</div>

1. Déterminer `exec.host` (paramètre de l’outil → surcharge d’agent → valeur globale par défaut).
2. Déterminer `exec.security` et `exec.ask` (même ordre de priorité).
3. Si l’hôte est `sandbox`, procéder à une exécution locale en sandbox.
4. Si l’hôte est `gateway` ou `node`, appliquer la politique de sécurité et de demande sur cet hôte.

<div id="default-safety">
  ## Sécurité par défaut
</div>

* Par défaut `exec.host = sandbox`.
* Par défaut `exec.security = deny` pour `gateway` et `node`.
* Par défaut `exec.ask = on-miss` (pertinent uniquement si la configuration de sécurité l’autorise).
* Si aucune liaison à un nœud n’est définie, **l’agent peut cibler n’importe quel nœud**, mais uniquement si la politique le permet.

<div id="config-surface">
  ## Périmètre de configuration
</div>

<div id="tool-parameters">
  ### Paramètres de l’outil
</div>

* `exec.host` (facultatif) : `sandbox | gateway | node`.
* `exec.security` (facultatif) : `deny | allowlist | full`.
* `exec.ask` (facultatif) : `off | on-miss | always`.
* `exec.node` (facultatif) : ID ou nom de nœud à utiliser lorsque `host=node`.

<div id="config-keys-global">
  ### Clés de configuration (globales)
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node` (liaison au nœud par défaut)

<div id="config-keys-per-agent">
  ### Clés de configuration (pour chaque agent)
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### Alias
</div>

* `/elevated on` = définit `tools.exec.host=gateway`, `tools.exec.security=full` pour la session de l&#39;agent.
* `/elevated off` = rétablit les paramètres d&#39;exécution précédents pour la session de l&#39;agent.

<div id="approvals-store-json">
  ## Stockage des approbations (JSON)
</div>

Chemin : `~/.openclaw/exec-approvals.json`

But :

* Stratégie locale + listes d’autorisation pour l’**hôte d’exécution** (Gateway ou runner de nœud).
* Mécanisme de demande de confirmation de secours lorsqu’aucune UI n’est disponible.
* Identifiants IPC pour les clients UI.

Schéma proposé (v1) :

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notes :

* Aucun ancien format de liste d’autorisation.
* `askFallback` s’applique uniquement lorsque `ask` est requis et qu’aucune UI n’est accessible.
* Permissions de fichier : `0600`.

<div id="runner-service-headless">
  ## Service d’exécution (sans interface graphique)
</div>

<div id="role">
  ### Rôle
</div>

* Appliquer `exec.security` + `exec.ask` en local.
* Exécuter des commandes système et renvoyer le résultat.
* Émettre des événements Bridge pour le cycle de vie d&#39;exec (facultatif mais recommandé).

<div id="service-lifecycle">
  ### Cycle de vie du service
</div>

* Launchd/daemon sur macOS ; service système sur Linux/Windows.
* Le fichier JSON d&#39;approbations est local à l&#39;hôte d&#39;exécution.
* La UI héberge un socket Unix local ; les runners s&#39;y connectent à la demande.

<div id="ui-integration-macos-app">
  ## Intégration de l&#39;UI (application macOS)
</div>

<div id="ipc">
  ### IPC
</div>

* Socket Unix à `~/.openclaw/exec-approvals.sock` (0600).
* Jeton stocké dans `exec-approvals.json` (0600).
* Vérifications du pair : même UID uniquement.
* Challenge/réponse : nonce + HMAC(token, request-hash) pour empêcher les attaques par rejeu.
* TTL courte (p. ex. 10 s) + charge utile maximale + limite de débit.

<div id="ask-flow-macos-app-exec-host">
  ### Flux Ask (hôte d’exécution de l’app macOS)
</div>

1. Le service de nœud reçoit `system.run` du Gateway.
2. Le service de nœud se connecte au socket local et envoie la requête de prompt/exec.
3. L’app valide le pair + le jeton + le HMAC + le TTL, puis affiche une boîte de dialogue si nécessaire.
4. L’app exécute la commande dans le contexte UI et renvoie la sortie.
5. Le service de nœud renvoie la sortie au Gateway.

Si l’UI est indisponible :

* Appliquer `askFallback` (`deny|allowlist|full`).

<div id="diagram-sci">
  ### Schéma (SCI)
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## Identité du nœud + liaison
</div>

* Utilisez le `nodeId` existant issu de l’appairage Bridge.
* Modèle de liaison :
  * `tools.exec.node` restreint l’agent à un nœud spécifique.
  * S’il n’est pas défini, l’agent peut choisir n’importe quel nœud (la politique applique toujours les valeurs par défaut).
* Résolution de la sélection du nœud :
  * correspondance exacte de `nodeId`
  * `displayName` (normalisé)
  * `remoteIp`
  * préfixe de `nodeId` (&gt;= 6 caractères)

<div id="eventing">
  ## Événements
</div>

<div id="who-sees-events">
  ### Qui voit les événements
</div>

* Les événements système sont **au niveau de chaque session** et sont affichés à l’agent au prochain prompt.
* Stockés dans la file d’attente en mémoire du Gateway (`enqueueSystemEvent`).

<div id="event-text">
  ### Texte des événements
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + optional output tail
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### Transport
</div>

Option A (recommandée) :

* Le Runner envoie au Bridge des trames d’événement `exec.started` / `exec.finished`.
* La méthode Gateway `handleBridgeEvent` les convertit en `enqueueSystemEvent`.

Option B :

* L’outil `exec` de Gateway gère directement le cycle de vie (synchrone uniquement).

<div id="exec-flows">
  ## Flux d&#39;exécution
</div>

<div id="sandbox-host">
  ### Hôte sandbox
</div>

* Comportement `exec` actuel (Docker ou hôte lorsqu&#39;il n&#39;est pas en sandbox).
* PTY pris en charge uniquement en mode sans sandbox.

<div id="gateway-host">
  ### Hôte du Gateway
</div>

* Le processus Gateway s’exécute sur une machine dédiée.
* Fait respecter localement `exec-approvals.json` (sécurité/demande/liste d’autorisation).

<div id="node-host">
  ### Hôte de nœud
</div>

* Gateway appelle `node.invoke` avec `system.run`.
* Le Runner applique les approbations locales.
* Le Runner renvoie la sortie stdout/stderr agrégée.
* Événements Bridge facultatifs pour démarrage/fin/refus.

<div id="output-caps">
  ## Limites de sortie
</div>

* Limiter la sortie combinée stdout+stderr à **200k** ; ne conserver que les **20k derniers** pour les événements.
* Tronquer avec un suffixe explicite (par exemple : `"… (truncated)"`).

<div id="slash-commands">
  ## Commandes slash
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* Surcharges par agent et par session ; non persistantes, sauf si elles sont enregistrées via la configuration.
* `/elevated on|off|ask|full` reste un raccourci pour `host=gateway security=full` (avec `full` qui saute les demandes d’approbation).

<div id="cross-platform-story">
  ## Stratégie multiplateforme
</div>

* Le service runner est la cible d&#39;exécution portable.
* L&#39;UI est optionnelle ; si elle est absente, `askFallback` s&#39;applique.
* Windows et Linux utilisent le même JSON d&#39;approbations et le même protocole socket.

<div id="implementation-phases">
  ## Phases d’implémentation
</div>

<div id="phase-1-config-exec-routing">
  ### Phase 1 : configuration + routage d&#39;exécution
</div>

* Ajouter un schéma de configuration pour `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
* Mettre à jour l’infrastructure des outils pour prendre en compte `exec.host`.
* Ajouter la commande slash `/exec` et conserver l’alias `/elevated`.

<div id="phase-2-approvals-store-gateway-enforcement">
  ### Phase 2 : store d’approbations + application par le Gateway
</div>

* Implémenter la lecture et l’écriture de `exec-approvals.json`.
* Appliquer la liste d’autorisation + les modes `ask` pour l’hôte `gateway`.
* Ajouter des limites de sortie.

<div id="phase-3-node-runner-enforcement">
  ### Phase 3 : application par le node runner
</div>

* Mettre à jour le node runner pour appliquer la liste d’autorisation + ask.
* Ajouter une passerelle de prompt via socket Unix à l’UI de l’app macOS.
* Connecter `askFallback`.

<div id="phase-4-events">
  ### Phase 4 : événements
</div>

* Ajouter des événements de bridge nœud → Gateway pour le cycle de vie de l&#39;exécution.
* Mapper sur `enqueueSystemEvent` pour les prompts d&#39;agent.

<div id="phase-5-ui-polish">
  ### Phase 5 : affinage de l’UI
</div>

* Application Mac : éditeur de liste d’autorisation, sélecteur d’agent, UI de politique de requêtes.
* Contrôles de liaison de nœud (optionnels).

<div id="testing-plan">
  ## Plan de tests
</div>

* Tests unitaires : correspondance avec la liste d’autorisation (glob + insensible à la casse).
* Tests unitaires : ordre de priorité de résolution des stratégies (paramètre d’outil → surcharge d’agent → global).
* Tests d’intégration : scénarios refuser/autoriser/demander de l’exécuteur de nœud.
* Tests d’événements du bridge : routage événement de nœud → événement système.

<div id="open-risks">
  ## Risques identifiés
</div>

* Indisponibilité de l’UI : veille à ce que `askFallback` soit appliqué.
* Commandes de longue durée : s’appuyer sur un timeout + des limites de sortie.
* Ambiguïté multi-nœud : erreur, sauf en cas de liaison à un nœud ou de paramètre de nœud explicite.

<div id="related-docs">
  ## Documentation connexe
</div>

* [Outil Exec](/fr/tools/exec)
* [Approbations d’Exec](/fr/tools/exec-approvals)
* [Nœuds](/fr/nodes)
* [Mode avec privilèges élevés](/fr/tools/elevated)