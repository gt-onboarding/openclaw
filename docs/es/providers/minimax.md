---
title: MiniMax
summary: "Usa MiniMax M2.1 en OpenClaw"
read_when:
  - Quieres usar modelos MiniMax en OpenClaw
  - Necesitas ayuda para configurar MiniMax
---

<div id="minimax">
  # MiniMax
</div>

MiniMax es una empresa de IA que desarrolla la familia de modelos **M2/M2.1**. La versión actual orientada a la programación es **MiniMax M2.1** (23 de diciembre de 2025), diseñada para tareas complejas del mundo real.

Fuente: [Nota de lanzamiento de MiniMax M2.1](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## Descripción general del modelo (M2.1)
</div>

MiniMax destaca las siguientes mejoras en M2.1:

* Programación **multilenguaje** más sólida (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
* Mejor **desarrollo web y de aplicaciones** y mayor calidad estética de los resultados (incluidas apps móviles nativas).
* Manejo mejorado de **instrucciones compuestas** para flujos de trabajo de tipo oficina, basados en
  razonamiento intercalado y ejecución integrada de restricciones.
* **Respuestas más concisas** con menor uso de tokens y bucles de iteración más rápidos.
* Mayor compatibilidad del **framework de herramientas/agentes** y mejor gestión de contexto (Claude Code,
  Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
* Resultados de mayor calidad en **diálogo y redacción técnica**.

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 vs MiniMax M2.1 Lightning
</div>

* **Velocidad:** Lightning es la variante “rápida” según la documentación de precios de MiniMax.
* **Costo:** La tabla de precios muestra el mismo costo de entrada, pero Lightning tiene un costo de salida más alto.
* **Enrutamiento en el plan de programación:** El backend Lightning no está disponible directamente en el plan de programación de MiniMax. MiniMax enruta automáticamente la mayoría de las solicitudes a Lightning, pero vuelve al backend M2.1 normal durante los picos de tráfico.

<div id="choose-a-setup">
  ## Elige una configuración
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — recomendado
</div>

**Mejor para:** MiniMax alojado con API compatible con Anthropic.

Configúralo desde la CLI:

* Ejecuta `openclaw configure`
* Selecciona **Model/auth**
* Elige **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### MiniMax M2.1 como respaldo (Opus principal)
</div>

**Ideal para:** mantener Opus 4.5 como principal y conmutar automáticamente a MiniMax M2.1 en caso de fallo.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### Opcional: en local mediante LM Studio (manual)
</div>

**Ideal para:** inferencia local con LM Studio.
Hemos observado muy buenos resultados con MiniMax M2.1 en hardware potente (por ejemplo, un
equipo de escritorio o servidor) utilizando el servidor local de LM Studio.

Configura manualmente en `openclaw.json`:

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="configure-via-openclaw-configure">
  ## Configurar mediante `openclaw configure`
</div>

Usa el asistente interactivo de configuración para configurar MiniMax sin editar JSON:

1. Ejecuta `openclaw configure`.
2. Selecciona **Model/auth**.
3. Elige **MiniMax M2.1**.
4. Escoge tu modelo predeterminado cuando se te solicite.

<div id="configuration-options">
  ## Opciones de configuración
</div>

* `models.providers.minimax.baseUrl`: es preferible `https://api.minimax.io/anthropic` (compatible con Anthropic); `https://api.minimax.io/v1` es opcional para payloads compatibles con OpenAI.
* `models.providers.minimax.api`: es preferible `anthropic-messages`; `openai-completions` es opcional para payloads compatibles con OpenAI.
* `models.providers.minimax.apiKey`: clave de API de MiniMax (`MINIMAX_API_KEY`).
* `models.providers.minimax.models`: define `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
* `agents.defaults.models`: crea alias para los modelos que quieras en la lista de permitidos.
* `models.mode`: mantén `merge` si quieres añadir MiniMax junto con los modelos incorporados.

<div id="notes">
  ## Notas
</div>

* Las referencias de modelo son `minimax/<model>`.
* API para el uso de Coding Plan: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (requiere una clave de Coding Plan).
* Actualiza los valores de precios en `models.json` si necesitas un seguimiento preciso de los costos.
* Enlace de recomendación para MiniMax Coding Plan (10% de descuento): https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* Consulta [/concepts/model-providers](/es/concepts/model-providers) para las reglas de los proveedores.
* Usa `openclaw models list` y `openclaw models set minimax/MiniMax-M2.1` para cambiar.

<div id="troubleshooting">
  ## Resolución de problemas
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### “Unknown model: minimax/MiniMax-M2.1”
</div>

Esto normalmente significa que el **proveedor MiniMax no está configurado** (no hay entrada de proveedor
y no se encontró ningún perfil de autenticación/clave de entorno de MiniMax). Hay una corrección para esta detección en
**2026.1.12** (aún no publicada en el momento de escribir esto). Para solucionarlo:

* Actualiza a **2026.1.12** (o ejecútalo desde `main`) y luego reinicia el Gateway.
* Ejecuta `openclaw configure` y selecciona **MiniMax M2.1**, o
* Añade manualmente el bloque `models.providers.minimax`, o
* Configura `MINIMAX_API_KEY` (o un perfil de autenticación de MiniMax) para poder inyectar el proveedor.

Asegúrate de que el ID del modelo respeta **mayúsculas y minúsculas**:

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

Luego vuelve a comprobar con:

```bash
openclaw models list
```
