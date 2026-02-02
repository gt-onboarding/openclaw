---
title: Approbations d'exécution
summary: "Approbations d'exécution, listes d’autorisation et prompts d'évasion de sandbox"
read_when:
  - Configuration des approbations d'exécution et des listes d’autorisation
  - Mise en œuvre de l’expérience utilisateur d’approbation d'exécution dans l’app macOS
  - Examen des prompts d'évasion de sandbox et de leurs implications
---

<div id="exec-approvals">
  # Approbations d’exécution
</div>

Les approbations d’exécution sont le **garde-fou de l’app compagnon / de l’hôte du nœud** pour autoriser un agent exécuté en sandbox à exécuter
des commandes sur un hôte réel (`gateway` ou `node`). Vous pouvez les voir comme un verrou de sécurité :
les commandes ne sont autorisées que lorsque la politique + la liste d’autorisation + (éventuellement) l’approbation de l’utilisateur sont toutes alignées.
Les approbations d’exécution viennent **en complément** de la politique des outils et du contrôle renforcé (sauf si `elevated` est défini sur `full`, ce qui ignore les approbations).
La politique effective est la **plus stricte** entre `tools.exec.*` et les valeurs par défaut des approbations ; si un champ d’approbations est omis, la valeur de `tools.exec` est utilisée.

Si l’UI de l’app compagnon **n’est pas disponible**, toute requête nécessitant une invite est
gérée par le **ask fallback** (par défaut : refus).

<div id="where-it-applies">
  ## Où cela s’applique
</div>

Les approbations d’exécution sont appliquées localement sur l’hôte d’exécution :

* **gateway host** → processus `openclaw` sur la machine Gateway
* **node host** → service de nœud (app compagnon macOS ou hôte de nœud sans interface)

Répartition sur macOS :

* **node host service** transmet `system.run` à l’**app macOS** via l’IPC locale.
* L’**app macOS** applique les approbations et exécute la commande dans le contexte de l’UI.

<div id="settings-and-storage">
  ## Paramètres et stockage
</div>

Les autorisations sont stockées dans un fichier JSON local sur l’hôte d’exécution :

`~/.openclaw/exec-approvals.json`

Exemple de schéma :

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## Réglages de la stratégie
</div>

<div id="security-execsecurity">
  ### Sécurité (`exec.security`)
</div>

* **deny** : bloque toutes les requêtes exec sur l’hôte.
* **allowlist** : autorise uniquement les commandes présentes dans la liste d’autorisation.
* **full** : autorise tout (équivalent à « elevated »).

<div id="ask-execask">
  ### Demande (`exec.ask`)
</div>

* **off** : ne jamais demander.
* **on-miss** : demander uniquement lorsque la liste d’autorisation ne correspond pas.
* **always** : demander pour chaque commande.

<div id="ask-fallback-askfallback">
  ### Repli de demande (`askFallback`)
</div>

Si une invite est requise mais qu’aucune UI n’est accessible, le mode de repli détermine :

* **deny** : bloquer.
* **allowlist** : autoriser uniquement si la liste d’autorisation correspond.
* **full** : autoriser.

<div id="allowlist-per-agent">
  ## Liste d’autorisation (par agent)
</div>

Les listes d’autorisation sont **propres à chaque agent**. S’il existe plusieurs agents, changez l’agent
que vous modifiez dans l’app macOS. Les motifs sont des **motifs glob insensibles à la casse**.
Les motifs doivent se résoudre en **chemins vers des exécutables** (les entrées ne contenant qu’un basename sont ignorées).
Les entrées héritées `agents.default` sont migrées vers `agents.main` au chargement.

Exemples :

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

Chaque entrée de la liste d’autorisation conserve :

* **id** UUID stable utilisé pour l’identité dans l’UI (optionnel)
* **last used** horodatage
* **last used command**
* **last resolved path**

<div id="auto-allow-skill-clis">
  ## Autorisation automatique des CLI de compétences
</div>

Quand **Auto-allow skill CLIs** est activé, les exécutables référencés par des compétences connues
sont considérés comme présents dans la liste d’autorisation sur les nœuds (node macOS ou hôte node sans interface). Cela utilise
`skills.bins` via le RPC du Gateway pour récupérer la liste des binaires de compétences. Désactivez cette option si vous souhaitez des listes d’autorisation strictement manuelles.

<div id="safe-bins-stdin-only">
  ## Binaires sûrs (stdin uniquement)
</div>

`tools.exec.safeBins` définit une petite liste de binaires **uniquement via stdin** (par exemple `jq`)
qui peuvent s’exécuter en mode liste d’autorisation **sans** entrées explicites dans la liste d’autorisation. Les binaires sûrs rejettent
les arguments positionnels de type fichier et les jetons de type chemin, ils ne peuvent donc opérer que sur le flux entrant.
L’enchaînement de commandes shell et les redirections ne sont pas automatiquement autorisés en mode liste d’autorisation.

L’enchaînement de commandes shell (`&&`, `||`, `;`) est autorisé lorsque chaque segment de premier niveau respecte la liste d’autorisation
(y compris les binaires sûrs ou l’auto-autorisation de skills). Les redirections restent non pris en charge en mode liste d’autorisation.

Binaires sûrs par défaut : `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`.

<div id="control-ui-editing">
  ## Modification dans la Control UI
</div>

Utilise la carte **Control UI → Nodes → Exec approvals** pour modifier les valeurs par défaut, les surcharges par agent et les listes d’autorisation. Choisis une portée (Defaults ou un agent), ajuste la politique, ajoute/supprime des motifs de liste d’autorisation, puis clique sur **Save**. L’UI affiche les métadonnées **last used** par motif pour t’aider à garder la liste propre.

Le sélecteur de cible choisit **Gateway** (approbations locales) ou un **Node**. Les nœuds doivent annoncer `system.execApprovals.get/set` (macOS app ou hôte de nœud sans interface). Si un nœud n’annonce pas encore les approbations d’exécution, modifie directement son fichier local
`~/.openclaw/exec-approvals.json`.

CLI : `openclaw approvals` prend en charge la modification côté Gateway ou côté nœud (voir [Approvals CLI](/fr/cli/approvals)).

<div id="approval-flow">
  ## Flux d’approbation
</div>

Lorsqu’une invite est requise, le Gateway diffuse `exec.approval.requested` aux clients opérateur.
La Control UI et l’app macOS la valident via `exec.approval.resolve`, puis le Gateway transfère la
requête approuvée à l’hôte du nœud.

Lorsque des approbations sont requises, l’outil exec renvoie immédiatement un identifiant d’approbation. Utilisez cet identifiant pour
corréler les événements système ultérieurs (`Exec finished` / `Exec denied`). Si aucune décision n’arrive avant
l’expiration du délai, la requête est traitée comme un dépassement de délai d’approbation et exposée comme motif de refus.

La boîte de dialogue de confirmation inclut :

* commande + args
* cwd
* agent id
* chemin exécutable résolu
* hôte + métadonnées de stratégie

Actions :

* **Allow once** → exécuter maintenant
* **Always allow** → ajouter à la liste d’autorisation et exécuter
* **Deny** → bloquer

<div id="approval-forwarding-to-chat-channels">
  ## Transfert des demandes d&#39;approbation vers les canaux de discussion
</div>

Vous pouvez transférer les demandes d&#39;approbation d&#39;exec vers n&#39;importe quel canal de discussion (y compris les canaux de plugin) et les approuver
avec `/approve`. Cela utilise le pipeline d&#39;envoi sortant habituel.

Config :

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // sous-chaîne ou regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

Répondre dans le chat :

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### Flux IPC sur macOS
</div>

```
Gateway -> Service de nœud (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Notes de sécurité :

* Socket Unix en mode `0600`, jeton stocké dans `exec-approvals.json`.
* Vérification d’un pair avec le même UID.
* Challenge-réponse (nonce + jeton HMAC + hachage de requête) + TTL court.

<div id="system-events">
  ## Événements système
</div>

Le cycle de vie d’Exec est exposé sous forme de messages système :

* `Exec running` (uniquement si la commande dépasse le seuil de notification d’exécution)
* `Exec finished`
* `Exec denied`

Ces messages sont publiés dans la session de l’agent après que le nœud a signalé l’événement.
Les approbations d’exécution hébergées sur le Gateway émettent les mêmes événements de cycle de vie lorsque la commande se termine (et éventuellement lorsqu’elle s’exécute plus longtemps que le seuil).
Les Exec soumis à approbation réutilisent l’identifiant d’approbation comme `runId` dans ces messages pour en faciliter la corrélation.

<div id="implications">
  ## Implications
</div>

* **full** est puissant ; privilégiez les listes d’autorisation lorsque c’est possible.
* **ask** vous maintient dans la boucle tout en permettant des approbations rapides.
* Les listes d’autorisation par agent empêchent les approbations d’un agent de « fuiter » vers d’autres.
* Les approbations s’appliquent uniquement aux requêtes d’exec sur l’hôte provenant d’**expéditeurs autorisés**. Les expéditeurs non autorisés ne peuvent pas émettre `/exec`.
* `/exec security=full` est un raccourci au niveau de la session pour les opérateurs autorisés et contourne les approbations par conception.
  Pour bloquer strictement exec sur l’hôte, définissez la sécurité des approbations sur `deny` ou interdisez l’outil `exec` via la politique d’outils.

Voir aussi :

* [Exec tool](/fr/tools/exec)
* [Elevated mode](/fr/tools/elevated)
* [Skills](/fr/tools/skills)