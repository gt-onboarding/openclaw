---
title: Rpc
summary: "Adaptateurs RPC pour des CLI externes (signal-cli, imsg) et les schémas du Gateway"
read_when:
  - Ajout ou modification d’intégrations CLI externes
  - Débogage d’adaptateurs RPC (signal-cli, imsg)
---

<div id="rpc-adapters">
  # Adaptateurs RPC
</div>

OpenClaw intègre des CLI externes via JSON-RPC. Deux schémas sont utilisés aujourd’hui.

<div id="pattern-a-http-daemon-signal-cli">
  ## Modèle A : démon HTTP (signal-cli)
</div>

* `signal-cli` s’exécute comme un démon avec JSON-RPC sur HTTP.
* Le flux d’événements utilise SSE (`/api/v1/events`).
* Sonde de contrôle d’état : `/api/v1/check`.
* OpenClaw gère le cycle de vie lorsque `channels.signal.autoStart=true`.

Voir [Signal](/fr/channels/signal) pour la configuration et les points de terminaison.

<div id="pattern-b-stdio-child-process-imsg">
  ## Modèle B : processus enfant stdio (imsg)
</div>

* OpenClaw lance `imsg rpc` en tant que processus enfant.
* JSON-RPC est délimité par des retours à la ligne sur stdin/stdout (un objet JSON par ligne).
* Aucun port TCP, aucun démon requis.

Méthodes principales utilisées :

* `watch.subscribe` → notifications (`method: "message"`)
* `watch.unsubscribe`
* `send`
* `chats.list` (sonde/diagnostic)

Voir [iMessage](/fr/channels/imessage) pour la configuration et l’adressage (de préférence `chat_id`).

<div id="adapter-guidelines">
  ## Directives pour les adaptateurs
</div>

* Le Gateway contrôle le processus (démarrage/arrêt liés au cycle de vie du fournisseur).
* Gardez les clients RPC résilients : délais d’expiration, redémarrage en cas de sortie.
* Préférez les identifiants stables (par ex. `chat_id`) aux chaînes d’affichage.