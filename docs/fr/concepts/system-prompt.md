---
title: Invite système
summary: "Ce que contient l'invite système OpenClaw et comment elle est construite"
read_when:
  - Modification du texte de l'invite système, de la liste des outils ou des sections temps/signal de vie
  - Modification du comportement d’initialisation de l’espace de travail ou d’injection des compétences
---

<div id="system-prompt">
  # Prompt système
</div>

OpenClaw construit un prompt système personnalisé pour chaque exécution d’agent. Le prompt est **contrôlé par OpenClaw** et n’utilise pas le prompt par défaut de p-coding-agent.

Le prompt est assemblé par OpenClaw et injecté dans chaque exécution d’agent.

<div id="structure">
  ## Structure
</div>

L’invite est volontairement compacte et utilise des sections fixes :

* **Tooling** : liste actuelle des outils et brèves descriptions.
* **Skills** (lorsqu’elles sont disponibles) : indique au modèle comment charger les instructions de compétences à la demande.
* **OpenClaw Self-Update** : comment exécuter `config.apply` et `update.run`.
* **Workspace** : répertoire de travail (`agents.defaults.workspace`).
* **Documentation** : chemin local vers la documentation OpenClaw (dépôt ou package npm) et moment où la consulter.
* **Workspace Files (injected)** : indique que les fichiers de bootstrap sont inclus ci‑dessous.
* **Sandbox** (lorsqu’elle est activée) : indique l’exécution en sandbox, les chemins de sandbox et si une exécution avec privilèges élevés est disponible.
* **Current Date &amp; Time** : heure locale de l’utilisateur, fuseau horaire et format de l’heure.
* **Reply Tags** : syntaxe optionnelle des balises de réponse pour les fournisseurs pris en charge.
* **Heartbeats** : invite de signal de vie et comportement d’acquittement.
* **Runtime** : hôte, OS, nœud, modèle, chemin racine du dépôt (lorsqu’il est détecté), niveau de raisonnement (une ligne).
* **Reasoning** : niveau de visibilité actuel + rappel sur le basculement /reasoning.

<div id="prompt-modes">
  ## Modes de prompt
</div>

OpenClaw peut générer des prompts système plus compacts pour les sous-agents. Le runtime définit un
`promptMode` pour chaque exécution (ce n’est pas un paramètre de configuration exposé à l’utilisateur) :

* `full` (par défaut) : inclut toutes les sections ci-dessus.
* `minimal` : utilisé pour les sous-agents ; omet **Compétences**, **Rappel de mémoire**, **Mise à jour automatique d’OpenClaw**, **Alias de modèles**, **Identité de l’utilisateur**, **Balises de réponse**, **Messagerie**, **Réponses silencieuses** et **Signaux de vie**. Les outils, l’espace de travail, la sandbox, la date et l’heure actuelles (lorsqu’elles sont connues), le runtime et le contexte injecté restent disponibles.
* `none` : renvoie uniquement la ligne d’identité de base.

Lorsque `promptMode=minimal`, les prompts supplémentaires injectés sont étiquetés **Contexte de sous-agent**
au lieu de **Contexte de discussion de groupe**.

<div id="workspace-bootstrap-injection">
  ## Injection de bootstrap de l&#39;espace de travail
</div>

Les fichiers de bootstrap sont réduits puis ajoutés sous **Contexte du projet** afin que le modèle voie le contexte d&#39;identité et de profil sans nécessiter de `read` explicites :

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (uniquement sur les espaces de travail tout juste créés)

Les gros fichiers sont tronqués avec un marqueur. La taille maximale par fichier est contrôlée par
`agents.defaults.bootstrapMaxChars` (valeur par défaut : 20000). Pour les fichiers manquants, un court
marqueur de fichier manquant est injecté.

Les hooks internes peuvent intercepter cette étape via `agent:bootstrap` pour modifier ou remplacer
les fichiers de bootstrap injectés (par exemple en remplaçant `SOUL.md` par une autre persona).

Pour inspecter la contribution de chaque fichier injecté (brut vs injecté, troncature, plus la surcharge liée au schéma d&#39;outils), utilisez `/context list` ou `/context detail`. Voir [Context](/fr/concepts/context).

<div id="time-handling">
  ## Gestion du temps
</div>

Le system prompt inclut une section dédiée **Current Date &amp; Time** lorsque le
fuseau horaire de l&#39;utilisateur est connu. Pour garder le prompt stable pour le cache, il n&#39;inclut désormais que le **fuseau horaire** (aucune horloge dynamique ni format d&#39;heure).

Utilisez `session_status` lorsque l&#39;agent a besoin de l&#39;heure actuelle ; la carte d&#39;état inclut une ligne d&#39;horodatage.

Configurez avec :

* `agents.defaults.userTimezone`
* `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Consultez [Date &amp; Time](/fr/date-time) pour une description complète du comportement.

<div id="skills">
  ## Compétences
</div>

Lorsque des compétences éligibles sont présentes, OpenClaw insère une **liste compacte de compétences disponibles**
(`formatSkillsForPrompt`) qui inclut le **chemin de fichier** pour chaque compétence. Le
prompt indique au modèle d&#39;utiliser `read` pour charger le SKILL.md à l&#39;emplacement indiqué
(espace de travail, géré ou embarqué). Si aucune compétence n&#39;est éligible, la
section Compétences est omise.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Cela maintient le prompt de base compact tout en permettant l’utilisation ciblée des compétences.

<div id="documentation">
  ## Documentation
</div>

Lorsque c&#39;est possible, l&#39;invite système inclut une section **Documentation** qui pointe vers le
répertoire de documentation local d&#39;OpenClaw (soit `docs/` dans l&#39;espace de travail du dépôt, soit la
documentation fournie avec le paquet npm) et mentionne également le miroir public, le dépôt source,
le Discord communautaire et ClawHub (https://clawhub.com) pour la découverte de compétences. L&#39;invite demande au modèle de consulter d&#39;abord la documentation locale
pour tout ce qui concerne le comportement, les commandes, la configuration ou l&#39;architecture d&#39;OpenClaw, et d&#39;exécuter
`openclaw status` lui‑même lorsque c&#39;est possible (en ne sollicitant l&#39;utilisateur que lorsqu&#39;il n&#39;y a pas accès).