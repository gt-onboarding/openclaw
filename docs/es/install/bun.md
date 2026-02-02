---
title: Bun
summary: "Flujo de trabajo con Bun (experimental): instalación y particularidades frente a pnpm"
read_when:
  - Quieres el ciclo de desarrollo local más rápido posible (bun + watch)
  - Te topas con problemas de instalación/parches/scripts de ciclo de vida de Bun
---

<div id="bun-experimental">
  # Bun (experimental)
</div>

Objetivo: ejecutar este repositorio con **Bun** (opcional, no recomendado para WhatsApp/Telegram)
sin apartarse de los flujos de trabajo con pnpm.

⚠️ **No se recomienda para el entorno de ejecución del Gateway** (errores en WhatsApp/Telegram). Usa Node para producción.

<div id="status">
  ## Estado
</div>

- Bun es un runtime local opcional para ejecutar TypeScript directamente (`bun run …`, `bun --watch …`).
- `pnpm` es el predeterminado para las compilaciones y sigue estando totalmente compatible (y utilizado por algunas herramientas de documentación).
- Bun no puede usar `pnpm-lock.yaml` y lo ignorará.

<div id="install">
  ## Instalación
</div>

Por defecto:

```sh
bun install
```

Nota: `bun.lock`/`bun.lockb` están ignorados por Git (`.gitignore`), así que no se generan cambios en el repositorio en ningún caso. Si no quieres que se escriba ningún *lockfile*:

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## Compilación / pruebas (Bun)
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Scripts de ciclo de vida de Bun (bloqueados por defecto)
</div>

Bun puede bloquear scripts de ciclo de vida de las dependencias a menos que se confíe explícitamente en ellos (`bun pm untrusted` / `bun pm trust`).
Para este repositorio, los scripts que Bun suele bloquear no son necesarios:

* `@whiskeysockets/baileys` `preinstall`: comprueba que la versión principal de Node sea &gt;= 20 (nosotros usamos Node 22+).
* `protobufjs` `postinstall`: muestra advertencias sobre esquemas de versiones incompatibles (sin artefactos de compilación).

Si te encuentras con un problema real en tiempo de ejecución que requiera estos scripts, márcalos explícitamente como de confianza:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## Advertencias
</div>

- Algunos scripts siguen teniendo `pnpm` hardcodeado (por ejemplo, `docs:build`, `ui:*`, `protocol:check`). Por ahora, ejecútalos con pnpm.