---
title: Creazione di abilità
---

<div id="creating-custom-skills">
  # Creare abilità personalizzate 🛠
</div>

OpenClaw è progettato per essere facilmente estensibile. Le “abilità” sono il principale modo per aggiungere nuove funzionalità al tuo assistente.

<div id="what-is-a-skill">
  ## Che cos'è una skill?
</div>

Una skill è una directory che contiene un file `SKILL.md` (che fornisce all'LLM istruzioni e definizioni di strumenti) e, eventualmente, alcuni script o risorse.

<div id="step-by-step-your-first-skill">
  ## Guida passo passo: la tua prima skill
</div>

<div id="1-create-the-directory">
  ### 1. Crea la directory
</div>

Le abilità si trovano nel tuo spazio di lavoro, di solito in `~/.openclaw/workspace/skills/`. Crea una nuova cartella per la tua abilità:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. Definisci il file `SKILL.md`
</div>

Crea un file `SKILL.md` in questa directory. Questo file utilizza il frontmatter YAML per i metadati e Markdown per le istruzioni.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
Quando l'utente chiede un saluto, usa il tool `echo` per dire "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. Aggiungi strumenti (facoltativo)
</div>

Puoi definire strumenti personalizzati nel frontmatter oppure istruire l'agente a utilizzare strumenti di sistema esistenti (come `bash` o `browser`).

<div id="4-refresh-openclaw">
  ### 4. Aggiorna OpenClaw
</div>

Chiedi al tuo agente di eseguire "refresh skills" oppure riavvia il Gateway. OpenClaw rileverà la nuova directory e indicizzerà il `SKILL.md`.

<div id="best-practices">
  ## Buone pratiche
</div>

- **Sii conciso**: Istruisci il modello su *cosa* deve fare, non su come essere un'IA.
- **Prima la sicurezza**: Se la tua skill usa `bash`, assicurati che i prompt non consentano l'injection arbitraria di comandi da input utente non attendibili.
- **Testa in locale**: Usa `openclaw agent --message "use my new skill"` per eseguire i test.

<div id="shared-skills">
  ## Abilità condivise
</div>

Puoi anche sfogliare le abilità disponibili e contribuire con le tue su [ClawHub](https://clawhub.com).