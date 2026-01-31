---
title: VPS
summary: "Hub d'hébergement VPS pour OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Vous souhaitez exécuter le Gateway dans le cloud
  - Vous avez besoin d'un aperçu rapide des guides d'hébergement VPS
---

<div id="vps-hosting">
  # Hébergement VPS
</div>

Cette section regroupe les guides d&#39;hébergement/VPS pris en charge et explique, de manière générale, le fonctionnement des déploiements dans le cloud.

<div id="pick-a-provider">
  ## Choisissez un fournisseur
</div>

* **Railway** (déploiement en un clic + via navigateur) : [Railway](/fr/railway)
* **Northflank** (déploiement en un clic + via navigateur) : [Northflank](/fr/northflank)
* **Oracle Cloud (Always Free)** : [Oracle](/fr/platforms/oracle) — 0 $/mois (Always Free, ARM ; capacité/inscription parfois aléatoires)
* **Fly.io** : [Fly.io](/fr/platforms/fly)
* **Hetzner (Docker)** : [Hetzner](/fr/platforms/hetzner)
* **GCP (Compute Engine)** : [GCP](/fr/platforms/gcp)
* **exe.dev** (VM + proxy HTTPS) : [exe.dev](/fr/platforms/exe-dev)
* **AWS (EC2/Lightsail/free tier)** : fonctionne bien aussi. Guide vidéo :
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## Fonctionnement des configurations cloud
</div>

* Le **Gateway s’exécute sur le VPS** et détient l’état et l’espace de travail.
* Vous vous connectez depuis votre ordinateur portable ou votre téléphone via le **Control UI** ou **Tailscale/SSH**.
* Considérez le VPS comme la source de vérité et **sauvegardez** l’état et l’espace de travail.
* Configuration sécurisée par défaut : gardez le Gateway sur l’interface de boucle locale (localhost) et accédez‑y via un tunnel SSH ou Tailscale Serve.
  Si vous l’exposez sur `lan`/`tailnet`, imposez l’usage de `gateway.auth.token` ou `gateway.auth.password`.

Accès à distance : [Gateway remote](/fr/gateway/remote)\
Plateforme centrale : [Platforms](/fr/platforms)

<div id="using-nodes-with-a-vps">
  ## Utilisation de nœuds avec un VPS
</div>

Vous pouvez garder le Gateway dans le cloud et associer des **nœuds** à vos appareils locaux
(Mac/iOS/Android/sans interface (headless)). Les nœuds fournissent des fonctionnalités locales pour l’écran, la caméra, le canevas et `system.run`,
tandis que le Gateway reste dans le cloud.

Docs : [Nœuds](/fr/nodes), [CLI des nœuds](/fr/cli/nodes)