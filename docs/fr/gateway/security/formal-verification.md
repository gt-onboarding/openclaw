---
title: Vérification formelle (modèles de sécurité)
summary: Modèles de sécurité vérifiés automatiquement pour les chemins d’exécution les plus à haut risque d’OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Vérification formelle (modèles de sécurité)
</div>

Cette page recense les **modèles de sécurité formels** d’OpenClaw (TLA+/TLC aujourd’hui ; plus au besoin).

> Remarque : certains liens plus anciens peuvent faire référence au nom précédent du projet.

**Objectif (étoile polaire)** : fournir un argument vérifié automatiquement montrant qu’OpenClaw applique sa
politique de sécurité définie (autorisation, isolation des sessions, contrôle d’accès aux outils et
sécurité face aux erreurs de configuration), sous des hypothèses explicites.

**Ce que c’est (aujourd’hui)** : une **suite de régression de sécurité** exécutable, pilotée par l’attaquant :

- Chaque assertion dispose d’une vérification de modèle exécutable sur un espace d’états fini.
- De nombreuses assertions disposent d’un **modèle négatif** associé qui produit une trace de contre‑exemple pour une catégorie de bugs réaliste.

**Ce que ce n’est pas (encore)** : une preuve que « OpenClaw est sécurisé à tous égards » ou que l’implémentation complète en TypeScript est correcte.

<div id="where-the-models-live">
  ## Emplacement des modèles
</div>

Les modèles sont gérés dans un dépôt séparé : [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Mises en garde importantes
</div>

- Il s’agit de **modèles**, pas de l’implémentation complète en TypeScript. Des divergences entre le modèle et le code sont possibles.
- Les résultats sont bornés par l’espace d’états exploré par TLC ; le statut « vert » n’implique pas un niveau de sécurité dépassant les hypothèses et bornes modélisées.
- Certaines affirmations reposent sur des hypothèses explicites concernant l’environnement (par exemple, déploiement correct, entrées de configuration correctes).

<div id="reproducing-results">
  ## Reproduire les résultats
</div>

Actuellement, pour reproduire les résultats, il faut cloner localement le dépôt des modèles et exécuter TLC (voir ci-dessous). Une future itération pourrait proposer :

* des modèles exécutés en CI avec des artefacts publics (traces de contre‑exemples, journaux d’exécution)
* un workflow hébergé « exécuter ce modèle » pour de petites vérifications bornées

Pour commencer :

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ requis (TLC s'exécute sur la JVM).
# Le dépôt inclut un `tla2tools.jar` épinglé (outils TLA+) et fournit `bin/tlc` + des cibles Make.

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Exposition du Gateway et mauvaise configuration d'un Gateway ouvert
</div>

**Constat :** le fait de lier le service à autre chose que l’interface loopback sans authentification peut permettre une compromission à distance / augmente l’exposition ; un jeton / mot de passe bloque les attaquants non authentifiés (conformément aux hypothèses du modèle).

- Exécutions vertes :
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rouge (attendu) :
  - `make gateway-exposure-v2-negative`

Voir aussi : `docs/gateway-exposure-matrix.md` dans le dépôt models.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Pipeline `nodes.run` (capacité la plus risquée)
</div>

**Affirmation :** `nodes.run` exige (a) une liste d’autorisation pour les commandes du nœud ainsi que des commandes déclarées et (b) une approbation en direct lorsqu’elle est configurée ; les approbations sont transformées en jetons pour empêcher les attaques par rejeu (dans le modèle).

- Exécutions en vert :
  - `make nodes-pipeline`
  - `make approvals-token`
- En rouge (attendu) :
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Stockage d'appairage (filtrage des DM)
</div>

**Propriété :** les requêtes d'appairage respectent le TTL et les limites de requêtes en attente.

- Exécutions en vert :
  - `make pairing`
  - `make pairing-cap`
- En rouge (attendus) :
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Filtrage d’ingress (mentions + contournement des commandes de contrôle)
</div>

**Assertion :** dans les contextes de groupe où une mention est requise, une « commande de contrôle » non autorisée ne peut pas contourner le filtrage par mention.

- Vert :
  - `make ingress-gating`
- Rouge (attendu) :
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### Isolation du routage et des clés de session
</div>

**Énoncé :** les DMs provenant de pairs distincts ne sont pas regroupés dans la même session, sauf s’ils sont explicitement liés/configurés.

- Vert :
  - `make routing-isolation`
- Rouge (attendu) :
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++ : modèles bornés supplémentaires (concurrence, nouvelles tentatives, correction des traces)
</div>

Ce sont des modèles complémentaires qui renforcent la fidélité vis-à-vis des modes de défaillance réels (mises à jour non atomiques, nouvelles tentatives et diffusion en éventail des messages).

<div id="pairing-store-concurrency-idempotency">
  ### Concurrence / idempotence du magasin d’appairage
</div>

**Affirmation :** un magasin d’appairage doit faire respecter `MaxPending` et l’idempotence même en cas d’entrelacement d’opérations (c.-à-d. que « check-then-write » doit être atomique / verrouillé ; un rafraîchissement ne doit pas créer de doublons).

Ce que cela signifie :

- Avec des requêtes concurrentes, vous ne pouvez pas dépasser `MaxPending` pour un canal.
- Des requêtes / rafraîchissements répétés pour le même `(channel, sender)` ne doivent pas créer de doublons de lignes en attente actives.

- Exécutions en vert :
  - `make pairing-race` (vérification de capacité maximale atomique / verrouillée)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- En rouge (attendu) :
  - `make pairing-race-negative` (condition de compétition sur le plafond début/commit non atomique)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Corrélation de traces d'ingress / idempotence
</div>

**Énoncé :** l'ingestion doit préserver la corrélation de traces lors du fan-out et être idempotente en cas de nouvelles tentatives du fournisseur.

Concrètement :

- Lorsqu'un événement externe se transforme en plusieurs messages internes, chaque partie conserve la même identité de trace/événement.
- Les nouvelles tentatives n'entraînent pas de double traitement.
- Si les ID d'événements du fournisseur sont absents, la déduplication se rabat sur une clé sûre (par exemple l'ID de trace) afin d'éviter d'écarter des événements distincts.

- Vert :
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rouge (attendu) :
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Routage : précédence de dmScope + identityLinks
</div>

**Énoncé :** le routage doit, par défaut, maintenir les sessions DM isolées et ne fusionner des sessions que lorsqu’il est explicitement configuré pour le faire (priorité de canal + identityLinks).

Ce que cela implique :

- Les remplacements de dmScope spécifiques à un canal doivent l’emporter sur les valeurs par défaut globales.
- identityLinks ne doivent regrouper les sessions qu’au sein de groupes explicitement liés, et jamais entre pairs non liés.

- Vert :
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rouge (attendu) :
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`