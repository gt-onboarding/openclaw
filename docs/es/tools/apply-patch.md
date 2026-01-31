---
title: Aplicar parche
summary: "Aplicar parches en varios archivos con la herramienta apply_patch"
read_when:
  - Necesitas cambios estructurados en varios archivos
  - Quieres documentar o depurar cambios basados en parches
---

<div id="apply_patch-tool">
  # apply_patch tool
</div>

Aplica cambios en archivos usando un formato de parche estructurado. Es ideal para ediciones en varios archivos
o con múltiples *hunks* donde una sola llamada a `edit` sería frágil.

La herramienta acepta una única cadena de texto `input` que contiene una o más operaciones de archivo:

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```


<div id="parameters">
  ## Parámetros
</div>

- `input` (obligatorio): Todo el contenido del parche, incluido `*** Begin Patch` y `*** End Patch`.

<div id="notes">
  ## Notas
</div>

- Las rutas se resuelven de forma relativa a la raíz del espacio de trabajo.
- Usa `*** Move to:` dentro de un bloque `*** Update File:` para renombrar archivos.
- `*** End of File` marca una inserción solo al final del archivo (EOF) cuando sea necesario.
- Experimental y deshabilitado de forma predeterminada. Actívalo con `tools.exec.applyPatch.enabled`.
- Solo para OpenAI (incluido OpenAI Codex). Opcionalmente se puede restringir por modelo mediante
  `tools.exec.applyPatch.allowModels`.
- La configuración solo está disponible bajo `tools.exec`.

<div id="example">
  ## Ejemplo
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
