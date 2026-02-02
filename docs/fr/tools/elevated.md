---
title: Mode élevé
summary: "Mode d’exécution élevé et directives /elevated"
read_when:
  - Ajustement des paramètres par défaut du mode élevé, des listes d’autorisation ou du comportement des commandes slash
---

<div id="elevated-mode-elevated-directives">
  # Mode avec privilèges élevés (/elevated directives)
</div>

<div id="what-it-does">
  ## Ce que cela fait
</div>

- `/elevated on` s’exécute sur l’hôte de Gateway et conserve les approbations d’exec (identique à `/elevated ask`).
- `/elevated full` s’exécute sur l’hôte de Gateway **et** approuve automatiquement exec (ignore les approbations d’exec).
- `/elevated ask` s’exécute sur l’hôte de Gateway mais conserve les approbations d’exec (identique à `/elevated on`).
- `on`/`ask` ne forcent **pas** `exec.security=full` ; la stratégie de sécurité/de demande (ask) configurée continue de s’appliquer.
- Ne modifie le comportement que lorsque l’agent est **en sandbox** (sinon exec s’exécute déjà sur l’hôte).
- Formes de directive : `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Seuls `on|off|ask|full` sont acceptés ; toute autre valeur renvoie une indication et ne change pas l’état.

<div id="what-it-controls-and-what-it-doesnt">
  ## Ce que cela contrôle (et ce que ça ne contrôle pas)
</div>

- **Garde-fous de disponibilité** : `tools.elevated` est la référence globale. `agents.list[].tools.elevated` peut restreindre davantage l’élévation par agent (les deux doivent l’autoriser).
- **État par session** : `/elevated on|off|ask|full` définit le niveau d’élévation pour la clé de session en cours.
- **Directive en ligne** : `/elevated on|ask|full` à l’intérieur d’un message s’applique uniquement à ce message.
- **Groupes** : dans les discussions de groupe, les directives elevated ne sont prises en compte que lorsque l’agent est mentionné. Les messages constitués uniquement d’une commande, qui contournent l’exigence de mention, sont traités comme si l’agent était mentionné.
- **Exécution sur l’hôte** : elevated force `exec` sur l’hôte de Gateway ; `full` définit aussi `security=full`.
- **Approbations** : `full` ignore les demandes d’approbation pour exec ; `on`/`ask` les respectent lorsque les règles de liste d’autorisation/demande l’exigent.
- **Agents non sandboxés** : aucun effet sur l’emplacement d’exécution ; n’affecte que les garde-fous, la journalisation et l’état.
- **La stratégie d’outil s’applique toujours** : si `exec` est refusé par la stratégie d’outil, elevated ne peut pas être utilisé.
- **Séparé de `/exec`** : `/exec` ajuste les paramètres par défaut de la session pour les émetteurs autorisés et ne nécessite pas elevated.

<div id="resolution-order">
  ## Ordre de résolution
</div>

1. Directive en ligne sur le message (s'applique uniquement à ce message).
2. Surcharge de session (définie en envoyant un message ne contenant qu'une directive).
3. Valeur par défaut globale (`agents.defaults.elevatedDefault` dans la configuration).

<div id="setting-a-session-default">
  ## Définir la valeur par défaut d’une session
</div>

- Envoyez un message constitué **uniquement** de la directive (espaces autorisés), par ex. `/elevated full`.
- Un message de confirmation est envoyé (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Si le mode avec élévation est désactivé ou si l’expéditeur ne figure pas dans la liste d’autorisation, la directive renvoie une erreur exploitable et ne modifie pas l’état de la session.
- Envoyez `/elevated` (ou `/elevated:`) sans argument pour voir le niveau d’élévation actuel.

<div id="availability-allowlists">
  ## Disponibilité + listes d’autorisation
</div>

- Commutateur de fonctionnalité : `tools.elevated.enabled` (la valeur par défaut peut être désactivée via la configuration même si le code la prend en charge).
- Liste d’autorisation des expéditeurs : `tools.elevated.allowFrom` avec des listes d’autorisation par fournisseur (par ex. `discord`, `whatsapp`).
- Commutateur par agent : `agents.list[].tools.elevated.enabled` (optionnel ; ne peut que restreindre davantage).
- Liste d’autorisation par agent : `agents.list[].tools.elevated.allowFrom` (optionnel ; lorsqu’elle est définie, l’expéditeur doit figurer **à la fois** sur les listes d’autorisation globale + par agent).
- Repli Discord : si `tools.elevated.allowFrom.discord` est omis, la liste `channels.discord.dm.allowFrom` est utilisée comme valeur de repli. Définissez `tools.elevated.allowFrom.discord` (même `[]`) pour la remplacer. Les listes d’autorisation par agent n’utilisent **pas** ce repli.
- Tous les contrôles doivent être validés ; sinon, la fonctionnalité elevated est considérée comme indisponible.

<div id="logging-status">
  ## Journalisation + statut
</div>

- Les appels exec avec élévation sont consignés au niveau info.
- Le statut de la session comprend le mode élevé (par exemple `elevated=ask`, `elevated=full`).