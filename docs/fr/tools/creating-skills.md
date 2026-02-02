---
title: Créer des compétences
---

<div id="creating-custom-skills">
  # Création de compétences personnalisées 🛠
</div>

OpenClaw est conçu pour être facilement extensible. Les « compétences » sont le principal moyen d’ajouter de nouvelles capacités à votre assistant.

<div id="what-is-a-skill">
  ## Qu’est-ce qu’une Skill ?
</div>

Une Skill est un répertoire qui contient un fichier `SKILL.md` (qui fournit des instructions et des définitions d’outils au LLM) et, éventuellement, des scripts ou des ressources.

<div id="step-by-step-your-first-skill">
  ## Guide pas à pas : votre première compétence
</div>

<div id="1-create-the-directory">
  ### 1. Créer le répertoire
</div>

Les compétences se trouvent dans votre espace de travail, généralement dans `~/.openclaw/workspace/skills/`. Créez un nouveau dossier pour votre compétence :

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. Définir le `SKILL.md`
</div>

Créez un fichier `SKILL.md` dans ce répertoire. Ce fichier utilise un en-tête (front matter) YAML pour les métadonnées et du Markdown pour les instructions.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. Ajouter des outils (facultatif)
</div>

Vous pouvez définir des outils personnalisés dans le frontmatter ou demander à l'agent d'utiliser des outils système existants (comme `bash` ou `browser`).

<div id="4-refresh-openclaw">
  ### 4. Actualiser OpenClaw
</div>

Demandez à votre agent d’exécuter la commande « refresh skills » ou redémarrez Gateway. OpenClaw détectera le nouveau répertoire et indexera le `SKILL.md`.

<div id="best-practices">
  ## Bonnes pratiques
</div>

- **Soyez concis** : indiquez au modèle *ce qu’il doit* faire, pas comment être une IA.
- **La sécurité avant tout** : si votre skill utilise `bash`, assurez-vous que les prompts n’autorisent pas l’injection de commandes arbitraires à partir d’une entrée utilisateur non fiable.
- **Testez en local** : utilisez `openclaw agent --message "use my new skill"` pour tester.

<div id="shared-skills">
  ## Compétences partagées
</div>

Vous pouvez également parcourir les compétences sur [ClawHub](https://clawhub.com) et y contribuer.