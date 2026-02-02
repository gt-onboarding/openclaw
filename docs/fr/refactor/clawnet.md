---
title: Clawnet
summary: "Refonte de Clawnet : unifier le protocole réseau, les rôles, l’authentification, les approbations et l’identité"
read_when:
  - Planification d’un protocole réseau unifié pour les nœuds et les clients opérateurs
  - Refonte des approbations, de l’appairage, de TLS et de la gestion de présence sur l’ensemble des appareils
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Refactorisation de Clawnet (unification du protocole et de l’authentification)
</div>

<div id="hi">
  ## Bonjour
</div>

Bonjour Peter — excellente approche ; cela permet à la fois de simplifier l’UX et de renforcer la sécurité.

<div id="purpose">
  ## Objectif
</div>

Document unique et rigoureux pour :

- État actuel : protocoles, flux, périmètres de confiance.
- Points de friction : validations, routage multi‑sauts, duplication de l’UI.
- Nouvel état proposé : un seul protocole, rôles à portée définie, authentification/appairage unifiés, épinglage TLS.
- Modèle d’identité : IDs stables + slugs sympa.
- Plan de migration, risques, questions ouvertes.

<div id="goals-from-discussion">
  ## Objectifs (issus de la discussion)
</div>

- Un protocole unique pour tous les clients (application macOS, CLI, iOS, Android, nœud headless).
- Chaque participant au réseau est authentifié et appairé.
- Clarté des rôles : nœuds vs opérateurs.
- Approbations centralisées, acheminées là où se trouve l’utilisateur.
- Chiffrement TLS + épinglage optionnel pour l’ensemble du trafic distant.
- Duplication de code minimale.
- Une machine donnée ne doit apparaître qu’une seule fois (aucune entrée en double dans l’UI/nœud).

<div id="nongoals-explicit">
  ## Non‑objectifs (explicites)
</div>

- Supprimer la séparation des capacités (le principe du moindre privilège doit rester appliqué).
- Exposer l’intégralité du plan de contrôle du Gateway sans vérification de la portée.
- Faire dépendre l’authentification d’étiquettes humaines (les *slugs* ne constituent toujours pas un mécanisme de sécurité).

---

<div id="current-state-asis">
  # État actuel (tel quel)
</div>

<div id="two-protocols">
  ## Deux protocoles
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket (plan de contrôle)
</div>

- Surface complète de l’API : config, canaux, modèles, sessions, exécutions d’agents, journaux, nœuds, etc.
- Liaison par défaut : loopback. Accès distant via SSH/Tailscale.
- Authentification : jeton/mot de passe via `connect`.
- Pas de TLS pinning (repose sur le loopback/tunnel).
- Code :
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge (transport de nœud)
</div>

- Surface d’exposition de la liste d’autorisation réduite, identité du nœud + appairage.
- JSONL sur TCP ; TLS facultatif + épinglage de l’empreinte du certificat.
- TLS publie l’empreinte dans l’enregistrement TXT de découverte.
- Code :
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## Clients actuels du plan de contrôle
</div>

- CLI → Gateway WS via `callGateway` (`src/gateway/call.ts`).
- UI de l’app macOS → Gateway WS (`GatewayConnection`).
- Web Control UI → Gateway WS.
- ACP → Gateway WS.
- Le contrôle via le navigateur s’appuie sur son propre serveur de contrôle HTTP.

<div id="nodes-today">
  ## Nœuds aujourd’hui
</div>

- L’app macOS en mode nœud se connecte au pont Gateway (`MacNodeBridgeSession`).
- Les apps iOS/Android se connectent au pont Gateway.
- Appairage + jeton par nœud, stocké sur le Gateway.

<div id="current-approval-flow-exec">
  ## Flux d'approbation actuel (exec)
</div>

- L'agent utilise `system.run` via Gateway.
- Gateway appelle le nœud via le bridge.
- Le runtime du nœud décide de la validation.
- Une invite dans l'UI est affichée par l'app macOS (quand le nœud == app macOS).
- Le nœud renvoie `invoke-res` à Gateway.
- Multi‑hop, UI rattachée à l’hôte du nœud.

<div id="presence-identity-today">
  ## Présence et identité aujourd’hui
</div>

- Entrées de présence du Gateway via les clients WS.
- Entrées de présence des nœuds via le bridge.
- L’app Mac peut afficher deux entrées pour la même machine (UI + nœud).
- Identité du nœud stockée dans le store d’appairage ; identité de l’UI distincte.

---

<div id="problems-pain-points">
  # Problèmes / points de friction
</div>

- Deux piles de protocoles à maintenir (WS + Bridge).
- Autorisations sur les nœuds distants : la demande apparaît sur l’hôte du nœud, pas là où se trouve l’utilisateur.
- Le pinning TLS n’existe que pour le Bridge ; WS dépend de SSH/Tailscale.
- Duplication d’identité : la même machine apparaît comme plusieurs instances.
- Rôles ambigus : les capacités de l’UI, du nœud et de la CLI ne sont pas clairement séparées.

---

<div id="proposed-new-state-clawnet">
  # Nouvel état proposé (Clawnet)
</div>

<div id="one-protocol-two-roles">
  ## Un protocole, deux rôles
</div>

Protocole WS unique avec rôle et portée.

- **Rôle : nœud** (hôte de capacités)
- **Rôle : opérateur** (plan de contrôle)
- **Portée** optionnelle pour l’opérateur :
  - `operator.read` (statut + consultation)
  - `operator.write` (exécution d’agents, envois)
  - `operator.admin` (configuration, canaux, modèles)

<div id="role-behaviors">
  ### Comportements des rôles
</div>

**Nœud**

- Peut enregistrer des capacités (`caps`, `commands`, permissions).
- Peut recevoir des commandes `invoke` (`system.run`, `camera.*`, `canvas.*`, `screen.record`, etc.).
- Peut envoyer des événements : `voice.transcript`, `agent.request`, `chat.subscribe`.
- Ne peut pas appeler les API du plan de contrôle pour config/models/channels/sessions/agent.

**Opérateur**

- Accès complet aux API du plan de contrôle, limité par la portée.
- Reçoit toutes les demandes d’approbation.
- N’exécute pas directement des actions du système d’exploitation ; les délègue aux nœuds.

<div id="key-rule">
  ### Règle principale
</div>

Le rôle est défini par connexion, et non par appareil. Un appareil peut assumer les deux rôles, séparément.

---

<div id="unified-authentication-pairing">
  # Authentification et appairage unifiés
</div>

<div id="client-identity">
  ## Identité du client
</div>

Chaque client fournit :

- `deviceId` (stable, dérivé de la clé de l’appareil).
- `displayName` (nom destiné aux utilisateurs).
- `role` + `scope` + `caps` + `commands`.

<div id="pairing-flow-unified">
  ## Flux d’appairage (unifié)
</div>

- Le client se connecte sans être authentifié.
- Le Gateway crée une **demande d’appairage** pour ce `deviceId`.
- L’opérateur reçoit une notification et approuve ou refuse.
- Le Gateway délivre des identifiants associés à :
  - la clé publique de l’appareil
  - le(s) rôle(s)
  - la(les) portée(s)
  - les capacités/commandes
- Le client stocke le jeton, puis se reconnecte en étant authentifié.

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## Authentification liée à l’appareil (éviter le rejeu de jetons bearer)
</div>

Préféré : paires de clés d’appareil.

- L’appareil génère une paire de clés une seule fois.
- `deviceId = fingerprint(publicKey)`.
- La Gateway envoie un nonce ; l’appareil le signe ; la Gateway vérifie.
- Les jetons sont émis pour une clé publique (preuve de possession), pas pour une chaîne de caractères.

Alternatives :

- mTLS (certificats client) : le plus robuste, mais complexité opérationnelle accrue.
- Jetons bearer de courte durée uniquement comme phase temporaire (rotation + révocation anticipée).

<div id="silent-approval-ssh-heuristic">
  ## Approbation silencieuse (heuristique SSH)
</div>

Définissez-la précisément pour éviter un maillon faible. Préférez l’une des options suivantes :

- **Local uniquement** : appariement automatique lorsque le client se connecte via boucle locale/socket Unix.
- **Challenge via SSH** : le Gateway émet un nonce ; le client prouve son accès SSH en le récupérant.
- **Fenêtre de présence physique** : après une approbation locale sur l’UI de l’hôte Gateway, autorisez l’appariement automatique pendant une courte fenêtre (par ex. 10 minutes).

Consignez et enregistrez toujours les approbations automatiques.

---

<div id="tls-everywhere-dev-prod">
  # TLS partout (dev et prod)
</div>

<div id="reuse-existing-bridge-tls">
  ## Réutiliser le TLS existant du bridge
</div>

Utilise le runtime TLS actuel + le « fingerprint pinning » :

- `src/infra/bridge/server/tls.ts`
- logique de vérification de l’empreinte dans `src/node-host/bridge-client.ts`

<div id="apply-to-ws">
  ## Pour WS
</div>

- Le serveur WS prend en charge TLS avec le même certificat, la même clé et la même empreinte.
- Les clients WS peuvent épingler l’empreinte (facultatif).
- Discovery annonce TLS et l’empreinte pour tous les points de terminaison.
  - Discovery fournit uniquement des indications de localisation ; jamais une autorité de confiance.

<div id="why">
  ## Pourquoi
</div>

- Réduire la dépendance à SSH/Tailscale pour garantir la confidentialité.
- Sécuriser par défaut les connexions mobiles à distance.

---

<div id="approvals-redesign-centralized">
  # Refonte du système d’approbation (centralisé)
</div>

<div id="current">
  ## État actuel
</div>

L’autorisation se fait sur l’hôte du node (runtime du node de l’app macOS). La demande s’affiche là où le node s’exécute.

<div id="proposed">
  ## Proposition
</div>

L'approbation est **hébergée dans le Gateway**, l'UI est fournie aux clients opérateurs.

<div id="new-flow">
  ### Nouveau flux
</div>

1) Gateway reçoit un intent `system.run` (agent).
2) Gateway crée un enregistrement d’approbation : `approval.requested`.
3) Les UI opérateur affichent la demande.
4) La décision d’approbation est envoyée au Gateway : `approval.resolve`.
5) Gateway invoque la commande du nœud si elle est approuvée.
6) Le nœud exécute et renvoie `invoke-res`.

<div id="approval-semantics-hardening">
  ### Sémantique des approbations (durcissement)
</div>

- Diffuser à tous les opérateurs ; seule l’UI active affiche une fenêtre modale (les autres reçoivent un toast).
- La première résolution l’emporte ; Gateway rejette les résolutions ultérieures comme déjà tranchées.
- Délai d’expiration par défaut : refuser après N secondes (par ex. 60 s) et consigner la raison dans les journaux.
- La résolution nécessite la portée `operator.approvals`.

<div id="benefits">
  ## Avantages
</div>

- L’invite apparaît là où se trouve l’utilisateur (Mac/téléphone).
- Processus d’approbation cohérent pour les nœuds distants.
- L’environnement d’exécution du nœud reste sans interface graphique ; aucune dépendance à une UI.

---

<div id="role-clarity-examples">
  # Exemples de définition des rôles
</div>

<div id="iphone-app">
  ## Application iPhone
</div>

- **Rôle de nœud** pour : micro, caméra, chat vocal, localisation, push‑to‑talk.
- **operator.read** facultatif pour la vue du statut et du chat.
- **operator.write/admin** facultatif, uniquement lorsqu’il est explicitement activé.

<div id="macos-app">
  ## Application macOS
</div>

- Rôle d’opérateur par défaut (Control UI).
- Rôle de nœud lorsque « Mac node » est activé (system.run, screen, camera).
- Même deviceId pour les deux connexions → entrée unique dans l’UI.

<div id="cli">
  ## CLI
</div>

- Toujours le rôle d’opérateur.
- Portée déterminée par la sous-commande :
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - approbations + appairage → `operator.approvals` / `operator.pairing`

---

<div id="identity-slugs">
  # Identité + slugs
</div>

<div id="stable-id">
  ## Identifiant stable
</div>

Obligatoire pour l'authentification ; ne change jamais.
Préféré :

- Empreinte de la paire de clés (hachage de la clé publique).

<div id="cute-slug-lobsterthemed">
  ## Slug sympathique (thème homard)
</div>

Libellé à usage humain uniquement.

- Exemple : `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Enregistré dans le registre du Gateway, modifiable.
- Gestion des collisions : `-2`, `-3`.

<div id="ui-grouping">
  ## Regroupement dans l’UI
</div>

Même `deviceId` pour tous les rôles → une seule ligne « Instance » :

- Badge : `operator`, `node`.
- Affiche les capacités + dernière activité.

---

<div id="migration-strategy">
  # Stratégie de migration
</div>

<div id="phase-0-document-align">
  ## Phase 0 : Documenter et aligner
</div>

- Publier ce document.
- Répertorier tous les appels de protocole et les flux d'approbation.

<div id="phase-1-add-rolesscopes-to-ws">
  ## Phase 1 : ajouter des rôles/portées au WS
</div>

- Étendre les paramètres de `connect` pour inclure `role`, `scope` et `deviceId`.
- Ajouter un filtrage par liste d’autorisation pour le rôle de nœud.

<div id="phase-2-bridge-compatibility">
  ## Phase 2 : compatibilité du bridge
</div>

- Garder le bridge en fonctionnement.
- Ajouter en parallèle la prise en charge des nœuds WS.
- Activer les fonctionnalités derrière un flag de configuration.

<div id="phase-3-central-approvals">
  ## Phase 3 : approbations centralisées
</div>

- Ajouter des événements de demande d’approbation + de résolution dans WS.
- Mettre à jour l’UI de l’app Mac pour demander + répondre.
- Le runtime Node cesse de solliciter l’UI.

<div id="phase-4-tls-unification">
  ## Phase 4 : unification TLS
</div>

- Ajouter la configuration TLS pour WS en utilisant le runtime TLS du bridge.
- Ajouter le pinning côté clients.

<div id="phase-5-deprecate-bridge">
  ## Phase 5 : marquer le bridge comme obsolète
</div>

- Migrer le nœud iOS/Android/mac vers WS.
- Conserver le bridge comme solution de repli ; le supprimer une fois l’ensemble stabilisé.

<div id="phase-6-devicebound-auth">
  ## Phase 6 : Authentification liée à l’appareil
</div>

- Exiger une identité basée sur des clés pour toutes les connexions distantes.
- Ajouter une UI pour la révocation et la rotation.

---

<div id="security-notes">
  # Notes de sécurité
</div>

- Rôle / liste d’autorisation appliqués au périmètre du Gateway.
- Aucun client n’obtient une API « complète » sans portée d’opérateur.
- Appairage requis pour *toutes* les connexions.
- TLS + pinning réduit le risque d’attaques MITM sur mobile.
- L’approbation silencieuse SSH est une commodité ; elle est tout de même enregistrée et révocable.
- La découverte n’est jamais une ancre de confiance.
- Les déclarations de capacités sont vérifiées par rapport aux listes d’autorisation du serveur, par plateforme/type.

<div id="streaming-large-payloads-node-media">
  # Streaming et charges utiles volumineuses (médias du nœud)
</div>

Le plan de contrôle WS convient pour les petits messages, mais les nœuds gèrent aussi :

- des séquences vidéo de la caméra
- des enregistrements d'écran
- des flux audio

Options :

1) Trames binaires WS + découpage en blocs (chunks) + règles de contre‑pression.
2) Endpoint de streaming séparé (toujours TLS + authentification).
3) Conserver le bridge plus longtemps pour les commandes à fort volume média, migrer en dernier.

Choisissez une option avant l'implémentation pour éviter les dérives.

<div id="capability-command-policy">
  # Politique de capacités et de commandes
</div>

- Les capacités et commandes signalées par le nœud sont traitées comme des **assertions**.
- Gateway applique des listes d’autorisation par plateforme.
- Toute nouvelle commande requiert l’approbation d’un opérateur ou une modification explicite de la liste d’autorisation.
- Consignez les modifications avec horodatage.

<div id="audit-rate-limiting">
  # Audit + limitation de débit
</div>

- Consigner : demandes d’appairage, approbations/refus, émission/rotation/révocation de jetons.
- Limiter le débit du spam d’appairage et des demandes d’approbation.

<div id="protocol-hygiene">
  # Hygiène du protocole
</div>

- Version explicite du protocole + codes d’erreur.
- Règles de reconnexion + politique de signal de vie.
- TTL de présence et sémantique de dernière activité.

---

<div id="open-questions">
  # Questions ouvertes
</div>

1) Appareil unique assurant les deux rôles : modèle de jetons
   - Recommandation : jetons séparés par rôle (nœud vs opérateur).
   - Même deviceId ; portées différentes ; révocation plus claire.

2) Granularité de la portée de l'opérateur
   - lecture/écriture/admin + approbations + appairage (minimum viable).
   - Envisager des portées par fonctionnalité plus tard.

3) UX pour la rotation et la révocation des jetons
   - Rotation automatique lors d'un changement de rôle.
   - UI pour révoquer par deviceId + rôle.

4) Découverte
   - Étendre le TXT Bonjour actuel pour inclure l'empreinte TLS WS + des indications de rôle.
   - Les traiter uniquement comme des indications de localisation.

5) Approbation inter‑réseaux
   - Diffuser à tous les clients opérateurs ; l'UI active affiche une fenêtre modale.
   - La première réponse l'emporte ; Gateway applique l'atomicité.

---

<div id="summary-tldr">
  # Résumé (TL;DR)
</div>

- Aujourd’hui : plan de contrôle WS + transport via le nœud Bridge.
- Problème : autorisations, duplication et deux stacks distinctes.
- Proposition : un seul protocole WS avec rôles et portées explicites, appairage unifié + épinglage TLS, autorisations gérées côté Gateway, identifiants d’appareils stables + petits slugs sympas.
- Résultat : UX plus simple, sécurité renforcée, moins de duplication, meilleur routage sur mobile.