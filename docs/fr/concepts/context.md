---
title: Contexte
summary: "Contexte : ce que le modèle voit, comment il est constitué et comment l'inspecter"
read_when:
  - Vous voulez comprendre ce que « contexte » signifie dans OpenClaw
  - Vous êtes en train de déboguer pourquoi le modèle « sait » quelque chose (ou l'a oublié)
  - Vous voulez réduire le surcoût lié au contexte (/context, /status, /compact)
---

<div id="context">
  # Contexte
</div>

Le « contexte » est **tout ce que OpenClaw envoie au modèle pour un run**. Il est limité par la **fenêtre de contexte** (limite de jetons) du modèle.

Modèle mental de base :

* **Invite système** (générée par OpenClaw) : règles, outils, liste de compétences, heure/runtime d’exécution et fichiers d’espace de travail injectés.
* **Historique de conversation** : vos messages + les messages de l’assistant pour cette session.
* **Appels/résultats d’outils + pièces jointes** : sortie de commande, lectures de fichiers, images/audio, etc.

Le contexte *n’est pas la même chose* que la « mémoire » : la mémoire peut être stockée sur disque et rechargée plus tard ; le contexte est ce qui se trouve dans la fenêtre actuelle du modèle.

<div id="quick-start-inspect-context">
  ## Démarrage rapide (inspecter le contexte)
</div>

* `/status` → vue rapide « à quel point ma fenêtre de contexte est pleine ? » + paramètres de session.
* `/context list` → ce qui est injecté + tailles approximatives (par fichier + totaux).
* `/context detail` → analyse plus détaillée : tailles par fichier, par outil (schémas), par entrée de compétence, et taille du prompt système.
* `/usage tokens` → ajoute, pour chaque réponse, un pied de page indiquant l’utilisation de tokens aux réponses normales.
* `/compact` → résume l’historique plus ancien en une entrée compacte pour libérer de l’espace dans la fenêtre de contexte.

Voir aussi : [Commandes slash](/fr/tools/slash-commands), [Utilisation des tokens et coûts](/fr/token-use), [Compaction](/fr/concepts/compaction).

<div id="example-output">
  ## Exemple de résultat
</div>

Les valeurs varient selon le modèle, le fournisseur, la politique d’utilisation des outils et le contenu de votre espace de travail.

<div id="context-list">
  ### `/context list`
</div>

```
🧠 Détail du contexte
Espace de travail : <workspaceDir>
Bootstrap max/fichier : 20 000 caractères
Sandbox : mode=non-main sandboxed=false
Prompt système (exécution) : 38 412 caractères (~9 603 jetons) (Contexte du projet 23 901 caractères (~5 976 jetons))

Fichiers de l'espace de travail injectés :
- AGENTS.md: OK | brut 1,742 caractères (~436 jetons) | injecté 1,742 caractères (~436 jetons)
- SOUL.md: OK | brut 912 caractères (~228 jetons) | injecté 912 caractères (~228 jetons)
- TOOLS.md: TRUNCATED | brut 54,210 caractères (~13,553 jetons) | injecté 20,962 caractères (~5,241 jetons)
- IDENTITY.md: OK | brut 211 caractères (~53 jetons) | injecté 211 caractères (~53 jetons)
- USER.md: OK | brut 388 caractères (~97 jetons) | injecté 388 caractères (~97 jetons)
- HEARTBEAT.md: MISSING | brut 0 | injecté 0
- BOOTSTRAP.md: OK | brut 0 caractères (~0 jetons) | injecté 0 caractères (~0 jetons)

Liste des compétences (texte du prompt système) : 2 184 caractères (~546 jetons) (12 compétences)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Liste des outils (texte du prompt système) : 1 032 caractères (~258 jetons)
Schémas des outils (JSON) : 31 988 caractères (~7 997 jetons) (comptabilisés dans le contexte ; non affichés en texte)
Outils : (identiques à ceux ci-dessus)

Jetons de session (en cache) : 14 250 total / ctx=32 000
```

<div id="context-detail">
  ### `/context detail`
</div>

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

<div id="what-counts-toward-the-context-window">
  ## Ce qui compte dans la fenêtre de contexte
</div>

Tout ce que le modèle reçoit est pris en compte, y compris :

* Invite système (toutes les sections).
* Historique de la conversation.
* Appels d’outils + résultats d’outils.
* Pièces jointes/transcriptions (images/audio/fichiers).
* Résumés de compaction et artéfacts d’élagage.
* « wrappers » ou en-têtes cachés du fournisseur (non visibles mais tout de même pris en compte).

<div id="how-openclaw-builds-the-system-prompt">
  ## Comment OpenClaw génère l&#39;invite système
</div>

L&#39;invite système est **propriété d&#39;OpenClaw** et recréée à chaque exécution. Elle inclut :

* Liste des outils avec une brève description.
* Liste des compétences (métadonnées uniquement ; voir ci-dessous).
* Emplacement de l&#39;espace de travail.
* Heure (UTC + heure de l&#39;utilisateur convertie si configurée).
* Métadonnées d&#39;exécution (hôte/OS/modèle/raisonnement).
* Fichiers d&#39;amorçage de l&#39;espace de travail injectés sous **Contexte du projet**.

Détail complet : [Invite système](/fr/concepts/system-prompt).

<div id="injected-workspace-files-project-context">
  ## Fichiers d’espace de travail injectés (Contexte du projet)
</div>

Par défaut, OpenClaw injecte un ensemble fixe de fichiers d’espace de travail (s’ils existent) :

* `AGENTS.md`
* `SOUL.md`
* `TOOLS.md`
* `IDENTITY.md`
* `USER.md`
* `HEARTBEAT.md`
* `BOOTSTRAP.md` (uniquement lors du premier lancement)

Les fichiers volumineux sont tronqués individuellement à l’aide de `agents.defaults.bootstrapMaxChars` (par défaut : `20000` caractères). `/context` affiche les tailles **brutes vs injectées** et indique si une troncation a eu lieu.

<div id="skills-whats-injected-vs-loaded-on-demand">
  ## Compétences : ce qui est injecté vs chargé à la demande
</div>

L’invite système inclut une **liste de compétences** compacte (nom + description + emplacement). Cette liste a un coût non négligeable.

Les instructions de la compétence ne sont *pas* incluses par défaut. Le modèle doit `read` le `SKILL.md` de la compétence **uniquement lorsque c’est nécessaire**.

<div id="tools-there-are-two-costs">
  ## Outils : il y a deux sources de coût
</div>

Les outils affectent le contexte de deux façons :

1. **Texte de la liste d’outils** dans le prompt système (ce que vous voyez sous « Tooling »).
2. **Schémas d’outils** (JSON). Ils sont envoyés au modèle pour qu’il puisse appeler des outils. Ils comptent dans le contexte même si vous ne les voyez pas en texte brut.

`/context detail` décompose les schémas d’outils les plus volumineux pour que vous puissiez voir ce qui domine.

<div id="commands-directives-and-inline-shortcuts">
  ## Commandes, directives et « raccourcis inline »
</div>

Les commandes commençant par une barre oblique sont traitées par le Gateway. Il existe plusieurs comportements différents :

* **Commandes autonomes** : un message qui ne contient que `/...` est exécuté comme une commande.
* **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` sont supprimées avant que le modèle ne voie le message.
  * Les messages ne contenant qu’une directive font persister les paramètres de la session.
  * Les directives en ligne dans un message normal servent d’indices au niveau du message.
* **« Inline shortcuts »** (expéditeurs autorisés figurant sur la liste d’autorisation uniquement) : certains jetons `/...` à l’intérieur d’un message normal peuvent s’exécuter immédiatement (exemple : « hey /status ») et sont supprimés avant que le modèle ne voie le texte restant.

Détails : [Commandes avec barre oblique](/fr/tools/slash-commands).

<div id="sessions-compaction-and-pruning-what-persists">
  ## Sessions, compactage et purge (ce qui persiste)
</div>

Ce qui est conservé entre les messages dépend du mécanisme :

* **L&#39;historique normal** est conservé dans le journal de session jusqu&#39;à ce qu&#39;il soit compacté/purgé selon la stratégie.
* **Le compactage** conserve un résumé dans le journal tout en gardant intacts les messages récents.
* **La purge** supprime les anciens résultats d&#39;outils de l&#39;invite *en mémoire* pour une exécution donnée, mais ne modifie pas le journal.

Docs : [Session](/fr/concepts/session), [Compactage](/fr/concepts/compaction), [Purge de session](/fr/concepts/session-pruning).

<div id="what-context-actually-reports">
  ## Ce que `/context` rapporte réellement
</div>

`/context` privilégie le dernier rapport de prompt système **généré lors d’un run** lorsqu’il est disponible :

* `System prompt (run)` = capturé à partir du dernier run embarqué (avec outils) et conservé dans le stockage de session.
* `System prompt (estimate)` = calculé à la volée lorsqu’aucun rapport de run n’existe (ou lorsqu’on s’exécute via un backend CLI qui ne génère pas ce rapport).

Dans tous les cas, il indique les tailles et les principaux contributeurs ; il **n’affiche** **pas** l’intégralité du prompt système ni les schémas des outils.