---
title: Github Copilot
summary: "Connectez-vous à GitHub Copilot à partir d’OpenClaw à l’aide du flux d’appareil (« device flow »)"
read_when:
  - Vous souhaitez utiliser GitHub Copilot comme fournisseur de modèles
  - Vous avez besoin du flux `openclaw models auth login-github-copilot`
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## Qu’est-ce que GitHub Copilot ?
</div>

GitHub Copilot est l’assistant de programmation basé sur l’IA de GitHub. Il offre un accès aux modèles Copilot pour votre compte et votre abonnement GitHub. OpenClaw peut utiliser Copilot comme fournisseur de modèles de deux manières différentes.

<div id="two-ways-to-use-copilot-in-openclaw">
  ## Deux manières d'utiliser Copilot avec OpenClaw
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) Fournisseur GitHub Copilot intégré (`github-copilot`)
</div>

Utilise la procédure native de connexion par appareil pour obtenir un jeton GitHub, puis l’échanger contre des jetons d’API Copilot lors de l’exécution d’OpenClaw. C’est l’option **par défaut** et la plus simple, car elle ne nécessite pas VS Code.

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Copilot Proxy plugin (`copilot-proxy`)
</div>

Utilisez l’extension **Copilot Proxy** pour VS Code comme pont local. OpenClaw communique avec
le point de terminaison `/v1` du proxy et utilise la liste de modèles que vous y configurez. Choisissez
cette option si vous exécutez déjà Copilot Proxy dans VS Code ou si vous devez faire transiter vos requêtes par celui‑ci.
Vous devez activer le plugin et laisser l’extension VS Code en cours d’exécution.

Utilisez GitHub Copilot comme fournisseur de modèles (`github-copilot`). La commande de connexion exécute
le flux d’authentification par appareil de GitHub, enregistre un profil d’authentification et met à jour votre configuration pour utiliser ce
profil.

<div id="cli-setup">
  ## Configuration de la CLI
</div>

```bash
openclaw models auth login-github-copilot
```

Vous serez invité à ouvrir une URL et à saisir un code à usage unique. Laissez le terminal
ouvert jusqu&#39;à la fin de l&#39;opération.


<div id="optional-flags">
  ### Options facultatives
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## Définir un modèle par défaut
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### Exemple de configuration
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## Notes
</div>

- Nécessite un TTY interactif ; exécutez la commande directement dans un terminal.
- La disponibilité des modèles Copilot dépend de votre abonnement ; si un modèle est refusé, essayez
  un autre identifiant (par exemple `github-copilot/gpt-4.1`).
- La connexion enregistre un jeton GitHub dans le stockage des profils d’authentification et l’échange contre un
  jeton d’API Copilot lorsque OpenClaw est en cours d’exécution.