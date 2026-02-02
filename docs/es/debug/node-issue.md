---
title: Problema con Node
summary: Notas y soluciones alternativas sobre el fallo de Node + tsx "__name is not a function"
read_when:
  - Depurar scripts de desarrollo exclusivos de Node o fallos en el modo watch
  - Investigar cierres inesperados del cargador tsx/esbuild en OpenClaw
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Bloqueo de Node + tsx con "__name is not a function"
</div>

<div id="summary">
  ## Resumen
</div>

La ejecución de OpenClaw con Node y `tsx` produce el siguiente error al arrancar:

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

Este problema empezó después de cambiar los scripts de desarrollo de Bun a `tsx` (commit `2871657e`, 2026-01-06). La misma ruta de ejecución funcionaba con Bun.


<div id="environment">
  ## Entorno
</div>

- Node: v25.x (observado en v25.3.0)
- tsx: 4.21.0
- SO: macOS (es probable que este problema también se reproduzca en otras plataformas que ejecuten Node 25)

<div id="repro-node-only">
  ## Repro (solo nodo)
</div>

```bash
# en la raíz del repositorio
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## Caso mínimo reproducible en el repositorio
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Comprobación de la versión de Node
</div>

- Node 25.3.0: falla
- Node 22.22.0 (Homebrew `node@22`): falla
- Node 24: aún no está instalado aquí; pendiente de verificación

<div id="notes-hypothesis">
  ## Notas / hipótesis
</div>

- `tsx` usa esbuild para transformar TS/ESM. La opción `keepNames` de esbuild emite un helper `__name` y envuelve las definiciones de funciones con `__name(...)`.
- El error indica que `__name` existe pero no es una función en tiempo de ejecución, lo que implica que el helper falta o se ha sobrescrito para este módulo en la ruta del cargador de Node 25.
- Se han reportado problemas similares con el helper `__name` en otros usuarios de esbuild cuando el helper falta o se reescribe.

<div id="regression-history">
  ## Historial de regresiones
</div>

- `2871657e` (2026-01-06): los scripts se cambiaron de Bun a tsx para que Bun pasara a ser opcional.
- Antes de eso (ruta basada en Bun), `openclaw status` y `gateway:watch` funcionaban.

<div id="workarounds">
  ## Soluciones alternativas
</div>

- Usa Bun para scripts de desarrollo (reversión temporal actual).
- Usa Node + tsc en modo watch y luego ejecuta el resultado compilado:
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- Confirmado localmente: `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` funciona en Node 25.
- Desactiva `keepNames` de esbuild en el TS loader si es posible (evita la inserción del helper `__name`); tsx actualmente no expone esta opción.
- Prueba Node LTS (22/24) con `tsx` para ver si el problema es específico de Node 25.

<div id="references">
  ## Referencias
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## Próximos pasos
</div>

- Reproducir el problema en Node 22/24 para confirmar que se trata de una regresión en Node 25.
- Probar la versión nightly de `tsx` o fijar la dependencia a una versión anterior si existe una regresión conocida.
- Si se reproduce en Node LTS, abrir un issue con un caso mínimo reproducible en el proyecto upstream incluyendo el stack trace de `__name`.