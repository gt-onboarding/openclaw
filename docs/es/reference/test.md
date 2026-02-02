---
title: Pruebas
summary: "Cómo ejecutar pruebas localmente (vitest) y cuándo usar los modos force/coverage"
read_when:
  - Ejecutar o corregir pruebas
---

<div id="tests">
  # Tests
</div>

* Kit de pruebas completo (suites, en vivo, Docker): [Testing](/es/testing)

* `pnpm test:force`: Mata cualquier proceso residual del Gateway que esté usando el puerto de control predeterminado y luego ejecuta toda la suite de Vitest con un puerto de Gateway aislado para que las pruebas de servidor no entren en conflicto con una instancia en ejecución. Usa este comando cuando una ejecución previa del Gateway haya dejado ocupado el puerto 18789.

* `pnpm test:coverage`: Ejecuta Vitest con cobertura V8. Los umbrales globales son 70% para líneas/ramas/funciones/declaraciones. La cobertura excluye puntos de entrada con mucha integración (conexión de la CLI, puentes Gateway/Telegram, servidor estático de webchat) para mantener el objetivo centrado en la lógica comprobable con tests unitarios.

* `pnpm test:e2e`: Ejecuta pruebas de humo de extremo a extremo del Gateway (emparejamiento multiinstancia WS/HTTP/nodo).

* `pnpm test:live`: Ejecuta pruebas en vivo de proveedores (minimax/zai). Requiere claves de API y `LIVE=1` (o `*_LIVE_TEST=1` específico del proveedor) para que no se omitan.

<div id="model-latency-bench-local-keys">
  ## Banco de pruebas de latencia de modelos (claves locales)
</div>

Script: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Uso:

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* Variables de entorno opcionales: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
* Prompt predeterminado: «Responde con una sola palabra: ok. Sin puntuación ni texto adicional».

Última ejecución (2025-12-31, 20 ejecuciones):

* minimax mediana 1279 ms (mín. 1114, máx. 2431)
* opus mediana 2454 ms (mín. 1224, máx. 3170)

<div id="onboarding-e2e-docker">
  ## Incorporación E2E (Docker)
</div>

Docker es opcional; solo lo necesitas para ejecutar pruebas de humo de incorporación en contenedores.

Flujo completo de arranque en frío en un contenedor Linux limpio:

```bash
scripts/e2e/onboard-docker.sh
```

Este script controla el asistente interactivo mediante un pseudo-tty, comprueba los archivos de configuración/espacio de trabajo/sesión y, a continuación, inicia el Gateway y ejecuta `openclaw health`.

<div id="qr-import-smoke-docker">
  ## Prueba de humo de importación de QR (Docker)
</div>

Verifica que `qrcode-terminal` se cargue en Node.js 22+ dentro de Docker:

```bash
pnpm test:docker:qr
```
