---
title: Suivi d'utilisation
summary: "Vues de suivi d'utilisation et exigences relatives aux identifiants"
read_when:
  - Vous raccordez les vues d'utilisation/quota des fournisseurs
  - Vous devez expliquer le fonctionnement du suivi d'utilisation ou les exigences en matière d'identifiants
---

<div id="usage-tracking">
  # Suivi d'utilisation
</div>

<div id="what-it-is">
  ## Ce que c'est
</div>

- Récupère l’utilisation et le quota du fournisseur directement à partir de ses endpoints d’utilisation.
- Aucun coût estimé ; uniquement les fenêtres temporelles rapportées par le fournisseur.

<div id="where-it-shows-up">
  ## Où ces informations apparaissent
</div>

- `/status` dans les conversations : carte d’état avec émojis, jetons de session + coût estimé (clé API uniquement). L’utilisation du fournisseur s’affiche pour le **fournisseur de modèle actuel** lorsqu’elle est disponible.
- `/usage off|tokens|full` dans les conversations : pied de page d’utilisation pour chaque réponse (OAuth affiche uniquement les jetons).
- `/usage cost` dans les conversations : récapitulatif local des coûts agrégé à partir des journaux de session OpenClaw.
- CLI : `openclaw status --usage` affiche une ventilation complète par fournisseur.
- CLI : `openclaw channels list` affiche le même instantané d’utilisation à côté de la configuration du fournisseur (utilisez `--no-usage` pour l’ignorer).
- Barre de menus macOS : section « Usage » sous « Context » (uniquement si disponible).

<div id="providers-credentials">
  ## Fournisseurs + identifiants
</div>

- **Anthropic (Claude)** : jetons OAuth dans les profils d’authentification.
- **GitHub Copilot** : jetons OAuth dans les profils d’authentification.
- **Gemini CLI** : jetons OAuth dans les profils d’authentification.
- **Antigravity** : jetons OAuth dans les profils d’authentification.
- **OpenAI Codex** : jetons OAuth dans les profils d’authentification (accountId utilisé le cas échéant).
- **MiniMax** : clé API (clé de forfait de codage ; `MINIMAX_CODE_PLAN_KEY` ou `MINIMAX_API_KEY`) ; utilise une fenêtre de 5 heures pour le forfait de codage.
- **z.ai** : clé API via le store env/config/auth.

Le suivi d’utilisation est masqué si aucun identifiant OAuth/API correspondant n’existe.