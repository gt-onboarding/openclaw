---
title: Creación de habilidades
---

<div id="creating-custom-skills">
  # Creación de habilidades personalizadas 🛠
</div>

OpenClaw está diseñado para ser fácilmente extensible. Las «habilidades» son la forma principal de incorporar nuevas capacidades a tu asistente.

<div id="what-is-a-skill">
  ## ¿Qué es una Skill?
</div>

Una Skill es un directorio que contiene un archivo `SKILL.md` (que proporciona instrucciones y definiciones de herramientas al LLM) y, de forma opcional, algunos scripts o recursos adicionales.

<div id="step-by-step-your-first-skill">
  ## Paso a paso: tu primera habilidad
</div>

<div id="1-create-the-directory">
  ### 1. Crea el directorio
</div>

Las habilidades se encuentran en tu espacio de trabajo, normalmente en `~/.openclaw/workspace/skills/`. Crea una nueva carpeta para tu habilidad:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. Define el `SKILL.md`
</div>

Crea un archivo `SKILL.md` en ese directorio. Este archivo usa front matter YAML para los metadatos y Markdown para las instrucciones.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. Añadir herramientas (opcional)
</div>

Puedes definir herramientas personalizadas en el frontmatter o indicarle al agente que utilice herramientas del sistema existentes (como `bash` o `browser`).

<div id="4-refresh-openclaw">
  ### 4. Actualizar OpenClaw
</div>

Pídele a tu agente que ejecute "refresh skills" o reinicia el Gateway. OpenClaw descubrirá el nuevo directorio e indexará el `SKILL.md`.

<div id="best-practices">
  ## Mejores prácticas
</div>

- **Sé conciso**: Indícale al modelo *qué* debe hacer, no cómo ser una IA.
- **Prioriza la seguridad**: Si tu skill usa `bash`, asegúrate de que los prompts no permitan inyección arbitraria de comandos desde entrada de usuario no confiable.
- **Prueba localmente**: Usa `openclaw agent --message "use my new skill"` para probar.

<div id="shared-skills">
  ## Habilidades compartidas
</div>

También puedes explorar y aportar habilidades a [ClawHub](https://clawhub.com).