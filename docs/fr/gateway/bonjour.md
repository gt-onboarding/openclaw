---
title: Bonjour
summary: "Découverte et débogage Bonjour/mDNS (balises Gateway, clients et défaillances courantes)"
read_when:
  - Débogage de problèmes de découverte Bonjour sur macOS/iOS
  - Modification des types de service mDNS, des enregistrements TXT ou de l’UX de découverte
---

<div id="bonjour-mdns-discovery">
  # Découverte Bonjour / mDNS
</div>

OpenClaw utilise Bonjour (mDNS / DNS‑SD) comme **commodité limitée au LAN** pour découvrir
un Gateway actif (point de terminaison WebSocket). Il fonctionne sur la base du principe de « best effort » et **ne** remplace **pas** la connectivité via SSH ou basée sur Tailnet.

<div id="widearea-bonjour-unicast-dnssd-over-tailscale">
  ## Bonjour étendu (DNS‑SD unicast) via Tailscale
</div>

Si le nœud et le Gateway sont sur des réseaux différents, le mDNS multicast ne
franchira pas cette frontière réseau. Vous pouvez conserver la même UX de découverte en passant
au **DNS‑SD unicast** (« Bonjour étendu ») via Tailscale.

Étapes générales :

1. Faire tourner un serveur DNS sur l’hôte du Gateway (accessible via le Tailnet).
2. Publier les enregistrements DNS‑SD pour `_openclaw-gw._tcp` dans une zone dédiée
   (par exemple : `openclaw.internal.`).
3. Configurer le **split DNS** de Tailscale afin que le domaine que vous avez choisi se résolve via ce
   serveur DNS pour les clients (y compris iOS).

OpenClaw prend en charge n’importe quel domaine de découverte ; `openclaw.internal.` n’est
qu’un exemple. Les nœuds iOS/Android effectuent une découverte à la fois sur `local.` et sur votre domaine étendu configuré.

<div id="gateway-config-recommended">
  ### Configuration du Gateway (recommandée)
</div>

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } } // active la publication DNS-SD à large zone
}
```

<div id="onetime-dns-server-setup-gateway-host">
  ### Configuration unique du serveur DNS (hôte du Gateway)
</div>

```bash
openclaw dns setup --apply
```

Cela installe CoreDNS et le configure pour :

* écouter sur le port 53 uniquement sur les interfaces Tailscale du Gateway
* servir le domaine que vous avez choisi (par exemple : `openclaw.internal.`) à partir de `~/.openclaw/dns/<domain>.db`

Vérifiez depuis une machine reliée au tailnet :

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

<div id="tailscale-dns-settings">
  ### Paramètres DNS Tailscale
</div>

Dans la console d’administration de Tailscale :

* Ajoutez un serveur de noms pointant vers l’IP Tailnet du Gateway (UDP/TCP 53).
* Configurez du DNS fractionné afin que votre domaine de découverte utilise ce serveur de noms.

Une fois que les clients acceptent le DNS Tailnet, les nœuds iOS peuvent découvrir
`_openclaw-gw._tcp` dans votre domaine de découverte sans multidiffusion.

<div id="gateway-listener-security-recommended">
  ### Sécurité de l’écoute Gateway (recommandé)
</div>

Le port WS de Gateway (par défaut `18789`) est lié à l’interface loopback par défaut. Pour un accès depuis le LAN ou le tailnet, associez explicitement ce port et laissez l’authentification activée.

Pour les configurations réservées au tailnet :

* Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json`.
* Redémarrez Gateway (ou redémarrez l’app de barre de menus macOS).

<div id="what-advertises">
  ## Qui annonce
</div>

Seul Gateway annonce `_openclaw-gw._tcp`.

<div id="service-types">
  ## Types de services
</div>

* `_openclaw-gw._tcp` — balise de transport du Gateway OpenClaw (utilisée par les nœuds macOS/iOS/Android).

<div id="txt-keys-nonsecret-hints">
  ## Clés TXT (indications non secrètes)
</div>

Le Gateway diffuse de petites indications non secrètes pour rendre les flux de l’UI plus pratiques :

* `role=gateway`
* `displayName=<friendly name>`
* `lanHost=<hostname>.local`
* `gatewayPort=<port>` (Gateway WS + HTTP)
* `gatewayTls=1` (uniquement lorsque TLS est activé)
* `gatewayTlsSha256=<sha256>` (uniquement lorsque TLS est activé et que l’empreinte est disponible)
* `canvasPort=<port>` (uniquement lorsque l’hôte Canvas est activé ; valeur par défaut `18793`)
* `sshPort=<port>` (par défaut 22 lorsqu’il n’est pas redéfini)
* `transport=gateway`
* `cliPath=<path>` (optionnel ; chemin absolu vers un point d’entrée `openclaw` exécutable)
* `tailnetDns=<magicdns>` (indication optionnelle lorsque Tailnet est disponible)

<div id="debugging-on-macos">
  ## Débogage sous macOS
</div>

Outils intégrés pratiques :

* Lister les instances :
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
* Résoudre une instance (remplacez `<instance>`) :
  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Si la découverte fonctionne mais que la résolution échoue, vous vous heurtez généralement à une règle de réseau local (LAN) ou à un problème de résolveur mDNS.

<div id="debugging-in-gateway-logs">
  ## Débogage dans les journaux de Gateway
</div>

Gateway écrit un fichier journal rotatif (affiché au démarrage sous la forme
`gateway log file: ...`). Recherchez les lignes commençant par `bonjour:`, en particulier :

* `bonjour: advertise failed ...`
* `bonjour: ... name conflict resolved` / `hostname conflict resolved`
* `bonjour: watchdog detected non-announced service ...`

<div id="debugging-on-ios-node">
  ## Débogage sur le nœud iOS
</div>

Le nœud iOS utilise `NWBrowser` pour découvrir `_openclaw-gw._tcp`.

Pour capturer les journaux :

* Réglages → Gateway → Avancé → **Journaux de débogage de découverte**
* Réglages → Gateway → Avancé → **Journaux de découverte** → reproduire → **Copier**

Le journal inclut les transitions d’état de `NWBrowser` et les modifications du jeu de résultats.

<div id="common-failure-modes">
  ## Modes de panne courants
</div>

* **Bonjour ne traverse pas les réseaux** : utilisez Tailnet ou SSH.
* **Multidiffusion bloquée** : certains réseaux Wi‑Fi désactivent mDNS.
* **Mise en veille / changements d’interface** : macOS peut temporairement perdre les résultats mDNS ; réessayez.
* **La découverte fonctionne mais la résolution échoue** : gardez des noms d’hôte simples (évitez les émojis ou la ponctuation), puis redémarrez Gateway. Le nom d’instance du service est dérivé du nom d’hôte, donc des noms trop complexes peuvent perturber certains résolveurs.

<div id="escaped-instance-names-032">
  ## Noms d’instance échappés (`\032`)
</div>

Bonjour/DNS‑SD échappe souvent des octets dans les noms d’instance de service sous forme de séquences `\DDD` en notation décimale
(par exemple, les espaces deviennent `\032`).

* Cela est normal au niveau du protocole.
* Les UI doivent les décoder pour l’affichage (iOS utilise `BonjourEscapes.decode`).

<div id="disabling-configuration">
  ## Désactivation / configuration
</div>

* `OPENCLAW_DISABLE_BONJOUR=1` désactive l’annonce (anciennement : `OPENCLAW_DISABLE_BONJOUR`).
* `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de bind du Gateway.
* `OPENCLAW_SSH_PORT` remplace le port SSH annoncé dans TXT (anciennement : `OPENCLAW_SSH_PORT`).
* `OPENCLAW_TAILNET_DNS` publie une indication MagicDNS dans TXT (anciennement : `OPENCLAW_TAILNET_DNS`).
* `OPENCLAW_CLI_PATH` remplace le chemin de la CLI annoncé (anciennement : `OPENCLAW_CLI_PATH`).

<div id="related-docs">
  ## Documentation associée
</div>

* Politique de découverte et sélection du transport : [Discovery](/fr/gateway/discovery)
* Appairage des nœuds et approbations : [Gateway pairing](/fr/gateway/pairing)