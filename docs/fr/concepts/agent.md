---
title: Agent
summary: "Runtime d'Agent (pi-mono embarqué), contrat d'espace de travail et initialisation de session"
read_when:
  - Modification du runtime d'agent, de l'initialisation de l'espace de travail ou du comportement de session
---

<div id="agent-runtime">
  # Runtime d&#39;agent 🤖
</div>

OpenClaw exécute un unique runtime d&#39;agent intégré dérivé de **pi-mono**.

<div id="workspace-required">
  ## Espace de travail (obligatoire)
</div>

OpenClaw utilise un seul répertoire d’espace de travail pour l’agent (`agents.defaults.workspace`) comme **seul** répertoire de travail (`cwd`) de l’agent pour les outils et le contexte.

Recommandé : utilisez `openclaw setup` pour créer `~/.openclaw/openclaw.json` s’il n’existe pas et initialiser les fichiers de l’espace de travail.

Structure complète de l’espace de travail + guide de sauvegarde : [Espace de travail d’Agent](/fr/concepts/agent-workspace)

Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent surcharger ce paramètre avec des espaces de travail par session sous `agents.defaults.sandbox.workspaceRoot` (voir
[Configuration du Gateway](/fr/gateway/configuration)).

<div id="bootstrap-files-injected">
  ## Fichiers de bootstrap (injectés)
</div>

Dans `agents.defaults.workspace`, OpenClaw s’attend à trouver ces fichiers modifiables par l’utilisateur :

* `AGENTS.md` — instructions de fonctionnement + « mémoire »
* `SOUL.md` — persona, limites, ton
* `TOOLS.md` — notes d’outils maintenues par l’utilisateur (par ex. `imsg`, `sag`, conventions)
* `BOOTSTRAP.md` — rituel de premier démarrage unique (supprimé après achèvement)
* `IDENTITY.md` — nom/ambiance/emoji de l’agent
* `USER.md` — profil utilisateur + forme d’adresse préférée

Au premier échange d’une nouvelle session, OpenClaw injecte le contenu de ces fichiers directement dans le contexte de l’agent.

Les fichiers vides sont ignorés. Les fichiers volumineux sont raccourcis et tronqués avec un marqueur afin que les prompts restent légers (lisez le fichier pour obtenir le contenu complet).

Si un fichier est manquant, OpenClaw injecte une seule ligne indiquant « fichier manquant » (et `openclaw setup` créera un modèle par défaut sécurisé).

`BOOTSTRAP.md` n’est créé que pour un **tout nouvel espace de travail** (aucun autre fichier de bootstrap présent). Si vous le supprimez après avoir terminé le rituel, il ne devrait pas être recréé lors de redémarrages ultérieurs.

Pour désactiver complètement la création de fichiers de bootstrap (pour des espaces de travail pré‑initialisés), définissez :

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## Outils intégrés
</div>

Les outils principaux (`read`/`exec`/`edit`/`write` et les outils système associés) sont toujours disponibles, sous réserve de la politique des outils. `apply_patch` est facultatif et conditionné par `tools.exec.applyPatch`. `TOOLS.md` ne détermine **pas** quels outils existent : il fournit des indications sur la façon dont *vous* voulez qu’ils soient utilisés.

<div id="skills">
  ## Compétences
</div>

OpenClaw charge des compétences depuis trois emplacements (l’espace de travail l’emporte en cas de conflit de nom) :

* Intégrées (livrées avec l’installation)
* Gérées/locales : `~/.openclaw/skills`
* Espace de travail : `<workspace>/skills`

Les compétences peuvent être contrôlées par la configuration ou les variables d’environnement (voir `skills` dans la [configuration du Gateway](/fr/gateway/configuration)).

<div id="pi-mono-integration">
  ## Intégration pi-mono
</div>

OpenClaw réutilise certaines parties du code de pi-mono (modèles/outils), mais **la gestion des sessions, la découverte et le raccordement des outils relèvent d’OpenClaw**.

* Aucun runtime d’agent pi-coding.
* Aucun paramètre `~/.pi/agent` ou `<workspace>/.pi` n’est pris en compte.

<div id="sessions">
  ## Sessions
</div>

Les transcriptions de sessions sont stockées au format JSONL à l’emplacement suivant :

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

L’ID de session est stable et choisi par OpenClaw.
Les anciens dossiers de sessions Pi/Tau ne sont **pas** lus.

<div id="steering-while-streaming">
  ## Pilotage pendant le streaming
</div>

Lorsque le mode de file d&#39;attente est `steer`, les messages entrants sont injectés dans l&#39;exécution en cours.
La file d&#39;attente est vérifiée **après chaque appel d&#39;outil** ; si un message en file d&#39;attente est présent,
les appels d&#39;outils restants du message assistant courant sont ignorés (résultats d&#39;outil en erreur avec « Skipped due to queued user message. »), puis le message utilisateur en file d&#39;attente est injecté avant la prochaine réponse de l&#39;assistant.

Lorsque le mode de file d&#39;attente est `followup` ou `collect`, les messages entrants sont conservés jusqu&#39;à la fin du tour en cours, puis un nouveau tour d&#39;agent démarre avec les charges utiles en file d&#39;attente. Voir [Queue](/fr/concepts/queue) pour le comportement du mode + debounce/cap.

Le block streaming envoie les blocs d&#39;assistant complétés dès qu&#39;ils se terminent ; il est
**désactivé par défaut** (`agents.defaults.blockStreamingDefault: "off"`).
Ajustez la limite via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end` ; par défaut `text_end`).
Contrôlez le découpage en blocs souples avec `agents.defaults.blockStreamingChunk` (par défaut
800–1200 caractères ; privilégie les sauts de paragraphe, puis les retours à la ligne, et les phrases en dernier).
Fusionnez les chunks streamés avec `agents.defaults.blockStreamingCoalesce` pour réduire
le spam de lignes uniques (fusion basée sur l&#39;inactivité avant envoi). Les canaux non‑Telegram nécessitent
un `*.blockStreaming: true` explicite pour activer les réponses par blocs.
Les résumés détaillés d&#39;outils sont émis au démarrage de l&#39;outil (sans debounce) ; Control UI
diffuse en streaming la sortie des outils via les événements d&#39;agent lorsque c&#39;est disponible.
Plus de détails : [Streaming + chunking](/fr/concepts/streaming).

<div id="model-refs">
  ## Références de modèle
</div>

Les références de modèle dans la configuration (par exemple `agents.defaults.model` et `agents.defaults.models`) sont analysées en scindant la chaîne sur le **premier** `/`.

* Utilisez `provider/model` lors de la configuration des modèles.
* Si l’ID du modèle lui‑même contient `/` (style OpenRouter), incluez le préfixe du fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
* Si vous omettez le fournisseur, OpenClaw interprète la valeur comme un alias ou comme un modèle pour le **fournisseur par défaut** (ne fonctionne que lorsqu’il n’y a pas de `/` dans l’ID du modèle).

<div id="configuration-minimal">
  ## Configuration (minimale)
</div>

Configure au minimum :

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom` (fortement recommandé)

***

*Ensuite : [Discussions de groupe](/fr/concepts/group-messages)* 🦞