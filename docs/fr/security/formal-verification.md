---
title: Vérification formelle (modèles de sécurité)
summary: Modèles de sécurité vérifiés automatiquement pour les chemins les plus critiques d’OpenClaw.
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # Vérification formelle (modèles de sécurité)
</div>

Cette page recense les **modèles de sécurité formels** d’OpenClaw (TLA+/TLC aujourd’hui ; davantage si nécessaire).

> Remarque : certains anciens liens peuvent faire référence au nom précédent du projet.

**Objectif (étoile polaire)** : fournir un argument vérifié automatiquement montrant qu’OpenClaw applique sa
politique de sécurité prévue (autorisation, isolation des sessions, contrôle d’accès aux outils et
protection contre les erreurs de configuration), sous des hypothèses explicites.

**Ce que c’est (aujourd’hui)** : une **suite de tests de régression de sécurité** exécutable, pilotée par l’attaquant :

- Chaque affirmation dispose d’une vérification de modèle exécutable sur un espace d’états fini.
- De nombreuses affirmations ont un **modèle négatif** associé qui produit une trace de contre‑exemple pour une catégorie de bugs réalistes.

**Ce que ce n’est pas (encore)** : une preuve que « OpenClaw est sécurisé à tous égards » ou que l’implémentation TypeScript complète est correcte.

<div id="where-the-models-live">
  ## Où se trouvent les modèles
</div>

Les modèles sont maintenus dans un dépôt Git séparé : [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

<div id="important-caveats">
  ## Avertissements importants
</div>

- Ce sont des **modèles**, pas l’implémentation TypeScript complète. Un écart entre le modèle et le code est possible.
- Les résultats sont bornés par l’espace d’états exploré par TLC ; le statut « vert » n’implique pas une sécurité au-delà des hypothèses et bornes modélisées.
- Certaines affirmations reposent sur des hypothèses environnementales explicites (par exemple, déploiement correct, entrées de configuration correctes).

<div id="reproducing-results">
  ## Reproduire les résultats
</div>

Aujourd’hui, les résultats sont reproduits en clonant localement le dépôt des modèles et en exécutant TLC (voir ci-dessous). Une future itération pourrait proposer :

* des modèles exécutés en CI avec des artefacts publics (traces de contre-exemples, journaux d’exécution)
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
  ### Exposition du Gateway et mauvaise configuration d’un Gateway ouvert
</div>

**Constat :** le fait de lier l’écoute du service à une interface autre que loopback sans authentification peut rendre une compromission à distance possible / augmente l’exposition ; un jeton ou mot de passe bloque les attaquants non authentifiés (selon les hypothèses du modèle).

- Exécutions réussies (vertes) :
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rouge (attendu) :
  - `make gateway-exposure-v2-negative`

Voir aussi : `docs/gateway-exposure-matrix.md` dans le dépôt des modèles.

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Pipeline Nodes.run (capacité la plus à risque)
</div>

**Énoncé :** `nodes.run` nécessite (a) une liste d’autorisation de commandes du nœud plus des commandes déclarées et (b) une approbation en direct lorsqu’il est configuré ; les approbations sont converties en jetons pour empêcher toute attaque par rejeu (dans le modèle).

- Vert :
  - `make nodes-pipeline`
  - `make approvals-token`
- Rouge (attendu) :
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### Stockage d'appairage (filtrage des DM)
</div>

**Assertion :** les requêtes d'appairage respectent le TTL et les limites de requêtes en attente.

- Exécutions vertes :
  - `make pairing`
  - `make pairing-cap`
- Rouges (attendues) :
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
  ### Isolation du routage/de la clé de session
</div>

**Assertion :** les DMs provenant de pairs distincts ne sont pas regroupés dans la même session, sauf s’ils sont explicitement liés ou configurés pour cela.

- Vert :
  - `make routing-isolation`
- Rouge (attendu) :
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++ : modèles bornés supplémentaires (concurrence, nouvelles tentatives, exactitude des traces)
</div>

Ce sont des modèles supplémentaires qui améliorent la fidélité par rapport aux modes de défaillance réels (mises à jour non atomiques, nouvelles tentatives et diffusion « fan-out » des messages).

<div id="pairing-store-concurrency-idempotency">
  ### Concurrence / idempotence du store d’appairage
</div>

**Assertion :** un store d’appairage doit faire respecter `MaxPending` et l’idempotence même en cas d’entrelacements (c.-à-d. que « check-then-write » doit être atomique / verrouillé ; un rafraîchissement ne doit pas créer de doublons).

Ce que cela implique :

- En présence de requêtes concurrentes, `MaxPending` ne doit pas être dépassé pour un canal.
- Des requêtes / rafraîchissements répétés pour le même `(channel, sender)` ne doivent pas créer de doublons de lignes en attente actives.

- Exécutions au vert :
  - `make pairing-race` (contrôle du plafond atomique / verrouillé)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- En rouge (attendu) :
  - `make pairing-race-negative` (course sur le plafond avec begin/commit non atomiques)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Corrélation de trace d’ingress / idempotence
</div>

**Énoncé :** l’ingestion doit préserver la corrélation de trace lors du fan-out et être idempotente lors des reprises côté fournisseur.

Ce que cela signifie :

- Lorsqu’un événement externe devient plusieurs messages internes, chaque partie conserve la même identité de trace/événement.
- Les reprises ne doivent pas entraîner de double traitement.
- Si les ID d’événements du fournisseur sont absents, la déduplication se rabat sur une clé sûre (par ex. l’ID de trace) afin d’éviter d’écarter des événements distincts.

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
  ### Priorité de routage dmScope + identityLinks
</div>

**Assertion :** le routage doit, par défaut, garder les sessions DM isolées et ne fusionner des sessions que lorsqu’il est explicitement configuré pour le faire (priorité de canal + identity links).

Ce que cela implique :

- Les surcharges dmScope spécifiques à un canal doivent prévaloir sur les valeurs globales par défaut.
- identityLinks ne doit fusionner les sessions qu’au sein de groupes explicitement liés, et jamais entre pairs sans lien entre eux.

- Vert :
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rouge (attendu) :
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`