---
title: Streaming
summary: "Comportement de streaming et de découpage (réponses par blocs, streaming de brouillons, limites)"
read_when:
  - Expliquer le fonctionnement du streaming ou du découpage sur les canaux
  - Modifier le streaming par blocs ou le comportement de découpage des canaux
  - Déboguer les réponses par blocs en double ou anticipées, ou le streaming de brouillons
---

<div id="streaming-chunking">
  # Streaming + découpage en blocs
</div>

OpenClaw dispose de deux couches de « streaming » distinctes :

- **Streaming par blocs (canaux) :** émet des **blocs** complets au fur et à mesure que l’assistant écrit. Ce sont des messages de canal standard (pas des deltas de tokens).
- **Streaming de type token (Telegram uniquement) :** met à jour une **bulle de brouillon** avec du texte partiel pendant la génération ; le message final est envoyé à la fin.

Il n’y a **pas de véritable streaming de tokens** vers les messages des canaux externes aujourd’hui. Le streaming de brouillons Telegram est le seul mécanisme de streaming partiel disponible.

<div id="block-streaming-channel-messages">
  ## Diffusion en blocs (messages du canal)
</div>

La diffusion en blocs envoie la sortie de l’assistant sous forme de gros blocs au fur et à mesure qu’elle devient disponible.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Légende :

* `text_delta/events` : événements de streaming du modèle (peuvent être clairsemés pour les modèles non diffusés en continu).
* `chunker` : `EmbeddedBlockChunker` appliquant les bornes min/max + préférence de coupure.
* `channel send` : messages sortants réels (réponses par bloc).

**Paramètres :**

* `agents.defaults.blockStreamingDefault` : `"on"`/`"off"` (désactivé par défaut).
* Surcharges par canal : `*.blockStreaming` (et variantes par compte) pour forcer `"on"`/`"off"` par canal.
* `agents.defaults.blockStreamingBreak` : `"text_end"` ou `"message_end"`.
* `agents.defaults.blockStreamingChunk` : `{ minChars, maxChars, breakPreference? }`.
* `agents.defaults.blockStreamingCoalesce` : `{ minChars?, maxChars?, idleMs? }` (fusionner les blocs diffusés avant envoi).
* Plafond strict du canal : `*.textChunkLimit` (par ex. `channels.whatsapp.textChunkLimit`).
* Mode de découpage du canal : `*.chunkMode` (`length` par défaut, `newline` découpe sur les lignes vides (délimitations de paragraphes) avant le découpage par longueur).
* Plafond souple Discord : `channels.discord.maxLinesPerMessage` (17 par défaut) découpe les réponses très longues pour éviter qu’elles ne soient rognées dans l’UI.

**Sémantique des délimitations :**

* `text_end` : diffuser les blocs dès que le chunker les émet ; vider à chaque `text_end`.
* `message_end` : attendre la fin du message de l’assistant, puis vider la sortie mise en mémoire tampon.

`message_end` utilise toujours le chunker si le texte mis en mémoire tampon dépasse `maxChars`, de sorte qu’il peut émettre plusieurs blocs à la fin.


<div id="chunking-algorithm-lowhigh-bounds">
  ## Algorithme de découpage (bornes basse/haute)
</div>

Le découpage en blocs est implémenté par `EmbeddedBlockChunker` :

- **Borne basse :** n’émet pas tant que le tampon &gt;= `minChars` (sauf si forcé).
- **Borne haute :** privilégie les séparations avant `maxChars` ; si forcé, coupe à `maxChars`.
- **Préférence de coupure :** `paragraph` → `newline` → `sentence` → `whitespace` → coupure forcée.
- **Blocs de code (code fences) :** ne jamais couper à l’intérieur des blocs ; en cas de découpe forcée à `maxChars`, fermer puis rouvrir le bloc pour conserver un Markdown valide.

`maxChars` est borné par le `textChunkLimit` du canal, donc vous ne pouvez pas dépasser les limites propres à chaque canal.

<div id="coalescing-merge-streamed-blocks">
  ## Fusion (regroupement des blocs diffusés en continu)
</div>

Lorsque la diffusion de blocs est activée, OpenClaw peut **fusionner des blocs consécutifs**
avant de les envoyer. Cela réduit le « spam de lignes uniques » tout en conservant
un affichage progressif de la sortie.

- La fusion attend des **périodes d’inactivité** (`idleMs`) avant de vider le tampon.
- Les tampons sont limités par `maxChars` et seront vidés s’ils dépassent cette valeur.
- `minChars` empêche l’envoi de minuscules fragments tant qu’assez de texte ne s’est pas accumulé
  (la vidange finale envoie toujours le texte restant).
- Le séparateur est dérivé de `blockStreamingChunk.breakPreference`
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espace).
- Des surcharges spécifiques par canal sont disponibles via `*.blockStreamingCoalesce` (y compris des configurations par compte).
- La valeur par défaut de `minChars` pour la fusion est portée à 1500 pour Signal/Slack/Discord, sauf si elle est surchargée.

<div id="human-like-pacing-between-blocks">
  ## Rythme humain entre les blocs
</div>

Lorsque le streaming par blocs est activé, vous pouvez ajouter une **pause aléatoire**
entre les réponses (après le premier bloc). Cela rend les réponses en plusieurs bulles
plus naturelles.

- Config : `agents.defaults.humanDelay` (peut être surchargée pour chaque agent via `agents.list[].humanDelay`).
- Modes : `off` (par défaut), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- S’applique uniquement aux **réponses par blocs**, pas aux réponses finales ni aux résumés d’outils.

<div id="stream-chunks-or-everything">
  ## « Diffuser par blocs ou tout diffuser »
</div>

Cela correspond à :

- **Diffuser par blocs :** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (émission au fil de l’eau). Les canaux non Telegram ont aussi besoin de `*.blockStreaming: true`.
- **Tout diffuser à la fin :** `blockStreamingBreak: "message_end"` (vidage en une fois, éventuellement en plusieurs blocs si c’est très long).
- **Aucune diffusion par blocs :** `blockStreamingDefault: "off"` (seulement la réponse finale).

**Note sur les canaux :** Pour les canaux non Telegram, la diffusion par blocs est **désactivée sauf si**
`*.blockStreaming` est explicitement défini à `true`. Telegram peut diffuser des brouillons
(`channels.telegram.streamMode`) sans réponses par blocs.

Rappel sur l’emplacement de la configuration : les valeurs par défaut `blockStreaming*` se trouvent sous
`agents.defaults`, et non à la racine de la configuration.

<div id="telegram-draft-streaming-token-ish">
  ## Streaming des brouillons Telegram (granularité type tokens)
</div>

Telegram est le seul canal avec streaming de brouillons :

* Utilise la Bot API `sendMessageDraft` dans les **discussions privées avec sujets**.
* `channels.telegram.streamMode: "partial" | "block" | "off"`.
  * `partial` : mises à jour du brouillon avec le dernier texte du flux.
  * `block` : mises à jour du brouillon par blocs segmentés (mêmes règles de découpage).
  * `off` : aucun streaming de brouillon.
* Configuration de découpage des brouillons (uniquement pour `streamMode: "block"`) : `channels.telegram.draftChunk` (valeurs par défaut : `minChars: 200`, `maxChars: 800`).
* Le streaming de brouillon est distinct du streaming par blocs ; les réponses par blocs sont désactivées par défaut et uniquement activées par `*.blockStreaming: true` sur les canaux autres que Telegram.
* La réponse finale reste un message normal.
* `/reasoning stream` écrit le raisonnement dans la bulle de brouillon (Telegram uniquement).

Lorsque le streaming de brouillon est actif, OpenClaw désactive le streaming par blocs pour cette réponse afin d’éviter un double streaming.

```
Telegram (privé + topics)
  └─ sendMessageDraft (bulle de brouillon)
       ├─ streamMode=partial → mise à jour du dernier texte
       └─ streamMode=block   → le chunker met à jour le brouillon
  └─ réponse finale → message normal
```

Légende :

* `sendMessageDraft` : bulle de brouillon Telegram (ce n’est pas un vrai message).
* `final reply` : envoi d’un message Telegram normal.
