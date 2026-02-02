---
title: Découverte
summary: "Découverte de nœuds et transports (Bonjour, Tailscale, SSH) pour localiser le Gateway"
read_when:
  - Mise en œuvre ou modification de la découverte/annonce Bonjour
  - Ajustement des modes de connexion à distance (direct vs SSH)
  - Conception de la découverte et de l’appairage de nœuds distants
---

<div id="discovery-transports">
  # Découverte &amp; transports
</div>

OpenClaw a deux problèmes distincts qui se ressemblent en apparence :

1. **Télécommande opérateur** : l’app de barre des menus macOS qui contrôle un Gateway fonctionnant ailleurs.
2. **Appairage de nœud** : iOS/Android (et de futurs nœuds) qui trouvent un Gateway et s’y appairent de manière sécurisée.

L’objectif de conception est de conserver toute la découverte et l’annonce réseau dans le **Node Gateway** (`openclaw gateway`) et de faire des clients (app Mac, iOS) de simples consommateurs.

<div id="terms">
  ## Termes
</div>

* **Gateway** : un processus Gateway unique et de longue durée qui détient l’état (sessions, appairage, registre des nœuds) et exécute les canaux. La plupart des configurations en utilisent un par hôte ; des configurations multi-Gateway isolées sont possibles.
* **Gateway WS (plan de contrôle)** : le point de terminaison WebSocket sur `127.0.0.1:18789` par défaut ; peut être associé au LAN/tailnet via `gateway.bind`.
* **Transport WS direct** : un point de terminaison Gateway WS exposé au LAN/tailnet (sans SSH).
* **Transport SSH (repli)** : contrôle à distance en transférant `127.0.0.1:18789` via SSH.
* **Legacy TCP bridge (obsolète/supprimé)** : ancien transport de nœud (voir [Bridge protocol](/fr/gateway/bridge-protocol)) ; n’est plus annoncé pour la découverte.

Détails des protocoles :

* [Gateway protocol](/fr/gateway/protocol)
* [Bridge protocol (legacy)](/fr/gateway/bridge-protocol)

<div id="why-we-keep-both-direct-and-ssh">
  ## Pourquoi nous conservons à la fois le « direct » et SSH
</div>

* **WS direct** offre la meilleure UX sur le même réseau et dans un même tailnet :
  * découverte automatique sur le LAN via Bonjour
  * jetons d’appairage + ACL détenus par le Gateway
  * aucun accès shell requis ; la surface du protocole peut rester réduite et vérifiable
* **SSH** demeure la solution de secours universelle :
  * fonctionne partout où vous avez un accès SSH (même entre réseaux totalement distincts)
  * reste fonctionnel malgré les problèmes de multidiffusion/mDNS
  * ne nécessite aucun port entrant supplémentaire en dehors de SSH

<div id="discovery-inputs-how-clients-learn-where-the-gateway-is">
  ## Sources de découverte (comment les clients déterminent où se trouve le Gateway)
</div>

<div id="1-bonjour-mdns-lan-only">
  ### 1) Bonjour / mDNS (LAN only)
</div>

Bonjour fonctionne en mode « best effort » et ne traverse pas les frontières de réseau. Il n’est utilisé que pour la commodité sur un même LAN.

Direction de découverte :

* Le **Gateway** annonce son endpoint WS via Bonjour.
* Les clients recherchent et affichent une liste « choisir un Gateway », puis enregistrent l’endpoint sélectionné.

Dépannage et détails de la balise de service : [Bonjour](/fr/gateway/bonjour).

<div id="service-beacon-details">
  #### Détails de la balise de service
</div>

* Types de service :
  * `_openclaw-gw._tcp` (balise d’annonce de transport du Gateway)
* Clés TXT (non secrètes) :
  * `role=gateway`
  * `lanHost=<hostname>.local`
  * `sshPort=22` (ou tout autre port annoncé)
  * `gatewayPort=18789` (WS + HTTP du Gateway)
  * `gatewayTls=1` (uniquement lorsque TLS est activé)
  * `gatewayTlsSha256=<sha256>` (uniquement lorsque TLS est activé et que l’empreinte est disponible)
  * `canvasPort=18793` (port hôte par défaut pour canvas ; sert `/__openclaw__/canvas/`)
  * `cliPath=<path>` (optionnel ; chemin absolu vers un point d’entrée ou binaire `openclaw` exécutable)
  * `tailnetDns=<magicdns>` (indice optionnel ; détecté automatiquement lorsque Tailscale est disponible)

Désactivation / remplacement :

* `OPENCLAW_DISABLE_BONJOUR=1` désactive l’annonce.
* `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de liaison du Gateway.
* `OPENCLAW_SSH_PORT` remplace le port SSH annoncé dans TXT (par défaut : 22).
* `OPENCLAW_TAILNET_DNS` publie un indice `tailnetDns` (MagicDNS).
* `OPENCLAW_CLI_PATH` remplace le chemin CLI annoncé.

<div id="2-tailnet-cross-network">
  ### 2) Tailnet (entre réseaux)
</div>

Pour des configurations de type Londres/Vienne, Bonjour ne sera d’aucune aide. La cible « directe » recommandée est :

* Nom MagicDNS Tailscale (de préférence) ou une IP Tailnet stable.

Si Gateway détecte qu’il fonctionne sous Tailscale, il publie `tailnetDns` comme indication facultative pour les clients (y compris pour les balises à large portée).

<div id="3-manual-ssh-target">
  ### 3) Cible manuelle / SSH
</div>

Lorsqu&#39;il n&#39;y a pas de route directe (ou que l&#39;accès direct est désactivé), les clients peuvent toujours se connecter via SSH en transférant (port forwarding) le port loopback du Gateway.

Voir [Accès à distance](/fr/gateway/remote).

<div id="transport-selection-client-policy">
  ## Sélection du transport (politique côté client)
</div>

Comportement client recommandé :

1. Si un endpoint direct appairé est configuré et joignable, utilisez-le.
2. Sinon, si Bonjour trouve un Gateway sur le LAN, proposez une option « Utiliser ce Gateway » en un seul appui et enregistrez-le comme endpoint direct.
3. Sinon, si un DNS/IP de tailnet est configuré, tentez une connexion directe.
4. Sinon, basculez sur SSH.

<div id="pairing-auth-direct-transport">
  ## Appairage + authentification (transport direct)
</div>

Le Gateway est la référence faisant foi pour l’admission des nœuds et des clients.

* Les demandes d’appairage sont créées, approuvées ou rejetées dans le Gateway (voir [Gateway pairing](/fr/gateway/pairing)).
* Le Gateway applique :
  * l’authentification (jeton / paire de clés)
  * les portées / ACL (le Gateway n’est pas un simple proxy vers chaque méthode)
  * les limites de débit

<div id="responsibilities-by-component">
  ## Responsabilités par composant
</div>

* **Gateway** : diffuse des balises de découverte, gère les décisions d’appairage et héberge le point de terminaison WS.
* **Application macOS** : vous aide à choisir un Gateway, affiche les fenêtres d’appairage et n’utilise SSH qu’en solution de repli.
* **Nœuds iOS/Android** : explorent Bonjour par commodité et se connectent au WS du Gateway appairé.