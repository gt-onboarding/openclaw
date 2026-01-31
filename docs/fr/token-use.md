---
title: Utilisation des jetons
summary: "Comment OpenClaw construit le contexte de l'invite et rend compte de l'utilisation des jetons et des coûts"
read_when:
  - Expliquer l'utilisation des jetons, les coûts ou les fenêtres de contexte
  - Déboguer la croissance ou le compactage du contexte
---

<div id="token-use-costs">
  # Utilisation des jetons et coûts
</div>

OpenClaw compte les **jetons**, pas les caractères. Les jetons sont spécifiques au modèle, mais la plupart des
modèles de type OpenAI tournent en moyenne autour de ~4 caractères par jeton pour le texte en anglais.

<div id="how-the-system-prompt-is-built">
  ## Comment l’invite système est générée
</div>

OpenClaw construit sa propre invite système à chaque exécution. Elle inclut :

* Liste des outils + courtes descriptions
* Liste des compétences (uniquement les métadonnées ; les instructions sont chargées à la demande avec `read`)
* Instructions d’auto‑mise à jour
* Espace de travail + fichiers de bootstrap (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` le cas échéant). Les fichiers volumineux sont tronqués par `agents.defaults.bootstrapMaxChars` (valeur par défaut : 20000).
* Heure (UTC + fuseau horaire de l’utilisateur)
* Balises de réponse + comportement du signal de vie
* Métadonnées d’exécution (hôte/OS/modèle/raisonnement)

Voir le détail complet dans [Invite système](/fr/concepts/system-prompt).

<div id="what-counts-in-the-context-window">
  ## Ce qui est pris en compte dans la fenêtre de contexte
</div>

Tout ce que le modèle reçoit compte dans la limite de contexte :

* Invite système (toutes les sections listées ci‑dessus)
* Historique de la conversation (messages utilisateur + assistant)
* Appels d’outils et résultats associés
* Pièces jointes/transcriptions (images, audio, fichiers)
* Résumés de compaction et artefacts de purge
* Wrappers de fournisseur ou en-têtes de sécurité (non visibles, mais tout de même comptés)

Pour une ventilation pratique (par fichier injecté, outils, compétences et taille de l’invite système), utilisez `/context list` ou `/context detail`. Voir [Contexte](/fr/concepts/context).

<div id="how-to-see-current-token-usage">
  ## Comment consulter l&#39;utilisation actuelle des tokens
</div>

Utilisez ces commandes dans le chat :

* `/status` → **carte d&#39;état riche en emojis** avec le modèle de la session, l&#39;utilisation du contexte,
  les tokens d&#39;entrée/sortie de la dernière réponse et le **coût estimé** (clé API uniquement).
* `/usage off|tokens|full` → ajoute un **pied de page récapitulant l&#39;utilisation par réponse** à chaque message.
  * Réglage conservé par session (stocké sous `responseUsage`).
  * L&#39;authentification OAuth **masque le coût** (tokens uniquement).
* `/usage cost` → affiche un résumé local des coûts à partir des journaux de session OpenClaw.

Autres interfaces :

* **TUI/Web TUI :** `/status` + `/usage` sont pris en charge.
* **CLI :** `openclaw status --usage` et `openclaw channels list` affichent
  les fenêtres de quota des fournisseurs (pas les coûts par réponse).

<div id="cost-estimation-when-shown">
  ## Estimation des coûts (lorsqu&#39;elle est affichée)
</div>

Les coûts sont estimés à partir de la configuration de tarification de votre modèle :

```
models.providers.<provider>.models[].cost
```

Les tarifs sont indiqués en **USD par 1M de tokens** pour `input`, `output`, `cacheRead` et
`cacheWrite`. Si aucun tarif n’est renseigné, OpenClaw affiche uniquement les tokens. Les jetons OAuth
n’affichent jamais de coût en dollars.

<div id="cache-ttl-and-pruning-impact">
  ## Impact du TTL de cache et de la purge
</div>

La mise en cache des prompts par le fournisseur ne s’applique que dans la fenêtre
de TTL de cache. OpenClaw peut, en option, exécuter une **purge cache-ttl** :
il purge la session une fois que le TTL de cache a expiré, puis réinitialise la
fenêtre de cache afin que les requêtes suivantes puissent réutiliser le contexte
récemment mis en cache au lieu de recréer tout l’historique dans le cache. Cela
réduit les coûts d’écriture dans le cache lorsqu’une session reste inactive au-delà
du TTL.

Configurez ce comportement dans la [configuration du Gateway](/fr/gateway/configuration)
et consultez les détails du fonctionnement dans [Purge de session](/fr/concepts/session-pruning).

Le signal de vie peut garder le cache **chaud** (actif) pendant les périodes d’inactivité.
Si le TTL de cache de votre modèle est de `1h`, définir l’intervalle du signal
de vie juste en dessous (par exemple `55m`) permet d’éviter de remettre en cache
le prompt complet, ce qui réduit les coûts d’écriture dans le cache.

Pour la tarification de l’API Anthropic, les lectures de cache sont
significativement moins chères que les jetons en entrée, tandis que les écritures
dans le cache sont facturées avec un multiplicateur plus élevé. Consultez la
tarification de la mise en cache des prompts d’Anthropic pour les tarifs les plus
récents et les multiplicateurs de TTL :
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### Exemple : maintenir le cache chaud pendant 1 h avec le signal de vie
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## Conseils pour réduire la pression sur les tokens
</div>

* Utilisez `/compact` pour résumer les longues sessions.
* Tronquez les sorties volumineuses des outils dans vos workflows.
* Gardez les descriptions de compétences courtes (la liste de compétences est injectée dans le prompt).
* Préférez des modèles de plus petite taille pour le travail verbeux et exploratoire.

Consultez [Skills](/fr/tools/skills) pour la formule exacte de surcoût liée à la liste de compétences.