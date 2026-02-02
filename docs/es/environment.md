---
title: Entorno
summary: "Dónde carga OpenClaw las variables de entorno y en qué orden de precedencia"
read_when:
  - Necesitas saber qué variables de entorno se cargan y en qué orden
  - Estás depurando claves de API que faltan en el Gateway
  - Estás documentando la autenticación del proveedor o los entornos de implementación
---

<div id="environment-variables">
  # Variables de entorno
</div>

OpenClaw toma variables de entorno de múltiples orígenes. La regla es: **nunca sobrescribas valores existentes**.

<div id="precedence-highest-lowest">
  ## Precedencia (de mayor a menor)
</div>

1. **Entorno del proceso** (lo que el proceso del Gateway ya hereda del shell/daemon padre).
2. **`.env` en el directorio de trabajo actual** (valor predeterminado de dotenv; no sobrescribe).
3. **`.env` global** en `~/.openclaw/.env` (también conocido como `$OPENCLAW_STATE_DIR/.env`; no sobrescribe).
4. **Bloque `env` de la configuración** en `~/.openclaw/openclaw.json` (se aplica solo si falta).
5. **Importación opcional del shell de inicio de sesión** (`env.shellEnv.enabled` o `OPENCLAW_LOAD_SHELL_ENV=1`), aplicada solo para claves esperadas que falten.

Si el archivo de configuración falta por completo, se omite el paso 4; la importación desde el shell sigue ejecutándose si está habilitada.

<div id="config-env-block">
  ## Bloque de configuración `env`
</div>

Dos formas equivalentes de definir variables de entorno en línea (ambas sin sobrescribir valores existentes):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

<div id="shell-env-import">
  ## Importación del entorno del shell
</div>

`env.shellEnv` ejecuta tu shell de inicio de sesión e importa únicamente las claves esperadas que **no estén definidas**:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

Variables de entorno equivalentes:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ## Sustitución de variables de entorno en la configuración
</div>

Puedes hacer referencia directamente a las variables de entorno en los valores de cadena de la configuración usando la sintaxis `${VAR_NAME}`:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

Consulta [Configuración: sustitución de variables de entorno](/es/gateway/configuration#env-var-substitution-in-config) para más detalles.

<div id="related">
  ## Contenido relacionado
</div>

* [Configuración del Gateway](/es/gateway/configuration)
* [Preguntas frecuentes: variables de entorno y carga de .env](/es/help/faq#env-vars-and-env-loading)
* [Información general sobre modelos](/es/concepts/models)