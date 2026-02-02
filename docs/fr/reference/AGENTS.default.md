---
title: AGENTS.default
summary: "Instructions par défaut de l'agent OpenClaw et liste des compétences pour la configuration de l'assistant personnel"
read_when:
  - Lors du démarrage d'une nouvelle session d'agent OpenClaw
  - Lors de l’activation ou du contrôle des compétences par défaut
---

<div id="agentsmd-openclaw-personal-assistant-default">
  # AGENTS.md — Assistant personnel OpenClaw (par défaut)
</div>

<div id="first-run-recommended">
  ## Première exécution (recommandé)
</div>

OpenClaw utilise un répertoire d’espace de travail dédié à l’agent. Par défaut : `~/.openclaw/workspace` (configurable via `agents.defaults.workspace`).

1. Créez l’espace de travail (s’il n’existe pas déjà) :

```bash
mkdir -p ~/.openclaw/workspace
```

2. Copiez les modèles d&#39;espace de travail par défaut dans l&#39;espace de travail :

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Facultatif : si vous voulez la liste des compétences de l&#39;assistant personnel, remplacez AGENTS.md par ce fichier :

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Facultatif : choisissez un autre espace de travail en définissant `agents.defaults.workspace` (prend en charge le symbole `~`) :

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```


<div id="safety-defaults">
  ## Règles de sécurité par défaut
</div>

- Ne déverse pas le contenu de répertoires ni de secrets dans le chat.
- N’exécute pas de commandes destructrices sauf si cela t’est explicitement demandé.
- N’envoie pas de réponses partielles ou en streaming vers des canaux de messagerie externes (uniquement les réponses finales).

<div id="session-start-required">
  ## Démarrage de la session (obligatoire)
</div>

- Lisez `SOUL.md`, `USER.md`, `memory.md`, ainsi que le contenu d’aujourd’hui et d’hier dans `memory/`.
- Faites-le avant de répondre.

<div id="soul-required">
  ## Soul (obligatoire)
</div>

- `SOUL.md` définit l'identité, le ton et les limites. Gardez-le à jour.
- Si vous modifiez `SOUL.md`, informez l'utilisateur.
- Vous êtes une nouvelle instance à chaque session ; la continuité est assurée par ces fichiers.

<div id="shared-spaces-recommended">
  ## Espaces partagés (recommandé)
</div>

- Vous n’êtes pas la voix de l’utilisateur ; faites attention dans les discussions de groupe ou les canaux publics.
- Ne partagez pas de données personnelles, de coordonnées ou de notes internes.

<div id="memory-system-recommended">
  ## Système de mémoire (recommandé)
</div>

- Journal quotidien : `memory/YYYY-MM-DD.md` (créez `memory/` si nécessaire).
- Mémoire à long terme : `memory.md` pour les faits, préférences et décisions durables.
- Au démarrage de la session, lisez les fichiers d’aujourd’hui et d’hier, ainsi que `memory.md` s’il existe.
- À consigner : décisions, préférences, contraintes, points en suspens.
- Évitez les secrets sauf demande explicite.

<div id="tools-skills">
  ## Outils et compétences
</div>

- Les outils vivent dans les compétences ; reportez-vous au fichier `SKILL.md` de chaque compétence lorsque vous en avez besoin.
- Conservez les notes spécifiques à l’environnement dans `TOOLS.md` (Notes pour les compétences).

<div id="backup-tip-recommended">
  ## Astuce de sauvegarde (recommandée)
</div>

Si vous considérez cet espace de travail comme la « mémoire » de Clawd, transformez-le en dépôt Git (idéalement privé) afin que `AGENTS.md` et vos fichiers de mémoire soient sauvegardés.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Optionnel : ajouter un dépôt distant privé et pousser les modifications
```


<div id="what-openclaw-does">
  ## Ce que fait OpenClaw
</div>

- Exécute une passerelle WhatsApp + un agent de développement sur Pi afin que l’assistant puisse lire/écrire dans les conversations, récupérer le contexte et exécuter des compétences via le Mac hôte.
- L’application macOS gère les autorisations (enregistrement d’écran, notifications, microphone) et expose la CLI `openclaw` via son binaire intégré.
- Les conversations directes sont, par défaut, regroupées dans la session `main` de l’agent ; les groupes restent isolés sous la forme `agent:<agentId>:<channel>:group:<id>` (salons/canaux : `agent:<agentId>:<channel>:channel:<id>`) ; les événements `heartbeat` maintiennent les tâches d’arrière-plan actives.

<div id="core-skills-enable-in-settings-skills">
  ## Compétences principales (à activer dans Réglages → Compétences)
</div>

- **mcporter** — Serveur d’outils (runtime/CLI) pour gérer des backends de compétences externes.
- **Peekaboo** — Captures d’écran macOS rapides avec analyse d’images par IA en option.
- **camsnap** — Capture d’images, de clips ou d’alertes de mouvement à partir de caméras de sécurité RTSP/ONVIF.
- **oracle** — CLI d’agent prête pour OpenAI avec relecture de session et contrôle du navigateur.
- **eightctl** — Gérez votre sommeil depuis le terminal.
- **imsg** — Envoyer, read, diffuser iMessage et SMS.
- **wacli** — CLI WhatsApp : synchroniser, rechercher, envoyer.
- **discord** — Actions Discord : réactions, stickers, sondages. Utilisez des cibles `user:<id>` ou `channel:<id>` (de simples ID numériques sont ambigus).
- **gog** — CLI Google Suite : Gmail, Calendar, Drive, Contacts.
- **spotify-player** — Client Spotify en ligne de commande pour rechercher/mettre en file d’attente/contrôler la lecture.
- **sag** — Synthèse vocale ElevenLabs avec UX de type `say` sur mac ; diffuse vers les haut‑parleurs par défaut.
- **Sonos CLI** — Contrôler les enceintes Sonos (découverte/état/lecture/volume/regroupement) depuis des scripts.
- **blucli** — Lire, regrouper et automatiser des lecteurs BluOS depuis des scripts.
- **OpenHue CLI** — Contrôle de l’éclairage Philips Hue pour les scènes et les automatisations.
- **OpenAI Whisper** — Reconnaissance vocale locale pour dictées rapides et transcriptions de messagerie vocale.
- **Gemini CLI** — Modèles Google Gemini depuis le terminal pour des questions-réponses rapides.
- **bird** — CLI X/Twitter pour tweeter, répondre, read des fils et rechercher sans navigateur.
- **agent-tools** — Boîte à outils utilitaire pour automatisations et scripts d’assistance.

<div id="usage-notes">
  ## Notes d’utilisation
</div>

- Pour les scripts, préférez la CLI `openclaw` ; l’app macOS gère les autorisations.
- Effectuez les installations depuis l’onglet Skills ; le bouton est masqué si un binaire est déjà présent.
- Laissez les signaux de vie (heartbeat) activés pour que l’assistant puisse programmer des rappels, surveiller les boîtes de réception et déclencher des captures caméra.
- La Canvas UI s’exécute en plein écran avec des surcouches natives. Évitez de placer des contrôles critiques dans les coins supérieur gauche/droit ou le long des bords inférieurs ; ajoutez des marges de sécurité explicites dans la mise en page et ne comptez pas sur les safe-area insets.
- Pour la vérification pilotée par navigateur, utilisez `openclaw browser` (tabs/status/screenshot) avec le profil Chrome géré par OpenClaw.
- Pour l’inspection du DOM, utilisez `openclaw browser eval|query|dom|snapshot` (et `--json`/`--out` lorsque vous avez besoin d’une sortie exploitable par une machine).
- Pour les interactions, utilisez `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type nécessitent des références de snapshot ; utilisez `evaluate` pour les sélecteurs CSS).