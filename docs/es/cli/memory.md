---
title: Memoria
summary: "Referencia de la CLI para `openclaw memory` (status/index/search)"
read_when:
  - Quieres indexar o buscar memoria semántica
  - Estás depurando problemas de disponibilidad o indexación de memoria
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

Administra la indexación y búsqueda de la memoria semántica.
Proporcionado por el complemento de memoria activo (valor predeterminado: `memory-core`; establece `plugins.slots.memory = "none"` para desactivarlo).

Relacionado:

* Concepto de memoria: [Memoria](/es/concepts/memory)
* Complementos: [Complementos](/es/plugins)

<div id="examples">
  ## Ejemplos
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## Opciones
</div>

Comunes:

* `--agent <id>`: limitar el ámbito a un único agente (predeterminado: todos los agentes configurados).
* `--verbose`: emite registros detallados durante los sondeos y la indexación.

Notas:

* `memory status --deep` sondea la disponibilidad de vectores + embeddings.
* `memory status --deep --index` ejecuta una reindexación si el almacén está sucio.
* `memory index --verbose` imprime detalles de cada fase (proveedor, modelo, orígenes, actividad por lotes).
* `memory status` incluye cualquier ruta adicional configurada mediante `memorySearch.extraPaths`.