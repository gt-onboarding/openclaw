---
title: Peekaboo
summary: "Intégration de PeekabooBridge pour l'automatisation de l'UI de macOS"
read_when:
  - Héberger PeekabooBridge dans OpenClaw.app
  - Intégrer Peekaboo via Swift Package Manager
  - Modifier le protocole et les chemins d'accès de PeekabooBridge
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge (automatisation de l’UI sur macOS)
</div>

OpenClaw peut héberger **PeekabooBridge** comme broker local d’automatisation
de l’UI, respectueux des autorisations. Cela permet à la CLI `peekaboo` de piloter
l’automatisation de l’UI tout en réutilisant les autorisations TCC de l’app macOS.

<div id="what-this-is-and-isnt">
  ## Ce que c’est (et ce que ça n’est pas)
</div>

- **Hôte** : OpenClaw.app peut servir d’hôte pour PeekabooBridge.
- **Client** : utilisez la CLI `peekaboo` (pas de surface `openclaw ui ...` séparée).
- **UI** : les surcouches visuelles restent dans Peekaboo.app ; OpenClaw sert uniquement d’hôte courtier léger.

<div id="enable-the-bridge">
  ## Activer le bridge
</div>

Dans l’app macOS :

- Settings → **Enable Peekaboo Bridge**

Lorsque cette option est activée, OpenClaw démarre un serveur de socket UNIX local. Si elle est désactivée, l’hôte est arrêté et `peekaboo` basculera vers d’autres hôtes disponibles.

<div id="client-discovery-order">
  ## Ordre de découverte du client
</div>

Les clients Peekaboo essaient généralement les hôtes dans cet ordre :

1. Peekaboo.app (interface utilisateur complète)
2. Claude.app (si installé)
3. OpenClaw.app (courtier léger)

Utilisez `peekaboo bridge status --verbose` pour voir quel hôte est actif et quel
chemin de socket est utilisé. Vous pouvez forcer un autre choix avec :

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## Sécurité et autorisations
</div>

- Le bridge valide les **signatures de code de l’appelant** ; une liste d’autorisation de TeamID est appliquée (TeamID de l’hôte Peekaboo + TeamID de l’app OpenClaw).
- Les requêtes expirent au bout d’environ 10 secondes.
- Si des autorisations requises manquent, le bridge renvoie un message d’erreur explicite au lieu de lancer Réglages Système.

<div id="snapshot-behavior-automation">
  ## Comportement des snapshots (automatisation)
</div>

Les snapshots sont stockés en mémoire et expirent automatiquement après une courte période.
Si vous avez besoin de les conserver plus longtemps, effectuez une nouvelle capture depuis le client.

<div id="troubleshooting">
  ## Dépannage
</div>

- Si `peekaboo` indique « bridge client is not authorized », assurez-vous que le client
  est correctement signé ou exécutez l’hôte avec `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  uniquement en mode **debug**.
- Si aucun hôte n’est trouvé, ouvrez l’une des applications hôtes (Peekaboo.app ou OpenClaw.app)
  et vérifiez que les autorisations sont accordées.