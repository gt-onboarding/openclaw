---
title: Verrou du Gateway
summary: "Mécanisme de verrouillage singleton du Gateway utilisant la liaison de l'écouteur WebSocket"
read_when:
  - Lors de l'exécution ou du débogage du processus Gateway
  - Lors de l'analyse du mécanisme d'application de l'instance unique
---

<div id="gateway-lock">
  # Verrouillage du Gateway
</div>

Dernière mise à jour : 2025-12-11

<div id="why">
  ## Pourquoi
</div>

- Garantir qu&#39;une seule instance de Gateway s&#39;exécute par port de base donné sur un même hôte ; les instances supplémentaires doivent utiliser des profils isolés et des ports uniques.
- Survivre aux crash/SIGKILL sans laisser de fichiers de verrouillage obsolètes.
- Échouer immédiatement avec une erreur claire lorsque le port de contrôle est déjà occupé.

<div id="mechanism">
  ## Mécanisme
</div>

- Le Gateway attache l’écouteur WebSocket (par défaut `ws://127.0.0.1:18789`) immédiatement au démarrage en utilisant un écouteur TCP exclusif.
- Si cet attachement échoue avec `EADDRINUSE`, le démarrage lève `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Le système d’exploitation libère automatiquement l’écouteur à la sortie de tout processus, y compris en cas de crash et de SIGKILL — aucun fichier de verrouillage distinct ni étape de nettoyage n’est nécessaire.
- À l’arrêt, le Gateway ferme le serveur WebSocket et le serveur HTTP sous-jacent afin de libérer le port rapidement.

<div id="error-surface">
  ## Surface d’erreur
</div>

- Si un autre processus occupe le port, le démarrage déclenche `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Les autres erreurs de liaison se manifestent sous la forme de `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

<div id="operational-notes">
  ## Remarques d’exploitation
</div>

- Si le port est occupé par *un autre* processus, l’erreur est la même ; libérez le port ou choisissez-en un autre avec `openclaw gateway --port <port>`.
- L’application macOS conserve toujours son propre mécanisme de garde PID léger avant de lancer le Gateway ; le verrou d’exécution est appliqué par la liaison WebSocket.