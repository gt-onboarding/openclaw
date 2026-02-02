---
title: Architecture
summary: "Architecture du Gateway WebSocket, composants et flux clients"
read_when:
  - Travail sur le protocole du Gateway, les clients ou les transports
---

<div id="gateway-architecture">
  # Architecture du Gateway
</div>

Dernière mise à jour : 2026-01-22

<div id="overview">
  ## Vue d’ensemble
</div>

* Un **Gateway** unique et pérenne gère tous les canaux de messagerie (WhatsApp via
  Baileys, Telegram via grammY, Slack, Discord, Signal, iMessage, WebChat).
* Les clients du plan de contrôle (application macOS, CLI, UI web, automatisations) se connectent au
  Gateway via **WebSocket** sur l’hôte d’écoute configuré (par défaut
  `127.0.0.1:18789`).
* Les **Nodes** (macOS/iOS/Android/sans interface) se connectent également via **WebSocket**, mais
  déclarent `role: node` avec des capacités/commandes explicites.
* Un seul Gateway par hôte ; c’est le seul endroit qui ouvre une session WhatsApp.
* Un **hôte de canvas** (par défaut `18793`) fournit du HTML modifiable par les agents et A2UI.

<div id="components-and-flows">
  ## Composants et flux
</div>

<div id="gateway-daemon">
  ### Gateway (daemon)
</div>

* Maintient les connexions aux fournisseurs.
* Expose une API WS typée (requêtes, réponses, événements émis par le serveur).
* Valide les trames entrantes au regard de JSON Schema.
* Émet des événements comme `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`.

<div id="clients-mac-app-cli-web-admin">
  ### Clients (app macOS / CLI / interface d&#39;administration web)
</div>

* Une connexion WS par client.
* Envoyer des requêtes (`health`, `status`, `send`, `agent`, `system-presence`).
* S&#39;abonner aux événements (`tick`, `agent`, `presence`, `shutdown`).

<div id="nodes-macos-ios-android-headless">
  ### Nœuds (macOS / iOS / Android / headless)
</div>

* Se connectent au **même serveur WS** avec `role: node`.
* Fournissent l’identité de l’appareil dans `connect` ; l’appairage est **basé sur l’appareil** (rôle `node`) et
  l’approbation est enregistrée dans le store d’appairage de l’appareil.
* Exposent des commandes comme `canvas.*`, `camera.*`, `screen.record`, `location.get`.

Détails du protocole :

* [Protocole du Gateway](/fr/gateway/protocol)

<div id="webchat">
  ### WebChat
</div>

* UI statique qui utilise l&#39;API WS du Gateway pour l&#39;historique des conversations et l&#39;envoi des messages.
* Dans les déploiements distants, se connecte via le même tunnel SSH/Tailscale que les autres
  clients.

<div id="connection-lifecycle-single-client">
  ## Cycle de vie de la connexion (client unique)
</div>

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   (or res error + close)
  |   (payload=hello-ok carries snapshot: presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   (streaming)
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

<div id="wire-protocol-summary">
  ## Protocole sur le fil (résumé)
</div>

* Transport : WebSocket, trames texte avec payloads JSON.
* La première trame **doit** être `connect`.
* Après la poignée de main :
  * Requêtes : `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  * Événements : `{type:"event", event, payload, seq?, stateVersion?}`
* Si `OPENCLAW_GATEWAY_TOKEN` (ou `--token`) est défini, `connect.params.auth.token`
  doit correspondre, sinon le socket est fermé.
* Les clés d&#39;idempotence sont requises pour les méthodes avec effets de bord (`send`, `agent`) afin
  de permettre des tentatives de ré-exécution en toute sécurité ; le serveur maintient un cache de déduplication à courte durée de vie.
* Les nœuds doivent inclure `role: "node"` ainsi que les capacités/commandes/autorisations dans `connect`.

<div id="pairing-local-trust">
  ## Appairage + confiance locale
</div>

* Tous les clients WS (opérateurs + nœuds) incluent une **identité d’appareil** lors de `connect`.
* Les nouveaux identifiants d’appareil nécessitent une approbation d’appairage ; Gateway émet un **jeton d’appareil**
  pour les connexions ultérieures.
* Les connexions **locales** (loopback ou adresse tailnet propre à l’hôte du Gateway) peuvent être
  approuvées automatiquement afin de conserver une UX fluide sur le même hôte.
* Les connexions **non locales** doivent signer le nonce `connect.challenge` et requièrent
  une approbation explicite.
* L’auth du Gateway (`gateway.auth.*`) s’applique toujours à **toutes** les connexions, qu’elles soient locales ou
  distantes.

Détails : [Protocole du Gateway](/fr/gateway/protocol), [Appairage](/fr/start/pairing),
[Sécurité](/fr/gateway/security).

<div id="protocol-typing-and-codegen">
  ## Typage du protocole et génération de code
</div>

* Les schémas TypeBox définissent le protocole.
* Des schémas JSON sont générés à partir de ces schémas.
* Les modèles Swift sont générés à partir des schémas JSON.

<div id="remote-access">
  ## Accès à distance
</div>

* Solution recommandée : Tailscale ou VPN.
* Alternative : tunnel SSH
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
* Le même handshake et le même jeton d’authentification s’appliquent via le tunnel.
* TLS et l’épinglage optionnel peuvent être activés pour WS dans les déploiements à distance.

<div id="operations-snapshot">
  ## Aperçu des opérations
</div>

* Démarrage : `openclaw gateway` (au premier plan, journalise sur stdout).
* Santé : `health` via WS (également inclus dans `hello-ok`).
* Supervision : launchd/systemd pour le redémarrage automatique.

<div id="invariants">
  ## Invariants
</div>

* Exactement un Gateway contrôle une seule session Baileys par hôte.
* La poignée de main est obligatoire ; toute première trame qui n’est ni du JSON ni une requête `connect` entraîne une fermeture brutale.
* Les événements ne sont pas rejoués ; les clients doivent se resynchroniser en cas de trous.