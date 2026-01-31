---
title: Ollama
summary: "Ejecutar OpenClaw con Ollama (entorno de ejecución local de LLM)"
read_when:
  - Quieres ejecutar OpenClaw con modelos locales mediante Ollama
  - Necesitas instrucciones para la instalación y configuración de Ollama
---

<div id="ollama">
  # Ollama
</div>

Ollama es un entorno de ejecución local para LLM que facilita la ejecución de modelos de código abierto en tu máquina. OpenClaw se integra con la API compatible con OpenAI de Ollama y puede **descubrir automáticamente modelos compatibles con herramientas** cuando habilitas esta opción con `OLLAMA_API_KEY` (o un perfil de autenticación) y no defines una entrada explícita `models.providers.ollama`.

<div id="quick-start">
  ## Inicio rápido
</div>

1. Instala Ollama: https://ollama.ai

2. Descarga un modelo:

```bash
ollama pull llama3.3
# o
ollama pull qwen2.5-coder:32b
# o
ollama pull deepseek-r1:32b
```

3. Activa Ollama para OpenClaw (cualquier valor sirve; Ollama no requiere una clave real):

```bash
# Establecer la variable de entorno
export OLLAMA_API_KEY="ollama-local"

# O configurar en el archivo de configuración
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Utiliza modelos de Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## Descubrimiento de modelos (proveedor implícito)
</div>

Cuando configuras `OLLAMA_API_KEY` (o un perfil de autenticación) y **no** defines `models.providers.ollama`, OpenClaw descubre modelos a partir de la instancia local de Ollama en `http://127.0.0.1:11434`:

* Consulta `/api/tags` y `/api/show`
* Mantiene solo los modelos que reportan la capacidad `tools`
* Marca `reasoning` cuando el modelo reporta `thinking`
* Lee `contextWindow` desde `model_info["<arch>.context_length"]` cuando está disponible
* Establece `maxTokens` a 10× la ventana de contexto
* Establece todos los costes en `0`

Esto evita definir modelos manualmente y mantiene el catálogo alineado con las capacidades de Ollama.

Para ver qué modelos están disponibles:

```bash
ollama list
openclaw models list
```

Para añadir un nuevo modelo, basta con descargarlo con Ollama:

```bash
ollama pull mistral
```

El nuevo modelo se detectará automáticamente y estará disponible para usarlo.

Si configuras `models.providers.ollama` explícitamente, se omite la detección automática y debes definir los modelos manualmente (ver más abajo).

<div id="configuration">
  ## Configuración
</div>

<div id="basic-setup-implicit-discovery">
  ### Configuración básica (descubrimiento implícito)
</div>

La forma más sencilla de habilitar Ollama es a través de una variable de entorno:

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### Configuración explícita (modelos configurados manualmente)
</div>

Usa una configuración explícita cuando:

* Ollama se ejecuta en otro host o puerto.
* Quieres forzar ventanas de contexto específicas o listas concretas de modelos.
* Quieres incluir modelos que no declaran compatibilidad con herramientas.

```json5
{
  models: {
    providers: {
      ollama: {
        // Usa un host que incluya /v1 para APIs compatibles con OpenAI
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Si `OLLAMA_API_KEY` está definida, puedes omitir `apiKey` en la entrada del proveedor y OpenClaw la rellenará automáticamente para las comprobaciones de disponibilidad.

<div id="custom-base-url-explicit-config">
  ### URL base personalizada (configuración explícita)
</div>

Si Ollama se está ejecutando en un host o puerto diferente (la configuración explícita desactiva el descubrimiento automático, así que define los modelos manualmente):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### Selección de modelos
</div>

Una vez que esté todo configurado, todos tus modelos de Ollama estarán disponibles:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## Opciones avanzadas
</div>

<div id="reasoning-models">
  ### Modelos de razonamiento
</div>

OpenClaw marca los modelos como con capacidad de razonamiento cuando Ollama indica `thinking` en `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### Costes de los modelos
</div>

Ollama es gratuito y se ejecuta localmente, por lo que todos los modelos tienen un coste de 0 $.

<div id="context-windows">
  ### Ventanas de contexto
</div>

Para modelos detectados automáticamente, OpenClaw usa la ventana de contexto informada por Ollama cuando está disponible; de lo contrario, usa `8192` por defecto. Puedes anular `contextWindow` y `maxTokens` en la configuración explícita del proveedor.

<div id="troubleshooting">
  ## Solución de problemas
</div>

<div id="ollama-not-detected">
  ### Ollama no detectado
</div>

Asegúrate de que Ollama se esté ejecutando, de que configuraste `OLLAMA_API_KEY` (o un perfil de autenticación) y de que **no** definiste ninguna entrada explícita para `models.providers.ollama`:

```bash
ollama serve
```

Y que la API esté accesible:

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### No hay modelos disponibles
</div>

OpenClaw solo detecta automáticamente modelos que declaran compatibilidad con herramientas. Si tu modelo no aparece en la lista, entonces:

* Descarga un modelo con soporte de herramientas, o
* Define el modelo explícitamente en `models.providers.ollama`.

Para añadir modelos:

```bash
ollama list  # Ver qué está instalado
ollama pull llama3.3  # Descargar un modelo
```

<div id="connection-refused">
  ### Conexión rechazada
</div>

Verifica que Ollama se esté ejecutando en el puerto correcto:

```bash
# Verificar si Ollama se está ejecutando
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

<div id="see-also">
  ## Consulta también
</div>

* [Proveedores de modelos](/es/concepts/model-providers) - Descripción general de todos los proveedores
* [Selección de modelos](/es/concepts/models) - Cómo elegir modelos
* [Configuración](/es/gateway/configuration) - Referencia completa de la configuración