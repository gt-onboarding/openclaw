---
title: Tableau de bord
summary: "Accès au tableau de bord Gateway (Control UI) et authentification"
read_when:
  - Modification des modes d'authentification ou d'exposition du tableau de bord
---

<div id="dashboard-control-ui">
  # Tableau de bord (Control UI)
</div>

Le tableau de bord de Gateway est la Control UI accessible via le navigateur, servie sur `/` par défaut
(modifiable avec `gateway.controlUi.basePath`).

Ouverture rapide (Gateway local) :

* http://127.0.0.1:18789/ (ou http://localhost:18789/)

Références clés :

* [Control UI](/fr/web/control-ui) pour l’utilisation et les fonctionnalités de l’UI.
* [Tailscale](/fr/gateway/tailscale) pour l’automatisation Serve/Funnel.
* [Surfaces Web](/fr/web) pour les modes de liaison (bind) et les notes de sécurité.

L’authentification est imposée lors de la poignée de main WebSocket via `connect.params.auth`
(token ou mot de passe). Voir `gateway.auth` dans la [configuration de Gateway](/fr/gateway/configuration).

Note de sécurité : la Control UI est une **surface d’administration** (chat, configuration, approbations d’exécution).
Ne l’exposez pas publiquement. L’UI stocke le token dans `localStorage` après le premier chargement.
Privilégiez localhost, Tailscale Serve ou un tunnel SSH.

<div id="fast-path-recommended">
  ## Procédure rapide (recommandée)
</div>

* Après l’onboarding, la CLI ouvre désormais automatiquement le tableau de bord avec votre jeton et affiche le même lien incluant ce jeton.
* Rouvrez-le à tout moment : `openclaw dashboard` (copie le lien, ouvre le navigateur si possible, affiche un rappel SSH si vous êtes en mode headless).
* Le jeton reste local (uniquement en paramètre de requête) : l’UI le retire après le premier chargement et l’enregistre dans localStorage.

<div id="token-basics-local-vs-remote">
  ## Notions de base sur les jetons (local vs distant)
</div>

* **Localhost** : ouvrez `http://127.0.0.1:18789/`. Si vous voyez « unauthorized », exécutez `openclaw dashboard` et utilisez le lien avec jeton (`?token=...`).
* **Source du jeton** : `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ; l’UI l’enregistre après le premier chargement.
* **En dehors du localhost** : utilisez Tailscale Serve (sans jeton si `gateway.auth.allowTailscale: true`), un bind Tailnet avec jeton ou un tunnel SSH. Voir [Interfaces Web](/fr/web).

<div id="if-you-see-unauthorized-1008">
  ## Si vous voyez « unauthorized » / 1008
</div>

* Exécutez `openclaw dashboard` pour obtenir un nouveau lien contenant un jeton.
* Assurez-vous que le Gateway est accessible (local : `openclaw status` ; distant : tunnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/?token=...`).
* Dans les paramètres du tableau de bord, collez le même jeton que celui que vous avez configuré dans `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).